# Nexus Calendar Endpoints

This document specifies the Nexus indexer endpoints for aggregating and querying
calendar posts. Calendar components use the existing PubkyAppPost with
respective new `kind`'s.

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

| Operation                          | Endpoint                                                       | Status                  |
| ---------------------------------- | -------------------------------------------------------------- | ----------------------- |
| List all calendars                 | `GET /v0/stream/posts?kind=calendar`                           | ✅ Works now            |
| Get single calendar                | `GET /v0/post/{author_id}/{post_id}`                           | ✅ Works now            |
| List all events                    | `GET /v0/stream/posts?kind=event`                              | ✅ Works now            |
| List events in calendar            | `GET /v0/stream/posts?source=calendar_events&...`              | ⚠️ Needs Redis index    |
| List RSVPs for event               | `GET /v0/stream/posts?source=event_attendees&...`              | ⚠️ Needs Redis index    |
| Get user's calendars (owner/admin) | `GET /v0/calendars/managed?user_id={user_id}`                  | ⚠️ New endpoint         |
| Get user's RSVPs                   | `GET /v0/stream/posts?kind=attendee&source=author`             | ✅ Works now            |
| Filter events by date              | `GET /v0/stream/posts?kind=event&event_start_after={iso_date}` | ⚠️ Needs date filtering |

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

**Base URL (Development/MVP)**: `https://nexus.riginode.xyz/v0` or
`http://localhost:3000/v0`

### Design Principle

All calendar components are posts. Use existing post endpoints with `kind`
parameter to filter:

- `kind=calendar` - Calendar collections (with admin list in jCal)
- `kind=event` - Events (created by owner or admins)
- `kind=attendee` - RSVPs
- `kind=alarm` - Reminders (Lower priority)

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
2. **Redis Indexes** (CRITICAL for performance): Add sorted sets for calendar
   queries following existing patterns:
   - `["Posts", "CalendarEvents", "{calendar_author}", "{calendar_id}"]`
   - `["Posts", "EventAttendees", "{event_author}", "{event_id}"]`
3. **Parent URI Filtering**: Add `parent` query parameter to `/v0/stream/posts`
   endpoint
4. **StreamSource Extensions**: Add `CalendarEvents` and `EventAttendees`
   sources to leverage Redis indexes
5. **jCal Parsing**: Parse jCal content during `sync_put` and extract metadata
   (dtstart, dtend, summary, location, status, partstat)
6. **Admin List Caching**: Cache admin lists in Redis to avoid repeated jCal
   parsing during event validation
7. **Admin Relationships**: Extract `X-PUBKY-ADMIN` properties and create
   `ADMIN_OF` Neo4j relationships
8. **Admin Validation**: Validate that event authors are calendar owners or
   admins
9. **Date Filtering**: Add `event_start_after`/`event_start_before` parameters
   (renamed to avoid conflict with existing `start`/`end` timestamp params)
10. **New Endpoint**: Add `/v0/calendars/managed` endpoint for querying
    calendars by admin status
11. **Post Response Extension**: Add `admins` field to calendar post responses
    (extracted from cached data)

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
            "attachments": [
                "pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"
            ],
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

### GET /v0/stream/posts?source=calendar_events

List all events for a specific calendar using optimized Redis index.

**⚠️ NEW STREAM SOURCE** - Requires implementation of `CalendarEvents` source
and Redis sorted set indexing.

**Query Parameters**:

| Parameter            | Type    | Default | Description                                                                           |
| -------------------- | ------- | ------- | ------------------------------------------------------------------------------------- |
| `source`             | object  | -       | `{"source": "calendar_events", "calendar_author": "{author}", "calendar_id": "{id}"}` |
| `skip`               | integer | `0`     | Pagination offset                                                                     |
| `limit`              | integer | `20`    | Number of results                                                                     |
| `viewer_id`          | string  | -       | Viewer ID for personalized responses                                                  |
| `event_start_after`  | string  | -       | Filter events starting after date (ISO 8601)                                          |
| `event_start_before` | string  | -       | Filter events starting before date (ISO 8601)                                         |

**Implementation Details**:

This leverages a **Redis sorted set** for fast access:

- **Key**: `["Posts", "CalendarEvents", "{calendar_author}", "{calendar_id}"]`
- **Score**: `indexed_at` timestamp for timeline ordering
- **Members**: Event post IDs

This pattern mirrors the existing `POST_REPLIES_PER_POST_KEY_PARTS` but filters
by `kind=event` during indexing, avoiding the need for client-side filtering.

