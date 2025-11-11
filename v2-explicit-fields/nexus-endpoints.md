# Nexus API Endpoints (v2.0 - Explicit Fields)

## Overview

This document defines the Nexus API endpoints for the explicit fields approach
to Pubky iCalendar integration. All endpoints return structured data with
explicit typed fields instead of jCal JSON.

## Basic Endpoint Overview

| Method | Endpoint                               | Description                                                                                 |
| ------ | -------------------------------------- | ------------------------------------------------------------------------------------------- |
| Get    | /v0/stream/calendars                   | Retrieve calendars with filtering (pagination, author, admins)                              |
| Get    | /v0/stream/events                      | Retrieve events with filtering (date range, calendar, status)                               |
| Get    | /v0/calendar/{author_id}/{calendar_id} | Retrieve a specific calendar with its events                                                |
| Get    | /v0/event/{author_id}/{event_id}       | Retrieve a specific event with attendees, alarms, and all related events (recurring series) |

**Note**: Individual entity endpoints (`/v0/calendar/{id}`, `/v0/event/{id}`)
are required for efficient single-page views without streaming full result sets.
The event endpoint automatically includes all events in the same recurring
series (master + overrides) to enable proper calendar navigation.

## Calendar Endpoints

### GET /v0/stream/calendars

Retrieve a list of calendars with optional filtering.

**Query Parameters:**

- `limit` (optional): Number of results to return (default: 50, max: 100)
- `skip` (optional): Number of results to skip (default: 0)
- `tags` (optional): Comma-separated list of tags to filter by
- `admin` (optional): Filter calendars where user is admin

**Response:**

```json
{
  "calendars": [
    {
      "id": "0033RCZXVEPNG",
      "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
      "author": "pubky://satoshi",
      "name": "Bitcoin Switzerland Events",
      "image_uri": "pubky://satoshi/pub/pubky.app/files/0033CALIMG01",
      "color": "#F7931A",
      "timezone": "Europe/Zurich",
      "x_pubky_admins": [
        "pubky://satoshi",
        "pubky://adam-back"
      ],
      "created": 1698753600000000
    }
  ],
  "total": 1,
  "limit": 50,
  "skip": 0
}
```

### GET /v0/calendar/{author_id}/{calendar_id}

Retrieve a specific calendar by ID with its events.

**Parameters:**

- `author_id`: Author Pubky ID (path)
- `calendar_id`: Calendar Crockford32 ID (path)

