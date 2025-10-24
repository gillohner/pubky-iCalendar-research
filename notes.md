# Just some unorganized notes to keep track of stuff for myself

- Should these Event and Calendar types just be a new type of PubkyAppPost
  instead of their own PubkyAppX?
  - JCal in Content or upload the ICS as PubkyAppFile instead of using JCal
    format?
- Meetup.com has a Community which has Events approach => Vcalendar could be
  extended to have "stuff" reference it and be a lot more like a Community than
  just a Calendar assortment. => Future idea and too complex for the MVP though

**Unsolved**

- How can we have multiple people edit the same Event?
  - Clients could do an edit request function where the owner can sign off or
    add admins that can override the event. When versioned, the admin saving the
    event would just update to the newest version. => Adds complexity and
    potential merge conflicts...
- Tagging, Comments, Bookmarks on recurring events?
  - **RSVP Solution (Solved)**: Using `recurrence-id` in VAttendee allows
    per-instance RSVPs
    - No `recurrence-id` = RSVP applies to all instances
    - With `recurrence-id` = RSVP for that specific occurrence
    - Users can create multiple VAttendee records for different instances with
      different statuses
  - **For Tags/Comments**: Similar approach could work
    - No `recurrence-id` = applies to the series
    - With `recurrence-id` = applies to specific instance
  - **Prototype approach**:
    - Implement `recurrence-id` in VAttendee for granular RSVPs
    - For tags/comments, start simple (apply to series), can extend later with
      `recurrence-id` if needed
