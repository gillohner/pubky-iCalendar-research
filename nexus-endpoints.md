# Nexus Calendar Endpoints (v2.0)

This document specifies the Nexus indexer endpoints for aggregating and querying
calendar posts. Calendar components leverage the existing post infrastructure
with new `kind` filters.

## Index

- [Quick Reference](#quick-reference)
- [Endpoint Overview](#endpoint-overview)
- [Calendar Endpoints](#calendar-endpoints)
- [Event Endpoints](#event-endpoints)
- [Attendee Endpoints](#attendee-endpoints)
- [Integration with Existing Post API](#integration-with-existing-post-api)

---

## Quick Reference

### Endpoint Mapping

| Operation                          | Endpoint                                                 | Status                  |
| ---------------------------------- | -------------------------------------------------------- | ----------------------- |
| List all calendars                 | `GET /v0/stream/posts?kind=calendar`                     | ✅ Works now            |
| Get single calendar                | `GET /v0/post/{author_id}/{post_id}`                     | ✅ Works now            |
| List all events                    | `GET /v0/stream/posts?kind=event`                        | ✅ Works now            |
| List events in calendar            | `GET /v0/stream/posts?kind=event&parent={calendar_uri}`  | ⚠️ Needs parent param   |
| List RSVPs for event               | `GET /v0/stream/posts?kind=attendee&parent={event_uri}`  | ⚠️ Needs parent param   |
| Get user's calendars (owner/admin) | `GET /v0/calendars/managed?user_id={user_id}`            | ⚠️ New endpoint         |
| Get user's RSVPs                   | `GET /v0/stream/posts?kind=attendee&source=author`       | ✅ Works now            |
| Filter events by date              | `GET /v0/stream/posts?kind=event&start_after={iso_date}` | ⚠️ Needs date filtering |

**Legend**:

- ✅ **Works now**: No Nexus changes needed, use existing API
- ⚠️ **Needs extension**: Requires Nexus implementation (see Summary section)

### Key Differences from Standard Posts

1. **Content format**: jCal JSON (not plain text or markdown)
2. **Parent relationships**: Events → Calendar, Attendees → Event
3. **Admin permissions**: Events can be created by calendar owner OR admins
4. **Metadata indexing**: Date, location, status extracted from jCal for
   filtering

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

### Current Status vs Required Changes

**✅ Works Now** (no Nexus changes needed):

- Basic post streaming with `GET /v0/stream/posts?kind={kind}`
- Filtering by author using `source=author`
- Getting individual posts with `GET /v0/post/{author_id}/{post_id}`
- Using `post_replies` source to get child posts (events under calendars, RSVPs
  under events)
- Standard post features: tags, bookmarks, replies, counts

**⚠️ Requires Nexus Changes**:

1. **New Post Kinds**: Add `calendar`, `event`, `attendee`, `alarm` to
   `PubkyAppPostKind` enum
2. **Parent URI Filtering**: Add `parent` query parameter to `/v0/stream/posts`
   endpoint
3. **jCal Parsing**: Parse jCal content and extract metadata (dtstart, dtend,
   summary, location, status, partstat)
4. **Admin Relationships**: Extract `X-PUBKY-ADMIN` properties and create
   `ADMIN_OF` Neo4j relationships
5. **Admin Validation**: Validate that event authors are calendar owners or
   admins
6. **Date Filtering**: Add `start_after`/`start_before` parameters for filtering
   by jCal date fields
7. **New Endpoint**: Add `/v0/calendars/managed` endpoint for querying calendars
   by admin status
8. **Post Response Extension**: Add `admins` field to calendar post responses
   (extracted from indexed data)

---

## Calendar Endpoints

### GET /v0/stream/posts?kind=calendar

List all calendar posts across the network.

**Query Parameters**:

| Parameter   | Type    | Default | Description                                       |
| ----------- | ------- | ------- | ------------------------------------------------- |
| `kind`      | string  | -       | **Required**: `calendar` to filter calendar posts |
| `skip`      | integer | `0`     | Pagination offset                                 |
| `limit`     | integer | `20`    | Number of results                                 |
| `source`    | string  | `all`   | Stream source (see StreamSource enum)             |
| `viewer_id` | string  | -       | Viewer ID for personalized responses              |
| `sorting`   | string  | -       | `timeline` or `total_engagement`                  |
| `start`     | integer | -       | Start timestamp/score for timeframe filtering     |
| `end`       | integer | -       | End timestamp/score for timeframe filtering       |

**Response** (PostStream - array of PostView objects):

```json
[
  {
    "details": {
      "id": "0033RCZXVEPNG",
      "author": "satoshi",
      "content": "[\"vcalendar\",[[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG\"],[\"name\",{},\"text\",\"Dezentralschweiz Meetups\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://adam-back\"],...],[]]",
      "kind": "calendar",
      "uri": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
      "attachments": ["pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"],
      "indexed_at": 1727785200000000
    },
    "counts": {
      "replies": 12,
      "tags": 2,
      "unique_tags": 2,
      "reposts": 0
    },
    "tags": [],
    "relationships": {
      "replied": null,
      "reposted": null,
      "mentioned": []
    },
    "bookmark": null
  }
]
```

**Note**:

- The response follows the standard `PostStream` format (array of `PostView`)
- The `admins` field will be extracted from `X-PUBKY-ADMIN` properties during
  indexing (requires Nexus implementation)
- To filter by author, use `source=author` with `author_id` parameter

---

### GET /v0/post/{author_id}/{post_id}

Retrieve a specific calendar post by author ID and post ID.

**Path Parameters**:

- `author_id` (string): Author's Pubky ID
- `post_id` (string): Post ID (Crockford32 encoded)

**Query Parameters**:

| Parameter       | Type    | Default | Description                        |
| --------------- | ------- | ------- | ---------------------------------- |
| `viewer_id`     | string  | -       | Viewer ID for relationship context |
| `limit_tags`    | integer | -       | Max number of tags to return       |
| `limit_taggers` | integer | -       | Max number of taggers per tag      |

**Response**: `PostView` object with `kind: "calendar"` and extracted `admins`
array (after implementation)

---

### GET /v0/stream/posts?kind=event&source=post_replies

List all events for a specific calendar (using the post_replies source to get
child posts).

**Query Parameters**:

| Parameter   | Type    | Default | Description                                                                                              |
| ----------- | ------- | ------- | -------------------------------------------------------------------------------------------------------- |
| `kind`      | string  | -       | **Required**: `event` to filter event posts                                                              |
| `source`    | object  | -       | **Required**: `{"source": "post_replies", "author_id": "{calendar_author}", "post_id": "{calendar_id}"}` |
| `skip`      | integer | `0`     | Pagination offset                                                                                        |
| `limit`     | integer | `20`    | Number of results                                                                                        |
| `viewer_id` | string  | -       | Viewer ID for personalized responses                                                                     |

**⚠️ IMPORTANT**: The current Nexus API uses `post_replies` source to get child
posts, but this returns ALL replies (including comments). Calendar
implementations need **additional filtering** to separate:

- `kind=event` posts (actual events in the calendar)
- `kind=short/long` posts (comments on the calendar)

**Alternative Approach** (requires Nexus extension): Add support for filtering
by `parent` URI directly:

```
GET /v0/stream/posts?kind=event&parent=pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG
```

**Response**: PostStream array with `kind: "event"` posts

```json
[
  {
    "details": {
      "id": "0033SCZXVEPNG",
      "author": "satoshi",
      "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Zürich\"],...],[]]",
      "kind": "event",
      "uri": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
      "indexed_at": 1727270200000000
    },
    "counts": {
      "replies": 3,
      "tags": 5,
      "unique_tags": 3,
      "reposts": 0
    },
    "tags": [],
    "relationships": {
      "replied": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
      "reposted": null,
      "mentioned": []
    },
    "bookmark": null
  }
]
```

**Note**: Events include those created by the calendar owner AND any user listed
in the calendar's `X-PUBKY-ADMIN` properties (requires admin validation in
Nexus).

---

## Event Endpoints

### GET /v0/stream/posts?kind=event

List all event posts across the network.

**Query Parameters**:

| Parameter   | Type    | Default | Description                                      |
| ----------- | ------- | ------- | ------------------------------------------------ |
| `kind`      | string  | -       | **Required**: `event` to filter events           |
| `skip`      | integer | `0`     | Pagination offset                                |
| `limit`     | integer | `20`    | Number of results                                |
| `source`    | string  | `all`   | Stream source (use `author` to filter by author) |
| `viewer_id` | string  | -       | Viewer ID for personalized responses             |
| `sorting`   | string  | -       | `timeline` or `total_engagement`                 |
| `start`     | integer | -       | Start timestamp for timeframe filtering          |
| `end`       | integer | -       | End timestamp for timeframe filtering            |

**Note**:

- To filter by specific author/organizer, use `source=author` with the source
  object format
- Date-based filtering (`start_after`, `start_before`) for event dtstart/dtend
  **requires Nexus extension** to parse and index jCal date fields
- Filtering by `parent` calendar URI **requires Nexus extension** (see
  alternative approach in Calendar Endpoints section)

**Response**: PostStream array with `kind: "event"`

---

### GET /v0/stream/posts?kind=attendee&source=post_replies

List all attendee RSVPs for a specific event.

**Query Parameters**:

| Parameter   | Type    | Default | Description                                                                                        |
| ----------- | ------- | ------- | -------------------------------------------------------------------------------------------------- |
| `kind`      | string  | -       | **Required**: `attendee` to filter RSVP posts                                                      |
| `source`    | object  | -       | **Required**: `{"source": "post_replies", "author_id": "{event_author}", "post_id": "{event_id}"}` |
| `skip`      | integer | `0`     | Pagination offset                                                                                  |
| `limit`     | integer | `50`    | Number of results                                                                                  |
| `viewer_id` | string  | -       | Viewer ID for personalized responses                                                               |

**Note**:

- Filtering by `partstat` (participation status) **requires Nexus extension** to
  parse and index jCal PARTSTAT values
- Same limitation as events: `post_replies` returns all child posts, so
  `kind=attendee` filter is essential

**Response**: PostStream array with `kind: "attendee"`

---

## Attendee Endpoints

### GET /v0/stream/posts?kind=attendee

List all attendee posts across the network.

**Query Parameters**:

| Parameter   | Type    | Default | Description                                    |
| ----------- | ------- | ------- | ---------------------------------------------- |
| `kind`      | string  | -       | **Required**: `attendee` to filter RSVPs       |
| `skip`      | integer | `0`     | Pagination offset                              |
| `limit`     | integer | `50`    | Number of results                              |
| `source`    | string  | `all`   | Stream source (use `author` to filter by user) |
| `viewer_id` | string  | -       | Viewer ID for personalized responses           |

**Note**:

- To get a specific user's RSVPs, use `source=author` with appropriate author_id
- Filtering by `parent` event URI **requires Nexus extension**

---

## Admin-Specific Endpoints

### GET /v0/calendars/managed

**⚠️ NEW ENDPOINT** - This endpoint does NOT exist in the current Nexus API and
needs to be implemented.

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
      "author": "satoshi",
      "content": "[\"vcalendar\",...[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"]...,[]]",
      "role": "owner",
      "uri": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG"
    },
    {
      "id": "0033XCZXVEPNG",
      "author": "alice",
      "content": "[\"vcalendar\",...[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"]...,[]]",
      "role": "admin",
      "uri": "pubky://alice/pub/pubky.app/posts/0033XCZXVEPNG"
    }
  ],
  "total": 2
}
```

**Implementation Requirements**:

- Query Neo4j for calendars where `author = user_id` OR user has `ADMIN_OF`
  relationship
- Requires indexing of `X-PUBKY-ADMIN` properties and creation of `ADMIN_OF`
  relationships
- See [Nexus Implementation Requirements](#nexus-implementation-requirements)
  section

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

**Get all calendars:**

```
GET /v0/stream/posts?kind=calendar&limit=50
```

**Get calendars by specific author:**

```
GET /v0/stream/posts?kind=calendar&source={"source":"author","author_id":"satoshi"}&limit=50
```

**Get all calendars where user is admin or owner** (⚠️ requires new endpoint):

```
GET /v0/calendars/managed?user_id={user_id}
```

**Get user's RSVP'd events:**

```
GET /v0/stream/posts?kind=attendee&source={"source":"author","author_id":"{user_id}"}&limit=100
```

**Get upcoming events** (⚠️ requires Nexus extension for date filtering):

```
GET /v0/stream/posts?kind=event&start_after=2025-10-14T00:00:00Z
```

_Note: Currently only `start`/`end` timestamp filtering is available, not jCal
date field filtering_

**Get events for a specific calendar** (⚠️ requires `parent` filter extension):

```
GET /v0/stream/posts?kind=event&parent=pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG
```

_Workaround: Use `source=post_replies` with calendar author_id and post_id, then
filter by kind=event_

**Get a specific calendar post:**

```
GET /v0/post/satoshi/0033RCZXVEPNG?viewer_id={viewer_id}
```

**Get RSVPs for a specific event:**

```
GET /v0/stream/posts?kind=attendee&source={"source":"post_replies","author_id":"satoshi","post_id":"0033SCZXVEPNG"}
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

