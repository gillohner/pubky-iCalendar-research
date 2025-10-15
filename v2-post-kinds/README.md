# V2: PubkyAppPost with New Kinds Approach

This folder contains the **updated specification** that uses `PubkyAppPost` with
new `kind` values for calendar functionality.

## Approach

This version **extends the existing PubkyAppPost** type with new `kind` enum
values:

```rust
pub enum PubkyAppPostKind {
    Short,
    Long,
    Image,
    Video,
    Link,
    File,
    Calendar,  // NEW
    Event,     // NEW
    Attendee,  // NEW
    Alarm,     // NEW
}
```

All calendar components are stored as **posts** at `/pub/pubky.app/posts/:id`
with calendar-specific content (jCal JSON).

## ✅ Advantages

- **Leverage Existing Infrastructure**: Reuses all existing post functionality
- **Simpler Implementation**: Minimal changes to pubky-app-specs (just enum
  values)
- **Built-in Features**: Automatically gets tags, bookmarks, replies, counts
- **Better Integration**: Calendar posts appear in feeds, can be shared/reposted
- **Nexus Ready**: Existing post endpoints work with `kind` filtering
- **Unified Storage**: All data in `/pub/pubky.app/posts/`

## ❌ Disadvantages

- **Less Type Safety**: Calendar data validated at runtime (jCal parsing), not
  compile-time
- **Content Opaque**: Post `content` field contains jCal JSON (generic string)
- **Validation Complexity**: Need to parse jCal to validate calendar-specific
  rules
- **Mixed Types**: All components stored in same path, differentiated only by
  `kind`

## Key Technical Details

### Redis Indexing (CRITICAL for Performance)

The specification includes a **hybrid Redis+Neo4j indexing strategy**:

1. **Redis sorted sets** for fast calendar/event/attendee queries
2. **Redis sets** for cached admin lists (O(1) validation)
3. **Neo4j** for complex filtering (date ranges, admin relationships)

This ensures calendar queries perform at the same level as existing post
queries.

### New Stream Sources

```rust
StreamSource::CalendarEvents { calendar_author, calendar_id }
StreamSource::EventAttendees { event_author, event_id }
```

These provide optimized access patterns without client-side filtering.

## Files in This Folder

**Version-Specific Files:**

- `pubky-ical-specification.md` - Complete specification using post kinds
- `nexus-endpoints.md` - **Comprehensive** Nexus API with Redis indexing
  strategy
- `event-examples.md` - Example calendar data (showing post-based structure with
  jCal)

**Shared Architecture Files** (in root directory):

- `../diagrams.md` - Architecture diagrams (shared between versions)
- `../prototype-implementation-scope.md` - Frontend implementation scope
  (shared)
- `../context-references.md` - External RFC references (shared)
- `../notes.md` - General development notes (shared)

---

**For comparison with the alternative approach, see**:
`../v1-separate-types/README.md`
