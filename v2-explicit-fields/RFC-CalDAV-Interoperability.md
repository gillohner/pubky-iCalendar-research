# RFC 5545 / CalDAV Interoperability Guide

## Overview

This document defines the bidirectional translation between Pubky calendar data
and standard iCalendar (RFC 5545) / CalDAV formats. It enables:

1. **Import**: Converting existing .ics files → Pubky calendar objects
2. **Export**: Converting Pubky calendars → .ics files for traditional clients
3. **Sync**: CalDAV bridge authenticating users via Pubky

## Core Principle

**Pubky stores the source of truth. Profile data comes from `profile.json`, not
calendar events.**

This prevents data duplication and ensures names/details stay current without
manual updates.

---

## Translation Matrix

### Calendar Properties

| Pubky Field        | RFC 5545 Property        | Direction  | Notes                                                                  |
| ------------------ | ------------------------ | ---------- | ---------------------------------------------------------------------- |
| `name`             | `X-WR-CALNAME`           | ↔          | Calendar display name                                                  |
| `timezone`         | `X-WR-TIMEZONE`          | ↔          | IANA timezone ID                                                       |
| `description`      | `X-WR-CALDESC`           | ↔          | Calendar description                                                   |
| `color`            | `X-APPLE-CALENDAR-COLOR` | ↔          | CSS hex color                                                          |
| `url`              | `URL`                    | ↔          | Calendar homepage                                                      |
| `image_uri`        | `IMAGE` (RFC 7986)       | ↔          | Calendar image                                                         |
| `created`          | `CREATED`                | ↔          | Creation timestamp (convert µs ↔ UTC)                                  |
| `x_pubky_admins`   | `X-PUBKY-ADMINS`         | Pubky only | Custom property (list of Pubky URIs)                                   |
| _(creator in URI)_ | `ORGANIZER`              | →          | Creator extracted from `pubky://<creator>/pub/pubky.app/calendar/<id>` |

### Event Properties

