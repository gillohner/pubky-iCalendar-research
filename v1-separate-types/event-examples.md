# Event Schema Examples

## Conference Event with Speakers (RFC 9073)

**Path**: `pubky://planb_lugano/pub/pubky.app/vevent/0033TD0XVEPNG`

```json
[
  "vevent",
  [
    [
      "uid",
      {},
      "text",
      "pubky://planb_lugano/pub/pubky.app/vevent/0033TD0XVEPNG"
    ],
    [
      "dtstamp",
      {},
      "date-time",
      "2025-10-01T09:00:00Z"
    ],
    [
      "dtstart",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-24T09:00:00"
    ],
    [
      "dtend",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-25T18:00:00"
    ],
    [
      "summary",
      {},
      "text",
      "Plan ₿ Forum Lugano 2025"
    ],
    [
      "description",
      {},
      "text",
      "The premier Bitcoin conference in Europe, bringing together Bitcoin builders, economists, and adopters. Two days of talks, workshops, and networking in the Bitcoin capital of Switzerland."
    ],
    [
      "location",
      {},
      "text",
      "Palazzo dei Congressi, Lugano, Switzerland"
    ],
    [
      "structured-location",
      {
        "name": "Palazzo dei Congressi",
        "osm": "way/987654321"
      },
      "text",
      "Via Ciseri 5, 6900 Lugano, Switzerland"
    ],
    [
      "geo",
      {},
      "float",
      [46.003677, 8.951052]
    ],
    [
      "url",
      {},
      "uri",
      "https://planb.lugano.ch"
    ],
    [
      "organizer",
      { "cn": "Plan ₿ Lugano" },
      "cal-address",
      "pubky://planb_lugano"
    ],
    [
      "categories",
      {},
      "text",
      "bitcoin",
      "conference",
      "adoption"
    ],
    [
      "status",
      {},
      "text",
      "CONFIRMED"
    ],
    [
      "sequence",
      {},
      "integer",
      0
    ],
    [
      "created",
      {},
      "date-time",
      "2025-09-15T10:00:00Z"
    ],
    [
      "last-modified",
      {},
      "date-time",
      "2025-10-01T09:00:00Z"
    ],
    [
      "color",
      {},
      "text",
      "#F7931A"
    ],
    [
      "image",
      {},
      "uri",
      "pubky://planb_lugano/pub/pubky.app/files/0033RCZXVEPNG"
    ],
    [
      "conference",
      { "feature": "AUDIO,VIDEO", "label": "Live Stream" },
      "uri",
      "https://livestream.planb.lugano.ch/2025"
    ],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://planb_lugano/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ]
  ],
  [
    [
      "participant",
      [
        [
          "uid",
          {},
          "text",
          "pubky://planb_lugano/pub/pubky.app/vevent/0033ZKQ8VEPNG"
        ],
        ["dtstamp", {}, "date-time", "2025-10-01T09:00:00Z"],
        [
          "participant-type",
          { "value": "SPEAKER" },
          "text",
          "SPEAKER"
        ],
        [
          "calendar-address",
          { "cn": "Samson Mow" },
          "cal-address",
          "pubky://samson_mow"
        ],
        [
          "summary",
          {},
          "text",
          "Bitcoin Nation State Adoption"
        ],
        [
          "description",
          {},
          "text",
          "CEO of Jan3, exploring how nation states are adopting Bitcoin as legal tender and strategic reserve asset."
        ],
        ["location", {}, "text", "Main Hall - Keynote Stage"],
        [
          "dtstart",
          { "tzid": "Europe/Zurich" },
          "date-time",
          "2025-10-24T10:00:00"
        ],
        [
          "dtend",
          { "tzid": "Europe/Zurich" },
          "date-time",
          "2025-10-24T11:00:00"
        ]
      ],
      []
    ],
    [
      "participant",
      [
        [
          "uid",
          {},
          "text",
          "pubky://planb_lugano/pub/pubky.app/vevent/0033ZKR2VEPNG"
        ],
        ["dtstamp", {}, "date-time", "2025-10-01T09:00:00Z"],
        [
          "participant-type",
          { "value": "SPEAKER" },
          "text",
          "SPEAKER"
        ],
        [
          "calendar-address",
          { "cn": "Jimmy Song" },
          "cal-address",
          "pubky://jimmy_song"
        ],
        [
          "summary",
          {},
          "text",
          "Bitcoin Programming Fundamentals"
        ],
        [
          "description",
          {},
          "text",
          "Author of 'Programming Bitcoin', teaching the technical foundations of Bitcoin development."
        ],
        ["location", {}, "text", "Technical Track - Room B"],
        [
          "dtstart",
          { "tzid": "Europe/Zurich" },
          "date-time",
          "2025-10-24T14:00:00"
        ],
        [
          "dtend",
          { "tzid": "Europe/Zurich" },
          "date-time",
          "2025-10-24T15:30:00"
        ]
      ],
      []
    ],
    [
      "participant",
      [
        [
          "uid",
          {},
          "text",
          "pubky://planb_lugano/pub/pubky.app/vevent/0033ZKS6VEPNG"
        ],
        ["dtstamp", {}, "date-time", "2025-10-01T09:00:00Z"],
        [
          "participant-type",
          { "value": "SPONSOR", "level": "PLATINUM" },
          "text",
          "SPONSOR"
        ],
        [
          "calendar-address",
          { "cn": "City of Lugano" },
          "cal-address",
          "pubky://lugano_city"
        ],
        [
          "summary",
          {},
          "text",
          "Platinum Sponsor - City of Lugano"
        ],
        [
          "description",
          {},
          "text",
          "Official host city, pioneering Bitcoin adoption in municipal services and the Plan ₿ initiative."
        ],
        ["url", {}, "uri", "https://lugano.ch/plan-b"]
      ],
      []
    ]
  ]
]
```

