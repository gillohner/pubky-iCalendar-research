# Prototype Implementation (v2.0)

## Executive Summary

Implement a proof-of-concept calendar application using **PubkyAppPost** with
new `kind` values (`calendar`, `event`, `attendee`, `alarm`). The implementation
leverages existing Nexus post infrastructure while extending it with
calendar-specific functionality. Calendar admins are defined by the calendar
owner through `X-PUBKY-ADMIN` properties in the jCal content.

## Architectural Changes from v1.0

**Key Implementation Differences:**

1. **No New Resource Types**: Extend `PubkyAppPostKind` enum instead of creating
   `PubkyAppVCalendar` types
2. **Reuse Post Handlers**: Modify existing
   `nexus-watcher/src/events/handlers/post.rs` instead of creating new handlers
3. **jCal-Based Admins**: Calendar owners define admins through `X-PUBKY-ADMIN`
   properties in jCal content (not via tags)
4. **Parent Relationships**: Use post `parent` field for event → calendar
   relationships
5. **Unified Storage**: All components under `/pub/pubky.app/posts/:id`
6. **Admin Graph Relationships**: Index `ADMIN_OF` relationships in Neo4j for
   efficient queries

---

## Roadmap (Updated)

### Phase 1: Extend pubky-app-specs (~1 week)

**Tasks:**

1. Add new post kinds to `PubkyAppPostKind` enum:
   - `Calendar`
   - `Event`
   - `Attendee`
   - `Alarm`

2. Update validation rules:
   - Allow jCal JSON in `content` for calendar kinds
   - Validate `parent` field for events (must point to calendar post)
   - No special validation for `X-PUBKY-ADMIN` (standard jCal extension
     property)

3. Build and test:

```bash
cd pubky-app-specs
cargo build
cargo test
```

---

### Phase 2: Extend Nexus Indexer (~2-3 weeks)

**2.1 Post Handler Extensions** (`nexus-watcher/src/events/handlers/post.rs`):

