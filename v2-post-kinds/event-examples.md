# Event Schema Examples (v2.0 - Post-Based with jCal Admins)

## Calendar Post Example

**Path**: `pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vcalendar\",[[\"prodid\",{},\"text\",\"-//Pubky//Pubky Calendar 1.0//EN\"],[\"version\",{},\"text\",\"2.0\"],[\"calscale\",{},\"text\",\"GREGORIAN\"],[\"method\",{},\"text\",\"PUBLISH\"],[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG\"],[\"name\",{},\"text\",\"Dezentralschweiz Meetups\"],[\"description\",{},\"text\",\"Bitcoin meetups and conferences across Switzerland\"],[\"color\",{},\"text\",\"#F7931A\"],[\"categories\",{},\"text\",[\"bitcoin\", \"meetups\", \"decentralization\"]],[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://adam-back\"]],[]]",
  "kind": "calendar",
  "parent": null,
  "embed": null,
  "attachments": ["pubky://satoshi/pub/pubky.app/files/0033RCZXVEPNG"]
}
```

**Parsed jCal Content** (for clarity):

```json
[
  "vcalendar",
  [
    ["prodid", {}, "text", "-//Pubky//Pubky Calendar 1.0//EN"],
    ["version", {}, "text", "2.0"],
    ["calscale", {}, "text", "GREGORIAN"],
    ["method", {}, "text", "PUBLISH"],
    ["uid", {}, "text", "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG"],
    ["name", {}, "text", "Dezentralschweiz Meetups"],
    [
      "description",
      {},
      "text",
      "Bitcoin meetups and conferences across Switzerland"
    ],
    ["color", {}, "text", "#F7931A"],
    ["categories", {}, "text", ["bitcoin", "meetups", "decentralization"]],
    ["x-pubky-admin", {}, "uri", "pubky://hal"],
    ["x-pubky-admin", {}, "uri", "pubky://adam-back"]
  ],
  []
]
```

**Admin Management**:

- Calendar owner: `satoshi` (post author)
- Admins: `hal` and `adam-back` (from `X-PUBKY-ADMIN` properties)
- Both owner and admins can create events for this calendar

**Indexed Neo4j Relationships**:

```
(satoshi:User)-[:AUTHORED]->(calendar:Post)
(hal:User)-[:ADMIN_OF]->(calendar:Post)
(adam-back:User)-[:ADMIN_OF]->(calendar:Post)
```

---

## Event Post by Calendar Owner

**Path**: `pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-01T12:00:00Z\"],[\"dtstart\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-09T19:00:00\"],[\"dtend\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-09T22:00:00\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Zürich\"],[\"description\",{},\"text\",\"Weekly Bitcoin meetup at Insider Bar\"],[\"location\",{},\"text\",\"Insider Bar, Zürich\"],[\"organizer\",{\"cn\":\"Satoshi\"},\"cal-address\",\"pubky://satoshi\"],[\"status\",{},\"text\",\"CONFIRMED\"]],[]]",
  "kind": "event",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": []
}
```

**Validation**: Event author `satoshi` is the calendar owner → ✅ Valid

---

## Event Post by Calendar Admin

**Path**: `pubky://hal/pub/pubky.app/posts/0033TD0XVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://hal/pub/pubky.app/posts/0033TD0XVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-02T10:00:00Z\"],[\"dtstart\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-11T18:30:00\"],[\"dtend\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-11T21:00:00\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Bern\"],[\"description\",{},\"text\",\"Monthly Bitcoin meetup in Bern\"],[\"location\",{},\"text\",\"Café des Pyrénées, Bern\"],[\"organizer\",{\"cn\":\"Hal\"},\"cal-address\",\"pubky://hal\"],[\"status\",{},\"text\",\"CONFIRMED\"]],[]]",
  "kind": "event",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": []
}
```

**Validation**: Event author `hal` is in calendar's `X-PUBKY-ADMIN` list → ✅
Valid

---

## Conference Event Post with Speakers

**Path**: `pubky://planb_lugano/pub/pubky.app/posts/0033TD0XVEPNG`

