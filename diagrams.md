# Diagrams

## Full Prototype Architecture Overview

```mermaid
graph TB
    subgraph "User Layer"
        U1[Event Creator]
        U2[Event Attendee]
        U3[Calendar Admin]
    end

    subgraph "Client Application - Next.js"
        subgraph "Authentication"
            AUTH["@synonymdev/pubky SDK"]
            QR[QR Code Login]
            RING[Pubky Ring Integration]
        end
        
        subgraph "Discovery Pages"
            ED["Event Discovery Page\nList/Calendar Views"]
            CD["Calendar Discovery Page\nList View"]
            FILTER["Filter Components\nTags/Location/Date"]
        end
        
        subgraph "Overview Pages"
            EO["Event Overview Page\nDetails/Map/Comments"]
            CO["Calendar Overview Page\nList/Calendar Views"]
        end
        
        subgraph "Creation & Edit"
            ECM["Event Creation Modal\nForm + Image Upload"]
            CCM["Calendar Creation Modal\nForm + Image Upload"]
        end
        
        subgraph "UI Components - Tailwind CSS"
            HEADER[Calendar/Event Header]
            ELIST[Event List Cards]
            ECAL[Event Calendar View]
            LOC[Location Display]
            OSM[OSM Map Integration]
            TAGS[Pubky Tags]
            ATT[Attendees Component]
        end
        
        API[Nexus API Client]
    end

    subgraph "Pubky Infrastructure"
        subgraph "Extended Nexus Indexer"
            NI[pubky-nexus<br/>Calendar Extensions]
            
            subgraph "New Calendar Endpoints"
                EP1[GET /vcalendars]
                EP2[GET /vcalendars/:id]
                EP3[GET /vevents]
                EP4[GET /vevents/:id]
                EP5[GET /vcalendars/:id/vevents]
                EP6[GET /vattendees]
            end
            
            DB[(Neo4j Database)]
        end
        
        subgraph "Homeservers"
            HS1[User 1 Homeserver<br/>pubky://user1z123...]
            HS2[User 2 Homeserver<br/>pubky://user2x456...]
            HS3[User 3 Homeserver<br/>pubky://user3y789...]
        end
        
        subgraph "Storage Structure"
            CAL[/pub/pubky.app/vcalendars/]
            EVT[/pub/pubky.app/vevents/]
            ATD[/pub/pubky.app/vattendees/]
            FILES[/pub/pubky.app/files/]
        end
    end

    subgraph "External Services"
        OSMAPI[Nominatim OSM API<br/>Location Geocoding]
    end

    %% User Authentication Flow
    U1 --> QR
    U2 --> QR
    U3 --> QR
    QR --> AUTH
    AUTH --> RING
    
    %% Discovery Flow
    U1 --> ED
    U1 --> CD
    U2 --> ED
    U3 --> CD
    ED --> FILTER
    CD --> FILTER
    
    %% Overview Flow
    ED --> EO
    CD --> CO
    
    %% Creation Flow
    U1 --> ECM
    U3 --> CCM
    ECM --> AUTH
    CCM --> AUTH
    
    %% Component Usage
    ED --> ELIST
    ED --> ECAL
    EO --> HEADER
    EO --> LOC
    EO --> OSM
    EO --> TAGS
    EO --> ATT
    CO --> HEADER
    CO --> ELIST
    CO --> ECAL
    
    %% API Communication
    API --> EP1
    API --> EP2
    API --> EP3
    API --> EP4
    API --> EP5
    API --> EP6
    
    ED --> API
    CD --> API
    EO --> API
    CO --> API
    FILTER --> API
    
    %% Nexus to Homeservers
    NI -.Indexes.-> HS1
    NI -.Indexes.-> HS2
    NI -.Indexes.-> HS3
    
    %% Nexus Internal
    EP1 --> DB
    EP2 --> DB
    EP3 --> DB
    EP4 --> DB
    EP5 --> DB
    EP6 --> DB
    
    %% Homeserver Storage
    HS1 --> CAL
    HS1 --> EVT
    HS1 --> ATD
    HS1 --> FILES
    HS2 --> CAL
    HS2 --> EVT
    HS2 --> ATD
    HS3 --> CAL
    HS3 --> EVT
    
    %% Write Operations
    AUTH -.Writes jCal JSON.-> HS1
    AUTH -.Writes jCal JSON.-> HS2
    AUTH -.Writes jCal JSON.-> HS3
    
    %% External Services
    OSM --> OSMAPI
    
    classDef userClass fill:#e1f5ff,stroke:#0066cc,stroke-width:2px
    classDef clientClass fill:#fff4e6,stroke:#ff9800,stroke-width:2px
    classDef nexusClass fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px
    classDef storageClass fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    classDef externalClass fill:#fce4ec,stroke:#e91e63,stroke-width:2px
    
    class U1,U2,U3 userClass
    class AUTH,QR,RING,ED,CD,EO,CO,ECM,CCM,API,HEADER,ELIST,ECAL,LOC,OSM,TAGS,ATT,FILTER clientClass
    class NI,EP1,EP2,EP3,EP4,EP5,EP6,DB nexusClass
    class HS1,HS2,HS3,CAL,EVT,ATD,FILES storageClass
    class OSMAPI externalClass
```

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
