# Pubky iCalendar Specification (v2.0 - Explicit Fields)

## Index

- [Introduction](#introduction)
- [Architectural Approach](#architectural-approach)
- [Background and Motivation](#background-and-motivation)
- [Type Definitions](#type-definitions)
- [RFC Field Tables](#rfc-field-tables)
- [Field Validation Rules](#field-validation-rules)
- [Homeserver Storage Structure](#homeserver-storage-structure)
- [Graph Relationships](#graph-relationships)
- [Future Extensions and Considerations](#future-extensions-and-considerations)

## Introduction

This specification defines the integration of industry-standard iCalendar
protocols (RFC 5545, RFC 7265, RFC 4791, RFC 5546, RFC 7986, RFC 9073) into the
Pubky ecosystem using **separate dedicated types** with **explicit typed
fields**.

**Key Characteristics:**

- Uses dedicated `PubkyAppCalendar`, `PubkyAppEvent`, `PubkyAppAttendee`,
  `PubkyAppAlarm` types
- Stores calendar data as explicit typed fields instead of jCal JSON
- Separate storage paths for each component type
- Provides compile-time type safety with clear field definitions

**Trade-offs:**

- ✅ **Pros**: Compile-time validation, explicit schema, no runtime parsing,
  better IDE support
- ❌ **Cons**: Less flexible for future RFC extensions, requires type updates
  for new fields

## Architectural Approach

### Core Types

```rust
pub struct PubkyAppCalendar {
    // Calendar fields (5 fields)
    pub name: Option<String>,
    pub color: Option<String>,
    pub x_pubky_admins: Option<Vec<String>>,
    pub timezone: Option<String>,
    pub created: Option<i64>,
}

pub struct PubkyAppEvent {
    // Event fields (13 fields)
    pub uid: Option<String>,
    pub dtstamp: Option<i64>,
    pub dtstart: Option<i64>,
    pub dtend: Option<i64>,
    pub summary: Option<String>,
    pub status: Option<String>,
    pub organizer: Option<String>,
    pub categories: Option<Vec<String>>,
    pub image_uri: Option<String>,
    pub created: Option<i64>,
    pub rrule: Option<String>,
    pub rdate: Option<Vec<String>>,
    pub exdate: Option<Vec<String>>,
    pub recurrence_id: Option<i64>,
    pub conference: Option<String>,
    pub structured_location: Option<String>,
    pub styled_description: Option<String>,
}

pub struct PubkyAppAttendee {
    // Attendee fields (4 fields)
    pub attendee_uri: Option<String>,
    pub attendee_name: Option<String>,
    pub partstat: Option<String>,
    pub role: Option<String>,
}

pub struct PubkyAppAlarm {
    // Alarm fields (4 fields)
    pub action: Option<String>,
    pub trigger: Option<String>,
    pub description: Option<String>,
    pub uid: Option<String>,
}
```

### Storage Paths

- Calendar: `/pub/pubky.app/calendar/:id`
- Event: `/pub/pubky.app/event/:id`
- Attendee: `/pub/pubky.app/attendee/:id`
- Alarm: `/pub/pubky.app/alarm/:id`

### Field Format Standards

- **Dates**: Unix timestamp microseconds (i64) - matches `indexed_at` format
- **Multi-value fields**: JSON array strings, e.g., `["category1", "category2"]`
- **RRULE**: RFC 5545 string format, e.g., `"FREQ=WEEKLY;BYDAY=MO,WE,FR"`
- **Structured data**: JSON object strings with standardized keys
- **URIs**: Pubky URIs for user/resource references, e.g.,
  `"pubky://<user_id>/pub/pubky.app/..."`

---

## Background and Motivation

By using explicit typed fields instead of jCal JSON storage, we gain:

- **Type Safety**: Compile-time validation of all calendar properties
- **Schema Clarity**: Field definitions are immediately visible in code
- **IDE Support**: Full autocomplete, type checking, and refactoring support
- **Performance**: No runtime JSON parsing required
- **Maintainability**: Clear field purposes and validation rules

The trade-off is reduced flexibility for future RFC extensions, as new fields
require type updates rather than just jCal property additions.

---

## Type Definitions

### PubkyAppCalendar

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppCalendar {
    // RFC 7986 - Calendar Properties
    pub name: Option<String>,              // Calendar display name
    pub color: Option<String>,             // CSS color value (hex)
    
    // Custom Pubky Extensions
    pub x_pubky_admins: Option<Vec<String>>,  // Pubky URIs of admin users
    
    // RFC 5545 - Calendar Metadata
    pub timezone: Option<String>,          // IANA timezone ID
    pub created: Option<i64>,              // Unix microseconds
}

// Example usage:
let calendar = PubkyAppCalendar {
    name: Some("Bitcoin Switzerland Events".to_string()),
    color: Some("#F7931A".to_string()),
    x_pubky_admins: Some(vec![
        "pubky://satoshi".to_string(),
        "pubky://adam-back".to_string(),
    ]),
    timezone: Some("Europe/Zurich".to_string()),
    created: Some(1698753600000000), // 2023-10-31T12:00:00Z
};
```

### PubkyAppEvent

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppEvent {
    // RFC 5545 - Core Event Properties
    pub uid: Option<String>,               // Globally unique identifier
    pub dtstamp: Option<i64>,              // Creation timestamp (microseconds)
    pub dtstart: Option<i64>,              // Start timestamp (microseconds)
    pub dtend: Option<i64>,                // End timestamp (microseconds)
    pub summary: Option<String>,           // Event title/subject
    pub status: Option<String>,            // CONFIRMED | TENTATIVE | CANCELLED
    pub organizer: Option<String>,         // JSON: {"uri": "...", "name": "..."}
    pub categories: Option<Vec<String>>,   // Event categories
    pub created: Option<i64>,              // Creation timestamp (microseconds)
    
    // RFC 5545 - Recurrence
    pub rrule: Option<String>,             // RFC 5545 RRULE string
    pub rdate: Option<Vec<String>>,        // Recurrence dates (JSON array)
    pub exdate: Option<Vec<String>>,       // Exception dates (JSON array)
    pub recurrence_id: Option<i64>,        // For recurrence instances
    
    // RFC 7986 - Modern Event Properties
    pub image_uri: Option<String>,         // Event image URI
    pub conference: Option<String>,        // JSON: {"uri": "...", "label": "..."}
    
    // RFC 9073 - Event Publishing Extensions
    pub structured_location: Option<String>, // JSON object (see below)
    pub styled_description: Option<String>, // JSON: {"fmttype": "text/html", "value": "..."}

    // Pubky Linkage - explicit references for aggregation
    pub x_pubky_calendar_uri: Option<String>, // URI of the calendar this event belongs to
}

// Example usage:
let event = PubkyAppEvent {
    uid: Some("pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string()),
    dtstamp: Some(1698753600000000),
    dtstart: Some(1698753600000000),
    dtend: Some(1698764400000000),
    summary: Some("Bitcoin Meetup Zürich".to_string()),
    status: Some("CONFIRMED".to_string()),
    organizer: Some(r#"{"uri": "pubky://satoshi", "name": "Satoshi"}"#.to_string()),
    categories: Some(vec!["bitcoin".to_string(), "meetup".to_string()]),
    created: Some(1698753600000000),
    rrule: Some("FREQ=WEEKLY;BYDAY=WE".to_string()),
    rdate: None,
    exdate: None,
    recurrence_id: None,
    image_uri: Some("pubky://satoshi/pub/pubky.app/files/0033EVENT01".to_string()),
    conference: Some(r#"{"uri": "https://meet.jit.si/bitcoin-zurich", "label": "Jitsi Meeting"}"#.to_string()),
    structured_location: Some(r#"{"uri": "geo:47.366667,8.550000", "name": "Insider Bar", "description": "Main venue"}"#.to_string()),
    styled_description: Some(r#"{"fmttype": "text/html", "value": "<p>Weekly Bitcoin meetup discussing <strong>Lightning Network</strong></p>"}"#.to_string()),
    x_pubky_calendar_uri: Some("pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG".to_string()),
};
```

### PubkyAppAttendee

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAttendee {
    // RFC 5545 / 5546 - Attendee/RSVP Properties
    pub attendee_uri: Option<String>,      // Pubky URI of attendee
    pub attendee_name: Option<String>,     // Display name
    pub partstat: Option<String>,          // NEEDS-ACTION | ACCEPTED | DECLINED | TENTATIVE | DELEGATED
    pub role: Option<String>,              // CHAIR | REQ-PARTICIPANT | OPT-PARTICIPANT | NON-PARTICIPANT

    // Pubky Linkage
    pub x_pubky_event_uri: Option<String>, // URI of the event this RSVP belongs to
}

// Example usage:
let attendee = PubkyAppAttendee {
    attendee_uri: Some("pubky://alice".to_string()),
    attendee_name: Some("Alice".to_string()),
    partstat: Some("ACCEPTED".to_string()),
    role: Some("REQ-PARTICIPANT".to_string()),
    x_pubky_event_uri: Some("pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string()),
};
```

### PubkyAppAlarm

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAlarm {
    // RFC 5545 - Alarm Properties
    pub action: Option<String>,            // AUDIO | DISPLAY | EMAIL
    pub trigger: Option<String>,           // Duration or absolute time
    pub description: Option<String>,       // Alarm message text
    pub uid: Option<String>,               // Globally unique identifier

    // Pubky Linkage
    pub x_pubky_target_uri: Option<String>, // Target event or calendar URI
}

// Example usage:
let alarm = PubkyAppAlarm {
    action: Some("DISPLAY".to_string()),
    trigger: Some("-PT15M".to_string()), // 15 minutes before
    description: Some("Bitcoin Meetup in 15 minutes".to_string()),
    uid: Some("pubky://bob/pub/pubky.app/alarm/0033WCZXVEPNG".to_string()),
    x_pubky_target_uri: Some("pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string()),
};
```

---

## RFC Field Tables

### Fields Used in MVP Implementation

| Field                  | RFC Source | Type        | Description                | Rationale                                  |
| ---------------------- | ---------- | ----------- | -------------------------- | ------------------------------------------ |
| **Calendar Fields**    |            |             |                            |                                            |
| `name`                 | RFC 7986   | String      | Calendar display name      | Essential for calendar identification      |
| `color`                | RFC 7986   | String      | CSS color value            | Important for visual calendar organization |
| `x_pubky_admins`       | Custom     | Vec<String> | Calendar admin URIs        | Core to Pubky's decentralized admin model  |
| `timezone`             | RFC 5545   | String      | IANA timezone ID           | Essential for proper time handling         |
| `created`              | RFC 5545   | i64         | Creation timestamp         | Standard metadata field                    |
| **Event Fields**       |            |             |                            |                                            |
| `uid`                  | RFC 5545   | String      | Globally unique identifier | Required for all calendar components       |
| `dtstamp`              | RFC 5545   | i64         | Creation timestamp         | Required for all calendar components       |
| `dtstart`              | RFC 5545   | i64         | Start timestamp            | Core event timing                          |
| `dtend`                | RFC 5545   | i64         | End timestamp              | Core event timing                          |
| `summary`              | RFC 5545   | String      | Event title                | Essential for event identification         |
| `status`               | RFC 5545   | String      | Event status               | Important for event lifecycle              |
| `organizer`            | RFC 5545   | String      | Event organizer            | Core to event ownership                    |
| `categories`           | RFC 5545   | Vec<String> | Event categories           | Useful for event classification            |
| `created`              | RFC 5545   | i64         | Creation timestamp         | Standard metadata field                    |
| `rrule`                | RFC 5545   | String      | Recurrence rule            | Essential for recurring events             |
| `rdate`                | RFC 5545   | Vec<String> | Recurrence dates           | For additional recurrence instances        |
| `exdate`               | RFC 5545   | Vec<String> | Exception dates            | For excluding recurrence instances         |
| `recurrence_id`        | RFC 5545   | i64         | Recurrence instance ID     | For recurrence overrides                   |
| `image_uri`            | RFC 7986   | String      | Event image URI            | Important for visual event representation  |
| `conference`           | RFC 7986   | String      | Conference link            | Essential for online events                |
| `structured_location`  | RFC 9073   | String      | Rich location data         | Better than simple location string         |
| `styled_description`   | RFC 9073   | String      | Rich text description      | Better than plain text description         |
| `x_pubky_calendar_uri` | Custom     | String      | Calendar URI               | Explicit linkage for aggregation           |
| **Attendee Fields**    |            |             |                            |                                            |
| `attendee_uri`         | RFC 5545   | String      | Attendee URI               | Core attendee identification               |
| `attendee_name`        | RFC 5545   | String      | Display name               | Human-readable attendee name               |
| `partstat`             | RFC 5546   | String      | Participation status       | Core RSVP functionality                    |
| `role`                 | RFC 5546   | String      | Participant role           | Important for attendee roles               |
| `x_pubky_event_uri`    | Custom     | String      | Event URI                  | Explicit linkage for aggregation           |
| **Alarm Fields**       |            |             |                            |                                            |
| `action`               | RFC 5545   | String      | Alarm action type          | Core alarm functionality                   |
| `trigger`              | RFC 5545   | String      | Alarm trigger              | Core alarm timing                          |
| `description`          | RFC 5545   | String      | Alarm message              | User-facing alarm text                     |
| `uid`                  | RFC 5545   | String      | Globally unique identifier | Required for all calendar components       |
| `x_pubky_target_uri`   | Custom     | String      | Target entity URI          | Explicit linkage for aggregation           |

### Fields Explicitly Not Used (with Rationale)

| Field                                      | RFC Source | Rationale for Exclusion                                               |
| ------------------------------------------ | ---------- | --------------------------------------------------------------------- |
| **RFC 5545 - Core Properties**             |            |                                                                       |
| `description`                              | RFC 5545   | Replaced by `styled_description` (RFC 9073) for better formatting     |
| `location`                                 | RFC 5545   | Replaced by `structured_location` (RFC 9073) for richer location data |
| `geo`                                      | RFC 5545   | Covered by `structured_location` which includes coordinates           |
| `url`                                      | RFC 5545   | Not essential for MVP, can be added later                             |
| `last_modified`                            | RFC 5545   | Not essential for MVP, can be added later                             |
| `sequence`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| `priority`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| `class`                                    | RFC 5545   | Not essential for MVP, can be added later                             |
| `transp`                                   | RFC 5545   | Not essential for MVP, can be added later                             |
| `duration`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| `related_to`                               | RFC 5545   | Not essential for MVP, can be added later                             |
| `resources`                                | RFC 5545   | Not essential for MVP, can be added later                             |
| `contact`                                  | RFC 5545   | Not essential for MVP, can be added later                             |
| `comment`                                  | RFC 5545   | Not essential for MVP, can be added later                             |
| `request_status`                           | RFC 5545   | Not essential for MVP, can be added later                             |
| `freebusy`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| `timezone`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| `method`                                   | RFC 5545   | Not essential for MVP, can be added later                             |
| `prodid`                                   | RFC 5545   | Not essential for MVP, can be added later                             |
| `version`                                  | RFC 5545   | Not essential for MVP, can be added later                             |
| `calscale`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| **RFC 7986 - New Properties**              |            |                                                                       |
| `description`                              | RFC 7986   | Replaced by `styled_description` (RFC 9073) for better formatting     |
| `image`                                    | RFC 7986   | Replaced by `image_uri` for consistency                               |
| `source`                                   | RFC 7986   | Not essential for MVP, can be added later                             |
| `refresh_interval`                         | RFC 7986   | Not essential for MVP, can be added later                             |
| `color` (calendar)                         | RFC 7986   | Included in MVP                                                       |
| `name` (calendar)                          | RFC 7986   | Included in MVP                                                       |
| `conference` (event)                       | RFC 7986   | Included in MVP                                                       |
| `image_uri` (event)                        | RFC 7986   | Included in MVP                                                       |
| **RFC 9073 - Event Publishing Extensions** |            |                                                                       |
| `participant`                              | RFC 9073   | Not essential for MVP, can be added later                             |
| `participant_type`                         | RFC 9073   | Not essential for MVP, can be added later                             |
| `structured_location`                      | RFC 9073   | Included in MVP                                                       |
| `styled_description`                       | RFC 9073   | Included in MVP                                                       |
| **RFC 5546 - iTIP Properties**             |            |                                                                       |
| `method`                                   | RFC 5546   | Not essential for MVP, can be added later                             |
| `rsvp`                                     | RFC 5546   | Not essential for MVP, can be added later                             |
| `delegated_from`                           | RFC 5546   | Not essential for MVP, can be added later                             |
| `delegated_to`                             | RFC 5546   | Not essential for MVP, can be added later                             |
| `member`                                   | RFC 5546   | Not essential for MVP, can be added later                             |
| `dir`                                      | RFC 5546   | Not essential for MVP, can be added later                             |
| `sent_by`                                  | RFC 5546   | Not essential for MVP, can be added later                             |
| `language`                                 | RFC 5546   | Not essential for MVP, can be added later                             |
| `cutype`                                   | RFC 5546   | Not essential for MVP, can be added later                             |
| `partstat`                                 | RFC 5546   | Included in MVP                                                       |
| `role`                                     | RFC 5546   | Included in MVP                                                       |
| **RFC 5545 - Alarm Properties**            |            |                                                                       |
| `repeat`                                   | RFC 5545   | Not essential for MVP, can be added later                             |
| `attach`                                   | RFC 5545   | Not essential for MVP, can be added later                             |
| `attendee`                                 | RFC 5545   | Not essential for MVP, can be added later                             |
| `summary`                                  | RFC 5545   | Not essential for MVP, can be added later                             |
| `x_properties`                             | RFC 5545   | Not essential for MVP, can be added later                             |

---

## Field Validation Rules

### Calendar Fields

- `name`: **Required**. 1-255 characters.
- `x_pubky_admins`: Must be valid Pubky URIs.
- `color`: Must be valid CSS color (hex, rgb, named).
- `timezone`: Must be valid IANA timezone identifier.

### Event Fields

- `uid`: **Required**. Globally unique, recommended format:
  `<timestamp>-<user_id>-<random>@pubky`
- `dtstart`: **Required**. Unix microseconds.
- `dtend`: If present, must be > `dtstart`.
- `summary`: **Required**. 1-255 characters.
- `status`: Must be one of: `CONFIRMED`, `TENTATIVE`, `CANCELLED`.
- `rrule`: Must be valid RFC 5545 RRULE string.
- `organizer`: JSON string with `uri` (required) and optional `name`.
- `conference`: JSON string with `uri` (required) and optional `label`.
- `styled_description`: JSON string with `fmttype` (MIME type, e.g.,
  "text/html", "text/markdown") and `value` (rich text content).

### Attendee Fields

- `attendee_uri`: **Required**. Valid Pubky URI.
- `partstat`: **Required**. Must be one of: `NEEDS-ACTION`, `ACCEPTED`,
  `DECLINED`, `TENTATIVE`, `DELEGATED`.
- `role`: Must be one of: `CHAIR`, `REQ-PARTICIPANT`, `OPT-PARTICIPANT`,
  `NON-PARTICIPANT`.

### Alarm Fields

- `action`: **Required**. Must be one of: `AUDIO`, `DISPLAY`, `EMAIL`.
- `trigger`: **Required**. Valid duration or absolute timestamp.
- `uid`: **Required**. Globally unique identifier.

---

## Homeserver Storage Structure

```
/pub/pubky.app/
├── calendar/:calendar_id           # Calendar metadata
├── event/:event_id                 # Individual events
├── attendee/:attendee_id           # RSVP records
└── alarm/:alarm_id                 # User reminders
```

**Example URIs:**

- Calendar: `pubky://<user_id>/pub/pubky.app/calendar/<calendar_id>`
- Event: `pubky://<user_id>/pub/pubky.app/event/<event_id>`
- Attendee: `pubky://<user_id>/pub/pubky.app/attendee/<attendee_id>`
- Alarm: `pubky://<user_id>/pub/pubky.app/alarm/<alarm_id>`

---

## Recurrence Overrides and Exceptions

Individual instances of recurring events can be modified or excluded using
standard iCalendar mechanisms (RFC 5545). This section details how recurrence
overrides work with explicit typed fields.

### Excluding Instances (EXDATE)

The `exdate` field contains an array of timestamps for instances to exclude from
the recurrence pattern:

```rust
// Example: Exclude October 16th and 23rd from weekly meetup
let event = PubkyAppEvent {
    // ... other fields ...
    rrule: Some("FREQ=WEEKLY;BYDAY=WE".to_string()),
    exdate: Some(vec![
        "2025-10-16T19:00:00+02:00".to_string(),  // October 16th
        "2025-10-23T19:00:00+02:00".to_string(),  // October 23rd
    ]),
    // ... other fields ...
};
```

### Adding Extra Instances (RDATE)

The `rdate` field contains an array of timestamps for additional instances not
covered by the RRULE:

```rust
// Example: Add extra meetup on October 15th
let event = PubkyAppEvent {
    // ... other fields ...
    rrule: Some("FREQ=WEEKLY;BYDAY=WE".to_string()),
    rdate: Some(vec![
        "2025-10-15T19:00:00+02:00".to_string(),  // Extra Tuesday meetup
    ]),
    // ... other fields ...
};
```

### Modifying Individual Instances (RECURRENCE-ID)

Create a separate `PubkyAppEvent` with modified properties for a specific
occurrence. The `recurrence_id` field identifies which instance is being
overridden.

**Master Recurring Event:**

```rust
let master_event = PubkyAppEvent {
    id: "0033SCZXVEPNG".to_string(),
    uid: "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string(),
    dtstart: 1698753600000000,  // October 9th, 19:00
    dtend: 1698764400000000,    // October 9th, 22:00
    summary: "Bitcoin Meetup Zürich".to_string(),
    rrule: Some("FREQ=WEEKLY;BYDAY=WE".to_string()),
    recurrence_id: None,  // Master event has no recurrence_id
    // ... other fields ...
};
```

**Override for October 16th instance (separate file):**

```rust
let override_event = PubkyAppEvent {
    id: "0033TCZXVEPNG".to_string(),
    uid: "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string(), // Same UID as master
    dtstart: 1699358400000000,  // October 16th, 20:00 (moved from 19:00)
    dtend: 1699372800000000,    // October 16th, 23:00 (extended)
    summary: "Bitcoin Meetup Zürich - Special Lightning Workshop".to_string(),
    recurrence_id: Some(1699358400000000), // Original occurrence time (19:00)
    rrule: None,  // Override events don't have RRULE
    // ... other fields ...
};
```

### Key Implementation Rules

1. **Shared UID**: Override events must share the same `uid` as the master
   recurring event
2. **Recurrence ID Timing**: The `recurrence_id` value must match the **original
   occurrence time** (October 16th at 19:00), even if the start time is moved to
   20:00
3. **Separate Storage**: Override events are stored as separate files on the
   homeserver
4. **Property Modification**: Any property can be modified in the override
   (time, location, summary, attendees, etc.)
5. **Nexus Aggregation**: Nexus aggregates both the master event and override
   instances when querying the calendar
6. **No RRULE in Overrides**: Override events should not have an `rrule` field

### Frontend Implementation Guidelines

**Calendar View Rendering:**

- Render master event instances according to RRULE
- Apply EXDATE exclusions
- Add RDATE extra instances
- Replace instances with matching RECURRENCE-ID overrides

**Event Creation/Editing:**

- When editing a recurring event, prompt user: "Edit this occurrence only" vs
  "Edit all occurrences"
- For single occurrence edits, create override event with proper `recurrence_id`
- For all occurrence edits, update master event and remove related overrides

**Data Consistency:**

- Validate that `recurrence_id` timestamps match actual occurrence times
- Ensure override events don't have conflicting RRULE patterns
- Handle timezone conversions properly for recurrence calculations

---

## Graph Relationships

### Neo4j Relationships

**OWNS** (User → Calendar/Event/Attendee/Alarm):

```cypher
(:User {id: "user_id"})-[:OWNS]->(:Calendar {id: "calendar_id"})
(:User {id: "user_id"})-[:OWNS]->(:Event {id: "event_id"})
(:User {id: "user_id"})-[:OWNS]->(:Attendee {id: "attendee_id"})
(:User {id: "user_id"})-[:OWNS]->(:Alarm {id: "alarm_id"})
```

**ADMIN_OF** (User → Calendar):

```cypher
(:User {id: "admin_id"})-[:ADMIN_OF]->(:Calendar {id: "calendar_id"})
```

**ATTENDING** (User → Event via Attendee):

```cypher
(:User {id: "attendee_id"})-[:ATTENDING {partstat: "ACCEPTED"}]->(:Event {id: "event_id"})
```

**BELONGS_TO** (Event → Calendar):

```cypher
(:Event {id: "event_id"})-[:BELONGS_TO]->(:Calendar {id: "calendar_id"})
```

**RSVP_FOR** (Attendee → Event):

```cypher
(:Attendee {id: "attendee_id"})-[:RSVP_FOR]->(:Event {id: "event_id"})
```

**ALARM_FOR** (Alarm → Event/Calendar):

```cypher
(:Alarm {id: "alarm_id"})-[:ALARM_FOR]->(:Event {id: "event_id"})
(:Alarm {id: "alarm_id"})-[:ALARM_FOR]->(:Calendar {id: "calendar_id"})
```

---

## Future Extensions and Considerations

### Potential Advanced Features

**Enhanced Access Control** (using RFC 4791 patterns):

- Calendar-level RSVP policies (`x-pubky-rsvp-policy`: open, approval-required,
  invite-only)
- Event submission workflows with approval queues
- Granular permissions (read, write, admin roles)
- Auto-approval lists for trusted contributors

**Rich Event Features** (RFC 9073):

- `PARTICIPANT` components for conference speakers, sponsors, performers
- Enhanced location data with OpenStreetMap integration. With location based
  event lookups in Nexus.
- Additional styled description formats

**Advanced Scheduling** (RFC 5546, RFC 6638):

- iTIP message flows for formal invitations
- Free/busy time aggregation
- Delegation and proxy workflows
- Server-side scheduling automation

**Extended Alarm Capabilities** (RFC 9074):

- `PROXIMITY` triggers (location-based reminders)
- Complex alarm relationships with `RELATED-TO`
- Snooze and acknowledgment tracking

### Implementation Notes

The current specification establishes the foundation for these features without
requiring their immediate implementation. The explicit field structure makes it
clear which fields are available while allowing for future extensions.

For the MVP implementation, focus is set on the core schemas (Calendar, Event,
Attendee) with essential fields. Advanced features can be layered on as the
ecosystem matures and user needs become clearer.

---

## References

### Core RFC Standards

- [RFC 5545 - Internet Calendaring and Scheduling Core Object Specification (iCalendar)](https://www.rfc-editor.org/rfc/rfc5545.html)
- [RFC 7265 - jCal: The JSON Format for iCalendar](https://www.rfc-editor.org/rfc/rfc7265.html)
- [RFC 4791 - Calendaring Extensions to WebDAV (CalDAV)](https://www.rfc-editor.org/rfc/rfc4791.html)
- [RFC 5546 - iCalendar Transport-Independent Interoperability Protocol (iTIP)](https://www.rfc-editor.org/rfc/rfc5546.html)
- [RFC 7986 - New Properties for iCalendar](https://www.rfc-editor.org/rfc/rfc7986.html)
- [RFC 9073 - Event Publishing Extensions to iCalendar](https://www.rfc-editor.org/rfc/rfc9073.html)

### Supporting Standards

- [RFC 6868 - Parameter Value Encoding in iCalendar and vCard](https://www.rfc-editor.org/rfc/rfc6868.html)
- [RFC 6638 - Scheduling Extensions to CalDAV](https://www.rfc-editor.org/rfc/rfc6638.html)
- [RFC 7529 - Non-Gregorian Recurrence Rules in iCalendar](https://www.rfc-editor.org/rfc/rfc7529.html)
- [RFC 9074 - VALARM Extensions for iCalendar](https://www.rfc-editor.org/rfc/rfc9074.html)

### Pubky Resources

- [Pubky Core Documentation](https://docs.pubky.org/)
- [Pubky-App Specs Repository](https://github.com/pubky/pubky-app-specs)
- [Pubky-Nexus Repository](https://github.com/pubky/pubky-nexus)