```rust
pub async fn sync_put(
    post: PubkyAppPost,
    author_id: PubkyId,
    post_id: String,
) -> Result<(), DynError> {
    // Existing post handling...

    // NEW: Calendar-specific handling
    match post.kind {
        PubkyAppPostKind::Calendar => {
            index_calendar_metadata(&post, &author_id, &post_id).await?;
            index_calendar_admins(&post, &author_id, &post_id).await?;
        },
        PubkyAppPostKind::Event => {
            validate_event_admin(&post, &author_id).await?;
            index_event_metadata(&post, &author_id, &post_id).await?;
        },
        PubkyAppPostKind::Attendee | PubkyAppPostKind::Alarm => {
            index_calendar_relationship(&post, &author_id, &post_id).await?;
        },
        _ => {}
    }

    // Continue with existing indexing...
}

async fn index_calendar_metadata(
    post: &PubkyAppPost,
    author_id: &PubkyId,
    post_id: &str
) -> Result<()> {
    // Parse jCal from content
    let jcal: Vec<serde_json::Value> = serde_json::from_str(&post.content)?;

    // Extract calendar name, color, etc.
    let properties = &jcal[1];
    let name = extract_jcal_property(properties, "name")?;
    let color = extract_jcal_property(properties, "color");

    // Store in Neo4j as post properties
    PostDetails::update_index_field(
        &[author_id.as_str(), post_id],
        "jcal_name",
        JsonAction::Set(name),
        None
    ).await?;

    Ok(())
}

async fn index_calendar_admins(
    post: &PubkyAppPost,
    author_id: &PubkyId,
    post_id: &str
) -> Result<()> {
    // Parse jCal from content
    let jcal: Vec<serde_json::Value> = serde_json::from_str(&post.content)?;
    let properties = &jcal[1];

    // Extract all X-PUBKY-ADMIN properties
    let admin_uris = extract_all_jcal_properties(properties, "x-pubky-admin")?;

    // Store as JSON array in post property
    PostDetails::update_index_field(
        &[author_id.as_str(), post_id],
        "admin_uris",
        JsonAction::Set(serde_json::to_string(&admin_uris)?),
        None
    ).await?;

    // Create ADMIN_OF relationships in Neo4j
    let calendar_uri = format!("pubky://{}/pub/pubky.app/posts/{}", author_id, post_id);

    for admin_uri in admin_uris {
        let admin_id = parse_pubky_user_id(&admin_uri)?;

        graph_client.execute(
            "MATCH (admin:User {id: $admin_id})
             MATCH (calendar:Post {uri: $calendar_uri})
             MERGE (admin)-[:ADMIN_OF]->(calendar)",
            params! {
                "admin_id" => admin_id,
                "calendar_uri" => calendar_uri
            }
        ).await?;
    }

    Ok(())
}

async fn validate_event_admin(
    post: &PubkyAppPost,
    author_id: &PubkyId
) -> Result<()> {
    let parent_uri = post.parent.as_ref()
        .ok_or("Event must have parent calendar")?;

    // Get parent calendar post
    let calendar = PostDetails::get_by_uri(parent_uri).await?
        .ok_or("Parent calendar not found")?;

    // Check if author is calendar owner
    if calendar.author_id == *author_id {
        return Ok(());
    }

    // Check if author is in admin list
    let admin_uris: Vec<String> = serde_json::from_str(&calendar.admin_uris)?;
    let author_uri = format!("pubky://{}", author_id);

    if !admin_uris.contains(&author_uri) {
        return Err("User is not admin or owner of parent calendar".into());
    }

    Ok(())
}

async fn index_event_metadata(
    post: &PubkyAppPost,
    author_id: &PubkyId,
    post_id: &str
) -> Result<()> {
    // Parse jCal from content
    let jcal: Vec<serde_json::Value> = serde_json::from_str(&post.content)?;

    // Extract event properties
    let properties = &jcal[1];
    let dtstart = extract_jcal_property(properties, "dtstart")?;
    let dtend = extract_jcal_property(properties, "dtend");
    let summary = extract_jcal_property(properties, "summary")?;
    let location = extract_jcal_property(properties, "location");
    let status = extract_jcal_property(properties, "status");

    // Store in Neo4j for efficient queries
    let updates = vec![
        ("dtstart", JsonAction::Set(dtstart)),
        ("dtend", JsonAction::Set(dtend.unwrap_or_default())),
        ("summary", JsonAction::Set(summary)),
        ("location", JsonAction::Set(location.unwrap_or_default())),
        ("status", JsonAction::Set(status.unwrap_or("CONFIRMED".to_string()))),
    ];

    for (field, action) in updates {
        PostDetails::update_index_field(
            &[author_id.as_str(), post_id],
            field,
            action,
            None
        ).await?;
    }

    Ok(())
}
```

**2.2 jCal Parsing Utilities** (new module `nexus-common/src/jcal.rs`):

```rust
use serde_json::Value;

pub fn extract_jcal_property(
    properties: &Value,
    property_name: &str
) -> Result<String, DynError> {
    let props = properties.as_array()
        .ok_or("Invalid jCal properties")?;

    for prop in props {
        if let Some(arr) = prop.as_array() {
            if arr.len() >= 4 && arr[0] == property_name {
                return Ok(arr[3].as_str()
                    .ok_or("Invalid property value")?
                    .to_string());
            }
        }
    }

    Err(format!("Property {} not found", property_name).into())
}

pub fn extract_all_jcal_properties(
    properties: &Value,
    property_name: &str
) -> Result<Vec<String>, DynError> {
    let props = properties.as_array()
        .ok_or("Invalid jCal properties")?;

    let mut values = vec![];

    for prop in props {
        if let Some(arr) = prop.as_array() {
            if arr.len() >= 4 && arr[0] == property_name {
                if let Some(val) = arr[3].as_str() {
                    values.push(val.to_string());
                }
            }
        }
    }

    Ok(values)
}

pub fn parse_jcal_date(date_str: &str) -> Result<i64, DynError> {
    // Parse ISO8601 to Unix timestamp
    // Implementation...
}

pub fn parse_pubky_user_id(uri: &str) -> Result<String, DynError> {
    // Extract user ID from pubky:// URI
    // pubky://userid => userid
    uri.strip_prefix("pubky://")
        .and_then(|s| s.split('/').next())
        .map(|s| s.to_string())
        .ok_or("Invalid pubky URI".into())
}
```