Extend the existing `/v0/stream/posts` endpoint handler to support additional
query parameters:

```rust
#[derive(Deserialize)]
pub struct PostStreamQuery {
    // Existing parameters
    pub skip: Option<i64>,
    pub limit: Option<i64>,
    pub kind: Option<PubkyAppPostKind>,  // Already exists
    pub source: Option<StreamSource>,     // Already exists
    pub viewer_id: Option<String>,        // Already exists
    pub sorting: Option<StreamSorting>,   // Already exists
    pub start: Option<i64>,               // Already exists (timestamp)
    pub end: Option<i64>,                 // Already exists (timestamp)
    
    // NEW: Calendar-specific parameters
    pub parent: Option<String>,           // Filter by parent URI
    pub start_after: Option<String>,      // Filter by jCal dtstart (ISO 8601)
    pub start_before: Option<String>,     // Filter by jCal dtstart (ISO 8601)
}

pub async fn stream_posts_handler(
    Query(query): Query<PostStreamQuery>
) -> Result<Json<Vec<PostView>>, ApiError> {
    let mut cypher_conditions = vec![];

    // Existing kind filtering (already works)
    if let Some(kind) = query.kind {
        cypher_conditions.push(format!("p.kind = '{}'", kind));
    }

    // NEW: Filter by parent URI (for events in calendar, RSVPs in event)
    if let Some(parent_uri) = query.parent {
        cypher_conditions.push(format!("p.parent = '{}'", parent_uri));
    }

    // NEW: Filter by jCal date range (requires indexed dtstart field)
    if let Some(after) = query.start_after {
        let timestamp = parse_iso_8601_to_timestamp(&after)?;
        cypher_conditions.push(format!("p.dtstart >= {}", timestamp));
    }

    if let Some(before) = query.start_before {
        let timestamp = parse_iso_8601_to_timestamp(&before)?;
        cypher_conditions.push(format!("p.dtstart <= {}", timestamp));
    }

    // Execute query with conditions
    PostStream::query_with_filters(cypher_conditions, query).await
}
```