| Pubky Field                            | RFC 5545 Property                                                  | Direction  | Notes                                                                                                                                                                                                                |
| -------------------------------------- | ------------------------------------------------------------------ | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Required Fields**                    |                                                                    |            |                                                                                                                                                                                                                      |
| `uid`                                  | `UID`                                                              | ↔          | Globally unique ID                                                                                                                                                                                                   |
| `dtstamp`                              | `DTSTAMP`                                                          | ↔          | Creation/modification timestamp (µs → UTC)                                                                                                                                                                           |
| `dtstart`                              | `DTSTART`                                                          | ↔          | Start time (µs → UTC, with TZID parameter from `dtstart_tzid`)                                                                                                                                                       |
| `summary`                              | `SUMMARY`                                                          | ↔          | Event title                                                                                                                                                                                                          |
| **Optional Core Fields**               |                                                                    |            |                                                                                                                                                                                                                      |
| `dtend`                                | `DTEND`                                                            | ↔          | End time (µs → UTC, with TZID parameter from `dtend_tzid`). Mutually exclusive with `duration`                                                                                                                       |
| `duration`                             | `DURATION`                                                         | ↔          | Event duration (ISO 8601). Mutually exclusive with `dtend`                                                                                                                                                           |
| `description`                          | `DESCRIPTION`                                                      | ↔          | Plain text description                                                                                                                                                                                               |
| `geo`                                  | `GEO`                                                              | ↔          | Geographic coordinates ("lat;lon")                                                                                                                                                                                   |
| `status`                               | `STATUS`                                                           | ↔          | CONFIRMED / TENTATIVE / CANCELLED                                                                                                                                                                                    |
| `categories`                           | `CATEGORIES`                                                       | ↔          | Comma-separated list                                                                                                                                                                                                 |
| `created`                              | `CREATED`                                                          | ↔          | Creation timestamp                                                                                                                                                                                                   |
| `url`                                  | `URL`                                                              | ↔          | Event homepage                                                                                                                                                                                                       |
| **Organizer (Special Handling)**       |                                                                    |            |                                                                                                                                                                                                                      |
| `organizer.uri`                        | `ORGANIZER` (CN param)                                             | ↔          | **Export**: Fetch name from `profile.json` at `organizer.uri` → `ORGANIZER;CN="Satoshi":mailto:satoshi@pubky.example`<br>**Import**: Extract URI, store in `organizer.uri`. Name is fetched dynamically from profile |
| _(organizer.name)_                     | `ORGANIZER` CN param                                               | → only     | **Not stored**. Frontend fetches from profile.json                                                                                                                                                                   |
| **Versioning & Sync**                  |                                                                    |            |                                                                                                                                                                                                                      |
| `sequence`                             | `SEQUENCE`                                                         | ↔          | Version number (increment on edits)                                                                                                                                                                                  |
| `last_modified`                        | `LAST-MODIFIED`                                                    | ↔          | Last edit timestamp (µs → UTC)                                                                                                                                                                                       |
| **Timezone Support**                   |                                                                    |            |                                                                                                                                                                                                                      |
| `dtstart_tzid`                         | `DTSTART;TZID=`                                                    | ↔          | IANA timezone parameter for dtstart                                                                                                                                                                                  |
| `dtend_tzid`                           | `DTEND;TZID=`                                                      | ↔          | IANA timezone parameter for dtend                                                                                                                                                                                    |
| **Recurrence**                         |                                                                    |            |                                                                                                                                                                                                                      |
| `rrule`                                | `RRULE`                                                            | ↔          | RFC 5545 RRULE string                                                                                                                                                                                                |
| `rdate`                                | `RDATE`                                                            | ↔          | Additional occurrences (ISO 8601 array)                                                                                                                                                                              |
| `exdate`                               | `EXDATE`                                                           | ↔          | Exception dates (ISO 8601 array)                                                                                                                                                                                     |
| `recurrence_id`                        | `RECURRENCE-ID`                                                    | ↔          | Instance identifier (µs → UTC)                                                                                                                                                                                       |
| **Rich Media**                         |                                                                    |            |                                                                                                                                                                                                                      |
| `image_uri`                            | `IMAGE` (RFC 7986)                                                 | ↔          | Event image                                                                                                                                                                                                          |
| `conference`                           | `CONFERENCE` (RFC 7986)                                            | ↔          | Video call details (structured → parameters)                                                                                                                                                                         |
| `styled_description`                   | `STYLED-DESCRIPTION` (RFC 9073)                                    | ↔          | HTML/Markdown content                                                                                                                                                                                                |
| **Location (Three-Tier Approach)**     |                                                                    |            |                                                                                                                                                                                                                      |
| `location`                             | `LOCATION`                                                         | ↔          | Primary text description                                                                                                                                                                                             |
| `geo`                                  | `GEO`                                                              | ↔          | Geographic coordinates ("lat;lon")                                                                                                                                                                                   |
| `structured_locations[].name`          | `VLOCATION` (RFC 9073) NAME                                        | ↔          | Location component name (REQUIRED in each VLOCATION)                                                                                                                                                                 |
| `structured_locations[].location_type` | `VLOCATION` LOCATION-TYPE                                          | ↔          | ARRIVAL, DEPARTURE, PARKING, VIRTUAL, etc.                                                                                                                                                                           |
| `structured_locations[].address`       | `VLOCATION` STRUCTURED-DATA with FMTTYPE=application/vnd.ical+json | ↔          | Street address in STRUCTURED-DATA object                                                                                                                                                                             |
| `structured_locations[].uri`           | `VLOCATION` GEO property                                           | ↔          | geo: URI or OSM link                                                                                                                                                                                                 |
| `structured_locations[].description`   | `VLOCATION` DESCRIPTION                                            | ↔          | Additional venue details                                                                                                                                                                                             |
| **Pubky Extensions**                   |                                                                    |            |                                                                                                                                                                                                                      |
| `x_pubky_calendar_uris`                | `X-PUBKY-CALENDAR-URI` (one per URI)                               | Pubky only | Parent calendar references (array)                                                                                                                                                                                   |
| `x_pubky_rsvp_access`                  | `X-PUBKY-RSVP-ACCESS`                                              | Pubky only | "PUBLIC" = open RSVP, "INVITE_ONLY" = requires invite (future)                                                                                                                                                       |