**2.3 API Extensions** (`nexus-webapi/src/routes/posts.rs`):

```rust
#[derive(Deserialize)]
pub struct CalendarPostQuery {
    pub skip: Option<i64>,
    pub limit: Option<i64>,
    pub kind: Option<String>,
    pub start_after: Option<String>,
    pub start_before: Option<String>,
    pub parent: Option<String>,
}

pub async fn get_posts(
    Query(query): Query<CalendarPostQuery>
) -> Result<Json<PostStreamResponse>, ApiError> {
    let mut conditions = vec![];

    // Filter by kind
    if let Some(kind) = query.kind {
        conditions.push(format!("p.kind = '{}'", kind));
    }

    // Filter by date range (for events)
    if let Some(after) = query.start_after {
        let timestamp = parse_iso_date(&after)?;
        conditions.push(format!("p.dtstart >= {}", timestamp));
    }

    if let Some(before) = query.start_before {
        let timestamp = parse_iso_date(&before)?;
        conditions.push(format!("p.dtstart <= {}", timestamp));
    }

    // Filter by parent (for events in specific calendar)
    if let Some(parent_uri) = query.parent {
        conditions.push(format!("p.parent = '{}'", parent_uri));
    }

    // Execute query with conditions
    PostStream::get_posts_with_filters(conditions, query.skip, query.limit).await
}
```

**2.4 New Calendars Endpoint** (`nexus-webapi/src/routes/calendars.rs`):

```rust
use axum::{extract::Query, Json};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
pub struct ManagedCalendarsQuery {
    pub user_id: String,
    pub skip: Option<i64>,
    pub limit: Option<i64>,
}

#[derive(Serialize)]
pub struct ManagedCalendar {
    pub id: String,
    pub author_id: String,
    pub uri: String,
    pub role: String, // "owner" or "admin"
    pub jcal_name: String,
    pub admin_uris: Vec<String>,
}

pub async fn get_managed_calendars(
    Query(query): Query<ManagedCalendarsQuery>
) -> Result<Json<Vec<ManagedCalendar>>, ApiError> {
    let result = graph_client.execute(
        "MATCH (calendar:Post {kind: 'calendar'})
         WHERE calendar.author_id = $user_id
            OR EXISTS {
              MATCH (user:User {id: $user_id})-[:ADMIN_OF]->(calendar)
            }
         RETURN calendar, 
                CASE WHEN calendar.author_id = $user_id 
                     THEN 'owner' 
                     ELSE 'admin' 
                END as role
         SKIP $skip
         LIMIT $limit",
        params! {
            "user_id" => query.user_id,
            "skip" => query.skip.unwrap_or(0),
            "limit" => query.limit.unwrap_or(20)
        }
    ).await?;

    let calendars = result.into_iter().map(|row| {
        let calendar: PostDetails = row.get("calendar")?;
        let role: String = row.get("role")?;

        Ok(ManagedCalendar {
            id: calendar.id,
            author_id: calendar.author_id,
            uri: calendar.uri,
            role,
            jcal_name: calendar.jcal_name,
            admin_uris: serde_json::from_str(&calendar.admin_uris)?
        })
    }).collect::<Result<Vec<_>>>()?;

    Ok(Json(calendars))
}
```

**2.5 Testing**:

