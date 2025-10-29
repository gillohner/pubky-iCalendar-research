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

## Architectural Approach

### Helper Types

```rust
use serde::{Deserialize, Serialize};

// RFC 5545 Organizer - represents event organizer
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Organizer {
    pub uri: String,                    // REQUIRED Pubky URI (name fetched from profile.json)
    pub sent_by: Option<String>,        // Pubky URI of delegate sending on behalf of organizer
}

// RFC 7986 Conference - video/audio conference details
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Conference {
    pub uri: String,                    // REQUIRED - Conference URL (Zoom, Meet, etc.)
    pub label: Option<String>,          // Human-readable label (e.g., "Zoom Meeting")
    pub features: Option<Vec<String>>, // Conference features: AUDIO, VIDEO, CHAT, SCREEN, etc.
}

// RFC 9073 VLOCATION - structured location component (repeatable)
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct StructuredLocation {
    pub name: String,                   // REQUIRED - Location name (e.g., "Insider Bar", "Parking Lot B")
    pub location_type: Option<String>,  // RFC 9073 LOCATION-TYPE: ARRIVAL, DEPARTURE, PARKING, etc.
    pub address: Option<String>,        // Street address (e.g., "Münstergasse 20, 8001 Zürich")
    pub uri: Option<String>,            // Reference URI (OSM node, website, etc.)
    pub description: Option<String>,    // Additional details (e.g., "Second floor, near the bar")
}

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
    pub timezone: Option<String>,          // IANA timezone ID (e.g., "Europe/Zurich")
    pub description: Option<String>,       // Calendar description
    pub url: Option<String>,               // Calendar homepage/details URL
    pub created: Option<i64>,              // Creation timestamp (Unix microseconds)

    // Pubky Extensions (all custom fields use x_pubky_ prefix)
    pub x_pubky_admins: Option<Vec<String>>,  // Pubky URIs of admin users
}

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
    pub structured_locations: Option<Vec<StructuredLocation>>, // RFC 9073 VLOCATION components (repeatable)
    
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

// Attendee - an RSVP/participation record for an event
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAttendee {
    // RFC 5545 - Attendee Properties
    pub attendee_uri: String,           // REQUIRED - Pubky URI of attendee (name from profile.json)
                                        // Note: Kept separate from storage path for delegation & import support
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

### Location Data Strategy

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
- **Format**: Array of `StructuredLocation` objects
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

## Type Definitions

### PubkyAppCalendar

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppCalendar {
    // RFC 7986 - Calendar Properties
    pub name: String,                      // REQUIRED - calendar display name
    pub color: Option<String>,             // CSS color value (hex format)
    pub image_uri: Option<String>,         // Calendar image/logo URI (pubky:// or https)
    
    // RFC 5545 - Calendar Metadata
    pub timezone: String,                  // REQUIRED - IANA timezone ID (e.g., "Europe/Zurich")
    pub description: Option<String>,       // Calendar description
    pub url: Option<String>,               // Calendar homepage URL
    pub created: Option<i64>,              // Creation timestamp (Unix microseconds)
    
    // Pubky Extensions
    pub x_pubky_admins: Option<Vec<String>>,  // Pubky URIs of admin users
}
```

