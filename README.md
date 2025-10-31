# Pubky iCalendar Integration Specification

## 🔀 Two Approaches Available

This repository now documents **two different approaches** for implementing
calendar functionality in Pubky:

### **[V1: Separate Types (jCal Storage)](v1-separate-types/README.md)**

Uses dedicated `PubkyAppCalendar`, `PubkyAppEvent`, `PubkyAppAttendee`,
`PubkyAppAlarm` types with separate storage paths. Stores all calendar data as
jCal JSON in a `content` field. Provides maximum flexibility for RFC extensions.

### **[V2: Explicit Fields (Typed Storage)](v2-explicit-fields/README.md)**

Uses dedicated `PubkyAppCalendar`, `PubkyAppEvent`, `PubkyAppAttendee`,
`PubkyAppAlarm` types with **explicit typed fields** from RFC standards (not
jCal JSON). Provides compile-time type safety with clear field definitions using
MVP subset (~25 fields) that can be extended with new fields as needed.

```text
For the prototype implementation, V2 (Explicit Fields) is currently favored by me because it provides
better developer experience with compile-time type safety, clearer schema definitions,
and better performance without runtime JSON parsing. Thus I recommend looking at V2 (Explicit Fields) for the prototype implementation.
```

## Overview

A specification for integrating industry-standard iCalendar protocols (RFC 5545,
RFC 7265, RFC 4791, RFC 5546, RFC 7986, RFC 9073) into the Pubky ecosystem,
enabling decentralized calendar functionality while maintaining possible
compatibility with existing CalDAV clients and calendar standards.

## Background

This project researches how events and calendars can be implemented on Pubky.
The existing iCalendar standards and their extensions provide a robust
foundation with comprehensive event management features. These standards have
been extensively tested in real-world scenarios across diverse calendar
applications, making them an ideal basis for a decentralized calendar system
with simple extensions.

By building upon established RFC specifications, we can potentially enable
seamless integration with existing calendar workflows while benefiting from
Pubky's decentralization. This research analyzes existing iCalendar standards,
documents their applicability to the Pubky ecosystem, and identifies which
specifications are most suitable for implementation—with consideration for
future protocol expansions.

The documentation includes mentions of a CalDAV bridge to implement Pubky-based
calendar management in existing calendar applications like Outlook or
Thunderbird. For the prototype phase, there is no plan to actually implement
this. I decided to document my ideas of how it could look, as the core structure
is designed with such a bridge in mind. Keeping this in mind for later stages
could enable deeper integration into existing workflows, as mentioned before.

## 📂 Repository Structure

### Variant-Specific Folders

| Folder                                         | Description                      |
| ---------------------------------------------- | -------------------------------- |
| **[v1-separate-types/](v1-separate-types/)**   | Separate types with jCal storage |
| **[v2-explicit-fields/](v2-explicit-fields/)** | explicit fields                  |

**Each variant folder contains:**

| File                          | Description                                               |
| ----------------------------- | --------------------------------------------------------- |
| `pubky-ical-specification.md` | Core technical specification (different for each variant) |
| `nexus-endpoints.md`          | Nexus API endpoints (different for each variant)          |
| `event-examples.md`           | Example calendar data (different structure per variant)   |
| `README.md`                   | Variant-specific overview and approach description        |

### Shared Architecture Files (Root Level)

These files are shared between both variants as they describe general
architecture and frontend scope:

| File                                                                       | Description                                    |
| -------------------------------------------------------------------------- | ---------------------------------------------- |
| **[diagrams.md](diagrams.md)**                                             | System architecture diagrams                   |
| **[prototype-implementation-scope.md](prototype-implementation-scope.md)** | Frontend implementation scope and roadmap      |
| **[context-references.md](context-references.md)**                         | RFC calendar standards and external references |
| **[notes.md](notes.md)**                                                   | Development notes and considerations           |

## Development Github Repositories

https://github.com/gillohner/pubky-app-specs/tree/feat/calendar-events
https://github.com/gillohner/pubky-nexus/tree/feat/calendar-events
https://github.com/gillohner/pubky-ical-poc