**Calendar**: `pubky://planb_lugano/pub/pubky.app/posts/0033RCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://planb_lugano/pub/pubky.app/posts/0033TD0XVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-01T09:00:00Z\"],[\"dtstart\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-24T09:00:00\"],[\"dtend\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-25T18:00:00\"],[\"summary\",{},\"text\",\"Plan ₿ Forum Lugano 2025\"],[\"description\",{},\"text\",\"The premier Bitcoin conference in Europe\"],[\"location\",{},\"text\",\"Palazzo dei Congressi, Lugano, Switzerland\"],[\"geo\",{},\"float\",[46.003677,8.951052]],[\"organizer\",{\"cn\":\"Plan ₿ Lugano\"},\"cal-address\",\"pubky://planb_lugano\"],[\"categories\",{},\"text\",[\"bitcoin\", \"conference\", \"adoption\"]],[\"status\",{},\"text\",\"CONFIRMED\"]],[[\"participant\",[[\"uid\",{},\"text\",\"pubky://planb_lugano/pub/pubky.app/posts/0033ZKQ8VEPNG\"],[\"participant-type\",{\"value\":\"SPEAKER\"},\"text\",\"SPEAKER\"],[\"calendar-address\",{\"cn\":\"Samson Mow\"},\"cal-address\",\"pubky://samson_mow\"],[\"summary\",{},\"text\",\"Bitcoin Nation State Adoption\"]],[]]]]]",
  "kind": "event",
  "parent": "pubky://planb_lugano/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": ["pubky://planb_lugano/pub/pubky.app/files/0033EVENT01"]
}
```

---

## Recurring Event Post

**Path**: `pubky://dezentralschweiz/pub/pubky.app/posts/0033SCZXVEPNG`

**Calendar**: `pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://dezentralschweiz/pub/pubky.app/posts/0033SCZXVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-01T12:00:00Z\"],[\"dtstart\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-02T19:00:00\"],[\"dtend\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-10-02T22:00:00\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Zürich (Weekly)\"],[\"description\",{},\"text\",\"Weekly Bitcoin meetup in Zürich\"],[\"location\",{},\"text\",\"Insider Bar, Zürich\"],[\"rrule\",{},\"recur\",{\"freq\":\"WEEKLY\",\"interval\":1,\"byday\":[\"WE\"],\"until\":\"2026-03-31T21:59:59Z\"}],[\"organizer\",{\"cn\":\"Dezentralschweiz\"},\"cal-address\",\"pubky://dezentralschweiz\"],[\"status\",{},\"text\",\"CONFIRMED\"]],[]]",
  "kind": "event",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": []
}
```

**Note**: Author `dezentralschweiz` must be either calendar owner or in
`X-PUBKY-ADMIN` list

---

## Event Override Post (Recurrence Exception)

**Path**: `pubky://dezentralschweiz/pub/pubky.app/posts/0033TCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://dezentralschweiz/pub/pubky.app/posts/0033SCZXVEPNG\"],[\"recurrence-id\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-11-20T19:00:00\"],[\"dtstamp\",{},\"date-time\",\"2025-11-15T10:00:00Z\"],[\"dtstart\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-11-20T18:30:00\"],[\"dtend\",{\"tzid\":\"Europe/Zurich\"},\"date-time\",\"2025-11-20T21:30:00\"],[\"summary\",{},\"text\",\"Bitcoin Meetup Zürich - Special Location\"],[\"description\",{},\"text\",\"This week at Colab Zürich for Lightning workshop\"],[\"location\",{},\"text\",\"Colab Zürich\"],[\"organizer\",{\"cn\":\"Dezentralschweiz\"},\"cal-address\",\"pubky://dezentralschweiz\"],[\"status\",{},\"text\",\"CONFIRMED\"]],[]]",
  "kind": "event",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": []
}
```

**Note**: Override events share the same UID as the master recurring event, but
have a `recurrence-id` property identifying which instance is being modified.

---

## Attendee Post (RSVP)

**Path**: `pubky://alice/pub/pubky.app/posts/0033ZCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"attendee\",[[\"uid\",{},\"text\",\"pubky://alice/pub/pubky.app/posts/0033ZCZXVEPNG\"],[\"dtstamp\",{},\"date-time\",\"2025-10-02T09:30:00Z\"],[\"attendee\",{\"cn\":\"Alice\",\"role\":\"REQ-PARTICIPANT\",\"partstat\":\"ACCEPTED\",\"rsvp\":\"TRUE\"},\"cal-address\",\"pubky://alice\"]],[]]",
  "kind": "attendee",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "embed": null,
  "attachments": null
}
```