### Attendee Properties

| Pubky Field         | RFC 5545 Property          | Direction  | Notes                                                                                                                                                                                                                         |
| ------------------- | -------------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `attendee_uri`      | `ATTENDEE` (CN param)      | ↔          | **Export**: Fetch name from `profile.json` at `attendee_uri` → `ATTENDEE;CN="Alice";PARTSTAT=ACCEPTED:mailto:alice@pubky.example`<br>**Import**: Extract URI from ATTENDEE, store in `attendee_uri`. Name fetched dynamically |
| _(attendee_name)_   | `ATTENDEE` CN param        | → only     | **Not stored**. Frontend fetches from profile.json                                                                                                                                                                            |
| `partstat`          | `PARTSTAT` parameter       | ↔          | NEEDS-ACTION / ACCEPTED / DECLINED / TENTATIVE                                                                                                                                                                                |
| `role`              | `ROLE` parameter           | ↔          | CHAIR / REQ-PARTICIPANT / OPT-PARTICIPANT / NON-PARTICIPANT                                                                                                                                                                   |
| `rsvp`              | `RSVP` parameter           | ↔          | TRUE / FALSE                                                                                                                                                                                                                  |
| `delegated_from`    | `DELEGATED-FROM` parameter | ↔          | Pubky URI of delegator                                                                                                                                                                                                        |
| `delegated_to`      | `DELEGATED-TO` parameter   | ↔          | Pubky URI of delegate                                                                                                                                                                                                         |
| `recurrence_id`     | _(matches event)_          | ↔          | Which instance this RSVP applies to (null = all)                                                                                                                                                                              |
| `x_pubky_event_uri` | `X-PUBKY-EVENT-URI`        | Pubky only | Which event this RSVP is for                                                                                                                                                                                                  |

**Important**: Attendee records are stored per-user in Pubky
(`pubky://<user>/pub/pubky.app/attendee/<id>`), not embedded in events. When
exporting to iCal, aggregate all attendee records for an event into the VEVENT's
ATTENDEE properties.

### Alarm Properties

| Pubky Field          | RFC 5545 Property    | Direction  | Notes                                      |
| -------------------- | -------------------- | ---------- | ------------------------------------------ |
| `action`             | `ACTION`             | ↔          | DISPLAY / AUDIO / EMAIL                    |
| `trigger`            | `TRIGGER`            | ↔          | Duration (e.g., "-PT15M") or absolute time |
| `description`        | `DESCRIPTION`        | ↔          | Alarm message                              |
| `uid`                | `UID`                | ↔          | Unique alarm ID                            |
| `summary`            | `SUMMARY`            | ↔          | Email subject (for EMAIL action)           |
| `attendees`          | `ATTENDEE`           | ↔          | Email recipients (for EMAIL action, array) |
| `repeat`             | `REPEAT`             | ↔          | Number of repeats                          |
| `duration`           | `DURATION`           | ↔          | Interval between repeats                   |
| `attach`             | `ATTACH`             | ↔          | Sound file URI                             |
| `x_pubky_target_uri` | `X-PUBKY-TARGET-URI` | Pubky only | Target event/calendar URI                  |

---

## Profile Integration Pattern

### The Problem

Traditional iCalendar stores names directly:

```ical
ORGANIZER;CN="Satoshi Nakamoto":mailto:satoshi@example.com
ATTENDEE;CN="Alice Smith":mailto:alice@example.com
```

If Alice changes her name, all past events show the old name. This creates stale
data.

### The Pubky Solution

**Store only URIs. Fetch names dynamically from `profile.json`.**

**Pubky Solution**:

Store only Pubky URIs. Fetch names dynamically from profile.json when rendering
UI or exporting .ics files.

```json
{
  "organizer": { "uri": "pubky://satoshi" },
  "attendees": [{ "attendee_uri": "pubky://alice", "partstat": "ACCEPTED" }]
}
```

