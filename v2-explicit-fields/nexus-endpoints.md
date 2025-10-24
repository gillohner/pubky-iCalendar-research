# Nexus API Endpoints (v2.0 - Explicit Fields)

## Overview

This document defines the Nexus API endpoints for the explicit fields approach
to Pubky iCalendar integration. All endpoints return structured data with
explicit typed fields instead of jCal JSON.

## Basic Endpoint overview

| Method | Endpoint                               | Description                                                    |
| ------ | -------------------------------------- | -------------------------------------------------------------- |
| Get    | /v0/stream/calendars                   | Retrieve a list of calendars with optional filtering.          |
| Get    | /v0/stream/events                      | Retrieve a list of events with optional filtering.             |
| Get    | /v0/calendar/{author_id}/{calendar_id} | Retrieve a specific calendar including its events and metadata |
| Get    | /v0/event/{author_id}/{event_id}       | Retrieve a specific event including its attendees and metadata |
| Get    | /v0/attendee/{author_id}/{attendee_id} | Retrieve a specific attendee                                   |
| Get    | /v0/alarm/{author_id}/{alarm_id}       | Retrieve a specific alarm                                      |

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
            "color": "#F7931A",
            "timezone": "Europe/Zurich",
            "admins": [
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
    "color": "#F7931A",
    "timezone": "Europe/Zurich",
    "admins": [
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
                "uri": "pubky://satoshi",
                "name": "Satoshi"
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
            "structured_location": {
                "uri": "geo:47.366667,8.550000",
                "name": "Insider Bar",
                "description": "Main venue"
            },
            "styled_description": {
                "fmttype": "text/html",
                "value": "<p>Weekly Bitcoin meetup discussing <strong>Lightning Network</strong></p>"
            }
        }
    ]
}
```

> Note: Tag listings for calendars and events may be added as optional
> expansions in a future version. They are not part of the MVP endpoints.

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
                "name": "Bitcoin Switzerland Events"
            },
            "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
            "dtstamp": 1698753600000000,
            "dtstart": 1698753600000000,
            "dtend": 1698764400000000,
            "summary": "Bitcoin Meetup Zürich",
            "status": "CONFIRMED",
            "organizer": {
                "uri": "pubky://satoshi",
                "name": "Satoshi"
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
            "structured_location": {
                "uri": "geo:47.366667,8.550000",
                "name": "Insider Bar",
                "description": "Main venue"
            },
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

### GET /v0/event/{author_id}/{event_id}

Retrieve a specific event by ID with attendees and alarms.

**Parameters:**

- `author_id`: Author Pubky ID (path)
- `event_id`: Event Crockford32 ID (path)
- `recurrence_id` (optional): Filter attendees by specific recurring event
  instance (Unix microseconds)

**Response:**

```json
{
    "id": "0033SCZXVEPNG",
    "uri": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "author": "pubky://satoshi",
    "calendar": {
        "id": "0033RCZXVEPNG",
        "uri": "pubky://satoshi/pub/pubky.app/calendar/0033RCZXVEPNG",
        "name": "Bitcoin Switzerland Events"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698753600000000,
    "dtend": 1698764400000000,
    "summary": "Bitcoin Meetup Zürich",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi",
        "name": "Satoshi"
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
    "structured_location": {
        "uri": "geo:47.366667,8.550000",
        "name": "Insider Bar",
        "description": "Main venue"
    },
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
            "attendee_name": "Alice",
            "partstat": "ACCEPTED",
            "role": "REQ-PARTICIPANT",
            "recurrence_id": null
        },
        {
            "id": "0033UDZXVEPNG",
            "uri": "pubky://bob/pub/pubky.app/attendee/0033UDZXVEPNG",
            "author": "pubky://bob",
            "attendee_uri": "pubky://bob",
            "attendee_name": "Bob",
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
}
```

> Note: Dedicated attendee listings by event may be added as an optional
> expansion in a future version. Not required for MVP.

> Note: Dedicated alarm listings by event may be added as an optional expansion
> in a future version. Not required for MVP.

> Note: Event tag listing may be added as an optional expansion in a future
> version. Not required for MVP.

## Attendee Endpoints

### GET /v0/attendee/{author_id}/{attendee_id}

Retrieve a specific attendee by ID.

**Parameters:**

- `author_id`: Author Pubky ID (path)
- `attendee_id`: Attendee Crockford32 ID (path)

**Response:**

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
    "attendee_name": "Alice",
    "partstat": "ACCEPTED",
    "role": "REQ-PARTICIPANT"
}
```

## Alarm Endpoints

### GET /v0/alarm/{author_id}/{alarm_id}

Retrieve a specific alarm by ID.

**Parameters:**

- `author_id`: Author Pubky ID (path)
- `alarm_id`: Alarm Crockford32 ID (path)

**Response:**

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
    "uid": "pubky://bob/pub/pubky.app/alarm/0033WCZXVEPNG"
}
```

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
```

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
        "name": "Bitcoin Switzerland Events"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1698753600000000,
    "dtend": 1698764400000000,
    "summary": "Bitcoin Meetup Zürich",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi",
        "name": "Satoshi"
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
    "structured_location": {
        "uri": "geo:47.366667,8.550000",
        "name": "Insider Bar",
        "description": "Main venue"
    },
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
        "name": "Bitcoin Switzerland Events"
    },
    "uid": "pubky://satoshi/pub/pubky.app/event/0033SCZXVEPNG",
    "dtstamp": 1698753600000000,
    "dtstart": 1699358400000000,
    "dtend": 1699372800000000,
    "summary": "Bitcoin Meetup Zürich - Special Lightning Workshop",
    "status": "CONFIRMED",
    "organizer": {
        "uri": "pubky://satoshi",
        "name": "Satoshi"
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
    "structured_location": {
        "uri": "geo:47.366667,8.550000",
        "name": "Tech Hub Zürich",
        "description": "Workshop venue for special event"
    },
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

## Tag Integration

Calendar and event entities support tagging through the existing PubkyAppTag
system:

- **Calendar Tags**: `/v0/calendar/{author_id}/{calendar_id}/tags`
- **Event Tags**: `/v0/event/{author_id}/{event_id}/tags`

Tags are included in Nexus responses for discoverability and filtering.

## Rate Limiting

All endpoints are subject to rate limiting:

- 1000 requests per hour per user
- 100 requests per minute per user

Rate limit headers are included in responses:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1698757200
```