## Recurring Event Example (RFC 5545 Section 3.8.5)

**Path**: `pubky://dezentralschweiz/pub/pubky.app/vevent/0033SCZXVEPNG`

```json
[
  "vevent",
  [
    [
      "uid",
      {},
      "text",
      "pubky://dezentralschweiz/pub/pubky.app/vevent/0033SCZXVEPNG"
    ],
    ["dtstamp", {}, "date-time", "2025-10-01T12:00:00Z"],
    [
      "dtstart",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-02T19:00:00"
    ],
    [
      "dtend",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-10-02T22:00:00"
    ],
    ["summary", {}, "text", "Bitcoin Meetup Zürich (Weekly)"],
    [
      "description",
      {},
      "text",
      "Weekly Bitcoin meetup in Zürich. Join us every Wednesday for Bitcoin discussions, Lightning demos, and networking. Location: Insider Bar unless otherwise noted."
    ],
    ["location", {}, "text", "Insider Bar, Zürich"],
    [
      "structured-location",
      {
        "name": "Insider Bar",
        "osm": "way/123456789"
      },
      "text",
      "Seefeldstrasse 63, 8008 Zürich, Switzerland"
    ],
    ["geo", {}, "float", [47.366667, 8.550000]],
    ["rrule", {}, "recur", {
      "freq": "WEEKLY",
      "interval": 1,
      "byday": ["WE"],
      "until": "2026-03-31T21:59:59Z"
    }],
    [
      "exdate",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-12-25T19:00:00"
    ],
    [
      "organizer",
      { "cn": "Dezentralschweiz" },
      "cal-address",
      "pubky://dezentralschweiz"
    ],
    ["categories", {}, "text", "bitcoin", "meetup", "recurring"],
    ["status", {}, "text", "CONFIRMED"],
    ["sequence", {}, "integer", 0],
    ["created", {}, "date-time", "2025-09-20T16:00:00Z"],
    ["last-modified", {}, "date-time", "2025-10-01T12:00:00Z"],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://dezentralschweiz/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ]
  ],
  []
]
```

## Event Override Example (Exception to Recurring Event)

**Path**: `pubky://dezentralschweiz/pub/pubky.app/vevent/0033TCZXVEPNG`

```json
[
  "vevent",
  [
    [
      "uid",
      {},
      "text",
      "pubky://dezentralschweiz/pub/pubky.app/vevent/0033SCZXVEPNG"
    ],
    ["dtstamp", {}, "date-time", "2025-11-15T10:00:00Z"],
    [
      "recurrence-id",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-11-20T19:00:00"
    ],
    [
      "dtstart",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-11-20T18:30:00"
    ],
    [
      "dtend",
      { "tzid": "Europe/Zurich" },
      "date-time",
      "2025-11-20T21:30:00"
    ],
    ["summary", {}, "text", "Bitcoin Meetup Zürich - Special Location"],
    [
      "description",
      {},
      "text",
      "This week's Bitcoin meetup is at a special location! We'll be at Colab Zürich for a workshop on running Lightning nodes. Earlier start time: 18:30."
    ],
    ["location", {}, "text", "Colab Zürich"],
    [
      "structured-location",
      {
        "name": "Colab Zurich",
        "osm": "way/987654321"
      },
      "text",
      "Gerbergasse 5, 8001 Zürich, Switzerland"
    ],
    ["geo", {}, "float", [47.373878, 8.545094]],
    [
      "organizer",
      { "cn": "Dezentralschweiz" },
      "cal-address",
      "pubky://dezentralschweiz"
    ],
    ["categories", {}, "text", "bitcoin", "meetup", "workshop"],
    ["status", {}, "text", "CONFIRMED"],
    ["sequence", {}, "integer", 1],
    ["created", {}, "date-time", "2025-09-20T16:00:00Z"],
    ["last-modified", {}, "date-time", "2025-11-15T10:00:00Z"],
    [
      "x-pubky-calendar",
      {},
      "uri",
      "pubky://dezentralschweiz/pub/pubky.app/vcalendar/0033RCZXVEPNG"
    ]
  ],
  []
]
```

---
