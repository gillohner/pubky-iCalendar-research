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
  - Should we treat events separately for this? => Design?
  - More complex
  - Getting rid of recurring (i.e. Homeserver documents are just generated for
    each)
    - Not a fan, as experience has shown that regular meetup hosts forget to add
      Meetups if they're every week/month, ... (Example: Bern, Thun, Zurich,
      Ostschweiz Meetups)
    - What if an instance of an event is overridden? Do tags/comments then
      change?
  - Prototype will keep it simple and just aggregate it as a single event, I
    think.