**Fallback**: When date filters are used, falls back to Neo4j Cypher query with
indexed `dtstart`/`dtend` properties.

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

### GET /v0/stream/posts?source=event_attendees

List all attendee RSVPs for a specific event using optimized Redis index.

**⚠️ NEW STREAM SOURCE** - Requires implementation of `EventAttendees` source
and Redis sorted set indexing.

**Query Parameters**:

| Parameter   | Type    | Default | Description                                                                     |
| ----------- | ------- | ------- | ------------------------------------------------------------------------------- |
| `source`    | object  | -       | `{"source": "event_attendees", "event_author": "{author}", "event_id": "{id}"}` |
| `skip`      | integer | `0`     | Pagination offset                                                               |
| `limit`     | integer | `50`    | Number of results                                                               |
| `viewer_id` | string  | -       | Viewer ID for personalized responses                                            |
| `partstat`  | string  | -       | Filter by participation status (ACCEPTED, DECLINED, TENTATIVE)                  |

**Implementation Details**:

This leverages a **Redis sorted set** for fast access:

- **Key**: `["Posts", "EventAttendees", "{event_author}", "{event_id}"]`
- **Score**: `indexed_at` timestamp for timeline ordering
- **Members**: Attendee post IDs

**Note**:

- Filtering by `partstat` requires parsing jCal during `sync_put` and indexing
  as a Neo4j property
- When `partstat` filter is used, falls back to Neo4j Cypher query

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

**Get upcoming events** (⚠️ requires date filtering extension):

```
GET /v0/stream/posts?kind=event&event_start_after=2025-10-14T00:00:00Z
```

_Note: Parameter renamed from `start_after` to `event_start_after` to avoid
conflict with existing `start` parameter_

**Get events for a specific calendar** (⚠️ requires new Redis index):

```
GET /v0/stream/posts?source={"source":"calendar_events","calendar_author":"satoshi","calendar_id":"0033RCZXVEPNG"}
```

_This uses dedicated Redis sorted set for fast access, no client-side filtering
needed_

**Get a specific calendar post:**

```
GET /v0/post/satoshi/0033RCZXVEPNG?viewer_id={viewer_id}
```

**Get RSVPs for a specific event** (⚠️ requires new Redis index):

```
GET /v0/stream/posts?source={"source":"event_attendees","event_author":"satoshi","event_id":"0033SCZXVEPNG"}
```

**Get accepted RSVPs only** (falls back to Neo4j query):

```
GET /v0/stream/posts?source={"source":"event_attendees","event_author":"satoshi","event_id":"0033SCZXVEPNG"}&partstat=ACCEPTED
```

---

## Redis Indexing Strategy

### Overview

Nexus uses a **hybrid Redis+Neo4j indexing strategy** for performance:

1. **Redis sorted sets** for fast access to common query patterns
2. **Neo4j Cypher queries** for complex filtering (dates, admin status, etc.)

Calendar endpoints follow the same pattern as existing post endpoints to ensure
consistent performance.

### Required Redis Indexes

#### 1. Calendar Events Index

**Purpose**: Fast retrieval of events in a specific calendar

**Key Pattern**:
`["Posts", "CalendarEvents", "{calendar_author}", "{calendar_id}"]`

**Example**: `Posts:CalendarEvents:satoshi:0033RCZXVEPNG`

**Members**: Event post IDs (e.g., `satoshi:0033SCZXVEPNG`)

**Score**: `indexed_at` timestamp (for timeline ordering)

**Population**: During `sync_put` of posts with `kind=event` and valid `parent`:

```rust
// Add to calendar's event sorted set
if post.kind == PubkyAppPostKind::Event && post.parent.is_some() {
    let (calendar_author, calendar_id) = parse_parent_uri(&post.parent)?;
    let key_parts = ["Posts", "CalendarEvents", &calendar_author, &calendar_id];
    redis_add_to_sorted_set(&key_parts, &post_id, indexed_at).await?;
}
```

#### 2. Event Attendees Index

**Purpose**: Fast retrieval of RSVPs for a specific event

**Key Pattern**: `["Posts", "EventAttendees", "{event_author}", "{event_id}"]`

**Example**: `Posts:EventAttendees:satoshi:0033SCZXVEPNG`

**Members**: Attendee post IDs (e.g., `alice:0033XYZABC`)

**Score**: `indexed_at` timestamp (for timeline ordering)