### PubkyAppEvent

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppEvent {
    // RFC 5545 - Core Event Properties (REQUIRED)
    pub uid: String,                       // Globally unique identifier
    pub dtstamp: i64,                      // Creation timestamp (microseconds)
    pub dtstart: i64,                      // Start timestamp (microseconds)
    pub summary: String,                   // Event title/subject
    
    // RFC 5545 - Time & Duration
    pub dtend: Option<i64>,                // End timestamp (mutually exclusive with duration)
    pub duration: Option<String>,          // RFC 5545 duration format (e.g., "PT1H30M")
    pub dtstart_tzid: Option<String>,      // IANA timezone for dtstart (e.g., "Europe/Zurich")
    pub dtend_tzid: Option<String>,        // IANA timezone for dtend
    
    // RFC 5545 - Event Details
    pub description: Option<String>,       // Plain text description
    pub status: Option<String>,            // CONFIRMED | TENTATIVE | CANCELLED
    pub organizer: Option<Organizer>,      // Event organizer (name from profile.json)
    pub categories: Option<Vec<String>>,   // Event categories/tags
    
    // RFC 5545 - Location
    pub location: Option<String>,          // Primary location text (RFC 5545 LOCATION property)
    pub geo: Option<String>,               // Geographic coordinates "lat;lon" (RFC 5545 GEO property)
    pub structured_locations: Option<Vec<StructuredLocation>>, // RFC 9073 VLOCATION components (repeatable)
    
    // RFC 7986 - Event Publishing Extensions
    pub image_uri: Option<String>,         // Event image/banner URI
    pub url: Option<String>,               // Event homepage/details link
    pub conference: Option<Conference>,    // Video/audio conference details
    
    // RFC 5545 - Change Management
    pub sequence: Option<i32>,             // Version number (increment on modifications)
    pub last_modified: Option<i64>,        // Last modification timestamp
    pub created: Option<i64>,              // Creation timestamp
    
    // RFC 5545 - Recurrence
    pub rrule: Option<String>,             // Recurrence rule (RFC 5545 format)
    pub rdate: Option<Vec<String>>,        // Additional recurrence dates
    pub exdate: Option<Vec<String>>,       // Excluded recurrence dates
    pub recurrence_id: Option<i64>,        // Identifies specific recurrence instance
    
    // RFC 9073 - Rich Content
    pub styled_description: Option<StyledDescription>, // Formatted description with metadata

    // Pubky Extensions
    pub x_pubky_calendar_uris: Option<Vec<String>>, // URIs of calendars containing this event
    pub x_pubky_rsvp_access: Option<String>, // "PUBLIC" (default) | "INVITE_ONLY" | "CONFIRMED_ONLY"
}
```

### Supporting Types

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Organizer {
    pub uri: String,                       // REQUIRED Pubky URI (name fetched from profile.json)
    pub sent_by: Option<String>,           // Pubky URI of delegate sending on behalf
}

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct Conference {
    pub uri: String,                       // REQUIRED - Conference URL (Zoom, Meet, etc.)
    pub label: Option<String>,             // Human-readable label
    pub features: Option<Vec<String>>,     // Conference features: AUDIO, VIDEO, CHAT, SCREEN, etc.
}

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct StructuredLocation {
    pub name: String,                      // REQUIRED - Location name
    pub location_type: Option<String>,     // RFC 9073 LOCATION-TYPE: ARRIVAL, DEPARTURE, PARKING, etc.
    pub address: Option<String>,           // Street address
    pub uri: Option<String>,               // Reference URI (OSM node, website, etc.)
    pub description: Option<String>,       // Additional instructions
}

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct VLocation {
    pub name: String,                      // REQUIRED - Location name (RFC 9073 NAME property)
    pub location_type: Option<String>,     // RFC 9073 LOCATION-TYPE: ARRIVAL, DEPARTURE, PARKING, VIRTUAL, etc.
    pub address: Option<String>,           // Street address
    pub uri: Option<String>,               // Reference URI (OSM node, geo: URI, website, etc.)
    pub description: Option<String>,       // Additional instructions or details
}

#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct StyledDescription {
    pub fmttype: Option<String>,           // Media type (e.g., "text/html", "text/markdown")
    pub value: String,                     // The styled content
    pub derived: Option<bool>,             // RFC 9073 - TRUE if auto-derived from DESCRIPTION
    pub altrep: Option<String>,            // RFC 9073 - Alternate representation URI
    pub language: Option<String>,          // RFC 9073 - Language tag (e.g., "en-US")
}

// Example usage:
let event = PubkyAppEvent {
    uid: "20231031-satoshi-001@pubky".to_string(),
    dtstamp: 1698753600000000,
    dtstart: 1698757200000000,
    summary: "Bitcoin Meetup Zürich".to_string(),
    dtend: Some(1698764400000000),
    duration: None,
    description: Some("Weekly Bitcoin meetup discussing Lightning Network".to_string()),
    location: Some("Insider Bar, Zürich".to_string()),
    geo: Some("47.366667;8.550000".to_string()),
    structured_locations: Some(vec![VLocation {
        name: "Insider Bar".to_string(),
        location_type: Some("ARRIVAL".to_string()),
        address: Some("Münstergasse 20, 8001 Zürich".to_string()),
        uri: Some("https://www.openstreetmap.org/node/123456789".to_string()),
        description: Some("Main venue, second floor".to_string()),
    }]),
    status: Some("CONFIRMED".to_string()),
    organizer: Some(Organizer {
        uri: "pubky://satoshi".to_string(),  // Name fetched from profile.json
        sent_by: None,
    }),
    categories: Some(vec!["bitcoin".to_string(), "meetup".to_string()]),
    created: Some(1698753600000000),
    sequence: Some(0),
    last_modified: Some(1698753600000000),
    url: Some("https://bitcoin.ch/meetup/zurich".to_string()),
    dtstart_tzid: Some("Europe/Zurich".to_string()),
    dtend_tzid: Some("Europe/Zurich".to_string()),
    rrule: Some("FREQ=WEEKLY;BYDAY=WE".to_string()),
    rdate: None,
    exdate: None,
    recurrence_id: None,
    image_uri: Some("pubky://satoshi/pub/pubky.app/files/0033EVENT01".to_string()),
    conference: Some(Conference {
        uri: "https://meet.jit.si/bitcoin-zurich".to_string(),
        label: Some("Jitsi Meeting".to_string()),
        features: Some(vec!["AUDIO".to_string(), "VIDEO".to_string()]),
    }),
    styled_description: Some(StyledDescription {
        fmttype: Some("text/html".to_string()),
        value: "<p>Weekly Bitcoin meetup discussing <strong>Lightning Network</strong></p>".to_string(),
        derived: None,
        altrep: None,
        language: Some("en-US".to_string()),
    }),
    x_pubky_calendar_uris: Some(vec!["pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG".to_string()]),
    x_pubky_rsvp_access: Some("PUBLIC".to_string()),
};
```