**Response:**

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
    "pubky://satoshi",
    "pubky://adam-back"
  ],
  "created": 1698753600000000,
  "events": [
    {
      "id": "0033SCZXVEPNG",
      "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
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
      "conference": {
        "uri": "https://meet.jit.si/bitcoin-zurich",
        "label": "Jitsi Meeting"
      },
      "location": "Insider Bar",
      "geo": "47.366667;8.550000",
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
      }
    }
  ]
}
```

## Event Endpoints

### GET /v0/stream/events

Retrieve a list of events with optional filtering.

**Query Parameters:**

- `limit` (optional): Number of results to return (default: 50, max: 100)
- `skip` (optional): Number of results to skip (default: 0)
- `tags` (optional): Comma-separated list of tags to filter by
- `calendar` (optional): Filter events by calendar ID
- `start_date` (optional): Filter events starting after this date (Unix
  microseconds)
- `end_date` (optional): Filter events starting before this date (Unix
  microseconds)
- `status` (optional): Filter by event status (CONFIRMED, TENTATIVE, CANCELLED)
- `location` (optional): Filter by location (text search)

**Behavior:**

- Returns **all events** (both master recurring events and override events)
  sorted by `dtstart`
- Master events have `recurrence_id: null` and may contain `rrule`
- Override events have non-null `recurrence_id` and `rrule: null`
- Events with the same `uid` represent a recurring series (master + overrides)
- Clients must handle recurrence expansion and override merging

**Response:**

```json
{
  "events": [
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
      "conference": {
        "uri": "https://meet.jit.si/bitcoin-zurich",
        "label": "Jitsi Meeting"
      },
      "location": "Insider Bar",
      "geo": "47.366667;8.550000",
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
      }
    }
  ],
  "total": 1,
  "limit": 50,
  "skip": 0
}
```

**Client Responsibilities:**

The stream endpoint returns raw event data. Clients are responsible for:

1. **Grouping by UID**: Group events with the same `uid` to identify recurring
   series
2. **Master/Override Detection**: Check `recurrence_id` (null = master, non-null
   = override)
3. **Recurrence Expansion**: Compute occurrences from master event's `rrule`
4. **Override Application**: Replace computed occurrences with override events
   where `recurrence_id` matches
5. **Exception Handling**: Filter out dates listed in master's `exdate`
6. **Additional Dates**: Include dates from master's `rdate`

**Example Response with Recurring Series:**

```json
{
  "events": [
    {
      "id": "evt-001",
      "uid": "pubky://alice/pub/pubky.app/event/evt-001",
      "recurrence_id": null,
      "rrule": "FREQ=WEEKLY;BYDAY=WE",
      "dtstart": 1698753600000000,
      "summary": "Weekly Bitcoin Meetup"
    },
    {
      "id": "evt-002",
      "uid": "pubky://bob/pub/pubky.app/event/evt-002",
      "recurrence_id": null,
      "rrule": null,
      "dtstart": 1698840000000000,
      "summary": "Lightning Workshop"
    },
    {
      "id": "evt-123",
      "uid": "pubky://alice/pub/pubky.app/event/evt-001",
      "recurrence_id": 1699358400000000,
      "rrule": null,
      "dtstart": 1699365600000000,
      "summary": "Weekly Bitcoin Meetup - Special Guest"
    }
  ],
  "total": 3
}
```

In this example:

- `evt-001` is a master recurring event (weekly on Wednesdays)
- `evt-002` is a standalone event (no recurrence)
- `evt-123` is an override for a specific occurrence of `evt-001` (same `uid`,
  different details)

### GET /v0/event/{author_id}/{event_id}

Retrieve a single specific event by its storage path (event_id). This endpoint
returns the requested event along with **all related events in the same
recurring series** (master + overrides) to enable proper calendar navigation and
occurrence display.

**Parameters:**

- `author_id`: Author Pubky ID (path)
- `event_id`: Event Crockford32 ID (path)
- `recurrence_id` (optional): Filter attendees by specific recurring event
  instance (Unix microseconds)

**Behavior:**

- Returns the **requested event** as the primary response object
- If the event is part of a recurring series (has `rrule` or non-null
  `recurrence_id`):
  - Automatically includes **all events with the same `uid`** in a
    `related_events` array
  - This includes the master event and all override events
  - Enables frontend to display previous/next occurrences and navigate the
    series
- For standalone events (no recurrence), `related_events` is empty

**Use Cases:**

- Single event page/detail view with series navigation
- Displaying "Next occurrence" / "Previous occurrence" buttons
- Showing all exceptions/overrides in a recurring series
- Calendar views that need complete series context

**Response Structure:**

````json
**Response Structure:**

```json
{
  "event": {
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
    "conference": {
      "uri": "https://meet.jit.si/bitcoin-zurich",
      "label": "Jitsi Meeting"
    },
    "location": "Insider Bar",
    "geo": "47.366667;8.550000",
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
    "attendees": [
      {
        "id": "0033UCZXVEPNG",
        "uri": "pubky://alice/pub/pubky.app/attendee/0033UCZXVEPNG",
        "author": "pubky://alice",
        "attendee_uri": "pubky://alice",
        "partstat": "ACCEPTED",
        "role": "REQ-PARTICIPANT",
        "recurrence_id": null
      },
      {
        "id": "0033UDZXVEPNG",
        "uri": "pubky://bob/pub/pubky.app/attendee/0033UDZXVEPNG",
        "author": "pubky://bob",
        "attendee_uri": "pubky://bob",
        "partstat": "ACCEPTED",
        "role": "REQ-PARTICIPANT",
        "recurrence_id": 1699358400000000
      }
    ],
    "alarms": [
      {
        "id": "0033WCZXVEPNG",
        "uri": "pubky://bob/pub/pubky.app/alarm/0033WCZXVEPNG",
        "author": "pubky://bob",
        "action": "DISPLAY",
        "trigger": "-PT15M",
        "description": "Bitcoin Meetup in 15 minutes",
        "uid": "pubky://bob/pub/pubky.app/alarm/0033WCZXVEPNG"
      }
    ]
  },
  "related_events": [
    {
      "id": "0033TCZXVEPNG",
      "uri": "pubky://satoshi/pub/pubky.app/event/0033TCZXVEPNG",
      "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
      "recurrence_id": 1699358400000000,
      "dtstart": 1699365600000000,
      "dtend": 1699380000000000,
      "summary": "Bitcoin Meetup Zürich - Special Workshop",
      "rrule": null,
      "location": "Tech Hub Zürich"
    },
    {
      "id": "0033UCZXVEPNG",
      "uri": "pubky://satoshi/pub/pubky.app/event/0033UCZXVEPNG",
      "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
      "recurrence_id": 1699963200000000,
      "dtstart": 1699963200000000,
      "dtend": 1699974000000000,
      "summary": "Bitcoin Meetup Zürich - Guest Speaker",
      "rrule": null,
      "location": "Insider Bar"
    }
  ]
}
````