---

## Alarm Post (User Reminder)

**Path**: `pubky://bob/pub/pubky.app/posts/0033WCZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"valarm\",[[\"uid\",{},\"text\",\"pubky://bob/pub/pubky.app/posts/0033WCZXVEPNG\"],[\"action\",{},\"text\",\"DISPLAY\"],[\"trigger\",{\"related\":\"START\"},\"duration\",\"-PT15M\"],[\"description\",{},\"text\",\"Bitcoin Meetup in 15 minutes\"]],[]]",
  "kind": "alarm",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "embed": null,
  "attachments": null
}
```

---

## Comment on Event (Regular Post Reply)

**Path**: `pubky://charlie/pub/pubky.app/posts/0033XCZXVEPNG`

**Post Structure**:

```json
{
  "content": "Looking forward to this meetup! Will there be pizza? 🍕",
  "kind": "short",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "embed": null,
  "attachments": null
}
```

**Note**: Comments on events are regular post replies, leveraging existing Nexus
post threading.

---

## Tag on Calendar (Regular Semantic Tag)

**Path**: `pubky://dave/pub/pubky.app/tags/J1Q2M4OPVW9`

**Tag Structure**:

```json
{
  "uri": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "label": "bitcoin",
  "created_at": 1727785400000000
}
```

**Note**: Regular tags on calendars/events work the same as tags on any other
post. This is for semantic tagging (e.g., #bitcoin), NOT for admin
relationships.

---

## Bookmark on Event (Regular Bookmark)

**Path**: `pubky://eve/pub/pubky.app/bookmarks/K2R3N5PQXYA`

**Bookmark Structure**:

```json
{
  "uri": "pubky://satoshi/pub/pubky.app/posts/0033SCZXVEPNG",
  "created_at": 1727785500000000
}
```

**Note**: Bookmarking events works identically to bookmarking any other post.

---

## Calendar with Multiple Admins Example

**Path**: `pubky://bitcoin-meetups/pub/pubky.app/posts/0033YYZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vcalendar\",[[\"prodid\",{},\"text\",\"-//Pubky//Pubky Calendar 1.0//EN\"],[\"version\",{},\"text\",\"2.0\"],[\"calscale\",{},\"text\",\"GREGORIAN\"],[\"method\",{},\"text\",\"PUBLISH\"],[\"uid\",{},\"text\",\"pubky://bitcoin-meetups/pub/pubky.app/posts/0033YYZXVEPNG\"],[\"name\",{},\"text\",\"Global Bitcoin Meetups\"],[\"description\",{},\"text\",\"Worldwide Bitcoin community events\"],[\"color\",{},\"text\",\"#F7931A\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://satoshi\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://hal\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://adam-back\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://jameson-lopp\"],[\"x-pubky-admin\",{},\"uri\",\"pubky://peter-todd\"]],[]]",
  "kind": "calendar",
  "parent": null,
  "embed": null,
  "attachments": []
}
```

**Admin List**:

- Owner: `bitcoin-meetups`
- Admins: `satoshi`, `hal`, `adam-back`, `jameson-lopp`, `peter-todd`

All 6 users (owner + 5 admins) can create events for this calendar.

---

## Invalid Event Example (Non-Admin)

**Path**: `pubky://random-user/pub/pubky.app/posts/0033ZZXVEPNG`

**Post Structure**:

```json
{
  "content": "[\"vevent\",[[\"uid\",{},\"text\",\"pubky://random-user/pub/pubky.app/posts/0033ZZXVEPNG\"],[\"summary\",{},\"text\",\"Fake Event\"],...],[]]",
  "kind": "event",
  "parent": "pubky://satoshi/pub/pubky.app/posts/0033RCZXVEPNG",
  "embed": null,
  "attachments": []
}
```

**Validation**: Event author `random-user` is NOT calendar owner AND NOT in
`X-PUBKY-ADMIN` list → ❌ **Rejected by Nexus**

**Error**: `"User is not admin or owner of parent calendar"`
