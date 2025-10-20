# V1: Separate Types (jCal Storage)

This approach uses **separate dedicated types** for each calendar component with
**jCal JSON storage**.

## Approach

This variant defines distinct data structures in `pubky-app-specs`:

- `PubkyAppCalendar` - For calendar collections
- `PubkyAppEvent` - For events
- `PubkyAppAttendee` - For RSVPs
- `PubkyAppAlarm` - For reminders

Each type stores its data as **jCal JSON** in a `content` field, providing
maximum flexibility for RFC extensions.

## Storage Structure

```
/pub/pubky.app/
├── calendar/:calendar_id     # Calendar metadata
├── event/:event_id           # Individual events
├── attendee/:attendee_id     # RSVP records
└── alarm/:alarm_id           # User reminders
```

## ✅ Advantages

- **Type Safety**: Strict compile-time validation of component types
- **Flexibility**: Easy to add new RFC properties via jCal
- **Standards Compliance**: Full jCal (RFC 7265) support
- **Clear Separation**: Each component type is completely independent
- **Future-Proof**: New RFC extensions require no type changes

## ❌ Disadvantages

- **More Code**: Requires defining multiple new types and handlers
- **Runtime Parsing**: jCal JSON parsing required at runtime
- **Less Integration**: Doesn't leverage existing post infrastructure
- **Hidden Structure**: Field definitions hidden in jCal JSON

## Files in This Folder

**Variant-Specific Files:**

- `pubky-ical-specification.md` - Complete specification using separate types
- `nexus-endpoints.md` - Nexus API endpoints for separate types
- `event-examples.md` - Example calendar data (showing separate types structure)

**Shared Architecture Files** (in root directory):

- `../diagrams.md` - Architecture diagrams (shared between variants)
- `../prototype-implementation-scope.md` - Frontend implementation scope
  (shared)
- `../context-references.md` - External RFC references (shared)
- `../notes.md` - General development notes (shared)

---

**For comparison with the explicit fields approach, see**:
`../v2-explicit-fields/README.md`
