# Context and References

## Relevant RFC Standards Overview

### Core Calendar Standards

| RFC          | Title & Purpose                                                                                                                               | Key Properties/Components Used                                                                                                                                    | Application in Pubky Calendar                                                     |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **RFC 5545** | **Internet Calendaring and Scheduling Core Object Specification (iCalendar)**<br/>Defines the fundamental calendar data format and components | `VEVENT`, `VTODO`, `VALARM`, `RRULE`, `DTSTART`, `DTEND`, `SUMMARY`, `DESCRIPTION`, `LOCATION`, `ORGANIZER`, `ATTENDEE`, `CATEGORIES`, `STATUS`, `UID`, `DTSTAMP` | Core event structure, recurrence patterns, participant roles, basic metadata      |
| **RFC 7265** | **jCal: The JSON Format for iCalendar**<br/>JSON representation of iCalendar data for web applications                                        | JSON array format `[component_name, [properties], [sub_components]]`                                                                                              | Native storage format on homeservers, API responses, client data exchange         |
| **RFC 4791** | **Calendaring Extensions to WebDAV (CalDAV)**<br/>WebDAV-based protocol for calendar access and synchronization                               | `calendar-query`, `calendar-multiget`, `free-busy-query`, `sync-collection`, ACL model                                                                            | Calendar collection structure, access control patterns, sync semantics for bridge |

### Scheduling and Interoperability

| RFC          | Title & Purpose                                                                                                 | Key Properties/Components Used                                                                                                                                                                  | Application in Pubky Calendar                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **RFC 5546** | **iCalendar Transport-Independent Interoperability Protocol (iTIP)**<br/>Scheduling messages and RSVP workflows | `METHOD` values: `PUBLISH`, `REQUEST`, `REPLY`, `CANCEL`, `REFRESH`<br/>`PARTSTAT` values: `ACCEPTED`, `DECLINED`, `TENTATIVE`<br/>`ROLE` values: `CHAIR`, `REQ-PARTICIPANT`, `OPT-PARTICIPANT` | RSVP flows, event invitations, status updates, organizer-attendee communication |
| **RFC 6638** | **Scheduling Extensions to CalDAV**<br/>Server-side scheduling automation                                       | `schedule-inbox`, `schedule-outbox`, `calendar-auto-schedule`, `schedule-calendar-transp`                                                                                                       | Future: automated scheduling, free/busy support, delegation workflows           |

### Enhanced Properties and Features

| RFC          | Title & Purpose                                                                                              | Key Properties/Components Used                                                                       | Application in Pubky Calendar                                                              |
| ------------ | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **RFC 7986** | **New Properties for iCalendar**<br/>Modern calendar features for web/mobile clients                         | `NAME`, `DESCRIPTION` (calendar-level), `COLOR`, `IMAGE`, `CONFERENCE`, `SOURCE`, `REFRESH-INTERVAL` | Visual calendar customization, conference integration, calendar metadata, refresh policies |
| **RFC 9073** | **Event Publishing Extensions to iCalendar**<br/>Enhanced event publishing for conferences and public events | `PARTICIPANT` component, `STRUCTURED-LOCATION`, `STYLED-DESCRIPTION`, `PARTICIPANT-TYPE` parameters  | Conference speakers/sponsors, OpenStreetMap location integration, rich event descriptions  |

### Supporting Standards

| RFC          | Title & Purpose                                                                                             | Key Properties/Components Used                                     | Application in Pubky Calendar                                         |
| ------------ | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------- |
| **RFC 6868** | **Parameter Value Encoding in iCalendar and vCard**<br/>Encoding for special characters in parameter values | `^n`, `^'`, `^^` escape sequences for newlines, quotes, and carets | Robust encoding of organizer names, locations with special characters |
| **RFC 7529** | **Non-Gregorian Recurrence Rules in iCalendar**<br/>Support for non-Gregorian calendar systems              | `RSCALE` parameter for recurrence rules                            | Future: multi-calendar system support                                 |
| **RFC 9074** | **VALARM Extensions for iCalendar**<br/>Enhanced alarm and notification features                            | `PROXIMITY` alarm trigger, `RELATED-TO` for alarm relationships    | Enhanced notification system, location-based reminders                |

## External References

### RFC Standards

- [RFC 5545 - Internet Calendaring and Scheduling Core Object Specification (iCalendar)](https://www.rfc-editor.org/rfc/rfc5545.html)
- [RFC 7265 - jCal: The JSON Format for iCalendar](https://www.rfc-editor.org/rfc/rfc7265.html)
- [RFC 4791 - Calendaring Extensions to WebDAV (CalDAV)](https://www.rfc-editor.org/rfc/rfc4791.html)
- [RFC 5546 - iCalendar Transport-Independent Interoperability Protocol (iTIP)](https://www.rfc-editor.org/rfc/rfc5546.html)
- [RFC 6638 - Scheduling Extensions to CalDAV](https://www.rfc-editor.org/rfc/rfc6638.html)
- [RFC 7986 - New Properties for iCalendar](https://www.rfc-editor.org/rfc/rfc7986.html)
- [RFC 9073 - Event Publishing Extensions to iCalendar](https://www.rfc-editor.org/rfc/rfc9073.html)
- [RFC 6868 - Parameter Value Encoding in iCalendar and vCard](https://www.rfc-editor.org/rfc/rfc6868.html)
- [RFC 7529 - Non-Gregorian Recurrence Rules in iCalendar](https://www.rfc-editor.org/rfc/rfc7529.html)
- [RFC 7953 - Calendar Availability](https://www.rfc-editor.org/rfc/rfc7953.html)
- [RFC 9074 - VALARM Extensions for iCalendar](https://www.rfc-editor.org/rfc/rfc9074.html)
- [RFC 9253 - Support for iCalendar Relationships](https://www.rfc-editor.org/rfc/rfc9253.html)

### Pubky Resources

- [Pubky Core Documentation](https://docs.pubky.org/)
- [Pubky-App Specs Repository](https://github.com/pubky/pubky-app-specs)
- [Pubky-Nexus Repository](https://github.com/pubky/pubky-nexus)
- [Pubky Homeserver Specification](https://docs.pubky.org/Explore/Pubky-App/App-Architectures/2.-Client---Homeserver)
- [Nexus Aggregator Architecture](https://docs.pubky.org/Explore/Pubky-App/App-Architectures/3.-Global-Aggregators)

### Related Resources

- [CalConnect: The Calendaring and Scheduling Consortium](https://www.calconnect.org/)
- [iCalendar.org Resources](https://icalendar.org/)
- [CalDAV Developer's Guide](https://devguide.calconnect.org/)