### PubkyAppAttendee

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAttendee {
    // Public RSVP model: anyone can create an attendee record for any public event
    // No invitation workflow - simple attendance tracking
    
    // RFC 5545 / 5546 - Attendee/RSVP Properties
    pub attendee_uri: String,              // REQUIRED - Pubky URI of attendee (name from profile.json)
    pub partstat: String,                  // REQUIRED - ACCEPTED | DECLINED | TENTATIVE | NEEDS-ACTION
    pub role: Option<String>,              // REQ-PARTICIPANT | OPT-PARTICIPANT | NON-PARTICIPANT
    pub rsvp: Option<bool>,                // true when RSVP requested/supported
    pub delegated_from: Option<String>,    // Pubky URI (for future delegation support)
    pub delegated_to: Option<String>,      // Pubky URI (for future delegation support)
    
    // RFC 5545 - Recurrence Support
    pub recurrence_id: Option<i64>,        // For recurring events, specifies which instance this RSVP applies to

    // Pubky Linkage
    pub x_pubky_event_uri: String,         // REQUIRED - URI of the event this RSVP belongs to
}

// Example usage - RSVP to all instances:
let attendee_all = PubkyAppAttendee {
    attendee_uri: "pubky://alice".to_string(),
    partstat: "ACCEPTED".to_string(),
    role: Some("REQ-PARTICIPANT".to_string()),
    rsvp: None,
    delegated_from: None,
    delegated_to: None,
    recurrence_id: None, // No recurrence_id = applies to all instances
    x_pubky_event_uri: "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string(),
};

// Example usage - RSVP to specific instance:
let attendee_specific = PubkyAppAttendee {
    attendee_uri: "pubky://bob".to_string(),
    partstat: "ACCEPTED".to_string(),
    role: Some("REQ-PARTICIPANT".to_string()),
    rsvp: None,
    delegated_from: None,
    delegated_to: None,
    recurrence_id: Some(1699358400000000), // Specific instance timestamp
    x_pubky_event_uri: "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string(),
};
```

### PubkyAppAlarm

```rust
#[derive(Serialize, Deserialize, Debug, Clone)]
pub struct PubkyAppAlarm {
    // RFC 5545 - Alarm Properties
    pub action: String,                    // REQUIRED - AUDIO | DISPLAY | EMAIL
    pub trigger: String,                   // REQUIRED - Duration (e.g., "-PT15M") or absolute time
    pub description: Option<String>,       // Alarm message text
    pub summary: Option<String>,           // Email subject (for EMAIL action)
    pub attendees: Option<Vec<String>>,    // Email recipients (for EMAIL action)
    
    // RFC 5545 - Repeat functionality
    pub repeat: Option<i32>,               // Number of times to repeat alarm
    pub duration: Option<String>,          // Interval between repeats (e.g., "PT5M")
    pub attach: Option<String>,            // Sound file or attachment URI

    // Pubky Linkage
    pub x_pubky_target_uri: String,        // REQUIRED - Target event or calendar URI
}