**Response Fields:**

- `event`: The requested event (primary response object) with full details
  including attendees and alarms
- `related_events`: Array of all other events with the same `uid` (sorted by
  `dtstart`)
  - For master events: contains all override events
  - For override events: contains the master event + all other overrides
  - Empty array for standalone events (no recurrence)
  - Related events include core fields but not attendees/alarms (those are
    fetched only for the primary event)

**Example Response for Override Event:**

When requesting an override event, the master is included in `related_events`:

```json
{
  "event": {
    "id": "0033TCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/event/0033TCZXVEPNG",
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "recurrence_id": 1699358400000000,
    "summary": "Bitcoin Meetup Zürich - Special Workshop",
    "rrule": null,
    "attendees": [...],
    "alarms": [...]
  },
  "related_events": [
    {
      "id": "0033SCZXVEPNG",
      "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
      "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
      "recurrence_id": null,
      "rrule": "FREQ=WEEKLY;BYDAY=WE",
      "summary": "Bitcoin Meetup Zürich"
    },
    {
      "id": "0033UCZXVEPNG",
      "recurrence_id": 1699963200000000,
      "summary": "Bitcoin Meetup Zürich - Guest Speaker"
    }
  ]
}
```

````
## Error Responses

All endpoints return standard HTTP status codes and error responses:

```json
{
  "error": {
    "code": "CALENDAR_NOT_FOUND",
    "message": "Calendar with ID '0033RCZXVEPNG' not found",
    "details": {
      "calendar_id": "0033RCZXVEPNG"
    }
  }
}
````

**Common Error Codes:**

- `CALENDAR_NOT_FOUND`: Calendar with specified ID not found
- `EVENT_NOT_FOUND`: Event with specified ID not found
- `ATTENDEE_NOT_FOUND`: Attendee with specified ID not found
- `ALARM_NOT_FOUND`: Alarm with specified ID not found
- `INVALID_PARAMETERS`: Invalid query parameters provided
- `UNAUTHORIZED`: Authentication required
- `FORBIDDEN`: Insufficient permissions

## RRULE Examples

The following are examples of RRULE strings used in event recurrence:

### Weekly Recurring Event

```json
{
  "rrule": "FREQ=WEEKLY;BYDAY=WE",
  "summary": "Bitcoin Meetup Zürich"
}
```

Creates an event that repeats every Wednesday.

### Monthly Recurring Event

```json
{
  "rrule": "FREQ=MONTHLY;BYMONTHDAY=15",
  "summary": "Monthly Bitcoin Talk"
}
```

Creates an event that repeats on the 15th of every month.

### Bi-weekly Recurring Event

```json
{
  "rrule": "FREQ=WEEKLY;INTERVAL=2;BYDAY=MO,WE,FR",
  "summary": "Bi-weekly Coding Session"
}
```

Creates an event that repeats every two weeks on Monday, Wednesday, and Friday.

### Recurring Event with End Date

```json
{
  "rrule": "FREQ=WEEKLY;BYDAY=SA;UNTIL=20251231T235959Z",
  "summary": "Weekend Bitcoin Study Group"
}
```

Creates an event that repeats every Saturday until December 31, 2025.

### Recurring Event with Count

```json
{
  "rrule": "FREQ=DAILY;COUNT=10",
  "summary": "10-day Bitcoin Bootcamp"
}
```

Creates an event that repeats daily for exactly 10 occurrences.

### Recurrence Override Examples

**Master Recurring Event Response:**

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
  "exdate": [
    "2025-10-16T19:00:00+02:00",
    "2025-10-23T19:00:00+02:00"
  ],
  "recurrence_id": null,
  "image_uri": "pubky://satoshi/pub/pubky.app/files/0033EVENT01",
  "conference": null,
  "location": "Insider Bar",
  "geo": "47.366667;8.550000",
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
  }
}
```

**Override Event Response:**

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
  "geo": "47.370000;8.545000",
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
  }
}
```

**Key Override Response Characteristics:**

- Same `uid` as master event
- `recurrence_id` set to original occurrence time
- `rrule` is null (overrides don't have recurrence rules)
- Modified properties (time, location, summary, etc.)
- Stored as separate event entity
