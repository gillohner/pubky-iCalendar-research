# Nexus Calendar Endpoints

This document specifies the Nexus indexer endpoints for aggregating and querying
Pubky calendar components. Nexus is responsible for discovering calendar
relationships, aggregating events from calendar admins, and providing efficient
query interfaces for calendar applications.

## Index

- [Endpoint Overview](#endpoint-overview)
- [Calendar Endpoints](#calendar-endpoints)
- [Event Endpoints](#event-endpoints)
- [Attendee Endpoints](#attendee-endpoints)
- [Alarm Endpoints](#alarm-endpoints)
- [Query Parameters](#query-parameters)
- [Response Formats](#response-formats)

---

## Endpoint Overview

Nexus provides REST endpoints for querying calendar components stored across
user homeservers. The indexer continuously scans homeservers for new calendar
components and maintains aggregated views based on relationships defined in the
specification.

**Base URL (MVP Development)**: `https://nexus.riginode.xyz/v0` or
`http://localhost:3000/v0`

---

## Calendar Endpoints

### GET /pub/pubky.app/vcalendar

List all calendars across the network.

**Query Parameters**:

| Parameter    | Type    | Default   | Description                                            |
| ------------ | ------- | --------- | ------------------------------------------------------ |
| `skip`       | integer | `0`       | Pagination offset                                      |
| `limit`      | integer | `20`      | Number of results                                      |
| `admin`      | string  | -         | Filter by admin pubky:// URI                           |
| `categories` | string  | -         | Filter by category (comma-separated)                   |
| `search`     | string  | -         | Full-text search in name/description                   |
| `sort`       | string  | `popular` | `created`, `modified`, `popular`, `name`, `eventcount` |

**Response**:

```json
{
  "calendars": [
    {
      "uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
      "name": "Dezentralschweiz Meetups",
      "description": "Bitcoin and decentralization meetups across Switzerland",
      "admins": ["pubky://satoshi", "pubky://adam-back"],
      "color": "#F7931A",
      "categories": ["bitcoin", "meetups", "decentralization"],
      "event_count": 12,
      "created_at": 1727785200000000,
      "updated_at": 1727871600000000
    }
  ],
  "total": 1,
  "skip": 0,
  "limit": 20
}
```

---

### GET /pub/pubky.app/vcalendar/:calendar_id

Retrieve a specific calendar by ID.

**Path Parameters**:

- `calendar_id` (string): Timestamp-based calendar ID

**Response (Trimmed jCal)**:

```json
{
  "calendar": {
    "uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
    "jcal": [
      "vcalendar",
      [
        ["prodid", {}, "text", "-//Pubky//Pubky Calendar 1.0//EN"],
        ["version", {}, "text", "2.0"],
        [
          "uid",
          {},
          "text",
          "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
        ],
        ["name", {}, "text", "Dezentralschweiz Meetups"],
        ["x-pubky-admins", {}, "uri", "pubky://satoshi", "pubky://adam-back"]
      ],
      []
    ],
    "metadata": {
      "event_count": 12,
      "attendee_count": 45,
      "created_at": 1727785200000000,
      "updated_at": 1727871600000000
    }
  }
}
```

---

### GET /pub/pubky.app/vcalendar/:calendar_id/events

List all events for a specific calendar (aggregated from admin homeservers).

**Path Parameters**:

- `calendar_id` (string): Timestamp-based calendar ID

**Query Parameters**:

| Parameter      | Type    | Default   | Description                                        |
| -------------- | ------- | --------- | -------------------------------------------------- |
| `skip`         | integer | `0`       | Pagination offset                                  |
| `limit`        | integer | `20`      | Number of results (max: 100)                       |
| `start_after`  | string  | -         | Filter events starting after this date (ISO 8601)  |
| `start_before` | string  | -         | Filter events starting before this date (ISO 8601) |
| `status`       | string  | -         | Filter by status (CONFIRMED, TENTATIVE, CANCELLED) |
| `organizer`    | string  | -         | Filter by organizer pubky:// URI                   |
| `sort`         | string  | `dtstart` | Sort order (`dtstart`, `created`, `modified`)      |

**Response**:

```json
{
  "events": [
    {
      "uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
      "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
      "summary": "Bitcoin Meetup Zürich",
      "dtstart": "2025-10-09T19:00:00+02:00",
      "dtend": "2025-10-09T22:00:00+02:00",
      "location": "Insider Bar, Zürich",
      "organizer": "pubky://satoshi",
      "status": "CONFIRMED",
      "attendee_count": 8,
      "created_at": 1727270200000000
    }
  ],
  "total": 12,
  "skip": 0,
  "limit": 20
}
```

---

## Event Endpoints

### GET /pub/pubky.app/vevent

List all events across the network.

**Query Parameters**:

| Parameter      | Type    | Default   | Description                                        |
| -------------- | ------- | --------- | -------------------------------------------------- |
| `skip`         | integer | `0`       | Pagination offset                                  |
| `limit`        | integer | `20`      | Number of results (max: 100)                       |
| `start_after`  | string  | -         | Filter events starting after this date (ISO 8601)  |
| `start_before` | string  | -         | Filter events starting before this date (ISO 8601) |
| `calendar`     | string  | -         | Filter by calendar pubky:// URI                    |
| `organizer`    | string  | -         | Filter by organizer pubky:// URI                   |
| `categories`   | string  | -         | Filter by category (comma-separated)               |
| `status`       | string  | -         | Filter by status (CONFIRMED, TENTATIVE, CANCELLED) |
| `sort`         | string  | `dtstart` | Sort order (`dtstart`, `created`, `modified`)      |

**Response**:

```json
{
  "events": [
    {
      "uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
      "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
      "calendar_name": "Dezentralschweiz Meetups",
      "summary": "Bitcoin Meetup Zürich",
      "description": "Weekly Bitcoin meetup discussing Lightning Network",
      "dtstart": "2025-10-09T19:00:00+02:00",
      "dtend": "2025-10-09T22:00:00+02:00",
      "location": "Insider Bar, Zürich",
      "organizer": "pubky://satoshi",
      "status": "CONFIRMED",
      "categories": ["bitcoin", "meetup", "networking"],
      "attendee_count": 8,
      "created_at": 1727270200000000,
      "updated_at": 1727785500000000
    }
  ],
  "total": 156,
  "skip": 0,
  "limit": 20
}
```

---

### GET /pub/pubky.app/vevent/:event_id

Retrieve a specific event by ID.

**Path Parameters**:

- `event_id` (string): Timestamp-based event ID

**Response (Trimmed jCal)**:

```json
{
  "event": {
    "uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
    "jcal": [
      "vevent",
      [
        [
          "uid",
          {},
          "text",
          "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"
        ],
        ["dtstamp", {}, "date-time", "2025-10-01T12:15:00Z"],
        [
          "dtstart",
          { "tzid": "Europe/Zurich" },
          "date-time",
          "2025-10-09T19:00:00"
        ],
        ["summary", {}, "text", "Bitcoin Meetup Zürich"],
        [
          "x-pubky-calendar",
          {},
          "uri",
          "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
        ]
      ],
      []
    ],
    "metadata": {
      "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
      "calendar_name": "Dezentralschweiz Meetups",
      "attendee_count": 8,
      "alarm_count": 3,
      "created_at": 1727270200000000,
      "updated_at": 1727785500000000
    }
  }
}
```

---

### GET /pub/pubky.app/vevent/:event_id/attendees

List all attendees for a specific event.

**Path Parameters**:

- `event_id` (string): Timestamp-based event ID

**Query Parameters**:

| Parameter  | Type    | Default | Description                                                              |
| ---------- | ------- | ------- | ------------------------------------------------------------------------ |
| `skip`     | integer | `0`     | Pagination offset                                                        |
| `limit`    | integer | `50`    | Number of results (max: 200)                                             |
| `partstat` | string  | -       | Filter by participation status (ACCEPTED, DECLINED, TENTATIVE, NEEDS-ACTION) |

**Response**:

```json
{
  "attendees": [
    {
      "uri": "pubky://alice/pub/pubky.app/vattendee/0033ZCZXVEPNG",
      "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
      "user": "pubky://alice",
      "cn": "Alice",
      "partstat": "ACCEPTED",
      "role": "REQ-PARTICIPANT",
      "created_at": 1727356800000000
    },
    {
      "uri": "pubky://bob/pub/pubky.app/vattendee/0033ZDZXVEPNG",
      "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
      "user": "pubky://bob",
      "cn": "Bob",
      "partstat": "TENTATIVE",
      "role": "OPT-PARTICIPANT",
      "created_at": 1727443200000000
    }
  ],
  "total": 8,
  "skip": 0,
  "limit": 50
}
```

---

## Attendee Endpoints

### GET /pub/pubky.app/vattendee

List all attendance records across the network.

**Query Parameters**:

| Parameter  | Type    | Default   | Description                                                              |
| ---------- | ------- | --------- | ------------------------------------------------------------------------ |
| `skip`     | integer | `0`       | Pagination offset                                                        |
| `limit`    | integer | `20`      | Number of results (max: 100)                                             |
| `user`     | string  | -         | Filter by user pubky:// URI                                              |
| `event`    | string  | -         | Filter by event pubky:// URI                                             |
| `calendar` | string  | -         | Filter by calendar pubky:// URI                                          |
| `partstat` | string  | -         | Filter by participation status (ACCEPTED, DECLINED, TENTATIVE, NEEDS-ACTION) |
| `sort`     | string  | `created` | Sort order (`created`, `modified`)                                       |

**Response**:

```json
{
  "attendees": [
    {
      "uri": "pubky://alice/pub/pubky.app/vattendee/0033ZCZXVEPNG",
      "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
      "event_summary": "Bitcoin Meetup Zürich",
      "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
      "calendar_name": "Dezentralschweiz Meetups",
      "user": "pubky://alice",
      "cn": "Alice",
      "partstat": "ACCEPTED",
      "role": "REQ-PARTICIPANT",
      "created_at": 1727356800000000
    }
  ],
  "total": 234,
  "skip": 0,
  "limit": 20
}
```

---

### GET /pub/pubky.app/vattendee/:attendee_id

Retrieve a specific attendance record by ID.

**Path Parameters**:

- `attendee_id` (string): Timestamp-based attendee ID

**Response**:

```json
{
  "attendee": {
    "uri": "pubky://alice/pub/pubky.app/vattendee/0033ZCZXVEPNG",
    "jcal": [
      "attendee",
      [
        [
          "uid",
          {},
          "text",
          "pubky://alice/pub/pubky.app/vattendee/0033ZCZXVEPNG"
        ],
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
    ],
    "metadata": {
      "event_summary": "Bitcoin Meetup Zürich",
      "calendar_name": "Dezentralschweiz Meetups",
      "created_at": 1727356800000000
    }
  }
}
```

---

## Alarm Endpoints

### GET /pub/pubky.app/valarm

List all alarms across the network.

**Query Parameters**:

| Parameter  | Type    | Default   | Description                                         |
| ---------- | ------- | --------- | --------------------------------------------------- |
| `skip`     | integer | `0`       | Pagination offset                                   |
| `limit`    | integer | `20`      | Number of results (max: 100)                        |
| `user`     | string  | -         | Filter by user pubky:// URI (alarm creator)         |
| `event`    | string  | -         | Filter by event pubky:// URI                        |
| `calendar` | string  | -         | Filter by calendar pubky:// URI                     |
| `action`   | string  | -         | Filter by action type (DISPLAY, AUDIO, EMAIL)       |
| `sort`     | string  | `created` | Sort order (`created`, `modified`)                  |

**Response**:

```json
{
  "alarms": [
    {
      "uri": "pubky://bob/pub/pubky.app/valarm/0033WCZXVEPNG",
      "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
      "event_summary": "Bitcoin Meetup Zürich",
      "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
      "user": "pubky://bob",
      "action": "DISPLAY",
      "trigger": "-PT15M",
      "description": "Bitcoin Meetup in 15 minutes",
      "created_at": 1727529600000000
    }
  ],
  "total": 89,
  "skip": 0,
  "limit": 20
}
```

---

### GET /pub/pubky.app/valarm/:alarm_id

Retrieve a specific alarm by ID.

**Path Parameters**:

- `alarm_id` (string): Timestamp-based alarm ID

**Response**:

```json
{
  "alarm": {
    "uri": "pubky://bob/pub/pubky.app/valarm/0033WCZXVEPNG",
    "jcal": [
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
    ],
    "metadata": {
      "event_summary": "Bitcoin Meetup Zürich",
      "calendar_name": "Dezentralschweiz Meetups",
      "created_at": 1727529600000000
    }
  }
}
```

---

## Query Parameters

### Common Parameters

All list endpoints support these common parameters:

- `skip` (integer): Pagination offset (default: 0)
- `limit` (integer): Number of results per page (default: 20, max varies by
  endpoint)
- `sort` (string): Sort order for results

### Date/Time Filtering

Event-related endpoints support temporal queries:

- `start_after` (string, ISO 8601): Events starting after this timestamp
- `start_before` (string, ISO 8601): Events starting before this timestamp
- `end_after` (string, ISO 8601): Events ending after this timestamp
- `end_before` (string, ISO 8601): Events ending before this timestamp

**Example**:

```
GET /pub/pubky.app/vevent?start_after=2025-10-01T00:00:00Z&start_before=2025-10-31T23:59:59Z
```

### Filtering by Relationships

Components can be filtered by their relationships:

- `calendar` (string): Filter by calendar pubky:// URI
- `event` (string): Filter by event pubky:// URI
- `user` (string): Filter by user pubky:// URI
- `organizer` (string): Filter by organizer pubky:// URI
- `admin` (string): Filter by calendar admin pubky:// URI

### Status and Category Filtering

- `status` (string): Event status (CONFIRMED, TENTATIVE, CANCELLED)
- `partstat` (string): Participation status (ACCEPTED, DECLINED, TENTATIVE,
  NEEDS-ACTION)
- `categories` (string): Comma-separated category list
- `action` (string): Alarm action type (DISPLAY, AUDIO, EMAIL)

---

## Response Formats

### Standard Response Structure

All Nexus endpoints return JSON with this structure:

```json
{
  "calendars|events|attendees|alarms": [...],
  "total": <integer>,
  "skip": <integer>,
  "limit": <integer>
}
```

### Single Item Response

Single item endpoints return the item with metadata:

```json
{
  "calendar|event|attendee|alarm": {
    "uri": "pubky://...",
    "jcal": [...],
    "metadata": {...}
  }
}
```

### Error Responses

Errors follow standard HTTP status codes:

```json
{
  "error": {
    "code": 404,
    "message": "Calendar not found",
    "details": "No calendar with ID 0033RCZXVEPNG exists"
  }
}
```

Common status codes:

- `200 OK`: Successful request
- `400 Bad Request`: Invalid query parameters
- `404 Not Found`: Resource does not exist
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server-side error

---

## Indexing Logic

### Calendar Aggregation

Nexus indexes calendars by:

1. Scanning all homeservers for `/pub/pubky.app/vcalendar/*` paths
2. Extracting `x-pubky-admins` property from each calendar
3. Building an index mapping calendar URIs to admin lists

### Event Aggregation

Nexus aggregates events for each calendar by:

1. For each calendar, identify all admin pubky:// URIs from `x-pubky-admins`
2. Scan each admin's homeserver for `/pub/pubky.app/vevent/*` paths
3. Filter events where `x-pubky-calendar` matches the calendar URI
4. Aggregate all matching events into the calendar's event collection

**Example**:

```
Calendar: pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG
Admins: ["pubky://satoshi", "pubky://hal"]

Nexus scans:
- pubky://satoshi/pub/pubky.app/vevent/* → finds event 0033SCZXVEPNG
- pubky://hal/pub/pubky.app/vevent/* → finds event 0033TD0XVEPNG

Both events have x-pubky-calendar pointing to calendar 0033RCZXVEPNG
→ Both events are aggregated into the calendar's event list
```

### Attendee and Alarm Aggregation

Attendees and alarms are indexed globally:

1. Scan all homeservers for `/pub/pubky.app/vattendee/*` and
   `/pub/pubky.app/valarm/*`
2. Extract `x-pubky-calendar` and `x-pubky-event` references
3. Build reverse indexes for efficient querying (event → attendees, calendar →
   alarms)

---

### Future Enhancements

Potential endpoint additions for advanced features:

- `/pub/pubky.app/vcalendar/:id/feed` - Combined feed of all calendar activity
- `/pub/pubky.app/vevent/upcoming` - Global upcoming events view
- `/pub/pubky.app/user/:user_id/calendars` - User's owned/subscribed calendars
- `/pub/pubky.app/user/:user_id/events` - User's event participation history
- WebSocket endpoints for real-time calendar updates
- Batch query endpoints for efficient multi-component fetching
