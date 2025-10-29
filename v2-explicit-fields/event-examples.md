# Event Examples (v2.0 - Explicit Fields)

## Overview

This document provides example data for the explicit fields approach to Pubky
iCalendar integration. All examples show the structured data format with
explicit typed fields.

## Calendar Examples

### Bitcoin Switzerland Events Calendar

```json
{
    "id": "0033RCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
    "author": "pubky://satoshi",
    "name": "Bitcoin Switzerland Events",
    "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01",
    "color": "#F7931A",
    "timezone": "Europe/Zurich",
    "x_pubky_admins": [
        "pubky://adam-back",
        "pubky://decentralschweiz"
    ],
    "created": 1698753600000000
}
```

### Lightning Network Meetups Calendar

```json
{
    "id": "0033RDZXVEPNG",
    "uri": "pubky://hal/pub/pubky.app/calendar/0033RDZXVEPNG",
    "author": "pubky://hal",
    "name": "Lightning Network Meetups",
    "image_uri": "pubky://hal/pub/pubky.app/files/0033CALIMG02",
    "color": "#FFD700",
    "timezone": "Europe/Zurich",
    "x_pubky_admins": [
        "pubky://lightning-dev"
    ],
    "created": 1698753600000000
}
```

## Event Examples

### Weekly Bitcoin Meetup

```json
{
    "id": "0033SCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "author": "pubky://satoshi",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Switzerland Events",
        "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698753600000000,
    "dtend": 1698764400000000,
    "summary": "Bitcoin Meetup Zürich",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi"
    },
    "categories": ["bitcoin", "meetup", "networking"],
    "created": 1698753600000000,
    "rrule": "FREQ=WEEKLY;BYDAY=WE",
    "rdate": null,
    "exdate": null,
    "recurrence_id": null,
    "image_uri": "pubky://satoshi/pub/pubky.app/files/0033EVENT01",
    "conference": {
        "uri": "https://meet.jit.si/bitcoin-zurich",
        "label": "Jitsi Meeting"
    },
    "location": "Insider Bar, Zürich",
    "geo": {
        "lat": 47.366667,
        "lon": 8.550000
    },
    "structured_locations": [
        {
            "name": "Insider Bar",
            "location_type": "ARRIVAL",
            "address": "Münstergasse 20, 8001 Zürich",
            "uri": "https://www.openstreetmap.org/node/123456789",
            "description": "Main venue in Zürich city center"
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>Weekly Bitcoin meetup discussing <strong>Lightning Network</strong> developments and Bitcoin adoption in Switzerland.</p><p>Topics include:</p><ul><li>Lightning Network updates</li><li>Bitcoin adoption stories</li><li>Technical discussions</li></ul>"
    },
    "x_pubky_calendar_uris": [
        "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG"
    ]
}
```

### Lightning Network Workshop

```json
{
    "id": "0033SDZXVEPNG",
    "uri": "pubky://hal/pub/pubky.app/event/0033SDZXVEPNG",
    "author": "pubky://hal",
    "calendar": {
        "id": "0033RDZXVEPNG",
        "uri": "pubky://hal/pub/pubky.app/calendar/0033RDZXVEPNG",
        "name": "Lightning Network Meetups",
        "image_uri": "pubky://hal/pub/pubky.app/files/0033CALIMG02"
    },
    "uid": "pubky://hal/pub/pubky.app/event/0033SDZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698840000000000,
    "dtend": 1698854400000000,
    "summary": "Lightning Network Workshop",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://hal"
    },
    "categories": ["lightning", "workshop", "education"],
    "created": 1698753600000000,
    "rrule": null,
    "rdate": null,
    "exdate": null,
    "recurrence_id": null,
    "image_uri": "pubky://hal/pub/pubky.app/files/0033EVENT02",
    "conference": {
        "uri": "https://meet.jit.si/lightning-workshop",
        "label": "Lightning Workshop Room"
    },
    "location": "Tech Hub Zürich",
    "geo": {
        "lat": 47.370000,
        "lon": 8.545000
    },
    "structured_locations": [
        {
            "name": "Tech Hub Zürich",
            "location_type": "ARRIVAL",
            "address": "Technoparkstrasse 1, 8005 Zürich",
            "uri": "https://www.openstreetmap.org/node/987654321",
            "description": "Workshop room with presentation equipment"
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>Hands-on Lightning Network workshop covering:</p><ul><li>Setting up a Lightning node</li><li>Creating payment channels</li><li>Routing payments</li><li>Best practices for security</li></ul><p><strong>Prerequisites:</strong> Basic understanding of Bitcoin and command line usage.</p>"
    },
    "x_pubky_calendar_uris": [
        "pubky://hal/pub/pubky.app/calendar/0033RDZXVEPNG"
    ]
}
```

### Bitcoin Conference 2024

