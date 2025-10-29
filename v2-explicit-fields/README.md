# V2: Explicit Fields (Typed Storage)

Separate dedicated types with explicit typed fields instead of jCal JSON.

## Core Types

- `PubkyAppCalendar` - Calendar collections
- `PubkyAppEvent` - Events
- `PubkyAppAttendee` - RSVP records
- `PubkyAppAlarm` - Reminders/notifications

## Storage Structure

```
/pub/pubky.app/
├── calendar/:id     # Calendar metadata
├── event/:id        # Individual events
├── attendee/:id     # RSVP records
└── alarm/:id        # User reminders
```

## Key Fields

**Calendar:** `name` (required), `timezone` (required), `color`, `image_uri`,
`description`, `url`, `created`, `x_pubky_admins`

**Event:** `uid` (required), `dtstamp` (required), `dtstart` (required),
`summary` (required), `dtend`/`duration`, `description`, `location`, `geo`,
`structured_locations`, `status`, `organizer`, `categories`, `image_uri`, `url`,
`conference`, `sequence`, `last_modified`, `created`, `dtstart_tzid`,
`dtend_tzid`, `rrule`, `rdate`, `exdate`, `recurrence_id`, `styled_description`,
`x_pubky_calendar_uris`, `x_pubky_rsvp_access`

**Attendee:** `attendee_uri` (required), `partstat` (required), `role`, `rsvp`,
`delegated_from`, `delegated_to`, `recurrence_id`, `x_pubky_event_uri`
(required)

**Alarm:** `action` (required), `trigger` (required), `description`, `summary`,
`attendees`, `repeat`, `duration`, `attach`, `x_pubky_target_uri` (required)

## Design Principles

- **Profile Integration**: Names fetched from `profile.json`, not duplicated
- **Location Layering**: `location` (text), `geo` (coordinates),
  `structured_locations` (RFC 9073 VLOCATION array)
- **Public RSVP**: Anyone can create attendee records (no invite workflow in
  MVP)
- **Type Safety**: Explicit Rust structs with compile-time validation
- **RFC Compliance**: Full RFC 5545/7986/9073 compatibility

## Documentation

- **[Specification](pubky-ical-specification.md)** - Complete technical details
- **[RFC Interoperability](RFC-CalDAV-Interoperability.md)** - Translation guide
  for iCalendar/CalDAV
- **[API Endpoints](nexus-endpoints.md)** - Nexus API documentation
- **[Examples](event-examples.md)** - Sample calendar data
