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

### Design Principle: jCal-First Responses

All Nexus endpoints return calendar components in jCal format (RFC 7265) with
optional metadata. This ensures:

- **Consistency**: Frontends handle parsing and formatting uniformly.
- **Extensibility**: New iCalendar properties are automatically supported.
- **Standards Compliance**: Direct mapping to/from iCalendar without data loss.
- **Frontend Flexibility**: UIs decide presentation logic, not the indexer.

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
          [
            "description",
            {},
            "text",
            "Bitcoin and decentralization meetups across Switzerland"
          ],
          ["color", {}, "text", "#F7931A"],
          ["categories", {}, "text", "bitcoin,meetups,decentralization"],
          ["x-pubky-admins", {}, "uri", "pubky://satoshi"],
          ["x-pubky-admins", {}, "uri", "pubky://adam-back"],
          ["created", {}, "date-time", "2024-10-01T15:00:00Z"],
          ["last-modified", {}, "date-time", "2024-10-02T12:00:00Z"]
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

**Response**:

```json
{
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
      [
        "description",
        {},
        "text",
        "Bitcoin and decentralization meetups across Switzerland"
      ],
      ["color", {}, "text", "#F7931A"],
      ["categories", {}, "text", "bitcoin,meetups,decentralization"],
      ["x-pubky-admins", {}, "uri", "pubky://satoshi"],
      ["x-pubky-admins", {}, "uri", "pubky://adam-back"],
      ["created", {}, "date-time", "2024-10-01T15:00:00Z"],
      ["last-modified", {}, "date-time", "2024-10-02T12:00:00Z"]
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
| `location`     | string  | -         | Filter using OSM structured location               |
| `sort`         | string  | `dtstart` | Sort order (`dtstart`, `created`, `modified`)      |

**Response**:

```json
{
  "events": [
    {
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
          ["dtstamp", {}, "date-time", "2024-10-01T12:15:00Z"],
          [
            "dtstart",
            { "tzid": "Europe/Zurich" },
            "date-time",
            "2024-10-09T19:00:00"
          ],
          [
            "dtend",
            { "tzid": "Europe/Zurich" },
            "date-time",
            "2024-10-09T22:00:00"
          ],
          ["summary", {}, "text", "Bitcoin Meetup Zürich"],
          [
            "description",
            {},
            "text",
            "Weekly Bitcoin meetup discussing Lightning Network"
          ],
          ["location", {}, "text", "Insider Bar, Zürich"],
          ["status", {}, "text", "CONFIRMED"],
          ["categories", {}, "text", "bitcoin,meetup,networking"],
          [
            "organizer",
            { "cn": "Satoshi" },
            "cal-address",
            "pubky://satoshi"
          ],
          [
            "x-pubky-calendar",
            {},
            "uri",
            "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
          ],
          ["created", {}, "date-time", "2024-09-25T10:30:00Z"],
          ["last-modified", {}, "date-time", "2024-10-01T14:05:00Z"]
        ],
        []
      ],
      "metadata": {
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "attendee_count": 8,
        "alarm_count": 0,
        "created_at": 1727270200000000,
        "updated_at": 1727785500000000
      }
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
      "jcal": [
        "vevent",
        [
          [
            "uid",
            {},
            "text",
            "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG"
          ],
          ["dtstamp", {}, "date-time", "2024-10-01T12:15:00Z"],
          [
            "dtstart",
            { "tzid": "Europe/Zurich" },
            "date-time",
            "2024-10-09T19:00:00"
          ],
          [
            "dtend",
            { "tzid": "Europe/Zurich" },
            "date-time",
            "2024-10-09T22:00:00"
          ],
          ["summary", {}, "text", "Bitcoin Meetup Zürich"],
          [
            "description",
            {},
            "text",
            "Weekly Bitcoin meetup discussing Lightning Network"
          ],
          ["location", {}, "text", "Insider Bar, Zürich"],
          ["status", {}, "text", "CONFIRMED"],
          ["categories", {}, "text", "bitcoin,meetup,networking"],
          [
            "organizer",
            { "cn": "Satoshi" },
            "cal-address",
            "pubky://satoshi"
          ],
          [
            "x-pubky-calendar",
            {},
            "uri",
            "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
          ],
          ["created", {}, "date-time", "2024-09-25T10:30:00Z"],
          ["last-modified", {}, "date-time", "2024-10-01T14:05:00Z"]
        ],
        []
      ],
      "metadata": {
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "calendar_name": "Dezentralschweiz Meetups",
        "attendee_count": 8,
        "alarm_count": 0,
        "created_at": 1727270200000000,
        "updated_at": 1727785500000000
      }
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

**Response**:

```json
{
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
      ["dtstamp", {}, "date-time", "2024-10-01T12:15:00Z"],
      [
        "dtstart",
        { "tzid": "Europe/Zurich" },
        "date-time",
        "2024-10-09T19:00:00"
      ],
      [
        "dtend",
        { "tzid": "Europe/Zurich" },
        "date-time",
        "2024-10-09T22:00:00"
      ],
      ["summary", {}, "text", "Bitcoin Meetup Zürich"],
      [
        "description",
        {},
        "text",
        "Weekly Bitcoin meetup discussing Lightning Network"
      ],
      ["location", {}, "text", "Insider Bar, Zürich"],
      ["geo", {}, "float", 47.376888, 8.541694],
      ["status", {}, "text", "CONFIRMED"],
      ["categories", {}, "text", "bitcoin,meetup,networking"],
      [
        "organizer",
        { "cn": "Satoshi" },
        "cal-address",
        "pubky://satoshi"
      ],
      [
        "x-pubky-calendar",
        {},
        "uri",
        "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG"
      ],
      ["created", {}, "date-time", "2024-09-25T10:30:00Z"],
      ["last-modified", {}, "date-time", "2024-10-01T14:05:00Z"]
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
```

---

### GET /pub/pubky.app/vevent/:event_id/attendees

List all attendees for a specific event.

**Path Parameters**:

- `event_id` (string): Timestamp-based event ID

**Query Parameters**:

| Parameter  | Type    | Default | Description                                                                  |
| ---------- | ------- | ------- | ---------------------------------------------------------------------------- |
| `skip`     | integer | `0`     | Pagination offset                                                            |
| `limit`    | integer | `50`    | Number of results (max: 200)                                                 |
| `partstat` | string  | -       | Filter by participation status (ACCEPTED, DECLINED, TENTATIVE, NEEDS-ACTION) |

**Response**:

```json
{
  "attendees": [
    {
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
          ["dtstamp", {}, "date-time", "2024-10-02T09:30:00Z"],
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
          ],
          ["created", {}, "date-time", "2024-09-26T14:00:00Z"]
        ],
        []
      ],
      "metadata": {
        "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "created_at": 1727356800000000
      }
    },
    {
      "uri": "pubky://bob/pub/pubky.app/vattendee/0033ZDZXVEPNG",
      "jcal": [
        "attendee",
        [
          [
            "uid",
            {},
            "text",
            "pubky://bob/pub/pubky.app/vattendee/0033ZDZXVEPNG"
          ],
          ["dtstamp", {}, "date-time", "2024-10-03T08:20:00Z"],
          [
            "attendee",
            {
              "cn": "Bob",
              "role": "OPT-PARTICIPANT",
              "partstat": "TENTATIVE",
              "rsvp": "TRUE"
            },
            "cal-address",
            "pubky://bob"
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
          ],
          ["created", {}, "date-time", "2024-09-27T11:00:00Z"]
        ],
        []
      ],
      "metadata": {
        "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "created_at": 1727443200000000
      }
    }
  ],
  "total": 8,
  "skip": 0,
  "limit": 50
}
```

---

### GET /pub/pubky.app/vevent/:event_id/alarms

List all alarms for a specific event.

**Path Parameters**:

- `event_id` (string): Timestamp-based event ID

**Query Parameters**:

| Parameter | Type    | Default | Description                                   |
| --------- | ------- | ------- | --------------------------------------------- |
| `skip`    | integer | `0`     | Pagination offset                             |
| `limit`   | integer | `50`    | Number of results (max: 200)                  |
| `action`  | string  | -       | Filter by action type (DISPLAY, AUDIO, EMAIL) |
| `user`    | string  | -       | Filter by alarm owner pubky:// URI            |

**Response**:

```json
{
  "alarms": [
    {
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
          ],
          ["created", {}, "date-time", "2024-09-28T16:00:00Z"]
        ],
        []
      ],
      "metadata": {
        "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "owner": "pubky://bob",
        "created_at": 1727529600000000
      }
    }
  ],
  "total": 3,
  "skip": 0,
  "limit": 50
}
```

---

## Attendee Endpoints

### GET /pub/pubky.app/vattendee

List all attendance records across the network.

**Query Parameters**:

| Parameter  | Type    | Default   | Description                                                                  |
| ---------- | ------- | --------- | ---------------------------------------------------------------------------- |
| `skip`     | integer | `0`       | Pagination offset                                                            |
| `limit`    | integer | `20`      | Number of results (max: 100)                                                 |
| `user`     | string  | -         | Filter by user pubky:// URI                                                  |
| `event`    | string  | -         | Filter by event pubky:// URI                                                 |
| `calendar` | string  | -         | Filter by calendar pubky:// URI                                              |
| `partstat` | string  | -         | Filter by participation status (ACCEPTED, DECLINED, TENTATIVE, NEEDS-ACTION) |
| `sort`     | string  | `created` | Sort order (`created`, `modified`)                                           |

**Response**:

```json
{
  "attendees": [
    {
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
          ["dtstamp", {}, "date-time", "2024-10-02T09:30:00Z"],
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
          ],
          ["created", {}, "date-time", "2024-09-26T14:00:00Z"]
        ],
        []
      ],
      "metadata": {
        "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
        "event_summary": "Bitcoin Meetup Zürich",
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "calendar_name": "Dezentralschweiz Meetups",
        "created_at": 1727356800000000
      }
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
      ["dtstamp", {}, "date-time", "2024-10-02T09:30:00Z"],
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
      ],
      ["created", {}, "date-time", "2024-09-26T14:00:00Z"]
    ],
    []
  ],
  "metadata": {
    "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
    "event_summary": "Bitcoin Meetup Zürich",
    "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
    "calendar_name": "Dezentralschweiz Meetups",
    "created_at": 1727356800000000
  }
}
```

---

## Alarm Endpoints

### GET /pub/pubky.app/valarm

List all alarms across the network.

**Query Parameters**:

| Parameter  | Type    | Default   | Description                                   |
| ---------- | ------- | --------- | --------------------------------------------- |
| `skip`     | integer | `0`       | Pagination offset                             |
| `limit`    | integer | `20`      | Number of results (max: 100)                  |
| `user`     | string  | -         | Filter by user pubky:// URI (alarm owner)     |
| `event`    | string  | -         | Filter by event pubky:// URI                  |
| `calendar` | string  | -         | Filter by calendar pubky:// URI               |
| `action`   | string  | -         | Filter by action type (DISPLAY, AUDIO, EMAIL) |
| `sort`     | string  | `created` | Sort order (`created`, `modified`)            |

**Response**:

```json
{
  "alarms": [
    {
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
          ],
          ["created", {}, "date-time", "2024-09-28T16:00:00Z"]
        ],
        []
      ],
      "metadata": {
        "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
        "event_summary": "Bitcoin Meetup Zürich",
        "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
        "calendar_name": "Dezentralschweiz Meetups",
        "owner": "pubky://bob",
        "created_at": 1727529600000000
      }
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
      ],
      ["created", {}, "date-time", "2024-09-28T16:00:00Z"]
    ],
    []
  ],
  "metadata": {
    "event_uri": "pubky://satoshi/pub/pubky.app/vevent/0033SCZXVEPNG",
    "event_summary": "Bitcoin Meetup Zürich",
    "calendar_uri": "pubky://satoshi/pub/pubky.app/vcalendar/0033RCZXVEPNG",
    "calendar_name": "Dezentralschweiz Meetups",
    "owner": "pubky://bob",
    "created_at": 1727529600000000
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
  "calendars|events|attendees|alarms": [
    {
      "uri": "pubky://...",
      "jcal": [...],
      "metadata": {...}
    }
  ],
  "total": <integer>,
  "skip": <integer>,
  "limit": <integer>
}
```

### Single Item Response

Single item endpoints return the item directly:

```json
{
  "uri": "pubky://...",
  "jcal": [...],
  "metadata": {...}
}
```

### jCal Format

All calendar components are returned in jCal format (RFC 7265):

```json
[
  "component_type",
  [
    ["property_name", { "param": "value" }, "value_type", "value"],
    ...
  ],
  [
    /* sub-components if any */
  ]
]
```

**Key Points**:

- Component types: `vcalendar`, `vevent`, `attendee`, `valarm`
- Property format: `[name, parameters_object, value_type, ...values]`
- Parameters can be empty objects `{}` if no parameters exist.
- Multiple values are comma-separated or as additional array elements.
- The sub-components array is always present (empty `[]` if none).

### Metadata Object

The `metadata` object provides computed/aggregated information not in the
original iCalendar component:

**Common metadata fields**:

- `created_at` (integer): Unix timestamp in microseconds
- `updated_at` (integer): Unix timestamp in microseconds (for mutable items)
- `event_count` (integer): Count of events (for calendars)
- `attendee_count` (integer): Count of attendees (for events/calendars)
- `alarm_count` (integer): Count of alarms (for events)
- `calendar_uri` (string): Parent calendar pubky:// URI
- `calendar_name` (string): Human-readable calendar name (optional)
- `event_uri` (string): Parent event pubky:// URI
- `event_summary` (string): Human-readable event summary (optional)
- `owner` (string): pubky:// URI of the item owner (for user-specific items)

**Rationale**:

Metadata provides indexer-computed information that helps UIs make decisions
without parsing jCal. For example:

- Display "8 attendees" without parsing all attendee records.
- Sort/filter by creation time efficiently.
- Show parent context ("Bitcoin Meetup Zürich" in "Dezentralschweiz Meetups").

Frontends should prefer jCal data for display but can use metadata for
performance optimizations and aggregated views.

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

1. Scanning all homeservers for `/pub/pubky.app/vcalendar/*` paths.
2. Extracting the `x-pubky-admins` property from each calendar.
3. Building an index mapping calendar URIs to admin lists.
4. Computing metadata (event counts, attendee counts, timestamps).

### Event Aggregation

Nexus aggregates events for each calendar by:

1. For each calendar, identify all admin pubky:// URIs from `x-pubky-admins`.
2. Scan each admin's homeserver for `/pub/pubky.app/vevent/*` paths.
3. Filter events where `x-pubky-calendar` matches the calendar URI.
4. Aggregate all matching events into the calendar's event collection.
5. Build temporal indexes for efficient date-range queries.

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
   `/pub/pubky.app/valarm/*`.
2. Extract `x-pubky-calendar` and `x-pubky-event` references.
3. Build reverse indexes for efficient querying:
   - event → attendees mapping
   - event → alarms mapping
   - user → attendances mapping
   - calendar → all attendees/alarms

---

## Future Enhancements

### User-Centric Views

User-specific aggregation endpoints:

- `GET /pub/pubky.app/users/:user_id/calendars` - User's owned/subscribed
  calendars
- `GET /pub/pubky.app/users/:user_id/events` - User's organized/attending events
- `GET /pub/pubky.app/users/:user_id/alarms` - User's personal alarms

### Advanced Queries

Additional query capabilities:

- Geospatial queries on location/geo properties
- Free/busy time calculations for users based on attendance

### CalDAV Bridge Integration

- Special endpoints for CalDAV bridge support

---

## Notes on Design Decisions

### Why jCal Instead of Formatted Objects?

**Consistency**: Every component type (calendar, event, attendee, alarm) uses
the same jCal structure. Frontends write one parser, not four.

**Extensibility**: When new iCalendar properties are added (e.g., RFC 9073
conference URIs, RFC 9074 valarms, future extensions), Nexus automatically
supports them without API changes.

**Standards Compliance**: jCal (RFC 7265) is a ratified standard with
well-defined semantics. Direct mapping ensures no information loss.

**Frontend Control**: UIs decide how to format dates, localize text, and present
properties. The indexer doesn't impose formatting decisions.

### Metadata vs jCal

Metadata provides:

- **Performance**: Aggregated counts (event_count, attendee_count) without
  parsing child components.
- **Context**: Parent names (calendar_name, event_summary) for display in lists.
- **Indexer State**: Timestamps (created_at, updated_at) track indexing, not
  just component semantics.

Metadata should **never** duplicate information in jCal. It augments with
computed/contextual data.

### URI as Primary Identifier

Every component returns its `uri` field at the top level for:

- **Direct Access**: Clients can construct URLs without parsing jCal.
- **Caching**: URI-based cache keys.
- **Relationships**: Reference components by URI in joins/links.

The URI is also in jCal as the `uid` property, but top-level duplication
improves ergonomics.