```json
{
    "id": "0033SEZXVEPNG",
    "uri": "pubky://adam-back/pub/pubky.app/event/0033SEZXVEPNG",
    "author": "pubky://adam-back",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Switzerland Events",
        "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01"
    },
    "uid": "pubky://adam-back/pub/pubky.app/event/0033SEZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1704067200000000,
    "dtend": 1704153600000000,
    "summary": "Bitcoin Conference 2024",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://adam-back"
    },
    "categories": ["bitcoin", "conference", "speakers"],
    "created": 1698753600000000,
    "rrule": null,
    "rdate": null,
    "exdate": null,
    "recurrence_id": null,
    "image_uri": "pubky://adam-back/pub/pubky.app/files/0033EVENT03",
    "conference": {
        "uri": "https://bitcoin-conference-2024.ch",
        "label": "Conference Website"
    },
    "location": "Kongresshaus Zürich",
    "geo": {
        "lat": 47.366667,
        "lon": 8.543333
    },
    "structured_locations": [
        {
            "name": "Kongresshaus Zürich",
            "location_type": "ARRIVAL",
            "address": "Claridenstrasse 5, 8002 Zürich",
            "uri": "https://www.openstreetmap.org/node/456789123",
            "description": "Main conference venue"
        },
        {
            "name": "Parkhaus Opéra",
            "location_type": "PARKING",
            "address": "Claridenstrasse 22, 8002 Zürich",
            "uri": "geo:47.366000,8.543000",
            "description": null
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>The premier Bitcoin conference in Switzerland featuring:</p><ul><li>Keynote speakers from the Bitcoin community</li><li>Technical presentations on Bitcoin development</li><li>Panel discussions on Bitcoin adoption</li><li>Networking opportunities</li></ul><p><strong>Speakers include:</strong> Satoshi Nakamoto, Hal Finney, Adam Back, and more.</p>"
    },
    "x_pubky_calendar_uris": [
        "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG"
    ]
}
```

## Attendee Examples

### Alice's RSVP for All Instances of Bitcoin Meetup