**Population**: During `sync_put` of posts with `kind=attendee` and valid
`parent`:

```rust
// Add to event's attendee sorted set
if post.kind == PubkyAppPostKind::Attendee && post.parent.is_some() {
    let (event_author, event_id) = parse_parent_uri(&post.parent)?;
    let key_parts = ["Posts", "EventAttendees", &event_author, &event_id];
    redis_add_to_sorted_set(&key_parts, &post_id, indexed_at).await?;
}
```

#### 3. Admin List Cache

**Purpose**: Fast validation of event authors without parsing jCal

**Key Pattern**: `["Calendar", "Admins", "{calendar_author}", "{calendar_id}"]`

**Example**: `Calendar:Admins:satoshi:0033RCZXVEPNG`

**Type**: Redis SET (not sorted set)

**Members**: Admin user IDs (e.g., `hal`, `adam-back`)

**Population**: During `sync_put` of calendar posts:

```rust
// Cache admin list for fast validation
if post.kind == PubkyAppPostKind::Calendar {
    let jcal = parse_jcal(&post.content)?;
    let admin_uris = extract_jcal_properties(&jcal[1], "x-pubky-admin")?;
    let admin_ids: Vec<String> = admin_uris.iter()
        .map(|uri| parse_user_id_from_uri(uri))
        .collect();
    
    let key_parts = ["Calendar", "Admins", &author_id, &post_id];
    redis_set_members(&key_parts, &admin_ids).await?;
}
```

### Query Optimization Path

When processing a calendar query, Nexus follows this decision tree:

```
1. Can use Redis index?
   └─ Yes (no date filters, no partstat filter)
      └─ Query Redis sorted set → Fast O(log N) retrieval
   └─ No (has date/partstat filters OR index doesn't exist)
      └─ Fall back to Neo4j Cypher → Query with indexed properties
```

This is implemented in `can_use_index()` function extension:

```rust
fn can_use_index(
    sorting: &StreamSorting,
    source: &StreamSource,
    tags: &Option<Vec<String>>,
    kind: &Option<PubkyAppPostKind>,
    has_date_filter: bool,
    has_partstat_filter: bool,
) -> bool {
    // Can't use Redis if we need to filter by jCal fields
    if has_date_filter || has_partstat_filter {
        return false;
    }
    
    // ... existing checks ...
    
    // NEW: Calendar-specific indexes
    match (sorting, source) {
        (StreamSorting::Timeline, StreamSource::CalendarEvents { .. }) => true,
        (StreamSorting::Timeline, StreamSource::EventAttendees { .. }) => true,
        _ => false,
    }
}
```

### Performance Characteristics

| Query Type                     | Data Structure   | Time Complexity | Notes                           |
| ------------------------------ | ---------------- | --------------- | ------------------------------- |
| Get events in calendar         | Redis Sorted Set | O(log N + M)    | N = total events, M = page size |
| Get RSVPs for event            | Redis Sorted Set | O(log N + M)    | N = total RSVPs, M = page size  |
| Validate event author is admin | Redis Set        | O(1)            | Cached admin list lookup        |
| Filter events by date          | Neo4j Cypher     | O(N log N)      | Uses dtstart index              |
| Filter RSVPs by partstat       | Neo4j Cypher     | O(N log N)      | Uses partstat index             |

---

## Nexus Implementation Requirements

### Post Handler Modifications

**In `nexus-watcher/src/events/handlers/post.rs`:**

The `sync_put` handler needs to be extended to handle calendar posts. Follow the
existing dual-write pattern (Neo4j + Redis).

#### 1. Parse jCal Content

When `kind` is `calendar`, `event`, `attendee`, or `alarm`, parse jCal and
extract metadata:

```rust
async fn sync_put(
    post: PubkyAppPost,
    author_id: PubkyId,
    post_id: String,
) -> Result<(), DynError> {
    // ... existing code ...
    
    // NEW: Handle calendar posts
    if matches!(post.kind, PubkyAppPostKind::Calendar | PubkyAppPostKind::Event | PubkyAppPostKind::Attendee) {
        handle_calendar_post(&post, &author_id, &post_id, &post_details).await?;
    }
    
    // ... existing code ...
}
```

#### 2. Calendar Post Handler

