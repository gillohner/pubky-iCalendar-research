# Pubky iCalendar: Three Approaches Comparison

Quick reference for comparing the three documented approaches.

## 📊 At-a-Glance Comparison

| Aspect            | V1: Separate Types                                                             | V2: Post Kinds                                 | V3: Explicit Fields                                      |
| ----------------- | ------------------------------------------------------------------------------ | ---------------------------------------------- | -------------------------------------------------------- |
| **Architecture**  |                                                                                |                                                |                                                          |
| Type System       | `PubkyAppCalendar`<br>`PubkyAppEvent`<br>`PubkyAppAttendee`<br>`PubkyAppAlarm` | `PubkyAppPost` with<br>`PubkyAppPostKind` enum | `PubkyAppEventPost` with<br>`PubkyAppEventPostKind` enum |
| Storage Paths     | `/calendars/:id`<br>`/events/:id`<br>`/attendees/:id`<br>`/alarms/:id`         | `/posts/:id`<br>(all unified)                  | `/event-posts/:id`<br>(unified)                          |
| Content Format    | jCal JSON in `content`                                                         | jCal JSON in `content`                         | Explicit typed fields                                    |
|                   |                                                                                |                                                |                                                          |
| **Type Safety**   |                                                                                |                                                |                                                          |
| Field Validation  | Compile-time                                                                   | Runtime                                        | Compile-time                                             |
| Field Visibility  | Hidden in jCal                                                                 | Hidden in jCal                                 | Explicit structs                                         |
| Schema Clarity    | Medium                                                                         | Low                                            | High                                                     |
| IDE Support       | Medium                                                                         | Low                                            | High                                                     |
|                   |                                                                                |                                                |                                                          |
| **Extensibility** |                                                                                |                                                |                                                          |
| Add RFC Fields    | Update jCal                                                                    | Update jCal                                    | Update struct                                            |
| Custom Fields     | x-pubky-* props                                                                | x-pubky-* props                                | Struct extension                                         |
| Future RFCs       | Easy                                                                           | Easy                                           | Requires types update                                    |

The Explicit Fields from V3 can be used in combination with the approaches
outlined in V1 or V2 as well instead of storing the data as jCal JSON in a
`content` field.

## 📝 Code Examples

### V1: Separate Types (jCal content)

```rust
pub struct PubkyAppCalendar {
    pub content: String,  // jCal JSON
}

// Storage: /pub/pubky.app/calendars/:id
```

### V2: Post Kinds (jCal content)

```rust
pub struct PubkyAppPost {
    pub content: String,  // jCal JSON
    pub kind: PubkyAppPostKind,  // calendar | event | attendee
}

// Storage: /pub/pubky.app/posts/:id
```

### V3: Explicit Fields (typed)

```rust
pub struct PubkyAppEventPost {
    pub kind: PubkyAppEventPostKind,  // calendar | event | attendee
    
    // Calendar fields
    pub name: Option<String>,
    pub description: Option<String>,
    pub x_pubky_admins: Option<Vec<String>>,
    
    // Event fields  
    pub summary: Option<String>,
    pub dtstart: Option<i64>,  // microseconds
    pub dtend: Option<i64>,
    pub location: Option<String>,
    pub status: Option<String>,
    pub rrule: Option<String>,
    // ... ~25 more fields for MVP
}

// Storage: /pub/pubky.app/event-posts/:id
```

## 📚 Detailed Documentation (V3 is kept limited to README.md and pubky-ical-specification.md for simplicity)

- **V1**: [v1-separate-types/README.md](v1-separate-types/README.md)
- **V2**: [v2-post-kinds/README.md](v2-post-kinds/README.md) ⭐ Current
  recommendation
- **V3**: [v3-explicit-fields/README.md](v3-explicit-fields/README.md)

Each folder contains:

- Complete specification (`pubky-ical-specification.md`)
- Nexus API endpoints (`nexus-endpoints.md`)
- Example data (`event-examples.md`)