```json
{
    "id": "0033UCZXVEPNG",
    "uri": "pubky://alice/pub/pubky.app/attendee/0033UCZXVEPNG",
    "author": "pubky://alice",
    "event": {
        "id": "0033SCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
        "summary": "Bitcoin Meetup Zürich"
    },
    "attendee_uri": "pubky://alice",
    "partstat": "ACCEPTED",
    "role": "REQ-PARTICIPANT",
    "recurrence_id": null,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

### Bob's RSVP for Specific Instance of Bitcoin Meetup

```json
{
    "id": "0033UDZXVEPNG",
    "uri": "pubky://bob/pub/pubky.app/attendee/0033UDZXVEPNG",
    "author": "pubky://bob",
    "event": {
        "id": "0033SCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
        "summary": "Bitcoin Meetup Zürich"
    },
    "attendee_uri": "pubky://bob",
    "partstat": "ACCEPTED",
    "role": "REQ-PARTICIPANT",
    "recurrence_id": 1699358400000000,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

### Charlie's Different RSVPs for Different Instances

**Accepted for October 9th:**

```json
{
    "id": "0033UEZXVEPNG",
    "uri": "pubky://charlie/pub/pubky.app/attendee/0033UEZXVEPNG",
    "author": "pubky://charlie",
    "event": {
        "id": "0033SCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
        "summary": "Bitcoin Meetup Zürich"
    },
    "attendee_uri": "pubky://charlie",
    "partstat": "ACCEPTED",
    "role": "REQ-PARTICIPANT",
    "recurrence_id": 1698753600000000,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

**Declined for October 16th:**

```json
{
    "id": "0033UFZXVEPNG",
    "uri": "pubky://charlie/pub/pubky.app/attendee/0033UFZXVEPNG",
    "author": "pubky://charlie",
    "event": {
        "id": "0033SCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
        "summary": "Bitcoin Meetup Zürich"
    },
    "attendee_uri": "pubky://charlie",
    "partstat": "DECLINED",
    "role": "REQ-PARTICIPANT",
    "recurrence_id": 1699358400000000,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

## Alarm Examples

### 15-minute reminder for Bitcoin Meetup

```json
{
    "id": "0033WCZXVEPNG",
    "uri": "pubky://bob/pub/pubky.app/alarm/0033WCZXVEPNG",
    "author": "pubky://bob",
    "event": {
        "id": "0033SCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
        "summary": "Bitcoin Meetup Zürich"
    },
    "action": "DISPLAY",
    "trigger": "-PT15M",
    "description": "Bitcoin Meetup in 15 minutes",
    "uid": "pubky://bob/pub/pubky.app/alarm/0033WCZXVEPNG",
    "x_pubky_target_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

### 1-day reminder for Lightning Workshop

```json
{
    "id": "0033WDZXVEPNG",
    "uri": "pubky://alice/pub/pubky.app/alarm/0033WDZXVEPNG",
    "author": "pubky://alice",
    "event": {
        "id": "0033SDZXVEPNG",
        "uri": "pubky://hal/pub/pubky.app/event/0033SDZXVEPNG",
        "summary": "Lightning Network Workshop"
    },
    "action": "DISPLAY",
    "trigger": "-P1D",
    "description": "Lightning Network Workshop tomorrow",
    "uid": "pubky://alice/pub/pubky.app/alarm/0033WDZXVEPNG",
    "x_pubky_target_uri": "pubky://hal/pub/pubky.app/event/0033SDZXVEPNG"
}
```

### Email reminder for Bitcoin Conference

```json
{
    "id": "0033WEZXVEPNG",
    "uri": "pubky://charlie/pub/pubky.app/alarm/0033WEZXVEPNG",
    "author": "pubky://charlie",
    "event": {
        "id": "0033SEZXVEPNG",
        "uri": "pubky://adam-back/pub/pubky.app/event/0033SEZXVEPNG",
        "summary": "Bitcoin Conference 2024"
    },
    "action": "EMAIL",
    "trigger": "-P7D",
    "description": "Bitcoin Conference 2024 in one week",
    "uid": "pubky://charlie/pub/pubky.app/alarm/0033WEZXVEPNG",
    "x_pubky_target_uri": "pubky://adam-back/pub/pubky.app/event/0033SEZXVEPNG"
}
```

## Recurring Event Examples

### Weekly Bitcoin Meetup with Exclusions and Extra Dates

**Master Event with EXDATE and RDATE:**

```json
{
    "id": "0033SCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "author": "pubky://satoshi",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Switzerland Events",
        "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698753600000000,
    "dtend": 1698764400000000,
    "summary": "Bitcoin Meetup Zürich",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi"
    },
    "categories": ["bitcoin", "meetup"],
    "created": 1698753600000000,
    "rrule": "FREQ=WEEKLY;BYDAY=WE",
    "rdate": [
        "2025-10-15T19:00:00+02:00"
    ],
    "exdate": [
        "2025-10-16T19:00:00+02:00",
        "2025-10-23T19:00:00+02:00"
    ],
    "recurrence_id": null,
    "image_uri": "pubky://satoshi/pub/pubky.app/files/0033EVENT01",
    "conference": null,
    "location": "Insider Bar",
    "geo": {
        "lat": 47.366667,
        "lon": 8.550000
    },
    "structured_locations": [
        {
            "name": "Insider Bar",
            "location_type": "ARRIVAL",
            "address": "Münstergasse 20, 8001 Zürich",
            "uri": "geo:47.366667,8.550000",
            "description": "Main venue"
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>Weekly Bitcoin meetup discussing <strong>Lightning Network</strong></p>"
    },
    "x_pubky_calendar_uris": [
        "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG"
    ]
}
```

### Weekly Bitcoin Meetup with Override

**Master Event:**

```json
{
    "id": "0033SCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "author": "pubky://satoshi",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Switzerland Events",
        "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698753600000000,
    "dtend": 1698764400000000,
    "summary": "Bitcoin Meetup Zürich",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi"
    },
    "categories": ["bitcoin", "meetup"],
    "created": 1698753600000000,
    "rrule": "FREQ=WEEKLY;BYDAY=WE",
    "rdate": null,
    "exdate": null,
    "recurrence_id": null,
    "image_uri": "pubky://satoshi/pub/pubky.app/files/0033EVENT01",
    "conference": null,
    "location": "Insider Bar",
    "geo": {
        "lat": 47.366667,
        "lon": 8.550000
    },
    "structured_locations": [
        {
            "name": "Insider Bar",
            "location_type": "ARRIVAL",
            "address": "Münstergasse 20, 8001 Zürich",
            "uri": "geo:47.366667,8.550000",
            "description": "Main venue"
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>Weekly Bitcoin meetup discussing <strong>Lightning Network</strong></p>"
    },
    "x_pubky_calendar_uris": [
        "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG"
    ]
}
```

**Override for Special Event:**

```json
{
    "id": "0033TCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/event/0033TCZXVEPNG",
    "author": "pubky://satoshi",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Switzerland Events",
        "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1699358400000000,
    "dtend": 1699372800000000,
    "summary": "Bitcoin Meetup Zürich - Special Lightning Workshop",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi"
    },
    "categories": ["bitcoin", "meetup", "workshop"],
    "created": 1698753600000000,
    "rrule": null,
    "rdate": null,
    "exdate": null,
    "recurrence_id": 1699358400000000,
    "image_uri": "pubky://satoshi/pub/pubky.app/files/0033EVENT01",
    "conference": {
        "uri": "https://meet.jit.si/bitcoin-zurich-workshop",
        "label": "Lightning Workshop Room"
    },
    "location": "Tech Hub Zürich",
    "geo": {
        "lat": 47.370000,
        "lon": 8.545000
    },
    "structured_locations": [
        {
            "name": "Tech Hub Zürich",
            "location_type": "ARRIVAL",
            "address": "Technoparkstrasse 1, 8005 Zürich",
            "uri": "geo:47.370000,8.545000",
            "description": "Workshop venue for special event"
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>Special Lightning Network workshop session. This week we'll have hands-on Lightning node setup and payment channel creation.</p>"
    },
    "x_pubky_calendar_uris": [
        "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG"
    ]
}
```

### Complex Recurrence Pattern Example

**Monthly Bitcoin Conference with Multiple Exceptions:**

```json
{
    "id": "0033MCZXVEPNG",
    "uri": "pubky://adam-back/pub/pubky.app/event/0033MCZXVEPNG",
    "author": "pubky://adam-back",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://adam-back/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Conference Series",
        "image_uri": "pubky://adam-back/pub/pubky.app/files/0033CALIMG03"
    },
    "uid": "pubky://adam-back/pub/pubky.app/event/0033MCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698753600000000,
    "dtend": 1698764400000000,
    "summary": "Bitcoin Conference Monthly",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://adam-back"
    },
    "categories": ["bitcoin", "conference"],
    "created": 1698753600000000,
    "rrule": "FREQ=MONTHLY;BYDAY=1SA",
    "rdate": [
        "2025-10-15T09:00:00+02:00",
        "2025-12-20T09:00:00+01:00"
    ],
    "exdate": [
        "2025-11-01T09:00:00+01:00",
        "2025-12-06T09:00:00+01:00"
    ],
    "recurrence_id": null,
    "image_uri": "pubky://adam-back/pub/pubky.app/files/0033EVENT03",
    "conference": {
        "uri": "https://bitcoin-conference-2024.ch",
        "label": "Conference Website"
    },
    "location": "Convention Center Zürich",
    "geo": {
        "lat": 47.410000,
        "lon": 8.540000
    },
    "structured_locations": [
        {
            "name": "Convention Center Zürich",
            "location_type": "ARRIVAL",
            "address": "Walchestrasse 9, 8006 Zürich",
            "uri": "geo:47.410000,8.540000",
            "description": "Main conference venue"
        }
    ],
    "styled_description": {
        "fmttype": "text/html",
        "value": "<p>Monthly Bitcoin conference with <strong>industry leaders</strong> and <em>technical workshops</em></p>"
    },
    "x_pubky_calendar_uris": [
        "pubky://adam-back/pub/pubky.app/calendar/0033RCZXVEPNG"
    ]
}
```

## Data Format Notes

### Timestamps

All timestamps are Unix microseconds (i64) for consistency with Pubky's
`indexed_at` format.

### JSON Objects

Structured data like `organizer`, `conference`, and `structured_locations` are
stored as JSON objects with standardized keys.

### Multi-value Fields

Fields like `categories`, `x_pubky_admins`, `x_pubky_calendar_uris`, and
`structured_locations` are stored as arrays.

### URIs

All resource references use Pubky URIs in the format
`pubky://<user_id>/pub/pubky.app/<type>/<id>`.