```rust
async fn handle_calendar_post(
    post: &PubkyAppPost,
    author_id: &str,
    post_id: &str,
    post_details: &PostDetails,
) -> Result<()> {
    let jcal: Vec<serde_json::Value> = serde_json::from_str(&post.content)?;
    let properties = &jcal[1];
    
    match post.kind {
        PubkyAppPostKind::Calendar => {
            // Extract and cache admin list in Redis
            let admin_uris = extract_jcal_properties(properties, "x-pubky-admin")?;
            let admin_ids: Vec<String> = admin_uris.iter()
                .map(|uri| parse_user_id_from_uri(uri))
                .collect::<Result<Vec<_>>>()?;
            
            // Cache in Redis for fast validation
            let key_parts = ["Calendar", "Admins", author_id, post_id];
            PostStream::cache_calendar_admins(&key_parts, &admin_ids).await?;
            
            // Create ADMIN_OF relationships in Neo4j
            for admin_id in &admin_ids {
                create_admin_relationship(admin_id, &post_details.uri).await?;
            }
            
            // Extract calendar name for metadata
            if let Ok(name) = extract_jcal_property(properties, "name") {
                post_details.update_metadata("jcal_name", name).await?;
            }
        },
        
        PubkyAppPostKind::Event => {
            // Validate author is owner or admin of parent calendar
            validate_event_author(post, author_id).await?;
            
            // Extract and index event metadata
            let dtstart = extract_jcal_date(properties, "dtstart")?;
            let dtend = extract_jcal_date(properties, "dtend")?;
            let summary = extract_jcal_property(properties, "summary")?;
            let location = extract_jcal_property(properties, "location").ok();
            let status = extract_jcal_property(properties, "status").ok();
            
            // Store in Neo4j for date-range queries
            post_details.update_event_metadata(dtstart, dtend, summary, location, status).await?;
            
            // Add to calendar's event Redis index
            if let Some(parent_uri) = &post.parent {
                let (cal_author, cal_id) = parse_post_uri(parent_uri)?;
                let key_parts = ["Posts", "CalendarEvents", &cal_author, &cal_id];
                PostStream::add_to_sorted_set(&key_parts, &format!("{}:{}", author_id, post_id), post_details.indexed_at).await?;
            }
        },
        
        PubkyAppPostKind::Attendee => {
            // Extract PARTSTAT from attendee property
            let partstat = extract_jcal_attendee_partstat(properties)?;
            post_details.update_metadata("partstat", &partstat).await?;
            
            // Add to event's attendee Redis index
            if let Some(parent_uri) = &post.parent {
                let (event_author, event_id) = parse_post_uri(parent_uri)?;
                let key_parts = ["Posts", "EventAttendees", &event_author, &event_id];
                PostStream::add_to_sorted_set(&key_parts, &format!("{}:{}", author_id, post_id), post_details.indexed_at).await?;
            }
        },
        
        _ => {}
    }
    
    Ok(())
}
```

#### 3. Admin Validation

```rust
async fn validate_event_author(event: &PubkyAppPost, author_id: &str) -> Result<()> {
    let parent_uri = event.parent.as_ref()
        .ok_or("Event must have parent calendar")?;
    
    let (calendar_author, calendar_id) = parse_post_uri(parent_uri)?;
    
    // Check if author is calendar owner
    if calendar_author == author_id {
        return Ok(());
    }
    
    // Check cached admin list in Redis (fast!)
    let key_parts = ["Calendar", "Admins", &calendar_author, &calendar_id];
    let is_admin = PostStream::check_set_membership(&key_parts, author_id).await?;
    
    if !is_admin {
        return Err("Event author must be calendar owner or admin".into());
    }
    
    Ok(())
}
```

#### 4. jCal Parsing Utilities

Add helper functions to parse jCal format:

```rust
fn extract_jcal_property(properties: &[Value], name: &str) -> Result<String> {
    properties.iter()
        .find(|prop| prop[0].as_str() == Some(name))
        .and_then(|prop| prop[3].as_str())
        .map(String::from)
        .ok_or_else(|| format!("Missing {} property", name).into())
}

fn extract_jcal_properties(properties: &[Value], name: &str) -> Result<Vec<String>> {
    Ok(properties.iter()
        .filter(|prop| prop[0].as_str() == Some(name))
        .filter_map(|prop| prop[3].as_str())
        .map(String::from)
        .collect())
}

fn extract_jcal_date(properties: &[Value], name: &str) -> Result<i64> {
    let date_str = extract_jcal_property(properties, name)?;
    // Parse ISO 8601 or other formats and convert to Unix timestamp
    parse_ical_datetime(&date_str)
}
```