**In `nexus-webapi/src/routes/calendars.rs` (NEW FILE):**

Create a new route module for calendar-specific endpoints:

```rust
use axum::{extract::Query, Json};
use neo4rs::query;
use serde::Deserialize;

#[derive(Deserialize)]
pub struct ManagedCalendarsQuery {
    pub user_id: String,
    pub skip: Option<i64>,
    pub limit: Option<i64>,
}

#[derive(Serialize)]
pub struct ManagedCalendarResponse {
    pub calendars: Vec<ManagedCalendar>,
    pub total: i64,
}

#[derive(Serialize)]
pub struct ManagedCalendar {
    pub id: String,
    pub author: String,
    pub content: String,
    pub role: String,  // "owner" or "admin"
    pub uri: String,
}

pub async fn get_managed_calendars(
    Query(query): Query<ManagedCalendarsQuery>
) -> Result<Json<ManagedCalendarResponse>, ApiError> {
    let graph = get_graph_client();
    
    // Query calendars where user is owner or has ADMIN_OF relationship
    let cypher = "
        MATCH (calendar:Post)
        WHERE calendar.kind = 'calendar'
          AND (calendar.author = $user_id 
               OR EXISTS {
                 MATCH (user:User {id: $user_id})-[:ADMIN_OF]->(calendar)
               })
        RETURN calendar, 
               CASE WHEN calendar.author = $user_id 
                    THEN 'owner' 
                    ELSE 'admin' 
               END as role
        ORDER BY calendar.indexed_at DESC
        SKIP $skip
        LIMIT $limit
    ";
    
    let mut result = graph.execute(
        query(cypher)
            .param("user_id", query.user_id.clone())
            .param("skip", query.skip.unwrap_or(0))
            .param("limit", query.limit.unwrap_or(20))
    ).await?;
    
    let mut calendars = vec![];
    while let Some(row) = result.next().await? {
        let calendar: neo4rs::Node = row.get("calendar")?;
        let role: String = row.get("role")?;
        
        calendars.push(ManagedCalendar {
            id: calendar.get("id")?,
            author: calendar.get("author")?,
            content: calendar.get("content")?,
            role,
            uri: calendar.get("uri")?,
        });
    }
    
    let total = calendars.len() as i64;
    
    Ok(Json(ManagedCalendarResponse { calendars, total }))
}
```

