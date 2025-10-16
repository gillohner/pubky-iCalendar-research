# Pubky iCalendar Specification (v3.0 - Explicit Fields)

## Index

- [Introduction](#introduction)
- [Architectural Approach](#architectural-approach)
- [Implementation Options](#implementation-options)
- [Type Definitions](#type-definitions)
- [RFC Field Tables](#rfc-field-tables)
- [Field Validation Rules](#field-validation-rules)
- [Homeserver Storage Structure](#homeserver-storage-structure)
- [Graph Relationships](#graph-relationships)
- [Integration with Existing Types](#integration-with-existing-types)

## Introduction

This specification defines the integration of industry-standard iCalendar
protocols (RFC 5545, RFC 7265, RFC 4791, RFC 5546, RFC 7986, RFC 9073) into the
Pubky ecosystem using **explicit typed fields** instead of jCal JSON content.

**Key Difference from v2.0:** Rather than storing calendar data as jCal JSON in
a `content` field, v3.0 defines explicit Rust struct fields for each RFC
property, providing compile-time type safety and clear schema visibility.

This approach can be used in combination with the approaches outlined in v1 or
v2 as well instead of storing the data as jCal JSON in a `content` field.

**Trade-offs:**

- ✅ **Pros**: Compile-time validation, explicit schema, no runtime jCal
  parsing, better IDE support
- ❌ **Cons**: Less flexible for future RFC extensions, requires type updates
  for new fields.

## Architectural Approach

### Core Type

```rust
pub struct PubkyAppEventPost {
    pub kind: PubkyAppEventPostKind,
    pub parent: Option<String>,  // URI to parent (event → calendar)
    // ... explicit fields based on implementation option chosen
}

pub enum PubkyAppEventPostKind {
    Calendar,
    Event,
    Attendee,
}
```

### Storage Path

All event-posts stored at: `/pub/pubky.app/event-posts/:id`

### Field Format Standards

- **Dates**: Unix timestamp microseconds (i64) - matches `indexed_at` format
- **Multi-value fields**: JSON array strings, e.g., `["category1", "category2"]`
- **RRULE**: RFC 5545 string format, e.g., `"FREQ=WEEKLY;BYDAY=MO,WE,FR"`
- **Structured data**: JSON object strings with standardized keys
- **URIs**: Pubky URIs for user/resource references, e.g.,
  `"pubky://<user_id>/pub/pubky.app/..."`

## Implementation Options

This specification provides **three implementation options**:

### Option 1: Complete RFC Implementation (~70 fields)

**Full compliance** with RFC 5545, 7986, 9073. Includes all standard fields.

### Option 2: MVP Subset (~25 fields)

**Essential fields only** covering most common use cases, but not all
iCalendar+Extensions fields.

### Option 3: Tiered with Feature Flags

**Core fields** (11) always present, **extended fields** behind Rust feature
flags.

---

## Type Definitions

### Option 1: Complete RFC Implementation

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppEventPost {
    // Meta fields
    pub kind: PubkyAppEventPostKind,
    pub parent: Option<String>,  // URI to parent entity
    
    // === CALENDAR FIELDS (kind = Calendar) ===
    
    // RFC 7986 - Calendar Properties
    pub name: Option<String>,              // Calendar display name
    pub description: Option<String>,       // Calendar description
    pub color: Option<String>,             // CSS color value
    pub image: Option<String>,             // URI to calendar image
    pub source: Option<String>,            // Source URL
    pub timezone: Option<String>,          // IANA timezone ID
    
    // Custom Pubky Extensions
    pub x_pubky_admins: Option<Vec<String>>,  // Pubky URIs of admin users
    
    // RFC 5545 - Calendar Metadata
    pub calscale: Option<String>,          // Calendar scale (usually "GREGORIAN")
    pub last_modified: Option<i64>,        // Unix microseconds
    pub created: Option<i64>,              // Unix microseconds
    
    // === EVENT FIELDS (kind = Event) ===
    
    // RFC 5545 - Core Event Properties
    pub uid: Option<String>,               // Globally unique identifier
    pub dtstamp: Option<i64>,              // Creation timestamp (microseconds)
    pub dtstart: Option<i64>,              // Start timestamp (microseconds)
    pub dtend: Option<i64>,                // End timestamp (microseconds)
    pub duration: Option<String>,          // ISO 8601 duration (if no dtend)
    pub summary: Option<String>,           // Event title/subject
    pub location: Option<String>,          // Simple location string
    pub geo: Option<String>,               // Lat;Lon coordinates
    pub status: Option<String>,            // CONFIRMED | TENTATIVE | CANCELLED
    pub transp: Option<String>,            // OPAQUE | TRANSPARENT
    pub sequence: Option<i32>,             // Revision number
    pub priority: Option<i32>,             // 0-9, 0 = undefined
    pub class: Option<String>,             // PUBLIC | PRIVATE | CONFIDENTIAL
    pub url: Option<String>,               // Associated URL
    
    // RFC 5545 - Recurrence
    pub rrule: Option<String>,             // RFC 5545 RRULE string
    pub rdate: Option<Vec<String>>,        // Recurrence dates (JSON array)
    pub exdate: Option<Vec<String>>,       // Exception dates (JSON array)
    pub recurrence_id: Option<i64>,        // For recurrence instances
    
    // RFC 5545 - Relationships
    pub organizer: Option<String>,         // JSON: {"uri": "...", "name": "..."}
    pub categories: Option<Vec<String>>,   // Event categories
    pub related_to: Option<String>,        // UID of related event
    
    // RFC 7986 - Modern Event Properties
    pub conference: Option<String>,        // JSON: {"uri": "...", "label": "..."}
    pub image_uri: Option<String>,         // Event image URI
    
    // RFC 9073 - Event Publishing Extensions
    pub structured_location: Option<String>, // JSON object (see below)
    pub participant: Option<Vec<String>>,  // Participant URIs
    pub styled_description: Option<String>, // JSON: {"fmttype": "text/html", "value": "..."}
    
    // Alarms (stored as JSON array of alarm objects)
    pub alarms: Option<String>,            // JSON: [{"action": "DISPLAY", "trigger": "-PT15M", ...}]
    
    // === ATTENDEE FIELDS (kind = Attendee) ===
    
    // RFC 5545 / 5546 - Attendee/RSVP Properties
    pub attendee_uri: Option<String>,      // Pubky URI of attendee
    pub attendee_name: Option<String>,     // Display name
    pub partstat: Option<String>,          // NEEDS-ACTION | ACCEPTED | DECLINED | TENTATIVE | DELEGATED
    pub role: Option<String>,              // CHAIR | REQ-PARTICIPANT | OPT-PARTICIPANT | NON-PARTICIPANT
    pub cutype: Option<String>,            // INDIVIDUAL | GROUP | RESOURCE | ROOM | UNKNOWN
    pub rsvp: Option<bool>,                // RSVP requested flag
    pub delegated_from: Option<String>,    // Delegator URI
    pub delegated_to: Option<String>,      // Delegatee URI
    pub member: Option<Vec<String>>,       // Group membership URIs
    pub dir: Option<String>,               // Directory entry reference
    pub sent_by: Option<String>,           // Sent by URI
    pub language: Option<String>,          // Language tag (e.g., "en-US")
    
    // Custom fields for additional metadata
    pub custom_fields: Option<String>,     // JSON object for x-* properties
}

#[derive(Serialize, Deserialize, Debug, Clone, PartialEq, Eq)]
#[serde(rename_all = "lowercase")]
pub enum PubkyAppEventPostKind {
    Calendar,
    Event,
    Attendee,
}

// Structured Location JSON Schema (RFC 9073)
// {
//   "uri": "geo:37.7749,-122.4194",
//   "name": "Conference Room A",
//   "description": "Main building, 3rd floor"
// }
//
// Styled Description JSON Schema (RFC 9073)
// {
//   "fmttype": "text/html",
//   "value": "<p>Rich <strong>HTML</strong> description</p>"
// }
```

### Option 2: MVP Subset (Ideal subset for Pubky.app MVP Event Publishing)

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppEventPost {
    // Meta fields
    pub kind: PubkyAppEventPostKind,
    pub parent: Option<String>, // URI to parent entity
    
    // === CALENDAR FIELDS (5 fields) ===
    pub name: Option<String>, // Calendar display name
    pub color: Option<String>, // CSS color value (hex)
    pub x_pubky_admins: Option<Vec<String>>, // Pubky URIs of calendar admins
    pub timezone: Option<String>, // IANA timezone ID
    
    // === EVENT FIELDS (13 fields) ===
    pub uid: Option<String>, // Globally unique identifier
    pub dtstamp: Option<i64>, // Creation timestamp (microseconds)
    pub dtstart: Option<i64>, // Start timestamp (microseconds)
    pub dtend: Option<i64>, // End timestamp (microseconds)
    pub summary: Option<String>, // Event title
    pub status: Option<String>, // CONFIRMED | TENTATIVE | CANCELLED
    pub organizer: Option<String>, // JSON: {"uri": "...", "name": "..."}
    pub categories: Option<Vec<String>>, // Event categories
    pub image_uri: Option<String>, // Event image URI
    pub created: Option<i64>, // Creation timestamp (microseconds)

    // RFC 5545 - Recurrence (Separate event-post for each overriden instance. Frontend should respect this and handle it properly based on the recurrence_id field.)
    pub rrule: Option<String>, // RFC 5545 RRULE string
    pub rdate: Option<Vec<String>>, // Recurrence dates (JSON array)
    pub exdate: Option<Vec<String>>, // Exception dates (JSON array)
    pub recurrence_id: Option<i64>, // For recurrence instances

    // RFC 7986 - Modern Event Properties
    pub image_uri: Option<String>, // Event image URI
    pub conference: Option<String>, // JSON: {"uri": "...", "label": "..."}

    // RFC 9073 - Event Publishing Extensions
    pub structured_location: Option<String>, // JSON object (see below)
    pub styled_description: Option<String>, // JSON: {"fmttype": "text/html", "value": "..."}
    
    // === ATTENDEE FIELDS (4 fields) ===
    pub attendee_uri: Option<String>,
    pub attendee_name: Option<String>,
    pub partstat: Option<String>,
    pub role: Option<String>,
}

#[derive(Serialize, Deserialize, Debug, Clone, PartialEq, Eq)]
#[serde(rename_all = "lowercase")]
pub enum PubkyAppEventPostKind {
    Calendar,
    Event,
    Attendee,
}
```

---

## RFC Field Tables

TODO: Add tables with all fields for each option

## Field Validation Rules

### Calendar (kind = Calendar)

- `name`: **Required**. 1-255 characters.
- `x_pubky_admins`: Must be valid Pubky URIs.
- `color`: Must be valid CSS color (hex, rgb, named).
- `timezone`: Must be valid IANA timezone identifier.

### Event (kind = Event)

- `uid`: **Required**. Globally unique, recommended format:
  `<timestamp>-<user_id>-<random>@pubky`
- `dtstart`: **Required**. Unix microseconds.
- `dtend`: If present, must be > `dtstart`.
- `summary`: **Required**. 1-255 characters.
- `status`: Must be one of: `CONFIRMED`, `TENTATIVE`, `CANCELLED`.
- `rrule`: Must be valid RFC 5545 RRULE string.
- `parent`: Must be a valid Calendar event-post URI.
- `organizer`: JSON string with `uri` (required) and optional `name`.
- `conference`: JSON string with `uri` (required) and optional `label`.
- `styled_description`: JSON string with `fmttype` (MIME type, e.g.,
  "text/html", "text/markdown") and `value` (rich text content).

### Attendee (kind = Attendee)

- `attendee_uri`: **Required**. Valid Pubky URI.
- `partstat`: **Required**. Must be one of: `NEEDS-ACTION`, `ACCEPTED`,
  `DECLINED`, `TENTATIVE`, `DELEGATED`.
- `parent`: **Required**. Must be a valid Event event-post URI.
- `role`: Must be one of: `CHAIR`, `REQ-PARTICIPANT`, `OPT-PARTICIPANT`,
  `NON-PARTICIPANT`.

---

## Homeserver Storage Structure

```
/pub/pubky.app/event-posts/
  ├── <calendar_id_1>         # Calendar event-post
  ├── <event_id_1>            # Event event-post
  ├── <event_id_2>            # Event event-post
  ├── <attendee_id_1>         # Attendee event-post (RSVP)
  └── ...
```

**Example URIs:**

- Calendar: `pubky://<user_id>/pub/pubky.app/event-posts/<calendar_id>`
- Event: `pubky://<user_id>/pub/pubky.app/event-posts/<event_id>`
- Attendee: `pubky://<user_id>/pub/pubky.app/event-posts/<attendee_id>`

**Parent Relationships:**

- Events: `parent` field contains calendar URI
- Attendees: `parent` field contains event URI

---

## Graph Relationships

### Neo4j Relationships

**OWNS** (User → EventPost):

```cypher
(:User {id: "user_id"})-[:OWNS]->(:EventPost {id: "event_post_id"})
```

**PARENT_OF** (Calendar → Event):

```cypher
(:EventPost {kind: "calendar"})-[:PARENT_OF]->(:EventPost {kind: "event"})
```

**PARENT_OF** (Event → Attendee):

```cypher
(:EventPost {kind: "event"})-[:PARENT_OF]->(:EventPost {kind: "attendee"})
```

**ADMIN_OF** (User → Calendar):

```cypher
(:User {id: "admin_id"})-[:ADMIN_OF]->(:EventPost {kind: "calendar"})
```

**ATTENDING** (User → Event via Attendee):

```cypher
(:User {id: "attendee_id"})-[:ATTENDING {partstat: "ACCEPTED"}]->(:EventPost {kind: "event"})
```

---

## Integration with Existing Types

### PubkyAppTags

Can tag event-posts (calendars, events):

```json
{
    "uri": "pubky://<user_id>/pub/pubky.app/tags/<tag_id>",
    "target": "pubky://<owner_id>/pub/pubky.app/event-posts/<event_id>",
    "label": "tech-meetup"
}
```

### PubkyAppBookmarks

Can bookmark event-posts:

```json
{
    "uri": "pubky://<user_id>/pub/pubky.app/bookmarks/<bookmark_id>",
    "target": "pubky://<owner_id>/pub/pubky.app/event-posts/<event_id>"
}
```

### PubkyAppFile

Can attach files to events via URI references:

```rust
pub struct PubkyAppEventPost {
    // ... other fields
    pub attachments: Option<Vec<String>>,  // PubkyAppFile URIs
}
```

---

For Nexus API endpoints, see: [nexus-endpoints.md](./nexus-endpoints.md)\
For example data, see: [event-examples.md](./event-examples.md)