```bash
# Set up test data
cargo run -p nexusd -- db mock

# Add calendar test data to docker/test-graph/mocks/
# Include calendars with X-PUBKY-ADMIN properties

# Run tests
cargo nextest run -p nexus-watcher calendar --no-fail-fast
cargo nextest run -p nexus-webapi calendar --no-fail-fast
```

---

### Phase 3: Frontend Application (Calky PWA) (~2 weeks)

**3.1 Project Setup**:

```bash
npx create-next-app@latest calky --typescript --tailwind --app
cd calky
npm install @synonymdev/pubky zustand axios date-fns
```

**3.2 Core Components**:

| Component        | Description                                             | Implementation Notes                                   |
| ---------------- | ------------------------------------------------------- | ------------------------------------------------------ |
| `CalendarList`   | Display calendars from `/v0/posts?kind=calendar`        | Card-based grid with admin badges                      |
| `EventList`      | Display events from `/v0/posts?kind=event&parent={cal}` | List/Calendar toggle, shows events from owner + admins |
| `EventDetail`    | Single event view                                       | Parse jCal, show map, comments, organizer info         |
| `CalendarForm`   | Create/edit calendar                                    | Build jCal JSON with admin list                        |
| `AdminManager`   | Manage calendar admins                                  | Add/remove X-PUBKY-ADMIN properties (owner only)       |
| `EventForm`      | Create/edit event                                       | Build jCal JSON, select from managed calendars         |
| `AttendeeButton` | RSVP to event                                           | Create attendee post                                   |

**3.3 State Management** (Zustand):

```typescript
// stores/calendarStore.ts
import create from "zustand";

interface CalendarState {
  calendars: Post[];
  managedCalendars: ManagedCalendar[];
  events: Post[];
  fetchCalendars: () => Promise<void>;
  fetchManagedCalendars: (userId: string) => Promise<void>;
  fetchEvents: (calendarUri: string) => Promise<void>;
  createCalendar: (jcal: any, admins: string[]) => Promise<void>;
  updateCalendarAdmins: (calendarId: string, admins: string[]) => Promise<void>;
  createEvent: (jcal: any, parentUri: string) => Promise<void>;
  rsvpEvent: (eventUri: string, status: string) => Promise<void>;
}

export const useCalendarStore = create<CalendarState>((set, get) => ({
  calendars: [],
  managedCalendars: [],
  events: [],

  fetchCalendars: async () => {
    const response = await fetch(
      "https://nexus.pubky.app/v0/posts?kind=calendar",
    );
    const data = await response.json();
    set({ calendars: data.posts });
  },

  fetchManagedCalendars: async (userId: string) => {
    const response = await fetch(
      `https://nexus.pubky.app/v0/calendars/managed?user_id=${userId}`,
    );
    const data = await response.json();
    set({ managedCalendars: data });
  },

  fetchEvents: async (calendarUri: string) => {
    const response = await fetch(
      `https://nexus.pubky.app/v0/posts?kind=event&parent=${
        encodeURIComponent(calendarUri)
      }`,
    );
    const data = await response.json();
    set({ events: data.posts });
  },

  createCalendar: async (jcal: any, admins: string[]) => {
    const postId = generateTimestampId();

    // Add X-PUBKY-ADMIN properties to jCal
    for (const adminUri of admins) {
      jcal[1].push(["x-pubky-admin", {}, "uri", adminUri]);
    }

    const post = {
      content: JSON.stringify(jcal),
      kind: "calendar",
      parent: null,
      embed: null,
      attachments: [],
    };

    await pubkyClient.put(`/pub/pubky.app/posts/${postId}`, post);
    get().fetchCalendars();
  },

  updateCalendarAdmins: async (calendarId: string, admins: string[]) => {
    // Fetch current calendar
    const calendar = await pubkyClient.get(
      `/pub/pubky.app/posts/${calendarId}`,
    );
    const jcal = JSON.parse(calendar.content);

    // Remove all existing X-PUBKY-ADMIN properties
    jcal[1] = jcal[1].filter((prop: any) => prop[0] !== "x-pubky-admin");

    // Add new admin properties
    for (const adminUri of admins) {
      jcal[1].push(["x-pubky-admin", {}, "uri", adminUri]);
    }

    // Update calendar
    calendar.content = JSON.stringify(jcal);
    await pubkyClient.put(`/pub/pubky.app/posts/${calendarId}`, calendar);

    get().fetchManagedCalendars(currentUserId);
  },

  createEvent: async (jcal: any, parentUri: string) => {
    const postId = generateTimestampId();
    const post = {
      content: JSON.stringify(jcal),
      kind: "event",
      parent: parentUri,
      embed: null,
      attachments: [],
    };

    await pubkyClient.put(`/pub/pubky.app/posts/${postId}`, post);
    get().fetchEvents(parentUri);
  },

  rsvpEvent: async (eventUri: string, status: string) => {
    const postId = generateTimestampId();
    const jcal = [
      "attendee",
      [
        ["uid", {}, "text", `pubky://${userId}/pub/pubky.app/posts/${postId}`],
        ["dtstamp", {}, "date-time", new Date().toISOString()],
        [
          "attendee",
          {
            "cn": currentUser.name,
            "partstat": status,
            "rsvp": "TRUE",
          },
          "cal-address",
          `pubky://${userId}`,
        ],
      ],
      [],
    ];

    const post = {
      content: JSON.stringify(jcal),
      kind: "attendee",
      parent: eventUri,
      embed: null,
      attachments: null,
    };

    await pubkyClient.put(`/pub/pubky.app/posts/${postId}`, post);
  },
}));
```

**3.4 jCal Utilities**:

```typescript
// utils/jcal.ts
export function parseJCal(jcalString: string) {
  const jcal = JSON.parse(jcalString);
  const [component, properties, subcomponents] = jcal;

  const parsed: any = { component };

  for (const prop of properties) {
    const [name, params, type, ...values] = prop;

    // Handle repeatable properties like X-PUBKY-ADMIN
    if (name === "x-pubky-admin") {
      if (!parsed.admins) parsed.admins = [];
      parsed.admins.push(values[0]);
    } else {
      parsed[name] = values.length === 1 ? values[0] : values;
    }
  }

  return parsed;
}

