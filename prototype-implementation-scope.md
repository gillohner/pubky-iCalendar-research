# Prototype Implementation

## Executive Summary

The Goal is to implement a first Proof of Concept application based on the
outlined specification. The main objective is to implement the minimal Nexus
Endpoints and have a basic frontend display the aggregated feeds to be able to
create public Events like Bitcoin Meetups using the "iCal on Pubky" standard.

This should serve as a base to extend Events/Calendars to implement more
functionalities at a later stage. This can include but is not limited to a
CalDAV bridge, extend to have encypted private Calendars, support for _all_ RFC
extensions, ...

## Notes

- Exact feature set taken from iCal standards is defined in other documents and
  may change throughout prototype development.

## Event/Calendar Lookup Features

- Design will be focused on a card-based feature layout across all screens. This
  allows the UI to be flexible in displaying optional fields.

### Calendar Discovery Page:

- Find different community calendars
- Allow filtering based on calandar metadata and Pubky Tags

### Event Discovery Page:

- Overview of Events happening.
- Allow filtering based on event metadata and Pubky Tags (Like in Calendar
  Overview)
- Use same component as Calendar Overview for list/calendar view => Event colors
  based on calendar metadata

### Calendar Overview:

Have a list/calendar view. Similar to how meetup.com does it:

![List/Calendar meetup.com](images/screenshots/meetupcom/list_calendar_toggle.png)

![List/Calendar meetup.com](images/screenshots/meetupcom/calendar_overview.png "")

- Recurring Events are rendered for each date inside of the list.
- Calendar Metadata is displayed at the top

### Event Overview:

- Page to view a single Event
- Location OSM Map integration => linking to Apple Maps, Google Maps, OSM
  - Show Bitcoin Payment tags
  - Example here:
    https://meetstr.com/event/naddr1qvzqqqrukvpzpzd4ye7z7x886as20jpf8xw46rdfywmg04f75f4xl566wu8entspqqy9yj63xyekx3ryaylwxs

    ![Location Card Meetstr](images/screenshots/meetstr/location_card.png)
- Comments, Tags using PubkyAppPost and PubkyAppTags

### Lower priority features

- Overview of own Calendars/Events
- Overview of own Attendance and history

### Main Components

```
Note: Will be extended further and divided into sub-components during Development.
```

| Name                     | Description                                                                                                                                                                                             |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Calendar/Event Header    | Header for a Calendar with Image, Summary, Host, Admins, Categories ... - Subcomponents are dinamically rendered based on if it's a calendar/event and what Metadata is present                         |
| Event List               | Card based list to display Events. (Supports single calendar or multiple as input). Clicking on an Event will open the Event detail page                                                                |
| Event Calendar           | Calendar-Style display of Events. (Supports single calendar or multiple as input). Clicking on an Event will open the Event detail page                                                                 |
| Location display         | Card displaying a text-based location                                                                                                                                                                   |
| OSM Location display     | Card displaying a structured location based from OSM (Nominatim API) - Displays payment possibility in Bitcoin                                                                                          |
| iCal Metadata components | Simple Card Components for other Metadata optionalities in iCal format                                                                                                                                  |
| Pubky Tags               | Show Tags like Pubky.app                                                                                                                                                                                |
| Attendees                | Component rendering attendees of an Event with their respective status => Invitng attendees to an Event will not be implemented yet in Event creation. Could be added with exisitng tags in the future. |