**Add route in `nexus-webapi/src/main.rs`:**

```rust
mod routes {
    pub mod calendars;  // NEW
    // ... existing modules
}

// In router setup:
let app = Router::new()
    // ... existing routes
    .route("/v0/calendars/managed", get(routes::calendars::get_managed_calendars));
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

---

## Summary: Required Nexus Changes

This section summarizes all changes needed to Nexus to fully support calendar
functionality.

### 1. pubky-app-specs Changes

**File: `pubky-app-specs/src/post.rs`**

Add new post kinds to the enum:

```rust
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
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
    Alarm,     // NEW (optional for MVP)
}
```

### 2. nexus-watcher Changes

**File: `nexus-watcher/src/events/handlers/post.rs`**

Add jCal parsing and metadata extraction:

```rust
async fn handle_calendar_post(post: &PubkyAppPost, uri: &str) -> Result<()> {
    if matches!(post.kind, PubkyAppPostKind::Calendar | PubkyAppPostKind::Event | PubkyAppPostKind::Attendee) {
        let jcal: Vec<serde_json::Value> = serde_json::from_str(&post.content)?;
        
        match post.kind {
            PubkyAppPostKind::Calendar => {
                // Extract X-PUBKY-ADMIN properties
                let admin_uris = extract_jcal_properties(&jcal[1], "x-pubky-admin")?;
                index_calendar_admins(uri, &admin_uris).await?;
                
                // Extract calendar name
                let name = extract_jcal_property(&jcal[1], "name")?;
                update_post_metadata(uri, "jcal_name", name).await?;
            },
            PubkyAppPostKind::Event => {
                // Validate author is owner or admin of parent calendar
                validate_event_author(post).await?;
                
                // Extract event metadata
                let dtstart = extract_jcal_property(&jcal[1], "dtstart")?;
                let dtend = extract_jcal_property(&jcal[1], "dtend")?;
                let summary = extract_jcal_property(&jcal[1], "summary")?;
                let location = extract_jcal_property(&jcal[1], "location").ok();
                let status = extract_jcal_property(&jcal[1], "status").ok();
                
                update_event_metadata(uri, dtstart, dtend, summary, location, status).await?;
            },
            PubkyAppPostKind::Attendee => {
                // Extract PARTSTAT from attendee property
                let partstat = extract_jcal_attendee_partstat(&jcal[1])?;
                update_post_metadata(uri, "partstat", partstat).await?;
            },
            _ => {}
        }
    }
    Ok(())
}
```

**Add Neo4j relationship creation:**

```rust
async fn index_calendar_admins(calendar_uri: &str, admin_uris: &[String]) -> Result<()> {
    for admin_uri in admin_uris {
        let admin_id = parse_pubky_id_from_uri(admin_uri)?;
        
        graph_client.execute(query(
            "MATCH (admin:User {id: $admin_id})
             MATCH (calendar:Post {uri: $calendar_uri})
             MERGE (admin)-[:ADMIN_OF]->(calendar)"
        )
        .param("admin_id", admin_id)
        .param("calendar_uri", calendar_uri)).await?;
    }
    Ok(())
}
```

**Add validation:**

```rust
async fn validate_event_author(event: &PubkyAppPost) -> Result<()> {
    let parent_uri = event.parent.as_ref()
        .ok_or("Event must have parent calendar")?;
    
    let calendar = get_post_by_uri(parent_uri).await?;
    
    // Check if author is calendar owner
    if calendar.author == event.author {
        return Ok(());
    }
    
    // Check if author is in admin list
    let is_admin = check_admin_relationship(&event.author, parent_uri).await?;
    if !is_admin {
        return Err("Event author must be calendar owner or admin".into());
    }
    
    Ok(())
}
```

### 3. nexus-webapi Changes

**File: `nexus-webapi/src/routes/posts.rs`**

Extend `PostStreamQuery` struct and handler (see API Endpoint Extensions section
above).

**File: `nexus-webapi/src/routes/calendars.rs` (NEW)**

Create new route module (see API Endpoint Extensions section above).

**File: `nexus-webapi/src/main.rs`**

Add calendar routes to router.

### 4. Neo4j Schema Changes

**Add indexes:**

```cypher
// Index for calendar kind filtering
CREATE INDEX post_kind_idx IF NOT EXISTS FOR (p:Post) ON (p.kind);

