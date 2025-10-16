# Pubky iCalendar Integration Specification

## 🔀 Three Approaches Available

This repository documents **three different approaches** for implementing
calendar functionality in Pubky:

### **[V1: Separate Types](v1-separate-types/README.md)**

Uses dedicated `PubkyAppCalendar`, `PubkyAppEvent`, `PubkyAppAttendee` types
with separate storage paths. Provides strong type safety but requires more
implementation work.

### **[V2: Post Kinds](v2-post-kinds/README.md)**

Extends `PubkyAppPost` with new `kind` enum values (`calendar`, `event`,
`attendee`). Content is jCal JSON. Leverages existing post infrastructure for
faster implementation and better integration.

### **[V3: Explicit RFC Fields](v3-explicit-fields/README.md)** ⭐ NEW

Single `PubkyAppEventPost` type with **explicit typed fields** from RFC
standards (not jCal JSON). Provides compile-time type safety with clear field
definitions. Includes three implementation options: Complete (~70 fields), MVP
(~25 fields), or Tiered with feature flags.

## 📊 Detailed Approach Comparison

For a comprehensive side-by-side comparison of all three approaches:

**See**: [APPROACH_COMPARISON.md](APPROACH_COMPARISON.md) ⭐

```text
For the prototype implementation a short discussion with someone from the Pubky
team on which approach to take and what other approaches could be taken (not all
potential approaches are documented here but the most important decision is on if PubkyAppPostKind should be used or new types should be created) would be helpful for making a decision. Currently I believe a mix between V1 and V3 where we define explicit fields in pubky-app-specs together with seperate types for RSVP and future Alarm extensions would be the best approach.
```

---

## Overview

A specification for integrating industry-standard iCalendar protocols (RFC 5545,
RFC 7265, RFC 4791, RFC 5546, RFC 7986, RFC 9073) into the Pubky ecosystem,
enabling decentralized calendar functionality while maintaining compatibility
with existing CalDAV clients and calendar standards.

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

### Version-Specific Folders

| Folder                                                | Description                             | Status                    |
| ----------------------------------------------------- | --------------------------------------- | ------------------------- |
| **[v1-separate-types/](v1-separate-types/)**          | Multiple dedicated types (jCal content) | Complete                  |
| **[v2-post-kinds/](v2-post-kinds/)**                  | PubkyAppPost with kinds (jCal content)  | Complete + Latest updates |
| **[v3-explicit-fields/](v3-explicit-fields/)** ⭐ NEW | Single type with explicit RFC fields    | Complete + RFC tables     |

**Each version folder contains:**

| File                          | Description                                               |
| ----------------------------- | --------------------------------------------------------- |
| `pubky-ical-specification.md` | Core technical specification (different for each version) |
| `nexus-endpoints.md`          | Nexus API endpoints (different for each version)          |
| `event-examples.md`           | Example calendar data (different structure per version)   |

### Shared Architecture Files (Root Level)

These files are shared between both versions as they describe general
architecture and frontend scope:

| File                                                                       | Description                                    |
| -------------------------------------------------------------------------- | ---------------------------------------------- |
| **[diagrams.md](diagrams.md)**                                             | System architecture diagrams                   |
| **[prototype-implementation-scope.md](prototype-implementation-scope.md)** | Frontend implementation scope and roadmap      |
| **[context-references.md](context-references.md)**                         | RFC calendar standards and external references |
| **[notes.md](notes.md)**                                                   | Development notes and considerations           |