### Styled Descriptions

Rich text content is stored as JSON objects with `fmttype` (MIME type) and
`value` (content) fields, supporting HTML, Markdown, and other formats.

## Recurring Event RSVP Patterns

### Pattern 1: RSVP to All Instances

User commits to attending all occurrences of the recurring event by omitting the
`recurrence_id` field:

```json
{
    "attendee_uri": "pubky://alice",
    "partstat": "ACCEPTED",
    "role": "REQ-PARTICIPANT",
    "recurrence_id": null,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

### Pattern 2: RSVP to Specific Instance

User RSVPs to a single occurrence by including the `recurrence_id` matching that
instance's timestamp:

```json
{
    "attendee_uri": "pubky://bob",
    "partstat": "ACCEPTED",
    "role": "REQ-PARTICIPANT",
    "recurrence_id": 1699358400000000,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

### Pattern 3: Different Status Per Instance

User creates separate attendee records with different `recurrence_id` values and
different `partstat`:

**October 9th - Accepted:**

```json
{
    "id": "0033UEZXVEPNG",
    "attendee_uri": "pubky://charlie",
    "partstat": "ACCEPTED",
    "recurrence_id": 1698753600000000,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```

**October 16th - Declined:**

```json
{
    "id": "0033UFZXVEPNG",
    "attendee_uri": "pubky://charlie",
    "partstat": "DECLINED",
    "recurrence_id": 1699358400000000,
    "x_pubky_event_uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG"
}
```
