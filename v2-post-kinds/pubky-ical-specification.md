# Pubky iCalendar Specification (v2.0)

## Index

- [Introduction](#introduction)
- [Architectural Changes from v1.0](#architectural-changes-from-v10)
- [Background and Motivation](#background-and-motivation)
- [RFC Tag Usage Throughout This Specification](#rfc-tag-usage-throughout-this-specification)
- [Homeserver Storage Structure](#homeserver-storage-structure)
- [Data Models and Schemas](#data-models-and-schemas)
- [Graph Relationships](#graph-relationships)
- [Future Extensions and Considerations](#future-extensions-and-considerations)

## Introduction

This specification defines the integration of industry-standard iCalendar
protocols (RFC 5545, RFC 7265, RFC 4791, RFC 5546, RFC 7986, RFC 9073) into the
Pubky ecosystem using **PubkyAppPost** as the base type. Calendar components are
stored as posts with specific `kind` values, enabling them to leverage existing
Pubky infrastructure for relationships, tags, bookmarks, and social
interactions.

This document follows the pubky-app-specs structure and conventions, extending
the post system to support calendar collections, events, scheduling, and RSVP
workflows. All calendar data uses the jCal JSON format (RFC 7265) stored in the
post `content` field, with pubky:// URLs for addressing users and resources.

## Architectural Changes from v1.0

**Key Changes:**

1. **Post-Based Architecture**: Calendar components are now `PubkyAppPost`
   entities with specific `kind` values instead of separate `PubkyAppX` types
2. **New Post Kinds**: `calendar`, `event`, `attendee`, `alarm`
3. **Relationship Model**: Uses post `parent` field for hierarchical
   relationships (event → calendar)
4. **jCal-Based Admins**: Calendar admins are stored as multiple `X-PUBKY-ADMIN`
   properties in the calendar's jCal content
5. **Unified Storage Path**: All calendar components stored under
   `/pub/pubky.app/posts/:post_id`
6. **Existing Infrastructure**: Leverages existing Nexus post indexing, graph
   relationships, and API patterns

**Benefits:**

- Reuses existing Nexus handlers for posts (create, delete, edit)
- Enables comments on events (replies to event posts)
- Supports tags on calendars/events using existing tag system
- Bookmarkable events
- Counts (replies, reposts, tags) work automatically
- Filtering by post kind in existing streams
- Admin list controlled by calendar owner through post edits

---

## Background and Motivation

By building calendar functionality on top of the existing post system, we gain:

- **Infrastructure Reuse**: Leverage existing post handlers, graph
  relationships, and API endpoints
- **Social Integration**: Events become first-class social objects with comments
  (replies), tags, and bookmarks
- **Interoperability**: Maintains iCalendar RFC compliance while using Pubky's
  native data structures
- **Simplicity**: No need for separate event processors, just extend existing
  post logic
- **Flexibility**: Calendar admins managed through jCal properties, enabling
  updates via standard post editing

The Nexus indexer aggregates calendar posts based on admin lists in jCal and
parent relationships, similar to how it handles post replies and threads.

---

## RFC Tag Usage Throughout This Specification

### Property Tags by RFC Source

**From RFC 5545 (Core iCalendar):**

- `UID`, `DTSTAMP`, `DTSTART`/`DTEND`, `SUMMARY`, `DESCRIPTION`, `LOCATION`,
  `ORGANIZER`, `ATTENDEE`, `CATEGORIES`, `STATUS`, `RRULE`

**From RFC 7265 (jCal JSON):**

- JSON array structure stored in post `content` field

**From RFC 5546 (iTIP Scheduling):**

- `METHOD`, `PARTSTAT`, `ROLE` (used in attendee posts)

**From RFC 7986 (New Properties):**

- `NAME`, `COLOR`, `IMAGE`, `CONFERENCE`

**From RFC 9073 (Event Publishing):**

- `PARTICIPANT`, `STRUCTURED-LOCATION`, `PARTICIPANT-TYPE`

**Pubky Extensions (X-Properties per RFC 5545 Section 3.8.8.2):**

- `X-PUBKY-ADMIN`: Repeatable property containing pubky:// URI of calendar admin
- Calendar reference via post `parent` field

---

## Homeserver Storage Structure

### Path Organization

All calendar components are stored as posts under the unified post path:

```
/pub/pubky.app/posts/:post_id
```

Each post has a `kind` field indicating its calendar type:

- `calendar` - Calendar collection
- `event` - Calendar event
- `attendee` - RSVP/attendance record
- `alarm` - User-defined reminder

### File Naming Conventions

- **Post ID**: Timestamp-based (13-character Crockford Base32)
- All calendar data stored in jCal format within the post `content` field
- UID preserved within jCal for iCalendar compatibility

### Component Relationships

Components reference each other through post relationships:

- **Event** → **Calendar**: Uses post `parent` field pointing to calendar post
  URI
- **Attendee** → **Event**: Uses post `parent` field pointing to event post URI
- **Alarm** → **Event/Calendar**: Uses post `parent` field

### Example File Paths

```
pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG  (calendar post with admin list)
pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG  (event post, parent=calendar, author=owner)
pubky://hal/pub/pubky.app/posts/0033TD0XVEPNG     (event post, parent=calendar, author=admin)
pubky://alice/pub/pubky.app/posts/0033UCZXVEPNG   (attendee post, parent=event)
pubky://bob/pub/pubky.app/posts/0033VCZXVEPNG     (alarm post, parent=event)
```

### Nexus Aggregation Logic

Nexus aggregates calendar components using existing post infrastructure:

1. **Calendar Discovery**: Query posts where `kind = "calendar"`
2. **Admin Discovery**: Parse `X-PUBKY-ADMIN` properties from calendar's jCal
   content
3. **Event Aggregation**: For each calendar, query posts where:
   - `kind = "event"`
   - `parent` matches the calendar URI
   - Post author is either:
     - Calendar owner (post author of calendar), OR
     - Listed in calendar's `X-PUBKY-ADMIN` properties
4. **Attendee/Alarm Aggregation**: Query posts where `kind = "attendee"/"alarm"`
   and `parent` matches event URI

---

## Data Models and Schemas

### Calendar Post (kind: "calendar")

**Path**: `/pub/pubky.app/posts/:post_id`

**PubkyAppPost Fields:**

- `content` (required): jCal JSON string with `X-PUBKY-ADMIN` properties
- `kind` (required): `"calendar"`
- `parent`: `null` (calendars are top-level)
- `embed`: `null`
- `attachments`: Optional array of image URIs (calendar logo)

**jCal Structure in `content` field:**

```json
{
  "content": "[\"vcalendar\",[[\"prodid\",{},\"text\",\"-//Pubky//Pubky Calendar 1.0//EN\"],[\"version\",{},\"text\",\"2.0\"],[\"calscale\",{},\"text\",\"GREGORIAN\"],[\"method\",{},\"text\",\"PUBLISH\"],[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG\"],[\"name\",{},\"text\",\"Dezentralschweiz Meetups\"],[\"description\",{},\"text\",\"Bitcoin meetups across Switzerland\"],[\"color\",{},\"text\",\"#F7931A\"],[\"categories\",{},\"text\",[\"bitcoin\", \"meetups\", \"decentralization\"]],[\"x-pubky-admins\",{},\"uri\",[\"pubky://satoshi\", \"pubky://adam-back\"]]],[]]",
  "kind": "calendar",
  "parent": null,
  "embed": null,
  "attachments": ["pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"]
}
```

**Admin Management:**

Admins are defined by the calendar owner (the post author) through repeatable
`X-PUBKY-ADMIN` properties in the jCal content. To add/remove admins, the
calendar owner edits the post and updates the property list.

**Admin Permissions:**

- Calendar owner (post author): Can edit calendar, add/remove admins, create
  events
- Listed admins (in `X-PUBKY-ADMIN`): Can create events for the calendar (but
  cannot edit calendar or manage admins)

**Example jCal with Multiple Admins:**

```json
[
  "vcalendar",
  [
    ["prodid", {}, "text", "-//Pubky//Pubky Calendar 1.0//EN"],
    ["version", {}, "text", "2.0"],
    ["uid", {}, "text", "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG"],
    ["name", {}, "text", "Bitcoin Switzerland Events"],
    ["x-pubky-admin", {}, "uri", "pubky://hal"],
    ["x-pubky-admin", {}, "uri", "pubky://adam-back"],
    ["x-pubky-admin", {}, "uri", "pubky://decentralschweiz"]
  ],
  []
]
```

---

### Event Post (kind: "event")

**Path**: `/pub/pubky.app/posts/:post_id`

**PubkyAppPost Fields:**

- `content` (required): jCal JSON string (VEVENT component)
- `kind` (required): `"event"`
- `parent` (required): URI of calendar post
  (`pubky://.../.../posts/:calendar_id`)
- `embed`: `null`
- `attachments`: Optional array of image/file URIs

**jCal Structure in `content` field:**

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-01T12:15:00Z\"],[\"dtstart\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-09T19:00:00\"],[\"dtend\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-09T22:00:00\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Zürich\"],[\"description\",{},\"text\",\"Weekly Bitcoin meetup\"],[\"location\",{},\"text\",\"Insider Bar\"],[\"organizer\",{\"cn\":\"Satoshi\"},\"cal-address\",\"pubky://satoshi\"],[\"status\",{},\"text\",\"CONFIRMED\"]],[]]",
  "kind": "event",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": ["pubky://satoshi/pub/pubky.app/files/0033EVENT01"]
}
```

**Validation**: Event creator (post author) must be either:

1. The calendar owner (author of parent calendar post), OR
2. Listed in the parent calendar's `X-PUBKY-ADMIN` properties

---

### Attendee Post (kind: "attendee")

**Path**: `/pub/pubky.app/posts/:post_id`

**PubkyAppPost Fields:**

- `content` (required): jCal JSON string (ATTENDEE component)
- `kind` (required): `"attendee"`
- `parent` (required): URI of event post
- `embed`: `null`
- `attachments`: `null`

**jCal Structure:**

```json
{
  "content": "[\"attendee\",[[\"uid\",{},\"text\",\"pubky://alice/pub/pubky.app/posts/0033ZCZXVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-02T09:30:00Z\"],[\"attendee\",{\"cn\":\"Alice\",\"role\":\"REQ-PARTICIPANT\",\"partstat\":\"ACCEPTED\",\"rsvp\":\"TRUE\"},\"cal-address\",\"pubky://alice\"]],[]]",
  "kind": "attendee",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "embed": null,
  "attachments": null
}
```

---

### Alarm Post (kind: "alarm")

**Path**: `/pub/pubky.app/posts/:post_id`

**PubkyAppPost Fields:**

- `content` (required): jCal JSON string (VALARM component)
- `kind` (required): `"alarm"`
- `parent` (required): URI of event or calendar post
- `embed`: `null`
- `attachments`: `null`

**jCal Structure:**

```json
{
  "content": "[\"valarm\",[[\"uid\",{},\"text\",\"pubky://bob/pub/pubky.app/posts/0033WCZXVEPNG\"],[\"action\",{},\"text\",\"DISPLAY\"],[\"trigger\",{\"related\":\"START\"},\"duration\",\"-PT15M\"],[\"description\",{},\"text\",\"Bitcoin Meetup in 15 minutes\"]],[]]",
  "kind": "alarm",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "embed": null,
  "attachments": null
}
```

---

## Graph Relationships

Calendar posts leverage existing Nexus graph relationships:

### Node Types

- `Post` nodes with `kind` property: `calendar`, `event`, `attendee`, `alarm`
- `User` nodes (existing)
- `Tag` nodes (existing, for semantic tagging like #bitcoin)

### Additional Properties on Calendar Posts

**Calendar Post Properties:**

```
- kind: "calendar"
- jcal_name: extracted from jCal
- admin_uris: JSON array of pubky:// URIs extracted from X-PUBKY-ADMIN properties
```

**Event Post Properties:**

```
- kind: "event"
- dtstart: ISO timestamp
- dtend: ISO timestamp
- summary: text
- location: text
- status: text
```

### Relationship Types (Existing)

- `AUTHORED`: User → Post (all calendar posts)
- `REPLIED`: Post → Post (for comments on events)
- `PARENT_OF`: Post → Post (calendar ← event ← attendee/alarm)
- `TAGGED`: User → Post (semantic tags like #bitcoin, NOT for admin
  relationships)
- `BOOKMARKED`: User → Post (bookmark events)

### New Relationship Type for Admin Indexing

- `ADMIN_OF`: User → Post (indexed from `X-PUBKY-ADMIN` properties)

This allows efficient Cypher queries for finding calendars where a user is admin
without parsing jCal each time.

### Calendar-Specific Queries

```cypher
// Find all events for a calendar (including from admins)
MATCH (calendar:Post {kind: "calendar", id: $calendar_id})
MATCH (event:Post {kind: "event"})-[:PARENT_OF]->(calendar)
WHERE (event)-[:AUTHORED]->(:User {id: calendar.author_id})
   OR EXISTS {
     MATCH (admin:User)-[:ADMIN_OF]->(calendar)
     WHERE (event)-[:AUTHORED]->(admin)
   }
RETURN event

// Find all calendars where user is admin or owner
MATCH (user:User {id: $user_id})
WHERE (user)-[:AUTHORED]->(calendar:Post {kind: "calendar"})
   OR (user)-[:ADMIN_OF]->(calendar)
RETURN calendar

// Find admins of a calendar
MATCH (admin:User)-[:ADMIN_OF]->(calendar:Post {id: $calendar_id})
RETURN admin
```

---

## Future Extensions and Considerations

### Enhanced Access Control

- Multiple admin levels via additional X-properties (X-PUBKY-ADMIN-LEVEL)
- Event submission approval workflows
- Granular permissions (read, write, admin)

### Rich Event Features (RFC 9073)

- `PARTICIPANT` sub-components for speakers/sponsors
- `STYLED-DESCRIPTION` for rich text
- `STRUCTURED-LOCATION` with OSM integration

### Advanced Scheduling (RFC 5546)

- iTIP message flows via special post kinds
- Free/busy time aggregation
- Delegation workflows

### Extended Alarm Capabilities (RFC 9074)

- `PROXIMITY` triggers (location-based)
- Complex alarm relationships
- Snooze tracking

---

## References

### Core RFC Standards

- [RFC 5545 - iCalendar](https://www.rfc-editor.org/rfc/rfc5545.html)
- [RFC 7265 - jCal](https://www.rfc-editor.org/rfc/rfc7265.html)
- [RFC 4791 - CalDAV](https://www.rfc-editor.org/rfc/rfc4791.html)
- [RFC 5546 - iTIP](https://www.rfc-editor.org/rfc/rfc5546.html)
- [RFC 7986 - New Properties](https://www.rfc-editor.org/rfc/rfc7986.html)
- [RFC 9073 - Event Publishing Extensions](https://www.rfc-editor.org/rfc/rfc9073.html)
