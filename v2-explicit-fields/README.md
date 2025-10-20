# V2: Explicit Fields (Typed Storage)

This approach uses **separate dedicated types** with **explicit typed fields**
instead of jCal JSON storage.

## Approach

This version defines distinct data structures in `pubky-app-specs` with explicit
RFC fields:

- `PubkyAppCalendar` - For calendar collections
- `PubkyAppEvent` - For events
- `PubkyAppAttendee` - For RSVPs
- `PubkyAppAlarm` - For reminders

Each type has **explicit typed fields** for RFC properties, providing
compile-time type safety and clear schema visibility.

## Storage Structure

```
/pub/pubky.app/
├── calendar/:calendar_id     # Calendar metadata
├── event/:event_id           # Individual events
├── attendee/:attendee_id     # RSVP records
└── alarm/:alarm_id           # User reminders
```

## Field Set

Uses **MVP subset** (~25 fields) covering essential calendar functionality:

- **Calendar fields**: name, color, admins, timezone
- **Event fields**: uid, timestamps, summary, status, organizer, categories,
  recurrence, location, description
- **Attendee fields**: uri, name, status, role

## ✅ Advantages

- **Type Safety**: Compile-time validation of all fields
- **Explicit Schema**: Clear field definitions visible in code
- **IDE Support**: Full autocomplete and type checking
- **Performance**: No runtime JSON parsing required
- **Clear Structure**: Field purposes are immediately obvious

## ❌ Disadvantages

- **Less Flexible**: Adding new RFC fields requires type updates
- **More Code**: Requires defining explicit field structures
- **Version Updates**: New RFC extensions need code changes

## Documentation

- **[Complete Specification](pubky-ical-specification.md)** - Full technical
  details with RFC field tables
- **[Nexus API Endpoints](nexus-endpoints.md)** - API documentation
- **[Example Data](event-examples.md)** - Sample calendar data

## Comparison

See [APPROACH_COMPARISON.md](../APPROACH_COMPARISON.md) for a detailed
comparison with the jCal storage approach.
