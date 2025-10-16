# V3: Explicit RFC Fields Approach

This folder contains the specification that uses a **single new
`PubkyAppEventPost` type** with explicit RFC-defined fields instead of jCal JSON
content.

## Approach

This version defines structured Rust types with explicit fields from RFC
standards:

- **New Type**: `PubkyAppEventPost`
- **New Enum**: `PubkyAppEventPostKind` with values: `calendar`, `event`,
  `attendee`
- **Storage Path**: `/pub/pubky.app/event-posts/:id`
- **Field Structure**: Explicit typed fields (not jCal JSON strings)

### Key Design Decisions

**Type Safety:**

- Fields are explicitly typed in Rust structs
- Compile-time validation of field types
- No runtime jCal parsing needed

**Field Sources:**

- RFC 5545 (iCalendar Core)
- RFC 7986 (Modern Calendar Properties)
- RFC 9073 (Event Publishing Extensions)
- Custom Pubky extensions (x-pubky-admins)

**Data Formats:**

- **Dates**: Unix timestamp microseconds (i64)
- **Multi-value**: JSON array strings
- **Structured data**: JSON object strings (location, organizer, etc.)
- **Recurrence**: RFC 5545 RRULE string format

### Three Implementation Options

This specification provides **three field table options** to choose from based
on your needs:

#### Option 1: Complete RFC Implementation (~70 fields)

- **All** fields from RFC 5545, 7986, 9073
- Full CalDAV bridge compatibility
- Maximum future-proofing
- Most complex to implement
- Unnecessary fields for Pubky.app Event Publishing use cases

#### Option 2: MVP Subset (~25 fields)

- Essential fields only
- Covers most common use cases
- Simplest to implement
- Easiest to extend later

#### Option 3: Tiered with Feature Flags

- Core fields (11) always present
- Extended fields behind Rust feature flags
- Flexible for different client capabilities
- Incremental complexity
- **Recommended for production**

## ✅ Advantages

- **Strong Type Safety**: All fields are explicitly typed and validated at
  compile-time
- **No Runtime Parsing**: No need to parse jCal JSON at runtime
- **Clear Schema**: Every field is documented and typed
- **IDE Support**: Full autocomplete and type checking
- **Explicit Validation**: Field-level validation rules
- **RFC Compliance**: Direct mapping to RFC standards

## ❌ Disadvantages

- **Larger Type Definition**: More fields to define and maintain
- **Less Flexible**: Can't easily add custom fields without type changes
- **Update Overhead**: Adding new RFC fields requires type updates $ $- **More
  Implementation**: New Nexus handlers and indexing logic

## Relationship to Existing Types

**Compatible With:**

- `PubkyAppTags` - Can tag event-posts
- `PubkyAppBookmarks` - Can bookmark event-posts
- `PubkyAppFile` - Can attach files via URIs

**Separate From:**

- `PubkyAppPost` - Different type, different path
- Does NOT get post features (comments, reposts)

**Parent/Child Relationships:**

- Events: `parent_id` points to calendar event-post URI
- Attendees: `parent_id` points to event event-post URI

## Files in This Folder

**Version-Specific Files:**

- `pubky-ical-specification.md` - Complete specification with all three RFC
  field table options
- `nexus-endpoints.md` - Nexus API endpoints for event-posts
- `event-examples.md` - Example data showing explicit field structure

**Shared Architecture Files** (in root directory):

- `../diagrams.md` - Architecture diagrams (shared between versions)
- `../prototype-implementation-scope.md` - Frontend implementation scope
  (shared)
- `../context-references.md` - External RFC references (shared)
- `../notes.md` - General development notes (shared)

## Comparison with Other Approaches

| Aspect               | V1: Separate Types | V2: Post Kinds      | V3: Explicit Fields (This) |
| -------------------- | ------------------ | ------------------- | -------------------------- |
| **Type Definition**  | Multiple types     | Single PubkyAppPost | Single PubkyAppEventPost   |
| **Content Format**   | jCal JSON          | jCal JSON           | Explicit fields            |
| **Type Safety**      | Compile-time ✅    | Runtime ⚠️          | Compile-time ✅            |
| **Field Visibility** | Hidden in jCal     | Hidden in jCal      | Explicit ✅                |
| **Post Integration** | No                 | Yes ✅              | No                         |
| **Implementation**   | Complex            | Simple              | Medium                     |
| **Extensibility**    | Medium             | High                | Low                        |

## Recommendation

**For Prototype:** Start with **Option 2 (MVP Subset)** of ~25 fields

- Fastest to implement
- Covers common use cases
- Can extend to Option 3 (Tiered) later

**For Discussion:** Consider if explicit fields vs jCal content (V2) is worth
the tradeoff:

- **Explicit fields (V3)**: Better type safety, clearer schema
- **jCal content (V2)**: Simpler implementation, better extensibility, post
  integration

---

**For comparison with alternative approaches, see**:

- `../v1-separate-types/README.md` - Multiple dedicated types
- `../v2-post-kinds/README.md` - Post with jCal content
