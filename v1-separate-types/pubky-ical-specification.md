# Pubky iCalendar Specification (v1.0)

## Index

- [Introduction](#introduction)
- [Background and Motivation](#background-and-motivation)
- [RFC Tag Usage Throughout This Specification](#rfc-tag-usage-throughout-this-specification)
- [Homeserver Storage Structure](#homeserver-storage-structure)
- [Data Models and Schemas](#data-models-and-schemas)
- [Future Extensions and Considerations](#future-extensions-and-considerations)

## Introduction

This specification defines the integration of industry-standard iCalendar
protocols (RFC 5545, RFC 7265, RFC 4791, RFC 5546, RFC 7986, RFC 9073) into the
Pubky ecosystem. By storing calendar components as jCal (JSON) files on user
homeservers and aggregating them through Nexus, this approach enables
decentralized calendar functionality while maintaining compatibility with
existing calendar standards.

This document follows the pubky-app-specs structure and conventions, extending
them to support calendar collections, events, scheduling, and RSVP workflows.
All calendar data uses the jCal JSON format (RFC 7265) as the native storage
format, with pubky:// URLs for addressing users and resources.

The design emphasizes simplicity and extensibility—the core specification is
lean to facilitate MVP development, while the underlying RFC standards provide a
path for future enhancements.

---

## Background and Motivation

Pubky homeservers provide decentralized, identity-based content hosting where
users maintain full control of their data. Integrating calendar functionality
following established RFC standards enables:

- **Interoperability**: Compatibility with existing calendar clients (Outlook,
  Thunderbird, Apple Calendar) via potential CalDAV bridges.
- **Decentralization**: Users store their calendar components on their own
  homeservers, eliminating dependency on centralized calendar services.
- **Social Calendaring**: Events become first-class social objects with
  potential for tagging, comments, and attendance aggregation.
- **Flexibility**: Support for public events, personal reminders, and various
  participation models.
- **Standards Compliance**: Leveraging decades of calendar protocol development
  and tooling.
- **Gradual Expansion**: Existing RFC standards are designed to be extendable,
  allowing us to start simple and expand functionality over time.

The Nexus indexer aggregates calendar components from user homeservers, enabling
discovery and feed generation similar to other pubky-app content types. Calendar
admins are identified through the `x-pubky-admins` property, and Nexus
aggregates events from these admin homeservers.

---

## RFC Tag Usage Throughout This Specification

### Property Tags by RFC Source

**From RFC 5545 (Core iCalendar):**

- `UID` - Universal identifier for calendar components (Required for all
  events/collections)
- `DTSTAMP` - Creation/modification timestamp (Required, maps to pubky
  microsecond timestamps)
- `DTSTART`/`DTEND` - Event timing (Core scheduling, VTIMEZONE aware)
- `SUMMARY` - Event title (Primary display name)
- `DESCRIPTION` - Event details (Supports multiple languages via LANGUAGE
  parameter)
- `LOCATION` - Event location (Enhanced by STRUCTURED-LOCATION from RFC 9073)
- `ORGANIZER` - Event creator (Maps to pubky:// URL)
- `ATTENDEE` - Event participants (Enhanced with pubky:// identifiers)
- `CATEGORIES` - Event classification (Bitcoin, Meetup, Conference, etc.)
- `STATUS` - Event status (`CONFIRMED`, `TENTATIVE`, `CANCELLED`)
- `RRULE` - Recurrence rules (FREQ, INTERVAL, BYDAY, etc.)

**From RFC 7265 (jCal JSON):**

- JSON array structure: `["vevent", [properties], [sub-components]]`
- Property format: `["property-name", parameters, "value-type", "value"]`
- Used as native storage format on Pubky homeservers

**From RFC 5546 (iTIP Scheduling):**

- `METHOD` - Message type (`PUBLISH`, `REQUEST`, `REPLY`, `CANCEL`)
- `PARTSTAT` - Participation status (`ACCEPTED`, `DECLINED`, `TENTATIVE`,
  `NEEDS-ACTION`)
- `ROLE` - Participant role (`CHAIR`, `REQ-PARTICIPANT`, `OPT-PARTICIPANT`)
- Used in VAttendee schema and event invitation flows

**From RFC 7986 (New Properties):**

- `NAME` - Calendar display name (Collection-level metadata)
- `COLOR` - Visual styling (`#FF5733` hex colors for calendar themes)
- `IMAGE` - Calendar/event imagery (Logo URLs for organizations)
- `CONFERENCE` - Meeting links (Jitsi, Zoom integration via URI with FEATURE
  parameter)

**From RFC 9073 (Event Publishing):**

- `PARTICIPANT` - Enhanced participant component (Conference speakers, sponsors)
- `STRUCTURED-LOCATION` - Rich location data (OpenStreetMap integration,
  coordinates)
- `PARTICIPANT-TYPE` - Role specification (`SPEAKER`, `SPONSOR`, `ORGANIZER`)

**Pubky Extensions:**

- `X-PUBKY-ADMINS` - Multi-value property listing calendar admin pubky:// URIs
- `X-PUBKY-CALENDAR` - Reference to parent calendar (used in VEVENT, VATTENDEE,
  VALARM)
- `X-PUBKY-EVENT` - Reference to event (used in VATTENDEE, VALARM)

---

## Homeserver Storage Structure

### Path Organization

Calendar components are stored in a flat structure, with Nexus responsible for
aggregating and relating components based on their reference properties:

```
/pub/pubky.app/
  ├── vcalendar/:calendar_id         # Calendar metadata (jCal JSON)
  ├── vevent/:event_id                # Individual events (jCal JSON)
  ├── vattendee/:attendee_id          # RSVP/attendance records (jCal JSON)
  └── valarm/:alarm_id                # User-defined alarms/reminders (jCal JSON)
```

### File Naming Conventions

- **Calendar ID**: Timestamp-based (13-character Crockford Base32)
- **Event ID**: Timestamp-based (13-character Crockford Base32)
- **Attendee ID**: Timestamp-based (13-character Crockford Base32)
- **Alarm ID**: Timestamp-based (13-character Crockford Base32)
- All files are stored as `.json` with jCal structure inside.
- UID is preserved within jCal for iCalendar compatibility.

### Component Relationships

Components reference each other through pubky:// URIs:

- **VEvent** → references VCalendar via `X-PUBKY-CALENDAR`
- **VAttendee** → references VCalendar via `X-PUBKY-CALENDAR` and VEvent via
  `X-PUBKY-EVENT`
- **VAlarm** → references VCalendar via `X-PUBKY-CALENDAR` and optionally VEvent
  via `X-PUBKY-EVENT`

### Example File Paths

```
pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG
pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG
pubky://hal/pub/pubky.app/vevent/0033TD0XVEPNG
pubky://alice/pub/pubky.app/vattendee/0033UCZXVEPNG
pubky://bob/pub/pubky.app/valarm/0033VCZXVEPNG
```

### Nexus Aggregation Logic

Nexus aggregates calendar components based on admin relationships:

1. **Calendar Discovery**: Index all VCalendar components and extract
   `X-PUBKY-ADMINS` property
2. **Event Aggregation**: For each calendar, aggregate VEvent components where:
   - Event's `X-PUBKY-CALENDAR` matches the calendar URI
   - Event creator's pubky:// is listed in calendar's `X-PUBKY-ADMINS`
3. **Attendee Aggregation**: Aggregate VAttendee components referencing the
   calendar/event
4. **Alarm Aggregation**: Aggregate VAlarm components from any user for any
   calendar/event

---

## Data Models and Schemas

### PubkyAppVCalendar

**Path**: `/pub/pubky.app/vcalendar/:calendar_id`

Calendar ID uses **Timestamp ID format** (13-character Crockford Base32 from
microsecond timestamp).

**UID Generation Strategy**: Calendar UIDs follow a standardized format:

```
pubky://<pubky_userid>/pub/pubky.app/vcalendar/<calendar_id>
```

**Schema Structure**:

```json
[
  "vcalendar",
  [
    ["prodid", {}, "text", "-//Pubky//Pubky Calendar 1.0//EN"],
    ["version", {}, "text", "2.0"],
    ["calscale", {}, "text", "GREGORIAN"],
    ["method", {}, "text", "PUBLISH"],
    [
      "uid",
      {},
      "text",
      "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ],
    ["name", {}, "text", "Dezentralschweiz Meetups"],
    [
      "description",
      {},
      "text",
      "Bitcoin meetups and conferences across Switzerland"
    ],
    ["color", {}, "text", "#F7931A"],
    ["image", {}, "uri", "pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"],
    ["url", {}, "uri", "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"],
    ["last-modified", {}, "date-time", "2025-10-01T12:00:00Z"],
    ["categories", {}, "text", "bitcoin", "meetups", "decentralization"],
    ["x-pubky-admins", {}, "uri", "pubky://satoshi", "pubky://adam-back"]
  ],
  []
]
```

**RFC Property Sources:**

- **RFC 7986**: `name`, `description`, `color`, `image`, `url`
- **RFC 5545**: `prodid`, `version`, `calscale`, `method`, `categories`,
  `last-modified`, `uid`
- **Pubky Extensions**: `x-pubky-admins` (multi-value property for calendar
  administrators)

**Field Descriptions:**

- `uid` (required): Globally unique identifier for the calendar.
- `name` (required): Human-readable calendar name.
- `description` (optional): Calendar description.
- `color` (optional): Hex color code for calendar theme.
- `image` (optional): pubky:// URI to calendar logo/image.
- `categories` (optional): Multi-value classification tags (TODO: Remove and
  replace with PubkyAppTags or not?)
- `x-pubky-admins` (required): Multi-value property listing pubky:// URIs of
  users who can add events to this calendar

---

### PubkyAppVEvent

**Path**: `/pub/pubky.app/vevent/:event_id`

Event ID uses **Timestamp ID format** (13-character Crockford Base32 from
microsecond timestamp).

**UID Generation Strategy**: Event UIDs follow a standardized format:

```
pubky://<pubky_userid>/pub/pubky.app/vevent/<event_id>
```

**Schema Structure**:

```json
[
  "vevent",
  [
    ["uid", {}, "text", "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"],
    ["dtstamp", {}, "date-time", "2025-10-01T12:15:00Z"],
    [
      "dtstart",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-09T19:00:00"
    ],
    ["dtend", { "tzid": "Europe/Zurich" }, "date-time", "2025-10-09T22:00:00"],
    ["summary", {}, "text", "Bitcoin Meetup Zürich"],
    [
      "description",
      {},
      "text",
      "Weekly Bitcoin meetup discussing Lightning Network"
    ],
    ["location", {}, "text", "Insider Bar, Zürich"],
    ["geo", {}, "float", [47.366667, 8.550000]],
    ["url", {}, "uri", "https://dezentralschweiz.ch/meetup/zurich"],
    ["organizer", { "cn": "Satoshi" }, "cal-address", "pubky://satoshi"],
    ["categories", {}, "text", "bitcoin", "meetup", "networking"],
    ["status", {}, "text", "CONFIRMED"],
    ["sequence", {}, "integer", 0],
    ["created", {}, "date-time", "2025-09-25T14:30:00Z"],
    ["last-modified", {}, "date-time", "2025-10-01T12:15:00Z"],
    ["color", {}, "text", "#F7931A"],
    ["image", {}, "uri", "pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ]
  ],
  []
]
```

**RFC Property Sources:**

- **RFC 5545**: `uid`, `dtstamp`, `dtstart`, `dtend`, `summary`, `description`,
  `location`, `organizer`, `categories`, `status`, `sequence`, `created`,
  `last-modified`, `geo`, `url`
- **RFC 7986**: `color`, `image`
- **Pubky Extensions**: `x-pubky-calendar` (reference to parent calendar)

**Field Descriptions:**

- `uid` (required): Globally unique identifier for the event.
- `dtstamp` (required): Timestamp of event creation/modification.
- `dtstart` (required): Event start time (with optional timezone).
- `dtend` (optional): Event end time.
- `summary` (required): Event title/name.
- `description` (optional): Detailed event description.
- `location` (optional): Event location text.
- `geo` (optional): Geographic coordinates [latitude, longitude].
- `organizer` (required): pubky:// URI of event organizer.
- `x-pubky-calendar` (required): pubky:// URI of parent calendar.
- `categories` (optional): Multi-value classification tags.
- `status` (optional): Event status (CONFIRMED, TENTATIVE, CANCELLED).

**Recurrence Support**: Events can include `rrule` property for recurring
patterns (RFC 5545 Section 3.3.10):

```json
["rrule", {}, "recur", {
  "freq": "WEEKLY",
  "byday": ["WE"],
  "until": "2025-12-31T23:59:59Z"
}]
```

**Recurrence Overrides and Exceptions**: Individual instances of recurring
events can be modified or excluded using standard iCalendar mechanisms.

**Excluding Instances (EXDATE)**:

```json
[
  "exdate",
  { "tzid": "Europe/Zurich" },
  "date-time",
  "2025-10-16T19:00:00",
  "2025-10-23T19:00:00"
]
```

**Adding Extra Instances (RDATE)**:

```json
[
  "rdate",
  { "tzid": "Europe/Zurich" },
  "date-time",
  "2025-10-15T19:00:00"
]
```

**Modifying Individual Instances (RECURRENCE-ID)**:

Create a separate VEVENT file with modified properties for a specific
occurrence. The recurrence-id property identifies which instance is being
overridden.

Path: `pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG`

```json
[
  "vevent",
  [
    ["uid", {}, "text", "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"],
    [
      "dtstart",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-09T19:00:00"
    ],
    ["summary", {}, "text", "Bitcoin Meetup Zürich"],
    ["rrule", {}, "recur", { "freq": "WEEKLY", "byday": ["WE"], "count": 10 }],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ]
  ],
  []
]
```

Override for October 16th instance (separate file):

Path: `pubky://satoshi/pub/pubky.app/vevent/0033TCZXVEPNG`

```json
[
  "vevent",
  [
    ["uid", {}, "text", "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"],
    [
      "recurrence-id",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-16T19:00:00"
    ],
    [
      "dtstart",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-16T20:00:00"
    ],
    ["dtend", { "tzid": "Europe/Zurich" }, "date-time", "2025-10-16T23:00:00"],
    [
      "summary",
      {},
      "text",
      "Bitcoin Meetup Zürich - Special Lightning Workshop"
    ],
    ["location", {}, "text", "Different Venue, Zürich"],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ]
  ],
  []
]
```

Key points for overrides:

- Override events must share the same uid as the master recurring event.
- The recurrence-id value must match the original occurrence time (October 16th
  at 19:00, even if the start time is moved to 20:00).
- Override events are stored as separate files on the homeserver.
- Any property can be modified in the override (time, location, summary,
  attendees, etc.).
- Nexus aggregates both the master event and override instances when querying
  the calendar

---

### PubkyAppVAttendee

**Path**: `/pub/pubky.app/vattendee/:attendee_id`

Attendee ID uses **Timestamp ID format** (13-character Crockford Base32 from
microsecond timestamp).

**Purpose**: Represents a user's RSVP/attendance record for an event. Any user
can create an attendee record for any event.

**Schema Structure**:

```json
[
  "attendee",
  [
    ["uid", {}, "text", "pubky://alice/pub/pubky.app/vattendee/0033ZCZXVEPNG"],
    ["dtstamp", {}, "date-time", "2025-10-02T09:30:00Z"],
    [
      "attendee",
      {
        "cn": "Alice",
        "role": "REQ-PARTICIPANT",
        "partstat": "ACCEPTED",
        "rsvp": "TRUE"
      },
      "cal-address",
      "pubky://alice"
    ],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ],
    [
      "x-pubky-event",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"
    ]
  ],
  []
]
```

**RFC Property Sources:**

- **RFC 5545**: `uid`, `dtstamp`, `attendee`
- **RFC 5546**: `attendee` parameters (`cn`, `role`, `partstat`, `rsvp`)
- **Pubky Extensions**: `x-pubky-calendar`, `x-pubky-event`

**Field Descriptions:**

- `uid` (required): Globally unique identifier for the attendance record.
- `dtstamp` (required): Timestamp of RSVP creation/modification.
- `attendee` (required): Attendee details with parameters:
  - `cn`: Common name (display name)
  - `role`: REQ-PARTICIPANT, OPT-PARTICIPANT, NON-PARTICIPANT, CHAIR
  - `partstat`: NEEDS-ACTION, ACCEPTED, DECLINED, TENTATIVE, DELEGATED
  - `rsvp`: TRUE/FALSE (whether a response is requested)
- `x-pubky-calendar` (required): pubky:// URI of parent calendar.
- `x-pubky-event` (required): pubky:// URI of the event.

---

### PubkyAppVAlarm

**Path**: `/pub/pubky.app/valarm/:alarm_id`

Alarm ID uses **Timestamp ID format** (13-character Crockford Base32 from
microsecond timestamp).

**Purpose**: Represents a user-defined reminder/alarm for a calendar or event.
Any user can create alarms for any calendar/event to manage their personal
notifications.

**Schema Structure**:

```json
[
  "valarm",
  [
    ["uid", {}, "text", "pubky://bob/pub/pubky.app/valarm/0033WCZXVEPNG"],
    ["action", {}, "text", "DISPLAY"],
    ["trigger", { "related": "START" }, "duration", "-PT15M"],
    ["description", {}, "text", "Bitcoin Meetup in 15 minutes"],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ],
    [
      "x-pubky-event",
      {},
      "uri",
      "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"
    ]
  ],
  []
]
```

**RFC Property Sources:**

- **RFC 5545**: `uid`, `action`, `trigger`, `description`
- **Pubky Extensions**: `x-pubky-calendar`, `x-pubky-event`

**Field Descriptions:**

- `uid` (required): Globally unique identifier for the alarm.
- `action` (required): AUDIO, DISPLAY, EMAIL (alarm type).
- `trigger` (required): When to trigger (duration relative to START/END, or
  absolute date-time).
- `description` (optional): Alarm message text.
- `x-pubky-calendar` (required): pubky:// URI of parent calendar.
- `x-pubky-event` (optional): pubky:// URI of specific event (if alarm is
  event-specific).

**Trigger Examples**:

```json
// 15 minutes before event start
["trigger", {"related": "START"}, "duration", "-PT15M"]

// 1 day before event start
["trigger", {"related": "START"}, "duration", "-P1D"]

// Absolute timestamp
["trigger", {}, "date-time", "2025-10-09T18:45:00Z"]
```

---

## Future Extensions and Considerations

This specification intentionally keeps the core protocol simple to facilitate
MVP development. The underlying RFC standards provide numerous extension points
for future enhancements:

### Potential Advanced Features

**Enhanced Access Control** (using RFC 4791 patterns):

- Calendar-level RSVP policies (`x-pubky-rsvp-policy`: open, approval-required,
  invite-only)
- Event submission workflows with approval queues
- Granular permissions (read, write, admin roles)
- Auto-approval lists for trusted contributors

**Rich Event Features** (RFC 9073):

- `PARTICIPANT` components for conference speakers, sponsors, performers
- `STYLED-DESCRIPTION` for rich text formatting
- Enhanced location data with `STRUCTURED-LOCATION` and OpenStreetMap
  integration

**Advanced Scheduling** (RFC 5546, RFC 6638):

- iTIP message flows for formal invitations
- Free/busy time aggregation
- Delegation and proxy workflows
- Server-side scheduling automation

**Extended Alarm Capabilities** (RFC 9074):

- `PROXIMITY` triggers (location-based reminders)
- Complex alarm relationships with `RELATED-TO`
- Snooze and acknowledgment tracking

**Calendar Subscriptions**:

- `REFRESH-INTERVAL` for periodic updates
- `SOURCE` property for calendar syndication
- CalDAV-style sync tokens and change tracking

### Implementation Notes

The current specification establishes the foundation for these features without
requiring their immediate implementation. Calendar and event components already
include standard RFC properties that support these extensions, allowing gradual
feature expansion without breaking changes to the core protocol.

For the MVP implementation, focus is set on the core schemas (VCalendar, VEvent,
VAttendee, VAlarm) and basic Nexus aggregation. Advanced features can be layered
on as the ecosystem matures and user needs become clearer. It is important to
have an extensible base structure.

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
