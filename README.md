# Pubky iCalendar Integration Specification

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

## 📋 Document Structure

### Core Specification Files

| File                                                                         | Purpose                         | Contents                                                             |
| ---------------------------------------------------------------------------- | ------------------------------- | -------------------------------------------------------------------- |
| **[pubky-iCal-specification.md](./pubky-ical-specification.md)**             | Pure technical specification    | Data schemas, storage paths, RFC mappings                            |
| **[nexus-endpoints.md](./nexus-endpoints.md)**                               | Outline minimal Nexus endpoints | Data schemas, storage paths, RFC mappings                            |
| **[prototype-implementation-scope.md](./prototype-implementation-scope.md)** | Initial PoC Prototype Scope     | Basic Screens, supported functionalities, what isn't supported, etc. |

### Supporting Documents

| File                                               | Purpose                                           | Contents                                                                                                                        |
| -------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **[context-reference.md](./context-reference.md)** | Context and external References used as Resources | Breakdown of RFC Calendar standards and URLs to external resources                                                              |
| **[diagrams.md](./diagrams.md)**                   | Collection of Diagrams for System overview        | Data flow and System Overview diagramms                                                                                         |
| **[event-examples.md](./event-examples.md)**       | Example Schemas                                   | Break down different event-types, calendars and such and provide specific examples of how Homeserver documents should look like |