// Index for parent URI filtering
CREATE INDEX post_parent_idx IF NOT EXISTS FOR (p:Post) ON (p.parent);

// Index for event date filtering
CREATE INDEX post_dtstart_idx IF NOT EXISTS FOR (p:Post) ON (p.dtstart);
CREATE INDEX post_dtend_idx IF NOT EXISTS FOR (p:Post) ON (p.dtend);

// Index for admin relationships
CREATE INDEX admin_of_idx IF NOT EXISTS FOR ()-[r:ADMIN_OF]-() ON (r.created_at);
```

**Add post properties:**

- `kind`: String (calendar, event, attendee, alarm)
- `jcal_name`: String (for calendars)
- `dtstart`: Integer timestamp (for events)
- `dtend`: Integer timestamp (for events)
- `summary`: String (for events)
- `location`: String (for events)
- `status`: String (for events)
- `partstat`: String (for attendees)

### 5. OpenAPI Specification Updates

Update `nexus-webapi` OpenAPI spec to include:

1. New `calendar`, `event`, `attendee`, `alarm` values in `PubkyAppPostKind`
   enum
2. New query parameters for `/v0/stream/posts`:
   - `parent` (string)
   - `start_after` (string, ISO 8601)
   - `start_before` (string, ISO 8601)
3. New endpoint `/v0/calendars/managed` with request/response schemas

### Implementation Priority

**Phase 1 (MVP - Required)**:

1. Add new post kinds to enum ✅
2. Add `parent` parameter to stream endpoint ✅
3. Basic jCal parsing (extract dtstart, dtend, summary) ✅
4. Neo4j indexes for kind and parent ✅

**Phase 2 (Enhanced)**: 5. Admin relationship indexing ✅ 6. Admin validation
for events ✅ 7. Date filtering (`start_after`, `start_before`) ✅ 8.
`/v0/calendars/managed` endpoint ✅

**Phase 3 (Optional)**: 9. Full jCal metadata extraction (location, status,
partstat) 10. Alarm support 11. Additional calendar-specific query optimizations

### Testing Requirements

1. **Unit Tests**: jCal parsing functions
2. **Integration Tests**:
   - Create calendar with admins
   - Create event as admin
   - Reject event from non-admin
   - Query events by parent
   - Query events by date range
   - Query managed calendars
3. **E2E Tests**: Full calendar workflow from frontend
