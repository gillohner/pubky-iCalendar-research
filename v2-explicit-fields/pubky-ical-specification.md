# Pubky iCalendar Specification (v2.0 - Explicit Fields)

## Index

- [Introduction](#introduction)
- [Architectural Approach](#architectural-approach)
- [Background and Motivation](#background-and-motivation)
- [Type Definitions](#type-definitions)
- [RFC Field Tables](#rfc-field-tables)
- [Field Validation Rules](#field-validation-rules)
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

## Architectural Approach

### Component Types

```rust
// Calendar container - collection of events
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppCalendar {
    // RFC 7986 - Calendar Properties
    pub name: String,                      // REQUIRED - calendar display name
    pub color: Option<String>,             // CSS color value (hex format)
    pub image_uri: Option<String>,         // Calendar image/logo URI (pubky:// or https)

    // RFC 5545 - Calendar Metadata
    pub timezone: String,                  // REQUIRED - IANA timezone ID (e.g., "Europe/Zurich")
    pub description: Option<String>,       // Calendar description
    pub url: Option<String>,               // Calendar homepage/details URL
    pub created: Option<i64>,              // Creation timestamp (Unix microseconds)

    // Pubky Extensions (all custom fields use x_pubky_ prefix)
    pub x_pubky_admins: Option<Vec<String>>,  // Pubky URIs of admin users
}
```

```rust
// Event - a scheduled activity or occasion
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppEvent {
    // RFC 5545 - Core Event Properties (REQUIRED)
    pub uid: String,                    // Globally unique identifier
    pub dtstamp: i64,                   // Creation/last-modified timestamp (Unix microseconds)
    pub dtstart: i64,                   // Start timestamp (Unix microseconds)
    pub summary: String,                // Event title/subject

    // RFC 5545 - Time & Duration
    pub dtend: Option<i64>,             // End timestamp (mutually exclusive with duration)
    pub duration: Option<String>,       // RFC 5545 duration format (mutually exclusive with dtend)
    pub dtstart_tzid: Option<String>,   // IANA timezone for dtstart (e.g., "Europe/Zurich")
    pub dtend_tzid: Option<String>,     // IANA timezone for dtend

    // RFC 5545 - Event Details
    pub description: Option<String>,    // Plain text description
    pub status: Option<String>,         // CONFIRMED | TENTATIVE | CANCELLED
    pub organizer: Option<Organizer>,   // Event organizer (name from profile.json)
    pub categories: Option<Vec<String>>, // Event categories/tags
    
    // RFC 5545 - Location
    pub location: Option<String>,       // Primary location text (RFC 5545 LOCATION property)
    pub geo: Option<String>,            // Geographic coordinates "lat;lon" (RFC 5545 GEO property)
    pub structured_locations: Option<Vec<Location>>, // RFC 9073 VLOCATION components (repeatable)
    
    // RFC 7986 - Event Publishing Extensions
    pub image_uri: Option<String>,      // Event image/banner URI
    pub url: Option<String>,            // Event homepage/details link
    pub conference: Option<Conference>, // Video/audio conference details

    // RFC 5545 - Change Management
    pub sequence: Option<i32>,          // Version number (increment on modifications)
    pub last_modified: Option<i64>,     // Last modification timestamp
    pub created: Option<i64>,           // Creation timestamp

    // RFC 5545 - Recurrence
    pub rrule: Option<String>,          // Recurrence rule (RFC 5545 format)
    pub rdate: Option<Vec<String>>,     // Additional recurrence dates
    pub exdate: Option<Vec<String>>,    // Excluded recurrence dates
    pub recurrence_id: Option<i64>,     // Identifies specific recurrence instance

    // RFC 9073 - Rich Content
    pub styled_description: Option<StyledDescription>, // Formatted description with metadata

    // Pubky Extensions (all custom fields use x_pubky_ prefix)
    pub x_pubky_calendar_uris: Option<Vec<String>>, // URIs of calendars containing this event
    pub x_pubky_rsvp_access: Option<String>, // RSVP access control: "PUBLIC" (default, anyone can RSVP)
                                                      // Future: "INVITE_ONLY", "CONFIRMED_ONLY"
}
```

```rust
// Attendee - an RSVP/participation record for an event
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAttendee {
    // RFC 5545 - Attendee Properties
    pub attendee_uri: String,           // REQUIRED - Pubky URI of the person attending
                                        // NOTE: Different from storage path author in these cases:
                                        //   1. Organizer creates invite records for invitees
                                        //   2. Future: Delegation (delegated_from/delegated_to)
                                        //   3. iCalendar import with external attendees
                                        // For self-RSVP: attendee_uri matches storage path author
    pub partstat: String,               // REQUIRED - NEEDS-ACTION | ACCEPTED | DECLINED | TENTATIVE
    pub role: Option<String>,           // CHAIR | REQ-PARTICIPANT | OPT-PARTICIPANT | NON-PARTICIPANT
    pub rsvp: Option<bool>,             // true when RSVP is requested/expected
    pub delegated_from: Option<String>, // Pubky URI (for future delegation support)
    pub delegated_to: Option<String>,   // Pubky URI (for future delegation support)

    // RFC 5545 - Recurrence Support
    pub recurrence_id: Option<i64>,     // For recurring events, specifies which instance

    // Pubky Extensions
    pub x_pubky_event_uri: String,      // REQUIRED - URI of the event this RSVP belongs to
}
```

```rust
// Alarm - a notification/reminder for an event
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAlarm {
    // RFC 5545 - Alarm Properties
    pub action: String,                 // REQUIRED - AUDIO | DISPLAY | EMAIL
    pub trigger: String,                // REQUIRED - Duration (e.g., "-PT15M") or absolute time
    pub description: Option<String>,    // REQUIRED for DISPLAY and EMAIL actions
    pub summary: Option<String>,        // REQUIRED for EMAIL action (email subject)
    pub attendees: Option<Vec<String>>, // REQUIRED for EMAIL action (email recipients)

    // RFC 5545 - Repeat Functionality
    pub repeat: Option<i32>,            // Number of additional repetitions
    pub duration: Option<String>,       // Delay between repetitions (RFC 5545 duration format)
    
    // RFC 5545 - Attachments
    pub attach: Option<Vec<String>>,    // URIs to attachments (audio file for AUDIO, files for EMAIL)

    // Pubky Extensions
    pub x_pubky_target_uri: String,     // REQUIRED - Target event or calendar URI
}
```

### Helper Types

```rust
use serde::{Deserialize, Serialize};

// RFC 5545 Organizer - represents event organizer
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Organizer {
    pub uri: String,                    // REQUIRED Pubky URI (name fetched from profile.json)
    pub sent_by: Option<String>,        // Pubky URI of delegate sending on behalf of organizer
}
```

```rust
// RFC 7986 Conference - video/audio conference details
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Conference {
    pub uri: String,                    // REQUIRED - Conference URL (Zoom, Meet, etc.)
    pub label: Option<String>,          // Human-readable label (e.g., "Zoom Meeting")
    pub features: Option<Vec<String>>, // Conference features: AUDIO, VIDEO, CHAT, SCREEN, etc.
}
```

```rust
// RFC 9073 VLOCATION - structured location component (repeatable)
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Location {
    pub name: String,                   // REQUIRED - Location name (e.g., "Insider Bar", "Parking Lot B")
    pub location_type: Option<String>,  // RFC 9073 LOCATION-TYPE: ARRIVAL, DEPARTURE, PARKING, etc.
    pub address: Option<String>,        // Street address (e.g., "Münstergasse 20, 8001 Zürich")
    pub uri: Option<String>,            // Reference URI (OSM node, website, etc.)
    pub description: Option<String>,    // Additional details (e.g., "Second floor, near the bar")
}
```

```rust
// RFC 9073 Styled Description - formatted content with metadata
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct StyledDescription {
    pub fmttype: Option<String>,        // Media type (e.g., "text/html", "text/markdown")
    pub value: String,                  // The styled content
    pub derived: Option<bool>,          // RFC 9073 - TRUE if auto-derived from plain DESCRIPTION
    pub altrep: Option<String>,         // RFC 9073 - Alternate representation URI
    pub language: Option<String>,       // RFC 9073 - Language tag (e.g., "en-US")
}
```

### Storage Paths

- Calendar: `/pub/pubky.app/calendar/:id`
- Event: `/pub/pubky.app/event/:id`
- Attendee: `/pub/pubky.app/attendee/:id`
- Alarm: `/pub/pubky.app/alarm/:id`

### Field Format Standards

- **Dates**: Unix timestamp microseconds (i64) - matches `indexed_at` format
  across Pubky homeserver documents. **All timestamps are stored in UTC**.
- **Timezone Handling**:
  - `dtstart`, `dtend`, `dtstamp`, `created`, `last_modified` are **always UTC**
  - `dtstart_tzid` and `dtend_tzid` are **display-only** IANA timezone IDs
  - Used for: (1) Converting UTC → local time for display, (2) Proper recurrence
    expansion across DST boundaries
  - Example: Event "Oct 26, 2025 19:00 Europe/Zurich" is stored as:
    - `dtstart: 1729965600000000` (UTC microseconds)
    - `dtstart_tzid: "Europe/Zurich"` (display timezone)
  - Calendar `timezone` field: Default timezone for events in that calendar
    (when event doesn't specify its own timezone)
- **Multi-value fields**: JSON array strings, e.g., `["category1", "category2"]`
- **RRULE**: RFC 5545 string format, e.g., `"FREQ=WEEKLY;BYDAY=MO,WE,FR"`
- **Structured data**: JSON object strings with standardized keys
- **URIs**: Pubky URIs for user/resource references, e.g.,
  `"pubky://<user_id>/pub/pubky.app/..."`

---

## Location Data Strategy

**Three-tier location approach** matching RFC 5545 and RFC 9073:

#### 1. `location` (RFC 5545 LOCATION property)

- **Purpose**: Primary human-readable location text
- **Format**: Plain text string
- **Use**: Single-line location description for event listings
- **Example**: `"Insider Bar, Münstergasse 20, Zürich"`

#### 2. `geo` (RFC 5545 GEO property)

- **Purpose**: Geographic coordinates for map display and proximity search
- **Format**: `"latitude;longitude"` (semicolon-separated decimals)
- **Use**: Map display, radius search, navigation
- **Example**: `"47.366667;8.550000"`

#### 3. `structured_locations` (RFC 9073 VLOCATION components)

- **Purpose**: Repeatable structured location data for complex events
- **Format**: Array of `Location` objects
- **Fields per location**:
  - `name`: Location name (REQUIRED)
  - `location_type`: ARRIVAL, DEPARTURE, PARKING, etc.
  - `address`: Street address
  - `uri`: OSM node, website, etc.
  - `description`: Additional instructions
- **Use cases**:
  - Events with multiple locations (arrival venue, parking, departure point)
  - OSM integration with full structured data
  - Detailed venue information

#### Frontend OSM Integration

**Simple event** (single location):

```typescript
// OSM lookup populates all three fields:
event.location = "Insider Bar, Münstergasse 20, Zürich"; // Primary text
event.geo = "47.366667;8.550000"; // Coordinates
event.structured_locations = [{ // Structured data
  name: "Insider Bar",
  location_type: "ARRIVAL",
  address: "Münstergasse 20, 8001 Zürich, Switzerland",
  uri: "https://www.openstreetmap.org/node/123456789",
  description: null,
}];
```

**Complex event** (multiple locations):

```typescript
event.location = "Insider Bar, Münstergasse 20, Zürich"; // Primary venue
event.geo = "47.366667;8.550000"; // Primary coordinates
event.structured_locations = [
  {
    name: "Insider Bar",
    location_type: "ARRIVAL", // Main event venue
    address: "Münstergasse 20, 8001 Zürich",
    uri: "https://www.openstreetmap.org/node/123456789",
  },
  {
    name: "Parking Garage Urania",
    location_type: "PARKING", // Parking location
    address: "Uraniastrasse 3, 8001 Zürich",
    uri: "https://www.openstreetmap.org/way/987654321",
    description: "Hourly parking available",
  },
];
```

#### RFC 9073 VLOCATION Compatibility

Our approach **implements RFC 9073 VLOCATION** with pragmatic simplifications:

**RFC 9073 VLOCATION:**

- Separate component with UID
- Multiple VLOCATIONs per event
- LOCATION-TYPE property

**Our implementation:**

- Embedded array (not separate components)
- No UID needed (locations bound to event)
- LOCATION-TYPE as field in struct
- **Fully convertible** to/from RFC 9073 VLOCATION components

---

## RFC Field Tables

### Fields Used in MVP Implementation

| Field                   | RFC Source | Type              | Description                | Rationale                                                |
| ----------------------- | ---------- | ----------------- | -------------------------- | -------------------------------------------------------- |
| **Calendar Fields**     |            |                   |                            |                                                          |
| `name`                  | RFC 7986   | String            | Calendar display name      | Essential for calendar identification                    |
| `color`                 | RFC 7986   | String            | CSS color value            | Important for visual calendar organization               |
| `image_uri`             | RFC 7986   | String            | Calendar image/logo URI    | Visual branding for calendars                            |
| `x_pubky_admins`        | Custom     | Vec<String>       | Calendar admin Pubky URIs  | Controls which users can have events indexed in calendar |
| `timezone`              | RFC 5545   | String            | IANA timezone ID           | Essential for proper time handling                       |
| `created`               | RFC 5545   | i64               | Creation timestamp         | Standard metadata field                                  |
| `description`           | RFC 7986   | String            | Calendar description       | Helpful for calendar discovery                           |
| `url`                   | RFC 7986   | String            | Calendar homepage          | Link to more information                                 |
| **Event Fields**        |            |                   |                            |                                                          |
| `uid`                   | RFC 5545   | String            | Globally unique identifier | Required for all calendar components                     |
| `dtstamp`               | RFC 5545   | i64               | Creation timestamp         | Required for proper iCalendar semantics                  |
| `dtstart`               | RFC 5545   | i64               | Start timestamp            | Required for all events                                  |
| `summary`               | RFC 5545   | String            | Event title                | Required - core event property                           |
| `dtend`                 | RFC 5545   | i64               | End timestamp              | Common for timed events                                  |
| `duration`              | RFC 5545   | String            | Duration (alternative)     | Alternative to dtend                                     |
| `description`           | RFC 5545   | String            | Event description          | Detailed information                                     |
| `geo`                   | RFC 5545   | String            | Geographic coordinates     | Lat;lon for mapping                                      |
| `status`                | RFC 5545   | String            | Event status               | CONFIRMED/TENTATIVE/CANCELLED                            |
| `organizer`             | RFC 5545   | Organizer         | Event organizer            | Who organized (name from profile.json)                   |
| `categories`            | RFC 5545   | Vec<String>       | Event categories           | Categorization/filtering                                 |
| `created`               | RFC 5545   | i64               | Creation timestamp         | When event was created                                   |
| `sequence`              | RFC 5545   | i32               | Version number             | Enables future iTIP support                              |
| `last_modified`         | RFC 5545   | i64               | Modification timestamp     | Track changes over time                                  |
| `url`                   | RFC 7986   | String            | Event homepage             | Link to event details                                    |
| `dtstart_tzid`          | RFC 5545   | String            | Start timezone             | IANA timezone for dtstart                                |
| `dtend_tzid`            | RFC 5545   | String            | End timezone               | IANA timezone for dtend                                  |
| `rrule`                 | RFC 5545   | String            | Recurrence rule            | Recurring events                                         |
| `rdate`                 | RFC 5545   | Vec<String>       | Recurrence dates           | Additional occurrences                                   |
| `exdate`                | RFC 5545   | Vec<String>       | Exception dates            | Skip specific occurrences                                |
| `recurrence_id`         | RFC 5545   | i64               | Instance identifier        | Identify specific recurrence                             |
| `image_uri`             | RFC 7986   | String            | Event image                | Visual representation                                    |
| `conference`            | RFC 7986   | Conference        | Conference details         | Video call/meeting info                                  |
| `location`              | RFC 5545   | String            | Location text              | Primary location description                             |
| `structured_locations`  | RFC 9073   | Vec<Location>     | VLOCATION components       | Repeatable locations (ARRIVAL, PARKING, etc.)            |
| `styled_description`    | RFC 9073   | StyledDescription | Formatted description      | HTML/Markdown content                                    |
| `x_pubky_calendar_uris` | Custom     | Vec<String>       | Calendar URIs              | Calendars containing this event (see permissions)        |
| `x_pubky_rsvp_access`   | Custom     | String            | RSVP access control        | "PUBLIC" (default) / "INVITE_ONLY"                       |
| **Attendee Fields**     |            |                   |                            |                                                          |
| `attendee_uri`          | RFC 5545   | String            | Attendee Pubky URI         | Required - who is attending (name from profile.json)     |
| `partstat`              | RFC 5545   | String            | Participation status       | Required - ACCEPTED/DECLINED/TENTATIVE/NEEDS-ACTION      |
| `role`                  | RFC 5545   | String            | Attendee role              | REQ-PARTICIPANT/OPT-PARTICIPANT                          |
| `rsvp`                  | RFC 5545   | bool              | RSVP requested             | true when RSVP requested                                 |
| `delegated_from`        | RFC 5545   | String            | Delegator URI              | Pubky URI (for future delegation)                        |
| `delegated_to`          | RFC 5545   | String            | Delegate URI               | Pubky URI (for future delegation)                        |
| `recurrence_id`         | RFC 5545   | i64               | Instance identifier        | RSVP for specific recurrence                             |
| `x_pubky_event_uri`     | Custom     | String            | Event URI                  | Required - which event this RSVP is for                  |
| **Alarm Fields**        |            |                   |                            |                                                          |
| `action`                | RFC 5545   | String            | Alarm action type          | Required - DISPLAY/AUDIO/EMAIL                           |
| `trigger`               | RFC 5545   | String            | When to trigger            | Required - relative or absolute                          |
| `description`           | RFC 5545   | String            | Alarm message              | What to display                                          |
| `summary`               | RFC 5545   | String            | Email subject              | For EMAIL action                                         |
| `attendees`             | RFC 5545   | Vec<String>       | Email recipients           | For EMAIL action                                         |
| `repeat`                | RFC 5545   | i32               | Repeat count               | How many times to repeat                                 |
| `duration`              | RFC 5545   | String            | Repeat interval            | Time between repeats                                     |
| `attach`                | RFC 5545   | String            | Attachment URI             | Sound file or other resource                             |
| `x_pubky_target_uri`    | Custom     | String            | Target URI                 | Required - event or calendar URI                         |

---

## Field Validation Rules

### Calendar Fields

- `name`: **Required**. 1-255 characters.
- `color`: Optional. CSS color value (hex format recommended, e.g., `#F7931A`).
- `timezone`: **Required**. Valid IANA timezone identifier (e.g.,
  `Europe/Zurich`). Default timezone for events in this calendar when event
  doesn't specify `dtstart_tzid`. Used for display purposes only - all
  timestamps stored in UTC.
- `description`: Optional. Plain text, up to 5000 characters.
- `url`: Optional. Valid HTTP(S) URL.
- `created`: Optional. Unix microseconds **in UTC**. Creation timestamp.
- `x_pubky_admins`: Optional. Array of valid Pubky URIs. Determines which users
  (besides the calendar owner) can create events that will be aggregated into
  this calendar by Nexus. The calendar owner (derived from the URI path) is
  implicitly an admin and does NOT need to be listed here. See
  [Calendar Permissions and Event Creation](#calendar-permissions-and-event-creation)
  for detailed behavior.

### Event Fields

- `uid`: **Required**. Globally unique identifier. **Must be the event's Pubky
  URI** (e.g., `pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG`). This
  ensures global uniqueness in the decentralized system and simplifies event
  identification.
- `dtstamp`: **Required**. Unix microseconds **in UTC**. Timestamp when event
  was created/modified. Consistent with Pubky homeserver timestamp format.
- `dtstart`: **Required**. Unix microseconds **in UTC**. Event start time. All
  timestamps stored in UTC per Pubky homeserver standards.
- `dtstart_tzid`: Optional. IANA timezone ID (e.g., "Europe/Zurich") for
  **display purposes only**. Used to convert UTC → local time for users and for
  proper recurrence expansion across DST boundaries. Does not affect stored
  `dtstart` value.
- `summary`: **Required**. 1-255 characters. Event title.
- `dtend`: Optional. Unix microseconds **in UTC**. If present, must be >
  `dtstart`. Mutually exclusive with `duration`.
- `dtend_tzid`: Optional. IANA timezone ID for displaying end time in local
  timezone. Typically same as `dtstart_tzid`.
- `duration`: Optional. RFC 5545 duration format (e.g., `PT1H30M`). Mutually
  exclusive with `dtend`.
- `description`: Optional. Plain text description.
- `geo`: Optional. Geographic coordinates in "latitude;longitude" format (RFC
  5545 GEO property). Example: "47.366667;8.550000"
- `sequence`: Optional. Default 0. Increment on each modification for future
  iTIP compatibility.
- `last_modified`: Optional. Unix microseconds **in UTC**. Should update
  whenever event changes.
- `status`: Optional. Must be one of: `CONFIRMED`, `TENTATIVE`, `CANCELLED`.
- `rrule`: Optional. Must be valid RFC 5545 RRULE string. Defines the recurrence
  pattern for master events. Must be `null` for override events.
- `rdate`: Optional. Array of RFC 5545 date-time strings specifying additional
  recurrence dates beyond what `rrule` generates.
- `exdate`: Optional. Array of RFC 5545 date-time strings specifying dates to
  exclude from the recurrence pattern.
- `recurrence_id`: Optional. Unix microseconds timestamp identifying which
  occurrence of a recurring event this event overrides. **`null` or omitted =
  master event** (defines the recurrence pattern). **Non-null = override event**
  (modifies a specific occurrence). See
  [Nexus Behavior for Recurring Events](#nexus-behavior-for-recurring-events)
  for detailed behavior.
- `created`: Optional. Unix microseconds **in UTC**. Creation timestamp.
  Consistent with Pubky homeserver timestamp format.
- `image_uri`: Optional. Event image/banner Pubky URI or HTTPS URL.
- `url`: Optional. Event homepage/details URL (HTTP/HTTPS).
- `categories`: Optional. Array of category/tag strings for event
  classification.
- `styled_description`: Optional. Structured type with `value` (required) and
  optional `fmttype` (e.g., "text/html", "text/markdown"), `derived`, `altrep`,
  `language` fields per RFC 9073.
- `x_pubky_calendar_uris`: Optional. Array of calendar Pubky URIs this event
  belongs to. See
  [Calendar Permissions and Event Creation](#calendar-permissions-and-event-creation).
- `x_pubky_rsvp_access`: Optional. RSVP access control string. Default:
  "PUBLIC". Future values: "INVITE_ONLY", "CONFIRMED_ONLY", "FOLLOWERS_ONLY".
- `organizer`: Optional. Structured type with required `uri` field. Name fetched
  from profile.json.
- `conference`: Optional. Structured type with required `uri` field.
- `location`: Optional. Primary location text description (RFC 5545 LOCATION
  property).
- `geo`: Optional. Geographic coordinates in "latitude;longitude" format (RFC
  5545 GEO property). Example: "47.366667;8.550000"
- `structured_locations`: Optional. Array of Location objects (RFC 9073
  VLOCATION components). Each object has:
  - `name`: REQUIRED - Location name (e.g., "Insider Bar")
  - `location_type`: Optional - ARRIVAL, DEPARTURE, PARKING, VIRTUAL, etc.
  - `address`: Optional - Street address (e.g., "Münstergasse 20, 8001 Zürich")
  - `uri`: Optional - geo: URI or OSM link
  - `description`: Optional - Additional details (e.g., "Second floor")

**Location Field Usage:**

Pubky uses a **three-tier location approach**:

1. **`location`** (optional): Simple text description - most calendar clients
   display this
2. **`geo`** (optional): Coordinates for mapping - simple "lat;lon" format
3. **`structured_locations`** (optional): Repeatable RFC 9073 VLOCATION
   components for complex events with multiple locations (parking, arrival,
   virtual, etc.)

**For CalDAV/iCal export:**

- Map `location` → LOCATION property
- Map `geo` → GEO property
- Map `structured_locations[]` → VLOCATION components with UID, NAME,
  LOCATION-TYPE, etc.

**For CalDAV/iCal import:**

- Map LOCATION → `location`
- Map GEO → `geo`
- Map VLOCATION components → `structured_locations[]` array

### Attendee Fields

- `attendee_uri`: **Required**. Valid Pubky URI of the person attending (name
  fetched from profile.json). **Note**: This field identifies WHO is attending,
  which may differ from the storage path author in cases like: (1) Organizer
  creating invite records for invitees, (2) Future delegation scenarios, (3)
  iCalendar import. For self-RSVP, `attendee_uri` matches storage path author.
- `partstat`: **Required**. Must be one of: `ACCEPTED`, `DECLINED`, `TENTATIVE`,
  `NEEDS-ACTION`. Default: `NEEDS-ACTION`.
- `x_pubky_event_uri`: **Required**. Valid Pubky URI pointing to an event.
- `role`: Optional. Must be one of: `REQ-PARTICIPANT`, `OPT-PARTICIPANT`,
  `NON-PARTICIPANT`.
- `rsvp`: Optional. Boolean indicating if RSVP is requested.
- `delegated_from`: Optional. Valid Pubky URI (for future delegation support).
- `delegated_to`: Optional. Valid Pubky URI (for future delegation support).
- `recurrence_id`: Optional. For recurring events, specifies which instance this
  RSVP applies to. If omitted, the RSVP applies to all instances.

### Alarm Fields

- `action`: **Required**. Must be one of: `AUDIO`, `DISPLAY`, `EMAIL`.
- `trigger`: **Required**. RFC 5545 duration (e.g., `-PT15M` for 15 minutes
  before) or absolute timestamp.
- `x_pubky_target_uri`: **Required**. Valid Pubky URI pointing to an event or
  calendar.
- `description`: Optional but recommended. Alarm message text.
- `summary`: Optional. Email subject (required for EMAIL action).
- `attendees`: Optional. Array of email addresses (required for EMAIL action).
- `repeat`: Optional. Must be non-negative integer.
- `duration`: Optional. RFC 5545 duration format. Required if `repeat` is
  present.
- `attach`: Optional. URI for sound file or other attachment.

---

### ID Generation

- **Calendar ID**: Base32-encoded timestamp + random suffix (e.g.,
  `0033RCZXVEPNG`)
- **Event ID**: Base32-encoded timestamp + random suffix (e.g., `0033SCZXVEPNG`)
- **Attendee ID**: Hash of `attendee_uri` + `event_uri` + optional
  `recurrence_id`
- **Alarm ID**: Base32-encoded timestamp + random suffix

### Storage Examples

```
pubky://alice/pub/pubky.app/calendar/cal-001
pubky://alice/pub/pubky.app/event/evt-001
pubky://bob/pub/pubky.app/attendee/att-alice-evt-001
pubky://bob/pub/pubky.app/alarm/alm-001
```

---

## Graph Relationships

### Event → Calendar

Events can reference one or more parent calendars via `x_pubky_calendar_uris`.
This is optional - events can exist independently.

```rust
event.x_pubky_calendar_uris = Some(vec!["pubky://alice/pub/pubky.app/calendar/cal-001".to_string()])
```

### Attendee → Event

Attendees reference the event they're RSVPing to via `x_pubky_event_uri`. This
is required.

```rust
attendee.x_pubky_event_uri = "pubky://alice/pub/pubky.app/event/evt-001".to_string()
```

### Alarm → Event/Calendar

Alarms reference their target (event or calendar) via `x_pubky_target_uri`.

```rust
alarm.x_pubky_target_uri = "pubky://alice/pub/pubky.app/event/evt-001".to_string()
```

### Nexus Indexing

Nexus indexes these relationships to provide:

- Events by calendar
- Attendees by event
- Alarms by event/calendar
- Events by category, time range, location

### Calendar Permissions and Event Creation

#### Decentralized Event Creation Model

In Pubky's decentralized architecture, **any user can create an event that
references any calendar**, since users have full control over their own
homeserver storage. However, calendar aggregation and display is controlled by
the `x_pubky_admins` field.

**Key principles:**

1. **Event Creation**: Any user can create an event on their homeserver that
   references calendar URIs via `x_pubky_calendar_uris`. There are no permission
   checks at the homeserver level - users have sovereignty over their own data.

2. **Nexus Aggregation**: The Nexus indexer determines which events are
   aggregated into a calendar's view based on **admin permissions**. For an
   event to appear in a calendar's aggregated view:
   - The event must reference the calendar via `x_pubky_calendar_uris`
   - The event creator must be listed in the calendar's `x_pubky_admins` array
   - **OR** the event creator must be the calendar owner (derived from the
     calendar's Pubky URI path)

3. **Multi-Calendar Events**: An event can reference multiple calendars in
   `x_pubky_calendar_uris`. The Nexus indexer will aggregate the event into each
   calendar **only if** the event creator has admin rights in that specific
   calendar.

#### Example Scenarios

**Scenario 1: Calendar Owner Creates Event**

```rust
// Calendar owned by alice
calendar.uri = "pubky://alice/pub/pubky.app/calendar/cal-001"
calendar.x_pubky_admins = Some(vec!["pubky://bob"])

// Alice creates event referencing her calendar
event.uri = "pubky://alice/pub/pubky.app/event/evt-001"
event.x_pubky_calendar_uris = Some(vec!["pubky://alice/pub/pubky.app/calendar/cal-001"])
// ✅ Indexed: Alice is the calendar owner
```

**Scenario 2: Admin Creates Event**

```rust
// Calendar owned by alice, bob is admin
calendar.uri = "pubky://alice/pub/pubky.app/calendar/cal-001"
calendar.x_pubky_admins = Some(vec!["pubky://bob"])

// Bob creates event referencing alice's calendar
event.uri = "pubky://bob/pub/pubky.app/event/evt-002"
event.x_pubky_calendar_uris = Some(vec!["pubky://alice/pub/pubky.app/calendar/cal-001"])
// ✅ Indexed: Bob is listed in x_pubky_admins
```

**Scenario 3: Non-Admin Creates Event**

```rust
// Calendar owned by alice, bob is admin
calendar.uri = "pubky://alice/pub/pubky.app/calendar/cal-001"
calendar.x_pubky_admins = Some(vec!["pubky://bob"])

// Charlie creates event referencing alice's calendar
event.uri = "pubky://charlie/pub/pubky.app/event/evt-003"
event.x_pubky_calendar_uris = Some(vec!["pubky://alice/pub/pubky.app/calendar/cal-001"])
// ❌ Not indexed: Charlie is neither owner nor admin
// Event exists on Charlie's homeserver but won't appear in alice's calendar view
```

**Scenario 4: Multi-Calendar Event with Partial Access**

```rust
// Alice's calendar - bob is admin
calendar1.uri = "pubky://alice/pub/pubky.app/calendar/cal-001"
calendar1.x_pubky_admins = Some(vec!["pubky://bob"])

// Dave's calendar - bob is NOT admin
calendar2.uri = "pubky://dave/pub/pubky.app/calendar/cal-002"
calendar2.x_pubky_admins = Some(vec!["pubky://eve"])

// Bob creates event referencing both calendars
event.uri = "pubky://bob/pub/pubky.app/event/evt-004"
event.x_pubky_calendar_uris = Some(vec![
    "pubky://alice/pub/pubky.app/calendar/cal-001",
    "pubky://dave/pub/pubky.app/calendar/cal-002"
])
// ✅ Indexed in alice's calendar (bob is admin)
// ❌ NOT indexed in dave's calendar (bob is not admin)
```

#### Future: RSVP-Type Permissions

The current implementation uses `x_pubky_rsvp_access: "PUBLIC"` as the default,
allowing anyone to create attendee records for events. **Future versions** will
expand this with different RSVP behaviors:

- `"PUBLIC"` - Anyone can RSVP (current default)
- `"INVITE_ONLY"` - Only users explicitly invited can RSVP
- `"CONFIRMED_ONLY"` - Only users with CONFIRMED attendee records can
  participate
- `"FOLLOWERS_ONLY"` - Only followers of the organizer can RSVP

These permission modes will be implemented in a later iteration and will affect
how Nexus indexes and displays attendee records.

### Nexus Behavior for Recurring Events

#### Master Events vs. Override Events

Nexus differentiates between **master events** (recurring event definitions) and
**override events** (exceptions to specific occurrences) using the
`recurrence_id` field:

- **Master Event**: `recurrence_id` is `null` (or not present), and contains an
  `rrule` field defining the recurrence pattern
- **Override Event**: `recurrence_id` is set to the Unix microsecond timestamp
  of the specific occurrence being overridden, and `rrule` is `null`

Both master and override events share the same `uid` but have different storage
paths (different `event_id` values in their URIs).

**Storage Example:**

```rust
// Master event
master.uri = "pubky://alice/pub/pubky.app/event/evt-001"
master.uid = "pubky://alice/pub/pubky.app/event/evt-001"
master.recurrence_id = None
master.rrule = Some("FREQ=WEEKLY;BYDAY=WE")

// Override event for a specific occurrence
override.uri = "pubky://alice/pub/pubky.app/event/evt-123"  // Different event_id
override.uid = "pubky://alice/pub/pubky.app/event/evt-001"  // Same UID as master
override.recurrence_id = Some(1699358400000000)  // Timestamp of overridden occurrence
override.rrule = None  // Overrides don't have recurrence rules
```

#### Nexus Query Patterns

Nexus provides two ways to retrieve event data:

**1. Single Event Retrieval (with Recurring Series Context)**

```
GET /v0/event/{author_id}/{event_id}
```

- Returns the **requested event** with full details (attendees, alarms)
- **Automatically includes all related events** in the same recurring series
- Response structure:
  - `event`: The primary event object (requested by ID)
  - `related_events`: All other events with the same `uid` (master + overrides)
- Enables frontend to:
  - Display the event with complete context
  - Navigate to previous/next occurrences
  - Show all exceptions/overrides in the series
- Example: `GET /v0/event/alice/evt-001` returns the master event plus all its
  overrides
- Example: `GET /v0/event/alice/evt-123` returns an override event plus the
  master and other overrides

**2. Event Stream Retrieval**

```
GET /v0/stream/events
```

- Returns **all events** (masters and overrides) matching filter criteria
- Results sorted by `dtstart` chronologically
- Supports filtering by calendar, date range, status, location, tags
- Returns both master events (`recurrence_id: null`) and override events
- Used for calendar views, event listings, and search results
- Clients must group by `uid` to reconstruct recurring series if needed
- Example: Filtering by calendar returns all events in that calendar, including
  both master recurring events and their overrides

**When to Use Each Endpoint:**

- **Single Event (`/v0/event/{id}`)**: Event detail pages, occurrence
  navigation, editing a specific event
- **Stream (`/v0/stream/events`)**: Calendar grid views, event lists, search
  results, date range queries

#### Orphaned Override Handling

**Orphaned overrides** occur when an override event exists but the master event
has been deleted. Nexus handles this scenario as follows:

1. **Indexing**: Override events are indexed independently and remain in the
   graph database even if the master is deleted
2. **Querying**: Override events appear in `/v0/stream/events` results like any
   other event
3. **Validation**: Nexus does NOT automatically delete orphaned overrides -
   they're treated as standalone events with recurrence metadata
4. **Client Handling**: Clients should handle orphaned overrides gracefully:
   - When grouping by `uid`, check if a master event exists
   - Display orphaned overrides as standalone events if no master found
   - Show warning/notice that the recurring series no longer exists
   - Allow users to delete orphaned overrides manually

**Orphaned Override Detection:**

```typescript
// Client-side logic for detecting orphaned overrides
const eventsByUid = groupBy(events, "uid");

for (const [uid, eventsInSeries] of Object.entries(eventsByUid)) {
  const masterEvent = eventsInSeries.find((e) => e.recurrence_id === null);
  const overrides = eventsInSeries.filter((e) => e.recurrence_id !== null);

  if (!masterEvent && overrides.length > 0) {
    // Orphaned overrides detected
    console.warn(
      `Found ${overrides.length} orphaned overrides for UID: ${uid}`,
    );
    // Display as standalone events with warning
  }
}
```

**Best Practices:**

- **Event Deletion**: When deleting a master event, clients should prompt users
  to also delete all associated overrides or just do it automatically
- **Override Creation**: Clients should validate that a master event exists
  before allowing override creation
- **Stream Processing**: When processing `/v0/stream/events`, always check for
  orphaned overrides before displaying recurring series

---

## RFC Compliance and iCalendar Interoperability

This specification implements a subset of iCalendar RFCs optimized for
decentralized social calendar use cases. The following sections document known
limitations and their mitigations for .ics import/export.

### Known Limitations & Mitigations

#### 1. VTIMEZONE Components Not Stored

**Issue**: Pubky stores timezone identifiers (`dtstart_tzid: "Europe/Zurich"`)
but does not store VTIMEZONE component definitions in the data model. RFC 5545
requires VTIMEZONE definitions when using TZID parameters.

**Impact**:

- Pubky homeserver data does not include timezone offset rules
- Direct export to .ics without VTIMEZONE generation will fail validation
- Importing apps may not have timezone definitions for custom/historical
  timezones

**Mitigation**:

- **By Design**: Storing full VTIMEZONE components (with all DST rules) is
  redundant when IANA timezone database is universally available
- **Export Solution**: .ics export functions MUST dynamically generate VTIMEZONE
  components from IANA tz database
  - Rust: Use `chrono-tz` crate to generate VTIMEZONE on export
  - JavaScript: Use `tzdata` or `luxon` to generate VTIMEZONE
  - Include VTIMEZONE for all timezones referenced in DTSTART/DTEND
- **Import Solution**: .ics import validates TZID against IANA database, ignores
  VTIMEZONE definition
- **Storage Savings**: Avoiding VTIMEZONE storage saves ~500-2000 bytes per
  event with timezones
- **Documentation**: Clearly documented in `RFC-CalDAV-Interoperability.md` with
  code examples

**Example Export Flow**:

```typescript
// Pubky Event
const event = {
  dtstart: 1729965600000000, // UTC microseconds
  dtstart_tzid: "Europe/Zurich",
};

// Export to .ics
import { IANAZone } from "luxon";
const tz = IANAZone.create(event.dtstart_tzid);
const vtz = generateVTIMEZONE(tz); // Generate from IANA database

const ics = `BEGIN:VCALENDAR
VERSION:2.0
${vtz}
BEGIN:VEVENT
DTSTART;TZID=${event.dtstart_tzid}:${
  formatDateTime(event.dtstart, event.dtstart_tzid)
}
...
END:VEVENT
END:VCALENDAR`;
```

#### 2. Microsecond Timestamp Precision

**Issue**: RFC 5545 uses DATE-TIME with second precision. Pubky uses Unix
microseconds (i64) for consistency with other Pubky data types.

**Impact**: Data loss on .ics export/import cycles (microseconds truncated to
seconds).

**Mitigation**:

- Acceptable trade-off - second precision is sufficient for calendar events
- Document that Pubky native format uses microseconds
- .ics export truncates to seconds per RFC 5545

#### 3. VLOCATION as Embedded Array

**Issue**: RFC 9073 specifies VLOCATION as separate components with UIDs. We
embed as array in event structure without per-location UIDs.

**Impact**: Some strict .ics parsers may not recognize our embedded format.

**Mitigation**:

- Fully convertible to/from RFC 9073 format
- .ics export generates proper VLOCATION components with generated UIDs
- .ics import converts VLOCATION components to array
- Documented in `RFC-CalDAV-Interoperability.md`

#### 4. VALARM Simplified

**Issue**: We support basic VALARM (DISPLAY, EMAIL, AUDIO) but don't fully
support AUDIO with sound file URIs or complex PROCEDURE actions.

**Impact**: Can't roundtrip complex alarms from external calendars.

**Mitigation**:

- Acceptable for MVP - most alarms are simple DISPLAY reminders
- Import: Preserve AUDIO/EMAIL alarms with basic fields
- Export: Generate valid VALARM components

#### 5. Pubky URIs Instead of mailto:

**Design Decision**: We use `pubky://` URIs for ORGANIZER and ATTENDEE instead
of `mailto:` URIs.

**Impact**: Breaks direct import to standard calendar apps.

**Mitigation**:

- By design - Pubky is a decentralized social network, not email-based
- .ics export can generate mailto: URIs from Pubky profiles (if email available)
- .ics import with mailto: URIs can be mapped to Pubky users (if resolution
  service exists)

### Acceptable Deviations

These are intentional decisions, not compliance issues:

- **VTODO/VJOURNAL not implemented**: Out of scope for social calendar MVP
- **iTIP METHOD not implemented**: Scheduling workflow deferred to V3
- **No VFREEBUSY**: Replaced by Nexus queries
- **No VAVAILABILITY**: Not needed for event-centric use case

### Future Compliance Improvements

For production .ics interoperability, implement:

1. **Export Library**: Generate RFC-compliant .ics with VTIMEZONE, proper
   VLOCATION components, mailto: fallbacks
2. **Import Library**: Parse .ics with VTIMEZONE resolution, VLOCATION component
   conversion, mailto: → Pubky URI mapping
3. **CalDAV Bridge** (optional): Proxy server translating CalDAV ↔ Pubky
   homeserver protocols

---

## Future Extensions and Considerations

### Extension to V3 (iTIP Scheduling)

V2 is designed to be **extendible without breaking changes** to allow iTIP
scheduling:

**Fields already present that enable iTIP:**

- `sequence` - Version tracking for event updates
- `last_modified` - Change detection
- `organizer` - Structured organizer information
- `uid` - Global event identification

### Additional Future Considerations

- **Attachments**: Add `attachments` field for files/documents
- **Custom properties**: Support for `X-` prefixed custom fields beyond existing
  x_pubky_ extensions
- **Localization**: Support for multiple language variants of
  summary/description
- **Accessibility**: Add fields for accessibility information
- **Capacity limits**: Add `max_attendees` for events with limited capacity
- **Advanced RSVP**: Expand `x_pubky_rsvp_access` to support "CONFIRMED_ONLY",
  "FOLLOWERS_ONLY", etc.