export function buildJCalCalendar(data: {
  name: string;
  description?: string;
  color?: string;
  categories?: string[];
  admins?: string[]; // Array of pubky:// URIs
}) {
  const properties: any[] = [
    ["prodid", {}, "text", "-//Pubky//Pubky Calendar 1.0//EN"],
    ["version", {}, "text", "2.0"],
    ["calscale", {}, "text", "GREGORIAN"],
    ["method", {}, "text", "PUBLISH"],
    ["uid", {}, "text", `pubky://${userId}/pub/pubky.app/posts/${postId}`],
    ["name", {}, "text", data.name],
  ];

  if (data.description) {
    properties.push(["description", {}, "text", data.description]);
  }

  if (data.color) {
    properties.push(["color", {}, "text", data.color]);
  }

  if (data.categories) {
    properties.push(["categories", {}, "text", ...data.categories]);
  }

  // Add admin properties
  if (data.admins) {
    for (const adminUri of data.admins) {
      properties.push(["x-pubky-admin", {}, "uri", adminUri]);
    }
  }

  return ["vcalendar", properties, []];
}

export function buildJCalEvent(data: {
  summary: string;
  description?: string;
  dtstart: string;
  dtend?: string;
  location?: string;
  organizer: string;
}) {
  return [
    "vevent",
    [
      ["uid", {}, "text", `pubky://${userId}/pub/pubky.app/posts/${postId}`],
      ["dtstamp", {}, "date-time", new Date().toISOString()],
      ["dtstart", { "tzid": "UTC" }, "date-time", data.dtstart],
      ...(data.dtend
        ? [["dtend", { "tzid": "UTC" }, "date-time", data.dtend]]
        : []),
      ["summary", {}, "text", data.summary],
      ...(data.description
        ? [["description", {}, "text", data.description]]
        : []),
      ...(data.location ? [["location", {}, "text", data.location]] : []),
      ["organizer", { "cn": currentUser.name }, "cal-address", data.organizer],
      ["status", {}, "text", "CONFIRMED"],
    ],
    [],
  ];
}
```

**3.5 Admin Management Component**:

```typescript
// components/AdminManager.tsx
import { useState } from "react";
import { useCalendarStore } from "@/stores/calendarStore";

