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

**Pubky Storage**:

```json
{
    "organizer": {
        "uri": "pubky://satoshi"
    },
    "attendees": [
        { "attendee_uri": "pubky://alice", "partstat": "ACCEPTED" }
    ]
}
```

**Frontend Resolution** (happens when rendering UI):

```typescript
async function getDisplayName(pubkyUri: string): Promise<string> {
    const profile = await fetch(`${pubkyUri}/pub/pubky.app/profile.json`);
    return profile.name || pubkyUri.split("//")[1]; // Fallback to ID
}

// Render:
const organizerName = await getDisplayName(event.organizer.uri); // "Satoshi Nakamoto"
const attendeeName = await getDisplayName(attendee.attendee_uri); // "Alice Smith"
```

**CalDAV Export** (when user exports to .ics):

```typescript
async function exportToICS(event: PubkyAppEvent): Promise<string> {
    const organizerName = await getDisplayName(event.organizer.uri);
    const attendees = await Promise.all(
        event.attendees.map(async (att) => ({
            uri: att.attendee_uri,
            name: await getDisplayName(att.attendee_uri),
            partstat: att.partstat,
        })),
    );

    return `BEGIN:VEVENT
UID:${event.uid}
ORGANIZER;CN="${organizerName}":mailto:${
        event.organizer.uri.replace("pubky://", "")
    }@pubky.example
${
        attendees.map((att) =>
            `ATTENDEE;CN="${att.name}";PARTSTAT=${att.partstat}:mailto:${
                att.uri.replace("pubky://", "")
            }@pubky.example`
        ).join("\n")
    }
SUMMARY:${event.summary}
...
END:VEVENT`;
}
```

**CalDAV Import** (when user imports .ics):

```typescript
function importFromICS(icalData: string): PubkyAppEvent {
  const event = parseICS(icalData);
  
  return {
    organizer: {
      uri: extractPubkyURI(event.ORGANIZER), // "pubky://satoshi"
      // CN parameter is IGNORED - will be fetched from profile.json
    },
    // Attendees created separately in /pub/pubky.app/attendee/
    ...
  };
}
```

**Key Benefits**:

- ✅ Names always current (even in old events)
- ✅ No data duplication
- ✅ Single source of truth (`profile.json`)
- ✅ Storage savings (don't duplicate names across thousands of events)

---

## Data Type Conversions

### Timestamps

**Pubky → iCalendar**:

```typescript
// Pubky stores Unix microseconds (i64)
const pubkyTimestamp = 1698753600000000; // µs

// Convert to UTC DateTime for iCalendar
const date = new Date(pubkyTimestamp / 1000); // ms
const icalDate = date.toISOString().replace(/[-:]/g, "").split(".")[0] + "Z";
// Result: "20231031T120000Z"
```

**With Timezone**:

```typescript
if (event.dtstart_tzid) {
    // Output: DTSTART;TZID=Europe/Zurich:20231031T130000
    const localTime = formatInTimeZone(
        date,
        event.dtstart_tzid,
        "yyyyMMdd'T'HHmmss",
    );
    icalString += `DTSTART;TZID=${event.dtstart_tzid}:${localTime}`;
} else {
    // UTC time
    icalString += `DTSTART:${icalDate}`;
}
```

**iCalendar → Pubky**:

```typescript
// Parse iCal DATE-TIME
const icalDate = "20231031T120000Z"; // or with TZID parameter
const date = parseISO(icalDate.replace(/Z$/, ""));
const pubkyTimestamp = date.getTime() * 1000; // ms → µs

// Extract timezone if present
const dtstart_tzid = params.TZID || null; // "Europe/Zurich" or null
```

### Multi-value Fields

**Pubky → iCalendar**:

```typescript
// Pubky: Vec<String>
const categories = ["bitcoin", "meetup", "networking"];

// iCalendar: Comma-separated
const icalCategories = `CATEGORIES:${categories.join(",")}`;
// Result: "CATEGORIES:bitcoin,meetup,networking"
```

**iCalendar → Pubky**:

```typescript
const icalCategories = "CATEGORIES:bitcoin,meetup,networking";
const categories = icalCategories.split(":")[1].split(",");
// Result: ["bitcoin", "meetup", "networking"]
```

### Location (Three-Tier Approach)

**Pubky → iCalendar**:

```typescript
const event = {
    location: "Insider Bar, Zürich", // Simple text
    geo: { lat: 47.3769, lon: 8.5417 }, // Coordinates
    structured_locations: [
        {
            name: "Insider Bar",
            location_type: "ARRIVAL",
            address: "Münstergasse 20, 8001 Zürich",
            uri: "https://www.openstreetmap.org/node/123456789",
            description: "Main venue, second floor",
        },
        {
            name: "Parking Garage Urania",
            location_type: "PARKING",
            address: "Uraniastrasse 3, 8001 Zürich",
            uri: "geo:47.3765,8.5410",
            description: null,
        },
    ],
};

// iCalendar export (RFC 9073 VLOCATION components)
const ical = `
BEGIN:VEVENT
UID:${event.uid}
...
LOCATION:Insider Bar, Zürich
GEO:47.3769;8.5417
END:VEVENT

BEGIN:VLOCATION
UID:vloc-${event.uid}-0
NAME:Insider Bar
LOCATION-TYPE:ARRIVAL
GEO:geo:47.3769,8.5417
STRUCTURED-DATA;FMTTYPE=application/vnd.ical+json:{"address":"Münstergasse 20, 8001 Zürich"}
DESCRIPTION:Main venue, second floor
END:VLOCATION

BEGIN:VLOCATION
UID:vloc-${event.uid}-1
NAME:Parking Garage Urania
LOCATION-TYPE:PARKING
GEO:geo:47.3765,8.5410
STRUCTURED-DATA;FMTTYPE=application/vnd.ical+json:{"address":"Uraniastrasse 3, 8001 Zürich"}
END:VLOCATION`;
```

**iCalendar → Pubky**:

```typescript
function importVLocationComponents(icalData: string): PubkyAppEvent {
    const vlocations = parseComponents(icalData, "VLOCATION");

    return {
        location: extractProperty("LOCATION") || null, // Simple text
        geo: parseGEO(extractProperty("GEO")), // Coordinates from main VEVENT
        structured_locations: vlocations.map((vloc) => ({
            name: vloc.NAME, // REQUIRED
            location_type: vloc["LOCATION-TYPE"] || null,
            address: vloc["STRUCTURED-DATA"]?.address || null,
            uri: vloc.GEO || null,
            description: vloc.DESCRIPTION || null,
        })),
    };
}
```

**Key Points**:

- `location`: Primary text description (RFC 5545 LOCATION property)
- `geo`: Coordinates for quick mapping (RFC 5545 GEO property)
- `structured_locations`: Array of VLOCATION components (RFC 9073) with types
  like ARRIVAL, DEPARTURE, PARKING
- Each VLOCATION has its own UID: `vloc-{event_uid}-{index}`
- STRUCTURED-DATA holds address as JSON object
- Frontend can use OSM integration to populate all three tiers

---

## Example: Full Event Translation

### Pubky Event JSON

```json
{
    "uid": "20231031-satoshi-001@pubky",
    "dtstamp": 1698753600000000,
    "dtstart": 1698757200000000,
    "dtstart_tzid": "Europe/Zurich",
    "dtend": 1698764400000000,
    "dtend_tzid": "Europe/Zurich",
    "summary": "Bitcoin Meetup Zürich",
    "description": "Weekly Bitcoin meetup discussing Lightning Network",
    "geo": "47.366667;8.550000",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi"
    },
    "categories": ["bitcoin", "meetup"],
    "created": 1698753600000000,
    "sequence": 0,
    "last_modified": 1698753600000000,
    "rrule": "FREQ=WEEKLY;BYDAY=WE",
    "image_uri": "pubky://satoshi/pub/pubky.app/files/0033EVENT01",
    "conference": {
        "uri": "https://meet.jit.si/bitcoin-zurich",
        "label": "Jitsi Meeting",
        "features": ["AUDIO", "VIDEO"]
    },
    "structured_location": {
        "name": "Insider Bar",
        "address": "Münstergasse 20, 8001 Zürich",
        "uri": "https://www.openstreetmap.org/node/123456789",
        "description": "Main venue, second floor"
    },
    "x_pubky_calendar_uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG"
}
```

### Exported iCalendar (.ics)

```ical
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//Pubky//Calendar v2.0//EN
CALSCALE:GREGORIAN
METHOD:PUBLISH
X-WR-TIMEZONE:Europe/Zurich

BEGIN:VEVENT
UID:20231031-satoshi-001@pubky
DTSTAMP:20231031T120000Z
DTSTART;TZID=Europe/Zurich:20231031T130000
DTEND;TZID=Europe/Zurich:20231031T150000
SUMMARY:Bitcoin Meetup Zürich
DESCRIPTION:Weekly Bitcoin meetup discussing Lightning Network
LOCATION:Insider Bar
GEO:47.366667;8.550000
STATUS:CONFIRMED
ORGANIZER;CN="Satoshi Nakamoto":mailto:satoshi@pubky.example
CATEGORIES:bitcoin,meetup
CREATED:20231031T120000Z
SEQUENCE:0
LAST-MODIFIED:20231031T120000Z
RRULE:FREQ=WEEKLY;BYDAY=WE
IMAGE;VALUE=URI:https://pubky.example/files/0033EVENT01
CONFERENCE;VALUE=URI;FEATURE=AUDIO,VIDEO;LABEL="Jitsi Meeting":https://meet.jit.si/bitcoin-zurich
X-APPLE-STRUCTURED-LOCATION;VALUE=URI;X-TITLE="Insider Bar";X-ADDRESS="Münstergasse 20, 8001 Zürich":https://www.openstreetmap.org/node/123456789
X-PUBKY-CALENDAR-URI:pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG
END:VEVENT

BEGIN:VTIMEZONE
TZID:Europe/Zurich
BEGIN:DAYLIGHT
TZOFFSETFROM:+0100
TZOFFSETTO:+0200
DTSTART:19700329T020000
RRULE:FREQ=YEARLY;BYMONTH=3;BYDAY=-1SU
END:DAYLIGHT
BEGIN:STANDARD
TZOFFSETFROM:+0200
TZOFFSETTO:+0100
DTSTART:19701025T030000
RRULE:FREQ=YEARLY;BYMONTH=10;BYDAY=-1SU
END:STANDARD
END:VTIMEZONE

END:VCALENDAR
```

**Notes**:

- `ORGANIZER;CN="Satoshi Nakamoto"` - Name fetched from
  `pubky://satoshi/pub/pubky.app/profile.json` at export time
- `DTSTART;TZID=Europe/Zurich` - Uses `dtstart_tzid` field
- `LOCATION:Insider Bar` - Mapped from `structured_location.name`
- `X-APPLE-STRUCTURED-LOCATION` - Preserves rich location data for Apple clients
- Attendees would be added from separate attendee records (not shown)

---

## Import Process

### Step 1: Parse iCalendar File

```typescript
import ical from "ical.js";

function parseICSFile(icsContent: string) {
    const jcalData = ical.parse(icsContent);
    const comp = new ical.Component(jcalData);
    const vevents = comp.getAllSubcomponents("vevent");

    return vevents.map((vevent) => parseVEvent(vevent));
}
```

### Step 2: Map to Pubky Structure

```typescript
function parseVEvent(vevent: ical.Component): PubkyAppEvent {
    const event = vevent.getFirstPropertyValue;

    // Extract basic properties
    const uid = event("uid");
    const dtstart = event("dtstart");
    const dtstartTzid = vevent.getFirstProperty("dtstart")?.getParameter(
        "tzid",
    );

    // Extract organizer (store URI only, ignore CN)
    const organizerProp = vevent.getFirstProperty("organizer");
    const organizerUri = extractPubkyURI(organizerProp.getFirstValue());

    // Extract location data
    const location = event("location");
    const geo = event("geo");
    const xLocation = vevent.getFirstProperty("x-apple-structured-location");

    return {
        uid,
        dtstamp: dateToMicroseconds(event("dtstamp")),
        dtstart: dateToMicroseconds(dtstart),
        dtstart_tzid: dtstartTzid || null,
        summary: event("summary"),
        description: event("description") || null,
        geo: geo ? `${geo.lat};${geo.lon}` : null,
        organizer: organizerUri ? { uri: organizerUri } : null,
        structured_location: location
            ? {
                name: location, // REQUIRED
                address: xLocation?.getParameter("x-address") || null,
                uri: xLocation?.getFirstValue() || null,
                description: null,
            }
            : null,
        // ... map remaining fields
    };
}

function extractPubkyURI(mailto: string): string | null {
    // Convert "mailto:satoshi@pubky.example" → "pubky://satoshi"
    if (mailto?.startsWith("mailto:")) {
        const username = mailto.split("@")[0].replace("mailto:", "");
        return `pubky://${username}`;
    }
    return null;
}
```

### Step 3: Create Attendee Records

```typescript
// Attendees are NOT embedded in events
// Create separate attendee records for each ATTENDEE property
function createAttendeeRecords(vevent: ical.Component, eventUri: string) {
    const attendees = vevent.getAllProperties("attendee");

    return attendees.map((attendeeProp) => {
        const uri = extractPubkyURI(attendeeProp.getFirstValue());
        const partstat = attendeeProp.getParameter("partstat") ||
            "NEEDS-ACTION";

        return {
            // Store at: pubky://<current-user>/pub/pubky.app/attendee/<id>
            attendee_uri: uri,
            partstat,
            role: attendeeProp.getParameter("role") || null,
            rsvp: attendeeProp.getParameter("rsvp") === "TRUE" || null,
            recurrence_id: null, // Or match specific instance
            x_pubky_event_uri: eventUri,
        };
    });
}
```

---

## Export Process

### Step 1: Fetch Event Data

```typescript
async function exportEvent(eventUri: string): Promise<string> {
    // 1. Fetch event
    const event = await fetch(`${eventUri}`).then((r) => r.json());

    // 2. Fetch organizer name from profile.json
    const organizerName = event.organizer?.uri
        ? await getDisplayName(event.organizer.uri)
        : null;

    // 3. Fetch attendees from Nexus (aggregates all users' attendee records)
    const attendees = await fetch(
        `/v0/event/${eventAuthorId}/${eventId}/attendees`,
    )
        .then((r) => r.json());

    // 4. Fetch names for all attendees
    const attendeesWithNames = await Promise.all(
        attendees.map(async (att) => ({
            ...att,
            name: await getDisplayName(att.attendee_uri),
        })),
    );

    return buildICS(event, organizerName, attendeesWithNames);
}