// Example usage:
let alarm = PubkyAppAlarm {
    action: "DISPLAY".to_string(),
    trigger: "-PT15M".to_string(), // 15 minutes before
    description: Some("Bitcoin Meetup in 15 minutes".to_string()),
    summary: None,
    attendees: None,
    repeat: Some(2),               // Repeat 2 more times
    duration: Some("PT5M".to_string()), // Every 5 minutes
    attach: None,
    x_pubky_target_uri: "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG".to_string(),
};
```

---

## RFC Field Tables

### Fields Used in MVP Implementation

| Field                   | RFC Source | Type              | Description                | Rationale                                            |
| ----------------------- | ---------- | ----------------- | -------------------------- | ---------------------------------------------------- |
| **Calendar Fields**     |            |                   |                            |                                                      |
| `name`                  | RFC 7986   | String            | Calendar display name      | Essential for calendar identification                |
| `color`                 | RFC 7986   | String            | CSS color value            | Important for visual calendar organization           |
| `image_uri`             | RFC 7986   | String            | Calendar image/logo URI    | Visual branding for calendars                        |
| `x_pubky_admins`        | Custom     | Vec<String>       | Calendar admin URIs        | Core to Pubky's decentralized admin model            |
| `timezone`              | RFC 5545   | String            | IANA timezone ID           | Essential for proper time handling                   |
| `created`               | RFC 5545   | i64               | Creation timestamp         | Standard metadata field                              |
| `description`           | RFC 7986   | String            | Calendar description       | Helpful for calendar discovery                       |
| `url`                   | RFC 7986   | String            | Calendar homepage          | Link to more information                             |
| **Event Fields**        |            |                   |                            |                                                      |
| `uid`                   | RFC 5545   | String            | Globally unique identifier | Required for all calendar components                 |
| `dtstamp`               | RFC 5545   | i64               | Creation timestamp         | Required for proper iCalendar semantics              |
| `dtstart`               | RFC 5545   | i64               | Start timestamp            | Required for all events                              |
| `summary`               | RFC 5545   | String            | Event title                | Required - core event property                       |
| `dtend`                 | RFC 5545   | i64               | End timestamp              | Common for timed events                              |
| `duration`              | RFC 5545   | String            | Duration (alternative)     | Alternative to dtend                                 |
| `description`           | RFC 5545   | String            | Event description          | Detailed information                                 |
| `geo`                   | RFC 5545   | String            | Geographic coordinates     | Lat;lon for mapping                                  |
| `status`                | RFC 5545   | String            | Event status               | CONFIRMED/TENTATIVE/CANCELLED                        |
| `organizer`             | RFC 5545   | Organizer         | Event organizer            | Who organized (name from profile.json)               |
| `categories`            | RFC 5545   | Vec<String>       | Event categories           | Categorization/filtering                             |
| `created`               | RFC 5545   | i64               | Creation timestamp         | When event was created                               |
| `sequence`              | RFC 5545   | i32               | Version number             | Enables future iTIP support                          |
| `last_modified`         | RFC 5545   | i64               | Modification timestamp     | Track changes over time                              |
| `url`                   | RFC 7986   | String            | Event homepage             | Link to event details                                |
| `dtstart_tzid`          | RFC 5545   | String            | Start timezone             | IANA timezone for dtstart                            |
| `dtend_tzid`            | RFC 5545   | String            | End timezone               | IANA timezone for dtend                              |
| `rrule`                 | RFC 5545   | String            | Recurrence rule            | Recurring events                                     |
| `rdate`                 | RFC 5545   | Vec<String>       | Recurrence dates           | Additional occurrences                               |
| `exdate`                | RFC 5545   | Vec<String>       | Exception dates            | Skip specific occurrences                            |
| `recurrence_id`         | RFC 5545   | i64               | Instance identifier        | Identify specific recurrence                         |
| `image_uri`             | RFC 7986   | String            | Event image                | Visual representation                                |
| `conference`            | RFC 7986   | Conference        | Conference details         | Video call/meeting info                              |
| `location`              | RFC 5545   | String            | Location text              | Primary location description                         |
| `structured_locations`  | RFC 9073   | Vec<VLocation>    | VLOCATION components       | Repeatable locations (ARRIVAL, PARKING, etc.)        |
| `styled_description`    | RFC 9073   | StyledDescription | Formatted description      | HTML/Markdown content                                |
| `x_pubky_calendar_uris` | Custom     | Vec<String>       | Calendar URIs              | Calendars containing this event                      |
| `x_pubky_rsvp_access`   | Custom     | String            | RSVP access control        | "PUBLIC" (default) / "INVITE_ONLY"                   |
| **Attendee Fields**     |            |                   |                            |                                                      |
| `attendee_uri`          | RFC 5545   | String            | Attendee Pubky URI         | Required - who is attending (name from profile.json) |
| `partstat`              | RFC 5545   | String            | Participation status       | Required - ACCEPTED/DECLINED/TENTATIVE/NEEDS-ACTION  |
| `role`                  | RFC 5545   | String            | Attendee role              | REQ-PARTICIPANT/OPT-PARTICIPANT                      |
| `rsvp`                  | RFC 5545   | bool              | RSVP requested             | true when RSVP requested                             |
| `delegated_from`        | RFC 5545   | String            | Delegator URI              | Pubky URI (for future delegation)                    |
| `delegated_to`          | RFC 5545   | String            | Delegate URI               | Pubky URI (for future delegation)                    |
| `recurrence_id`         | RFC 5545   | i64               | Instance identifier        | RSVP for specific recurrence                         |
| `x_pubky_event_uri`     | Custom     | String            | Event URI                  | Required - which event this RSVP is for              |
| **Alarm Fields**        |            |                   |                            |                                                      |
| `action`                | RFC 5545   | String            | Alarm action type          | Required - DISPLAY/AUDIO/EMAIL                       |
| `trigger`               | RFC 5545   | String            | When to trigger            | Required - relative or absolute                      |
| `description`           | RFC 5545   | String            | Alarm message              | What to display                                      |
| `summary`               | RFC 5545   | String            | Email subject              | For EMAIL action                                     |
| `attendees`             | RFC 5545   | Vec<String>       | Email recipients           | For EMAIL action                                     |
| `repeat`                | RFC 5545   | i32               | Repeat count               | How many times to repeat                             |
| `duration`              | RFC 5545   | String            | Repeat interval            | Time between repeats                                 |
| `attach`                | RFC 5545   | String            | Attachment URI             | Sound file or other resource                         |
| `x_pubky_target_uri`    | Custom     | String            | Target URI                 | Required - event or calendar URI                     |

---

## Field Validation Rules

### Calendar Fields

- `name`: **Required**. 1-255 characters.
- `color`: Optional. CSS color value (hex format recommended, e.g., `#F7931A`).
- `timezone`: **Required**. Valid IANA timezone identifier (e.g.,
  `Europe/Zurich`).
