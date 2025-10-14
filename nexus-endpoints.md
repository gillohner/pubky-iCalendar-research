# Nexus Calendar Endpoints (v2.0)

This document specifies the Nexus indexer endpoints for aggregating and querying
calendar posts. Calendar components leverage the existing post infrastructure
with new `kind` filters.

## Index

- [Endpoint Overview](#endpoint-overview)
- [Calendar Endpoints](#calendar-endpoints)
- [Event Endpoints](#event-endpoints)
- [Attendee Endpoints](#attendee-endpoints)
- [Integration with Existing Post API](#integration-with-existing-post-api)

---

## Endpoint Overview

Calendar posts use the **existing Nexus post endpoints** with `kind` filtering.
This enables reuse of existing infrastructure while providing calendar-specific
views.

**Base URL (Development/MVP)**: `https://nexus.riginode.xyz/v0` or
`http://localhost:3000/v0`

### Design Principle

All calendar components are posts. Use existing post endpoints with `kind`
parameter to filter:

- `kind=calendar` - Calendar collections (with admin list in jCal)
- `kind=event` - Events (created by owner or admins)
- `kind=attendee` - RSVPs
- `kind=alarm` - Reminders

---

## Calendar Endpoints

### GET /v0/posts?kind=calendar

List all calendar posts across the network.

**Query Parameters**:

| Parameter   | Type    | Default | Description                 |
| ----------- | ------- | ------- | --------------------------- |
| `skip`      | integer | `0`     | Pagination offset           |
| `limit`     | integer | `20`    | Number of results           |
| `author_id` | string  | -       | Filter by calendar owner    |
| `search`    | string  | -       | Full-text search in content |

**Response** (standard post response):

```json
{
  "posts": [
    {
      "id": "0033RCZXVEPNG",
      "author_id": "satoshi",
      "content": "[\"vcalendar\",[[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG\"],[\"name\",{},\"text\",\"Dezentralschweiz Meetups\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://adam-back\"],...],[]]",
      "kind": "calendar",
      "parent": null,
      "embed": null,
      "attachments": ["pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"],
      "indexed_at": 1727785200000000,
      "counts": {
        "replies": 12,
        "tags": 2
      },
      "uri": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
      "admins": ["pubky://hal", "pubky://adam-back"]
    }
  ],
  "total": 1,
  "skip": 0,
  "limit": 20
}
```

**Note**: The `admins` field is extracted from `X-PUBKY-ADMIN` properties in the
jCal content during indexing.

---

### GET /v0/posts/:post_id

Retrieve a specific calendar post by ID.

**Path Parameters**:

- `post_id` (string): Post ID

**Response**: Standard post object with `kind: "calendar"` and extracted
`admins` array

---

### GET /v0/posts/:calendar_id/posts?kind=event

List all events for a specific calendar (children posts with kind=event from
owner and admins).

**Path Parameters**:

- `calendar_id` (string): Calendar post ID

**Query Parameters**:

| Parameter      | Type    | Default | Description                                  |
| -------------- | ------- | ------- | -------------------------------------------- |
| `skip`         | integer | `0`     | Pagination offset                            |
| `limit`        | integer | `20`    | Number of results                            |
| `start_after`  | string  | -       | Filter events starting after date (ISO 8601) |
| `start_before` | string  | -       | Filter events starting before date           |
| `status`       | string  | -       | Filter by status (parsed from jCal content)  |

**Response**:

```json
{
  "posts": [
    {
      "id": "0033SCZXVEPNG",
      "author_id": "satoshi",
      "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Zürich\"],...],[]]",
      "kind": "event",
      "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
      "embed": null,
      "attachments": [],
      "indexed_at": 1727270200000000,
      "counts": {
        "replies": 3,
        "tags": 5
      },
      "uri": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG"
    },
    {
      "id": "0033TD0XVEPNG",
      "author_id": "hal",
      "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://hal/pub/pubky.app/posts/0033TD0XVEPNG\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Bern\"],...],[]]",
      "kind": "event",
      "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
      "embed": null,
      "attachments": [],
      "indexed_at": 1727356600000000,
      "counts": {
        "replies": 1,
        "tags": 3
      },
      "uri": "pubky://hal/pub/pubky.app/posts/0033TD0XVEPNG"
    }
  ],
  "total": 12,
  "skip": 0,
  "limit": 20
}
```

**Note**: Events include those created by the calendar owner AND any user listed
in the calendar's `X-PUBKY-ADMIN` properties.

---

## Event Endpoints

### GET /v0/posts?kind=event

List all event posts across the network.

**Query Parameters**:

| Parameter      | Type    | Default | Description            |
| -------------- | ------- | ------- | ---------------------- |
| `skip`         | integer | `0`     | Pagination offset      |
| `limit`        | integer | `20`    | Number of results      |
| `parent`       | string  | -       | Filter by calendar URI |
| `author_id`    | string  | -       | Filter by organizer    |
| `start_after`  | string  | -       | Filter by start date   |
| `start_before` | string  | -       | Filter by start date   |

**Response**: Standard post array with `kind: "event"`

---

### GET /v0/posts/:event_id/posts?kind=attendee

List all attendee posts for a specific event.

**Path Parameters**:

- `event_id` (string): Event post ID

**Query Parameters**:

| Parameter  | Type    | Default | Description                    |
| ---------- | ------- | ------- | ------------------------------ |
| `skip`     | integer | `0`     | Pagination offset              |
| `limit`    | integer | `50`    | Number of results              |
| `partstat` | string  | -       | Filter by participation status |

**Response**: Standard post array with `kind: "attendee"`

---

## Attendee Endpoints

### GET /v0/posts?kind=attendee

List all attendee posts across the network.

**Query Parameters**:

| Parameter   | Type    | Default | Description             |
| ----------- | ------- | ------- | ----------------------- |
| `skip`      | integer | `0`     | Pagination offset       |
| `limit`     | integer | `50`    | Number of results       |
| `parent`    | string  | -       | Filter by event URI     |
| `author_id` | string  | -       | Filter by attendee user |

---

## Admin-Specific Endpoints

### GET /v0/calendars/managed

List all calendars where the current user is owner or admin.

**Query Parameters**:

| Parameter | Type    | Default  | Description       |
| --------- | ------- | -------- | ----------------- |
| `user_id` | string  | required | User ID to check  |
| `skip`    | integer | `0`      | Pagination offset |
| `limit`   | integer | `20`     | Number of results |

**Response**:

```json
{
  "calendars": [
    {
      "id": "0033RCZXVEPNG",
      "author_id": "satoshi",
      "content": "[\"vcalendar\",...[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"]...,[]]",
      "role": "owner",
      "uri": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG"
    },
    {
      "id": "0033XCZXVEPNG",
      "author_id": "alice",
      "content": "[\"vcalendar\",...[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"]...,[]]",
      "role": "admin",
      "uri": "pubky://alice/pub/pubky.app/posts/0033XCZXVEPNG"
    }
  ],
  "total": 2
}
```

---

## Integration with Existing Post API

### Leveraging Existing Features

**Comments on Events**: Users can reply to event posts using standard post
replies:

```json
{
  "content": "Looking forward to this meetup!",
  "kind": "short",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "embed": null,
  "attachments": null
}
```

**Tags on Calendars/Events**: Use existing tag system for semantic tagging:

```json
{
  "uri": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "label": "bitcoin",
  "created_at": 1727785200000000
}
```

**Bookmarks**: Bookmark events using existing bookmark system:

```json
{
  "uri": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "created_at": 1727785200000000
}
```

### Query Examples

**Get all events where user is admin or owner:**

```
GET /v0/calendars/managed?user_id={user_id}
```

**Get user's RSVP'd events:**

```
GET /v0/posts?kind=attendee&author_id={user_id}
```

**Get upcoming events:**

```
GET /v0/posts?kind=event&start_after=2025-10-14T00:00:00Z
```

---

## Nexus Implementation Requirements

### Post Handler Modifications

**In `nexus-watcher/src/events/handlers/post.rs`:**

1. **Parse jCal Content**: When `kind` is `calendar`, `event`, `attendee`, or
   `alarm`, parse the jCal JSON from `content` field
2. **Extract Admin List**: For calendars, extract all `X-PUBKY-ADMIN` properties
   and create `ADMIN_OF` relationships in Neo4j
3. **Extract Metadata**: Index key properties (dtstart, dtend, summary) into
   separate Neo4j properties for efficient querying
4. **Validate Parent**: For events, verify author is either calendar owner OR
   listed in calendar's `X-PUBKY-ADMIN` properties

**Admin Validation Logic**:

```rust
async fn validate_event_admin(
    event: &PubkyAppPost,
    author_id: &PubkyId
) -> Result<bool> {
    let parent_uri = event.parent.as_ref()
        .ok_or("Event must have parent calendar")?;

    // Get parent calendar post
    let calendar = PostDetails::get_by_uri(parent_uri).await?
        .ok_or("Parent calendar not found")?;

    // Check if author is calendar owner
    if calendar.author_id == *author_id {
        return Ok(true);
    }

    // Parse jCal and check X-PUBKY-ADMIN properties
    let jcal: Vec<serde_json::Value> = serde_json::from_str(&calendar.content)?;
    let properties = &jcal[1];

    let admin_uris = extract_all_jcal_properties(properties, "x-pubky-admin")?;
    let author_uri = format!("pubky://{}", author_id);

    Ok(admin_uris.contains(&author_uri))
}
```

**Admin Relationship Indexing**:

```rust
async fn index_calendar_admins(
    calendar_uri: &str,
    jcal: &Vec<serde_json::Value>
) -> Result<()> {
    let properties = &jcal[1];
    let admin_uris = extract_all_jcal_properties(properties, "x-pubky-admin")?;

    // Create ADMIN_OF relationships in Neo4j
    for admin_uri in admin_uris {
        let admin_id = parse_pubky_uri(&admin_uri)?;

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
```

### Graph Schema Additions

**Neo4j Relationship Indexing:**

```cypher
// Create index for admin relationships
CREATE INDEX admin_of_idx FOR ()-[r:ADMIN_OF]-() ON (r.created_at);

// Create index for calendar/event metadata
CREATE INDEX post_kind_idx FOR (p:Post) ON (p.kind);
CREATE INDEX post_dtstart_idx FOR (p:Post) ON (p.dtstart);
CREATE INDEX post_dtend_idx FOR (p:Post) ON (p.dtend);
```

**Post Node Properties (for calendar posts):**

```
- kind: "calendar" | "event" | "attendee" | "alarm"
- jcal_name: extracted from jCal (for calendars)
- admin_uris: JSON array of admin URIs (for calendars)
- dtstart: ISO timestamp (for events)
- dtend: ISO timestamp (for events)
- summary: text (for events)
- location: text (for events)
- status: text (for events)
```

### API Endpoint Extensions

**In `nexus-webapi/src/routes/posts.rs`:**

Add query parameter handlers for calendar-specific filtering:

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

**In `nexus-webapi/src/routes/calendars.rs` (new):**

```rust
#[derive(Deserialize)]
pub struct ManagedCalendarsQuery {
    pub user_id: String,
    pub skip: Option<i64>,
    pub limit: Option<i64>,
}

pub async fn get_managed_calendars(
    Query(query): Query<ManagedCalendarsQuery>
) -> Result<Json<CalendarResponse>, ApiError> {
    // Query calendars where user is owner or has ADMIN_OF relationship
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

    Ok(Json(result))
}
```

---

## Migration Path

### Phase 1: Add Post Kinds to pubky-app-specs

```rust
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

### Phase 2: Extend Post Handlers

- Add jCal parsing logic in `sync_put`
- Extract and index `X-PUBKY-ADMIN` properties
- Create `ADMIN_OF` relationships in Neo4j
- Implement admin validation for events

### Phase 3: Add Query Filters

- Extend post query API with calendar filters
- Add date range queries
- Add `/v0/calendars/managed` endpoint

### Phase 4: Frontend Integration

- Update Franky/Calky to create posts with calendar kinds
- Parse jCal from post content for display
- Build admin management UI for calendar owners

---

## Example Frontend Workflow

**Creating a Calendar with Admins:**

```javascript
const calendar = {
  content: JSON.stringify([
    "vcalendar",
    [
      ["prodid", {}, "text", "-//Pubky//Pubky Calendar 1.0//EN"],
      ["version", {}, "text", "2.0"],
      [
        "uid",
        {},
        "text",
        `pubky://${userId}/pub/pubky.app/posts/${calendarId}`,
      ],
      ["name", {}, "text", "My Bitcoin Events"],
      ["color", {}, "text", "#F7931A"],
      ["x-pubky-admin", {}, "uri", "pubky://hal"],
      ["x-pubky-admin", {}, "uri", "pubky://adam-back"],
    ],
    [],
  ]),
  kind: "calendar",
  parent: null,
  embed: null,
  attachments: [],
};

await pubkyClient.put(`/pub/pubky.app/posts/${calendarId}`, calendar);
```

**Adding an Admin (Edit Calendar Post):**

```javascript
// Fetch current calendar
const calendar = await pubkyClient.get(`/pub/pubky.app/posts/${calendarId}`);
const jcal = JSON.parse(calendar.content);

// Add new admin property
jcal[1].push(["x-pubky-admin", {}, "uri", "pubky://newadmin"]);

// Update calendar
calendar.content = JSON.stringify(jcal);
await pubkyClient.put(`/pub/pubky.app/posts/${calendarId}`, calendar);
```

**Creating an Event as Admin:**

```javascript
const event = {
  content: JSON.stringify([
    "vevent",
    [
      ["uid", {}, "text", `pubky://${userId}/pub/pubky.app/posts/${eventId}`],
      [
        "dtstart",
        { "tzid": "Europe/Zurich" },
        "date-time",
        "2025-10-20T19:00:00",
      ],
      ["summary", {}, "text", "Bitcoin Meetup Zürich"],
      ["organizer", { "cn": "Hal" }, "cal-address", `pubky://${userId}`],
    ],
    [],
  ]),
  kind: "event",
  parent: `pubky://satoshi/pub/pubky.app/posts/${calendarId}`,
  embed: null,
  attachments: [],
};

await pubkyClient.put(`/pub/pubky.app/posts/${eventId}`, event);
// Nexus will validate that userId is in calendar's X-PUBKY-ADMIN list
```

**RSVP to Event:**

```javascript
const attendee = {
  content: JSON.stringify([
    "attendee",
    [
      [
        "uid",
        {},
        "text",
        `pubky://${userId}/pub/pubky.app/posts/${attendeeId}`,
      ],
      [
        "attendee",
        {
          "cn": "Alice",
          "partstat": "ACCEPTED",
          "rsvp": "TRUE",
        },
        "cal-address",
        `pubky://${userId}`,
      ],
    ],
    [],
  ]),
  kind: "attendee",
  parent: `pubky://satoshi/pub/pubky.app/posts/${eventId}`,
  embed: null,
  attachments: null,
};

await pubkyClient.put(`/pub/pubky.app/posts/${attendeeId}`, attendee);
```