async function getDisplayName(pubkyUri: string): Promise<string> {
    try {
        const profile = await fetch(`${pubkyUri}/pub/pubky.app/profile.json`);
        return profile.name || pubkyUri.split("//")[1];
    } catch {
        return pubkyUri.split("//")[1]; // Fallback to ID
    }
}
```

### Step 2: Build iCalendar String

```typescript
function buildICS(
    event: PubkyAppEvent,
    organizerName: string | null,
    attendees: Array<{ attendee_uri: string; name: string; partstat: string }>,
): string {
    let ics = "BEGIN:VCALENDAR\n";
    ics += "VERSION:2.0\n";
    ics += "PRODID:-//Pubky//Calendar v2.0//EN\n";
    ics += "BEGIN:VEVENT\n";

    // Basic fields
    ics += `UID:${event.uid}\n`;
    ics += `DTSTAMP:${formatDateTimeUTC(event.dtstamp)}\n`;

    // Start time with timezone
    if (event.dtstart_tzid) {
        ics += `DTSTART;TZID=${event.dtstart_tzid}:${
            formatDateTimeLocal(event.dtstart, event.dtstart_tzid)
        }\n`;
    } else {
        ics += `DTSTART:${formatDateTimeUTC(event.dtstart)}\n`;
    }

    // End time
    if (event.dtend) {
        if (event.dtend_tzid) {
            ics += `DTEND;TZID=${event.dtend_tzid}:${
                formatDateTimeLocal(event.dtend, event.dtend_tzid)
            }\n`;
        } else {
            ics += `DTEND:${formatDateTimeUTC(event.dtend)}\n`;
        }
    }

    ics += `SUMMARY:${escapeText(event.summary)}\n`;

    if (event.description) {
        ics += `DESCRIPTION:${escapeText(event.description)}\n`;
    }

    // Location
    if (event.structured_location) {
        ics += `LOCATION:${escapeText(event.structured_location.name)}\n`;
    }

    if (event.geo) {
        const [lat, lon] = event.geo.split(";");
        ics += `GEO:${lat};${lon}\n`;
    }

    // Organizer with fetched name
    if (event.organizer && organizerName) {
        const mailto = event.organizer.uri.replace("pubky://", "") +
            "@pubky.example";
        ics += `ORGANIZER;CN="${escapeText(organizerName)}":mailto:${mailto}\n`;
    }

    // Attendees with fetched names
    for (const att of attendees) {
        const mailto = att.attendee_uri.replace("pubky://", "") +
            "@pubky.example";
        ics += `ATTENDEE;CN="${
            escapeText(att.name)
        }";PARTSTAT=${att.partstat}:mailto:${mailto}\n`;
    }

    // ... add remaining fields

    ics += "END:VEVENT\n";
    ics += "END:VCALENDAR\n";

    return ics;
}
```

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
4. **Profile Caching**: Cache profile.json lookups with TTL
5. **Sync**: Use `sequence` and `last_modified` for efficient sync

---

## Best Practices

### For Implementers

1. **Always fetch names dynamically** - Never store `ORGANIZER` CN or `ATTENDEE`
   CN in Pubky data
2. **Cache profile lookups** - Cache `profile.json` responses with reasonable
   TTL (e.g., 5 minutes)
3. **Handle missing profiles gracefully** - Fallback to Pubky ID if profile
   fetch fails
4. **Preserve unknown X- properties** - Store custom iCal properties you don't
   understand for round-trip compatibility
5. **Validate timezones** - Ensure `dtstart_tzid` is a valid IANA timezone ID
6. **Use sequence for sync** - Increment `sequence` on every event modification

### For CalDAV Bridge Developers

1. **Implement ETags properly** - Use `sequence` + `last_modified` to generate
   ETags
2. **Support WebDAV locking** - Prevent concurrent modifications
3. **Handle recurrence properly** - Map Pubky's instance-based RSVP to iCal's
   RECURRENCE-ID
4. **Optimize profile fetches** - Batch fetch profiles for all attendees
5. **Support calendar subscriptions** - Read-only access via webcal:// URLs

---

## Migration Path

### Migrating from Traditional Calendars

```typescript
async function migrateICSToPubly(icsFile: File) {
    // 1. Parse ICS
    const events = parseICSFile(await icsFile.text());

    // 2. Create calendar in Pubky
    const calendar = await createPubkyCalendar({
        name: "Imported from " + icsFile.name,
        timezone: events[0]?.dtstart_tzid || "UTC",
    });

    // 3. Import events
    for (const event of events) {
        // Try to match organizer/attendees to Pubky users
        const organizer = await findPubkyUser(event.organizer?.uri);

        await createPubkyEvent({
            ...event,
            organizer: organizer ? { uri: organizer } : null,
            x_pubky_calendar_uri: calendar.uri,
        });

        // Create attendee records (only if current user is one)
        const myAttendance = event.attendees?.find((att) =>
            att.attendee_uri === currentUser.uri
        );

        if (myAttendance) {
            await createAttendeeRecord(myAttendance);
        }
    }
}
```

### Exporting to Traditional Calendars

```typescript
async function exportPubkyCalendarToICS(calendarUri: string): Promise<Blob> {
    // 1. Fetch calendar and all events
    const calendar = await fetch(calendarUri).then((r) => r.json());
    const events = await fetch(`/v0/calendar/${authorId}/${calendarId}/events`)
        .then((r) => r.json());

    // 2. Fetch all profiles in parallel
    const organizerUris = [
        ...new Set(events.map((e) => e.organizer?.uri).filter(Boolean)),
    ];
    const profileCache = await fetchProfiles(organizerUris);

    // 3. Build ICS
    let ics = buildCalendarHeader(calendar);

    for (const event of events) {
        const organizerName = profileCache[event.organizer?.uri] || null;
        const attendees = await fetchEventAttendees(event.uri);
        const attendeesWithNames = await enrichAttendeesWithNames(attendees);

        ics += buildVEvent(event, organizerName, attendeesWithNames);
    }

    ics += "END:VCALENDAR\n";

    return new Blob([ics], { type: "text/calendar; charset=utf-8" });
}
```

---

## Summary

This interoperability guide ensures Pubky calendars can:

1. ✅ **Import** existing .ics files from any RFC 5545 compliant source
2. ✅ **Export** to standard .ics format for use in traditional clients
3. ✅ **Sync** via CalDAV bridge without code changes to existing clients
4. ✅ **Preserve** RFC compliance while leveraging Pubky's profile system
5. ✅ **Prevent** data duplication by fetching names from profile.json

**The core innovation**: Separate identity (profile.json) from events, making
calendar data lightweight and always up-to-date.