- `description`: Optional. Plain text, up to 5000 characters.
- `url`: Optional. Valid HTTP(S) URL.
- `x_pubky_admins`: Optional. Array of valid Pubky URIs. Does NOT include
  creator by default (creator is implicit from calendar URI path).

### Event Fields

- `uid`: **Required**. Globally unique identifier. Recommended format:
  `<timestamp>-<user_id>-<random>@pubky`
- `dtstamp`: **Required**. Unix microseconds. Timestamp when event was
  created/modified.
- `dtstart`: **Required**. Unix microseconds. Event start time.
- `summary`: **Required**. 1-255 characters. Event title.
- `dtend`: Optional. If present, must be > `dtstart`. Mutually exclusive with
  `duration`.
- `duration`: Optional. RFC 5545 duration format (e.g., `PT1H30M`). Mutually
  exclusive with `dtend`.
- `description`: Optional. Plain text description.
- `geo`: Optional. Geographic coordinates in "latitude;longitude" format (RFC
  5545 GEO property). Example: "47.366667;8.550000"
- `dtstart_tzid`: Optional. IANA timezone ID for dtstart (e.g.,
  "Europe/Zurich").
- `dtend_tzid`: Optional. IANA timezone ID for dtend.
- `sequence`: Optional. Default 0. Increment on each modification for future
  iTIP compatibility.
- `last_modified`: Optional. Should update whenever event changes.
- `status`: Optional. Must be one of: `CONFIRMED`, `TENTATIVE`, `CANCELLED`.
- `rrule`: Optional. Must be valid RFC 5545 RRULE string.
- `organizer`: Optional. Structured type with required `uri` field. Name fetched
  from profile.json.
- `conference`: Optional. Structured type with required `uri` field.
- `location`: Optional. Primary location text description (RFC 5545 LOCATION
  property).
- `geo`: Optional. Geographic coordinates in "latitude;longitude" format (RFC
  5545 GEO property). Example: "47.366667;8.550000"
- `structured_locations`: Optional. Array of VLocation objects (RFC 9073
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

- `attendee_uri`: **Required**. Valid Pubky URI. Name fetched from profile.json.
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

## Homeserver Storage Structure

```
/pub/pubky.app/
├── calendar/:calendar_id     # Calendar metadata
├── event/:event_id           # Individual events
├── attendee/:attendee_id     # RSVP records
└── alarm/:alarm_id           # User reminders
```

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
