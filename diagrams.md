## Architecture Overview

### System Overview

```mermaid
graph TB
    subgraph "User Layer"
        U1[Satoshi - Event Creator]
        U2[Hal Finney - Attendee]
        U3[Nick Szabo - Calendar Owner]
    end
    
    subgraph "Client Layer"
        CC[Calendar Clients<br/>CalDAV/Native]
        WC[Web Clients]
        MC[Mobile Apps]
    end
    
    subgraph "Bridge Layer"
        CB[CalDAV/WebDAV Bridge<br/>jCal ↔ .ics]
    end
    
    subgraph "Pubky Infrastructure"
        HS1[Satoshi's Homeserver]
        HS2[Hal Finney's Homeserver]
        HS3[Nick Szabo's Homeserver]
        NI[Nexus Indexer]
        DB[Neo4j Database]
        CF[Calendar Feeds API]
    end
    
    U1 --> HS1
    U2 --> HS2
    U3 --> HS3
    
    CC --> CB
    WC --> CF
    MC --> CF
    
    CB --> CF
    HS1 --> NI
    HS2 --> NI
    HS3 --> NI
    NI --> DB
    NI --> CF
```

### Data Flow for a CalDAV Bridge

```
Note:
Below Diagram is an example outlining the flow for a potential CalDAV client. I currently
don't have a plan to implement this during the prototyping phase. The idea is
that this could be created at a later stage based on the curent core
specification.
```

```mermaid
sequenceDiagram
    participant U as User
    participant HS as Homeserver
    participant N as Nexus Indexer
    participant DB as Neo4j Database
    participant CB as CalDAV Bridge
    participant C as Calendar Client
    
    U->>HS: 1. Store jCal JSON files
    HS->>N: 2. Watch & notify changes
    N->>DB: 3. Index calendar data
    N->>DB: 4. Generate feeds
    
    Note over C,CB: Access Patterns
    CB->>N: Fetch jCal data
    CB->>C: Translate to .ics format
```

1. **Storage**: Users store PubkyAppVCalendar, PubkyAppVEvent,
   PubkyAppVAttendee, PubkyAppVAlarm, ... files as jCal files on their
   homeservers at `/pub/pubky.app/...`
2. **Indexing**: Nexus watches homeserver paths, indexes calendar data into
   Neo4j graph database
3. **Aggregation**: Nexus generates feeds based on follows, tags, date ranges,
   and social signals, ... (Exact endpoints TBD)
4. **Access**: Clients query Nexus REST API for aggregated feeds.
5. **Bridging**: CalDAV bridge translates between jCal JSON and .ics format for
   legacy client compatibility and maps iTip functionality to Pubky Homeserver
   Documents.

---