#### 5. Redis Helper Methods

Add to `nexus-common/src/models/post/stream.rs`:

````rust
impl PostStream {
    // Cache calendar admin list in Redis SET
    pub async fn cache_calendar_admins(
        key_parts: &[&str],
        admin_ids: &[String],
    ) -> Result<(), DynError> {
        let redis_key = key_parts.join(":");
        let mut conn = get_redis_client().get_multiplexed_async_connection().await?;
        
        // Store as Redis SET for fast membership checks
        for admin_id in admin_ids {
            conn.sadd::<_, _, ()>(&redis_key, admin_id).await?;
        }
        
        Ok(())
    }
    
    // Check if user is in calendar's admin set
    pub async fn check_set_membership(
        key_parts: &[&str],
        member: &str,
    ) -> Result<bool, DynError> {
        let redis_key = key_parts.join(":");
        let mut conn = get_redis_client().get_multiplexed_async_connection().await?;
        
        Ok(conn.sismember(&redis_key, member).await?)
    }
}

### Graph Schema Additions

**Neo4j Relationship Indexing:**

```cypher
// Create index for admin relationships
CREATE INDEX admin_of_idx FOR ()-[r:ADMIN_OF]-() ON (r.created_at);

// Create index for calendar/event metadata
CREATE INDEX post_kind_idx FOR (p:Post) ON (p.kind);
CREATE INDEX post_dtstart_idx FOR (p:Post) ON (p.dtstart);
CREATE INDEX post_dtend_idx FOR (p:Post) ON (p.dtend);
````

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

**In `pubky-app-specs/src/stream.rs`:**

First, extend the `StreamSource` enum to include calendar-specific sources:

```rust
#[derive(Debug, Clone, Deserialize, Serialize, ToSchema)]
#[serde(tag = "source", rename_all = "snake_case")]
pub enum StreamSource {
    // ... existing variants ...
    
    // NEW: Calendar-specific sources
    #[serde(rename = "calendar_events")]
    CalendarEvents {
        calendar_author: String,
        calendar_id: String,
    },
    
    #[serde(rename = "event_attendees")]
    EventAttendees {
        event_author: String,
        event_id: String,
    },
}
```

**In `nexus-webapi/src/routes/v0/stream/posts.rs`:**

Extend the existing `/v0/stream/posts` endpoint handler to support calendar
parameters:

```rust
#[derive(Deserialize, Debug, ToSchema)]
pub struct PostStreamQuery {
    // Existing parameters (no changes)
    #[serde(flatten, default)]
    pub source: Option<StreamSource>,     // Now includes CalendarEvents, EventAttendees
    #[serde(flatten)]
    pub pagination: Pagination,
    pub order: Option<SortOrder>,
    pub sorting: Option<StreamSorting>,
    pub viewer_id: Option<String>,
    #[serde(default, deserialize_with = "deserialize_comma_separated")]
    pub tags: Option<Vec<String>>,
    pub kind: Option<PubkyAppPostKind>,   // Now includes calendar, event, attendee
    
    // NEW: Calendar-specific date filtering (renamed to avoid conflicts)
    pub event_start_after: Option<String>,   // ISO 8601 date for dtstart filtering
    pub event_start_before: Option<String>,  // ISO 8601 date for dtstart filtering
    pub partstat: Option<String>,            // ACCEPTED, DECLINED, TENTATIVE
}