export function AdminManager({ calendar }: { calendar: ManagedCalendar }) {
  const [adminInput, setAdminInput] = useState("");
  const { updateCalendarAdmins } = useCalendarStore();

  const addAdmin = async () => {
    const newAdmins = [...calendar.admin_uris, adminInput];
    await updateCalendarAdmins(calendar.id, newAdmins);
    setAdminInput("");
  };

  const removeAdmin = async (adminUri: string) => {
    const newAdmins = calendar.admin_uris.filter((a) => a !== adminUri);
    await updateCalendarAdmins(calendar.id, newAdmins);
  };

  // Only show if user is calendar owner
  if (calendar.role !== "owner") {
    return null;
  }

  return (
    <div className="admin-manager">
      <h3>Manage Admins</h3>
      <ul>
        {calendar.admin_uris.map((adminUri) => (
          <li key={adminUri}>
            {adminUri}
            <button onClick={() => removeAdmin(adminUri)}>Remove</button>
          </li>
        ))}
      </ul>
      <input
        value={adminInput}
        onChange={(e) => setAdminInput(e.target.value)}
        placeholder="pubky://userid"
      />
      <button onClick={addAdmin}>Add Admin</button>
    </div>
  );
}
```

---

### Phase 4: Testing & Refinement (~1 week)

**Integration Testing:**

1. Create calendar via frontend with admins
2. Verify Nexus indexes post with `kind=calendar` and `ADMIN_OF` relationships
3. Admin creates event
4. Verify event appears under calendar
5. Owner edits calendar to add/remove admin
6. Verify `ADMIN_OF` relationships update
7. RSVP to event
8. Verify attendee count updates
9. Comment on event (reply to event post)
10. Tag event with categories
11. Bookmark event

---

## Core Features (Updated)

### Discovery Pages

**Calendar Discovery** (`/calendars`):

- Query: `GET /v0/posts?kind=calendar`
- Display: Grid of calendar cards with name, color, event count, admin count

**Event Discovery** (`/events`):

- Query: `GET /v0/posts?kind=event&start_after={today}`
- Display: List/Calendar view toggle
- Filters: Date range, location (from jCal), tags
- Show organizer badge (owner vs admin)

### Overview Pages

**Calendar Overview** (`/calendar/:id`):

- Query: `GET /v0/posts/:id` (calendar post)
- Query events: `GET /v0/posts?kind=event&parent={calendarUri}`
- Display: Calendar header + event list/calendar view
- Show admins (parsed from jCal `X-PUBKY-ADMIN`)
- Admin manager component (owner only)

**Event Overview** (`/event/:id`):

- Query: `GET /v0/posts/:id` (event post)
- Query attendees: `GET /v0/posts?kind=attendee&parent={eventUri}`
- Query comments: `GET /v0/posts?parent={eventUri}&kind=short`
- Display: Event details, location map, attendee list, comment thread
- Show organizer info and badge

### Creation Modals

**Calendar Creation**:

- Form fields: name, description, color, image upload
- Admin management: add pubky:// URIs
- Build jCal using `buildJCalCalendar()` with admins
- Create post with `kind=calendar`

**Event Creation**:

- Form fields: summary, description, dtstart, dtend, location
- Select parent calendar from user's managed calendars (owner or admin)
- Build jCal using `buildJCalEvent()`
- Create post with `kind=event` and `parent=calendarUri`

**RSVP**:

- Button: "Attending" / "Maybe" / "Not Attending"
- Build attendee jCal with `partstat` value
- Create post with `kind=attendee` and `parent=eventUri`

---

## Required Nexus Changes Summary

### 1. pubky-app-specs Changes

File: `pubky-app-specs/src/models/post.rs`

```rust
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq, Eq, Hash)]
#[serde(rename_all = "lowercase")]
pub enum PubkyAppPostKind {
    Short,
    Long,
    Image,
    Video,
    Link,
    File,
    Calendar,  // NEW
    Event,     // NEW
    Attendee,  // NEW
    Alarm,     // NEW
}
```

### 2. nexus-common Changes

**New module** `nexus-common/src/jcal.rs`:

- jCal parsing functions
- Property extraction utilities (single and multiple)
- Date parsing helpers
- Pubky URI parsing

**Post model extension** `nexus-common/src/models/post.rs`:

```rust
pub struct PostDetails {
    // Existing fields...

