# V1: Separate PubkyApp Types Approach

This folder contains the original specification that uses **separate PubkyApp
types** for calendar functionality.

## Approach

This version defines distinct data structures in `pubky-app-specs`:

- `PubkyAppCalendar` - For calendar collections
- `PubkyAppEvent` - For events
- `PubkyAppAttendee` - For RSVPs
- `PubkyAppAlarm` - For reminders

Each type has its own:

- Data structure definition
- Validation rules
- Storage path (e.g., `/pub/pubky.app/calendars/:id`,
  `/pub/pubky.app/events/:id`)
- Nexus indexing logic

## ✅ Advantages

- **Type Safety**: Strict compile-time validation of calendar data structures
- **Clear Separation**: Each component type is completely independent
- **Explicit Schema**: Clear field definitions for each calendar component
- **Dedicated Paths**: Separate homeserver paths for different components

## ❌ Disadvantages

- **More Code**: Requires defining multiple new types and handlers
- **Less Integration**: Doesn't leverage existing post infrastructure
- **Complexity**: More complex Nexus implementation with separate indexing logic

## Files in This Folder

**Version-Specific Files:**

- `pubky-ical-specification.md` - Complete specification using separate types
- `nexus-endpoints.md` - Nexus API endpoints for separate types
- `event-examples.md` - Example calendar data (showing separate types structure)

**Shared Architecture Files** (in root directory):

- `../diagrams.md` - Architecture diagrams (shared between versions)
- `../prototype-implementation-scope.md` - Frontend implementation scope
  (shared)
- `../context-references.md` - External RFC references (shared)
- `../notes.md` - General development notes (shared)

---

**For comparison with the alternative approach, see**:
`../v2-post-kinds/README.md`