**Implementation**:

- **Frontend**: Fetch profile.json when rendering event UI
- **Export**: Fetch profiles and populate CN parameters in .ics output
- **Import**: Extract URIs from ORGANIZER/ATTENDEE, ignore CN parameters
- **Attendees**: Store as separate per-user records at
  `/pub/pubky.app/attendee/<id>`, not embedded in events

---

## Data Type Conversions

### Timestamps and Timezones

**Storage Format**:

- All timestamps: UTC microseconds (i64)
- Optional timezone identifiers: IANA timezone IDs (e.g., "Europe/Zurich")

**Export Requirements**:

- Generate VTIMEZONE components dynamically from IANA database
- Convert microseconds to RFC 5545 DATE-TIME format
- Include TZID parameters when timezone specified

**Import Requirements**:

- Convert DATE-TIME to UTC microseconds
- Extract TZID parameter, store as timezone identifier
- Validate TZID against IANA database
- Ignore VTIMEZONE components (redundant with IANA database)

### Multi-value Fields

- **Pubky**: Arrays (`["bitcoin", "meetup"]`)
- **iCalendar**: Comma-separated (`CATEGORIES:bitcoin,meetup`)

### Location Data

**Three-tier approach**:

1. `location` → LOCATION property (simple text)
2. `geo` → GEO property (coordinates)
3. `structured_locations[]` → VLOCATION components (RFC 9073)

**VLOCATION mapping**:

- Each location in array becomes separate VLOCATION component
- Generate UIDs as `vloc-{event_uid}-{index}`
- LOCATION-TYPE: ARRIVAL, DEPARTURE, PARKING, VIRTUAL
- STRUCTURED-DATA holds address as JSON

---

## Example Translation

### Pubky Event

```json
{
  "uid": "20231031-satoshi-001@pubky",
  "dtstart": 1698757200000000,
  "dtstart_tzid": "Europe/Zurich",
  "summary": "Bitcoin Meetup Zürich",
  "organizer": { "uri": "pubky://satoshi" },
  "rrule": "FREQ=WEEKLY;BYDAY=WE"
}
```

### Exported .ics

```ical
BEGIN:VCALENDAR
VERSION:2.0
BEGIN:VTIMEZONE
TZID:Europe/Zurich
[...DST rules...]
END:VTIMEZONE
BEGIN:VEVENT
UID:20231031-satoshi-001@pubky
DTSTART;TZID=Europe/Zurich:20231031T130000
SUMMARY:Bitcoin Meetup Zürich
ORGANIZER;CN="Satoshi Nakamoto":mailto:satoshi@pubky.example
RRULE:FREQ=WEEKLY;BYDAY=WE
END:VEVENT
END:VCALENDAR
```

**Notes**:

- ORGANIZER CN fetched from profile.json at export time
- VTIMEZONE generated from IANA database
- Attendees aggregated from separate per-user records

---

## CalDAV Bridge Architecture

For full CalDAV compatibility, you'd build a bridge service:

```
┌─────────────────┐
│ CalDAV Client   │ (Thunderbird, Apple Calendar, etc.)
│ (RFC 4791)      │
└────────┬────────┘
         │ HTTP(S) + CalDAV protocol
         ▼
┌─────────────────────────┐
│ Pubky CalDAV Bridge     │
│ - Authenticates via     │
│   Pubky (no passwords)  │
│ - Translates CalDAV ↔   │
│   Pubky API calls       │
│ - Caches profile.json   │
│   names                 │
└────────┬────────────────┘
         │ Pubky protocol
         ▼
┌─────────────────────────┐
│ Pubky Homeserver        │
│ + Nexus API             │
└─────────────────────────┘
```

**Key Bridge Functions**:

1. **Authentication**: Pubky keypair instead of username/password
2. **Calendar Discovery**: Map Pubky calendars to CalDAV collections
3. **Event CRUD**: Translate VEVENT ↔ PubkyAppEvent
4. **Sync**: Use `sequence` and `last_modified` for efficient sync