    // NEW: Calendar-specific fields
    pub jcal_name: Option<String>,
    pub admin_uris: Option<String>, // JSON array
    pub dtstart: Option<i64>,
    pub dtend: Option<i64>,
    pub summary: Option<String>,
    pub location: Option<String>,
    pub status: Option<String>,
}
```

### 3. nexus-watcher Changes

**Post handler** `nexus-watcher/src/events/handlers/post.rs`:

- Add calendar kind handling in `sync_put()`
- Implement `index_calendar_metadata()`
- Implement `index_calendar_admins()` with `ADMIN_OF` relationship creation
- Implement `validate_event_admin()` checking owner OR admin list
- Implement `index_event_metadata()`

### 4. nexus-webapi Changes

**Post routes** `nexus-webapi/src/routes/posts.rs`:

- Extend `PostQuery` with calendar filters
- Add `start_after`, `start_before` date filters
- Implement date range queries on `dtstart`/`dtend`

**New routes** `nexus-webapi/src/routes/calendars.rs`:

```rust
// GET /v0/calendars/managed?user_id={id}
pub async fn get_managed_calendars() -> Result<Json<Vec<ManagedCalendar>>>
```

---

## Implementation Priority

**MVP Scope (4-5 weeks):**

1. ✅ Extend pubky-app-specs with calendar post kinds
2. ✅ Implement calendar metadata indexing in Nexus
3. ✅ Implement admin extraction and `ADMIN_OF` relationship indexing
4. ✅ Implement event admin validation (owner OR in admin list)
5. ✅ Extend API with date filters and managed calendars endpoint
6. ✅ Build Calky frontend (discovery + overview pages)
7. ✅ Implement calendar/event creation modals with admin management
8. ✅ Implement RSVP functionality

**Post-MVP Extensions:**

- Recurring event support (parse RRULE)
- OSM location indexing (structured-location)
- Alarm notifications (push notifications)
- CalDAV bridge (translate between jCal and .ics)
- Event history page per user
- Calendar subscription features
- Admin permission levels (X-PUBKY-ADMIN-LEVEL)

---

## Testing Strategy

**Unit Tests**:

- jCal parsing functions (single and multiple properties)
- Admin validation logic
- Date range filters
- Pubky URI parsing

**Integration Tests**:

- Create calendar with admins → verify indexed with `ADMIN_OF` relationships
- Create event as admin → verify admin check passes
- Create event as non-admin → verify rejected
- Edit calendar admins → verify `ADMIN_OF` relationships update
- RSVP → verify attendee count
- Query events by date range

**E2E Tests** (Frontend):

- User creates calendar with admins
- Admin creates event
- Owner edits calendar to add/remove admin
- User RSVPs
- User comments on event
- User bookmarks event
- Query managed calendars endpoint