pub async fn stream_posts_handler(
    Query(query): Query<PostStreamQuery>
) -> Result<Json<Vec<PostView>>, ApiError> {
    // ... existing validation ...
    
    // NEW: Determine if we need date/partstat filtering
    let has_date_filter = query.event_start_after.is_some() || query.event_start_before.is_some();
    let has_partstat_filter = query.partstat.is_some();
    
    // Check if we can use Redis index or need to fall back to Neo4j
    let use_redis = can_use_index(
        &sorting,
        &source,
        &query.tags,
        &query.kind,
        has_date_filter,
        has_partstat_filter,
    );
    
    if use_redis {
        // Use Redis sorted set (fast path)
        match source {
            StreamSource::CalendarEvents { calendar_author, calendar_id } => {
                PostStream::get_calendar_events(&calendar_author, &calendar_id, /* ... */).await?
            },
            StreamSource::EventAttendees { event_author, event_id } => {
                PostStream::get_event_attendees(&event_author, &event_id, /* ... */).await?
            },
            // ... existing sources ...
            _ => { /* fallback */ }
        }
    } else {
        // Fall back to Neo4j query (complex filtering)
        PostStream::query_with_filters(
            query.kind,
            query.event_start_after,
            query.event_start_before,
            query.partstat,
            /* ... */
        ).await?
    }
}
```

**In `nexus-common/src/models/post/stream.rs`:**

Extend the `can_use_index` function:

```rust
fn can_use_index(
    sorting: &StreamSorting,
    source: &StreamSource,
    tags: &Option<Vec<String>>,
    kind: &Option<PubkyAppPostKind>,
    has_date_filter: bool,
    has_partstat_filter: bool,
) -> bool {
    // Can't use Redis if we need jCal field filtering
    if has_date_filter || has_partstat_filter {
        return false;
    }
    
    // ... existing checks ...
    
    // NEW: Calendar-specific indexes (timeline sorting only)
    match (sorting, source, tags) {
        (StreamSorting::Timeline, StreamSource::CalendarEvents { .. }, None) => true,
        (StreamSorting::Timeline, StreamSource::EventAttendees { .. }, None) => true,
        _ => false,
    }
}
```

Add new retrieval methods:

```rust
impl PostStream {
    pub async fn get_calendar_events(
        calendar_author: &str,
        calendar_id: &str,
        order: SortOrder,
        start: Option<f64>,
        end: Option<f64>,
        skip: Option<usize>,
        limit: Option<usize>,
    ) -> Result<Vec<String>, DynError> {
        let key_parts = ["Posts", "CalendarEvents", calendar_author, calendar_id];
        let events = Self::try_from_index_sorted_set(
            &key_parts,
            start,
            end,
            skip,
            limit,
            order,
            None,
        ).await?;
        
        Ok(events.map_or(Vec::new(), |entries| {
            entries.into_iter().map(|(post_id, _)| post_id).collect()
        }))
    }
    
    pub async fn get_event_attendees(
        event_author: &str,
        event_id: &str,
        order: SortOrder,
        start: Option<f64>,
        end: Option<f64>,
        skip: Option<usize>,
        limit: Option<usize>,
    ) -> Result<Vec<String>, DynError> {
        let key_parts = ["Posts", "EventAttendees", event_author, event_id];
        let attendees = Self::try_from_index_sorted_set(
            &key_parts,
            start,
            end,
            skip,
            limit,
            order,
            None,
        ).await?;
        
        Ok(attendees.map_or(Vec::new(), |entries| {
            entries.into_iter().map(|(post_id, _)| post_id).collect()
        }))
    }
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
            [
                "uid",
                {},
                "text",
                `pubky://${userId}/pub/pubky.app/posts/${eventId}`,
            ],
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

1. **Add new post kinds to enum** - `calendar`, `event`, `attendee` to
   `PubkyAppPostKind`
2. **Add Redis sorted set indexes** - `CalendarEvents` and `EventAttendees` keys
3. **Add StreamSource variants** - `CalendarEvents` and `EventAttendees`
4. **Basic jCal parsing** - Extract dtstart, dtend, summary during `sync_put`
5. **Redis dual-write** - Update sorted sets when indexing calendar posts
6. **Extend `can_use_index()`** - Recognize calendar query patterns
7. **Neo4j indexes** - Add indexes for `kind`, `dtstart`, `dtend` properties
8. **Admin list caching** - Cache admin lists in Redis SETs
9. **Admin validation** - Validate event authors using cached admin lists
10. **ADMIN_OF relationships** - Create Neo4j relationships for admin queries
11. **`/v0/calendars/managed` endpoint** - Query calendars by admin status
12. **Date range filtering** - Add `event_start_after`/`event_start_before`
    parameters
13. **PARTSTAT filtering** - Add `partstat` parameter for RSVP filtering
14. **Neo4j fallback queries** - Implement Cypher queries for complex filters
15. **Full jCal metadata** - Extract location, status, and other optional fields
16. Alarm support (`kind=alarm`)
17. Recurring event support
18. Timezone handling improvements
19. Performance tuning and caching optimizations

### Critical Performance Notes

⚠️ **DO NOT skip Redis indexing (Phase 1)** - Without Redis sorted sets,
calendar queries will be slow and put unnecessary load on Neo4j. The hybrid
indexing strategy is what makes Nexus performant.

⚠️ **DO NOT use `post_replies` workaround in production** - This returns all
child posts (including comments), requiring client-side filtering. Always use
dedicated `CalendarEvents` and `EventAttendees` sources.

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
