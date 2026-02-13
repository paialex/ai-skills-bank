# AEM Diagram Patterns

Reusable diagram templates for AEM architecture documentation.

---

## Table of Contents

1. [When to Use Each Format](#when-to-use-each-format)
2. [C4 Model Diagrams](#c4-model-diagrams)
3. [Flow Diagrams](#flow-diagrams)
4. [Caching Diagrams](#caching-diagrams)
5. [Sequence Diagrams](#sequence-diagrams)
6. [State Diagrams](#state-diagrams)
7. [ASCII Patterns](#ascii-patterns)

---

## When to Use Each Format

| Diagram Type | Format | Complexity | Best For |
|--------------|--------|------------|----------|
| Context | Mermaid | Simple-Medium | Stakeholder communication |
| Container | Mermaid | Medium | Technical overview |
| Request flow | Mermaid Flowchart | Medium | Developer understanding |
| Cache decision | Mermaid Flowchart | Simple | Cache strategy |
| API sequence | Mermaid Sequence | Medium-Complex | Integration design |
| Layer stack | ASCII | Simple | Quick documentation |
| File structure | ASCII | Simple | Component structure |
| Boundary boxes | ASCII | Simple | Responsibility mapping |

---

## C4 Model Diagrams

### Level 1: System Context

Shows AEM in relation to users and external systems.

```mermaid
flowchart TB
    subgraph Users
        Author[Content Author]
        EndUser[End User]
    end

    subgraph External Systems
        CRM[CRM System]
        PIM[Product System]
        Analytics[Analytics]
    end

    subgraph AEM System
        AEM[AEM as a Cloud Service]
    end

    Author -->|Creates content| AEM
    EndUser -->|Views content| AEM
    AEM -->|Fetches products| PIM
    AEM -->|Gets customer data| CRM
    AEM -->|Sends events| Analytics

    style AEM fill:#ff6b35,color:#fff
```

### Level 2: Container Diagram

Shows major containers within AEM ecosystem.

```mermaid
flowchart TB
    subgraph Users
        Author[Content Author]
        EndUser[End User]
    end

    subgraph AEM Cloud["AEM as a Cloud Service"]
        subgraph Author Tier
            AEMAuthor[AEM Author]
        end
        subgraph Publish Tier
            AEMPublish[AEM Publish]
        end
    end

    subgraph Delivery
        CDN[CDN / Fastly]
        Dispatcher[Dispatcher]
    end

    subgraph Backend
        API[Backend API]
        DB[(Database)]
    end

    Author --> AEMAuthor
    AEMAuthor -->|Replication| AEMPublish
    EndUser --> CDN
    CDN --> Dispatcher
    Dispatcher --> AEMPublish
    AEMPublish --> API
    API --> DB

    style AEMAuthor fill:#4a148c,color:#fff
    style AEMPublish fill:#4a148c,color:#fff
    style CDN fill:#00796b,color:#fff
```

### Level 3: Component Diagram

Shows internal structure of AEM Publish.

```mermaid
flowchart TB
    subgraph AEM Publish
        subgraph Sling
            Resolver[Resource Resolver]
            Servlets[Servlets]
        end

        subgraph Components
            HTL[HTL Templates]
            Models[Sling Models]
            Services[OSGi Services]
        end

        subgraph Content
            JCR[(JCR Repository)]
            DAM[DAM Assets]
        end
    end

    Request[Request] --> Resolver
    Resolver --> Servlets
    Servlets --> HTL
    HTL --> Models
    Models --> Services
    Services --> JCR
    Models --> DAM

    style Models fill:#4a148c,color:#fff
    style Services fill:#00796b,color:#fff
```

---

## Flow Diagrams

### Content Delivery Flow

```mermaid
flowchart TD
    A[Browser Request] --> B{CDN Cache}
    B -->|HIT| C[Return from CDN]
    B -->|MISS| D{Dispatcher Cache}
    D -->|HIT| E[Return from Dispatcher]
    D -->|MISS| F[AEM Publish]
    F --> G[Sling Resolution]
    G --> H[Component Rendering]
    H --> I[Sling Model]
    I --> J{External API?}
    J -->|Yes| K[Backend Call]
    J -->|No| L[JCR Content]
    K --> M[Response]
    L --> M
    M --> N[Cache at Dispatcher]
    N --> O[Cache at CDN]
    O --> P[Return to Browser]

    style F fill:#4a148c,color:#fff
    style B fill:#00796b,color:#fff
    style D fill:#00796b,color:#fff
```

### Authoring Flow

```mermaid
flowchart LR
    A[Author] --> B[AEM Author]
    B --> C{Preview?}
    C -->|Yes| D[Preview Mode]
    D --> B
    C -->|No| E{Publish?}
    E -->|Yes| F[Workflow]
    F --> G{Approved?}
    G -->|Yes| H[Replication Queue]
    G -->|No| B
    H --> I[AEM Publish]
    I --> J[Dispatcher Flush]

    style B fill:#4a148c,color:#fff
    style I fill:#4a148c,color:#fff
```

---

## Caching Diagrams

### Cache Decision Tree

```mermaid
flowchart TD
    A[Request] --> B{Personalized?}
    B -->|Yes| C{Client-side?}
    C -->|Yes| D[Cache + JS personalization]
    C -->|No| E[No cache / Private]
    B -->|No| F{Static asset?}
    F -->|Yes| G[Long TTL + Hash]
    F -->|No| H{HTML page?}
    H -->|Yes| I[Moderate TTL + statfileslevel]
    H -->|No| J{API response?}
    J -->|Yes| K{Public data?}
    K -->|Yes| L[TTL based on freshness]
    K -->|No| E

    style D fill:#00796b,color:#fff
    style G fill:#00796b,color:#fff
    style I fill:#00796b,color:#fff
    style L fill:#00796b,color:#fff
    style E fill:#c62828,color:#fff
```

### Cache Invalidation Flow

```mermaid
sequenceDiagram
    participant Author as AEM Author
    participant Publish as AEM Publish
    participant Dispatcher
    participant CDN

    Author->>Publish: Activate Content
    Publish->>Dispatcher: Invalidation Request
    Dispatcher->>Dispatcher: Touch .stat file
    Note over Dispatcher: statfileslevel=3
    Dispatcher->>Dispatcher: Mark files stale
    Note over Dispatcher: Next request fetches fresh

    alt Purge CDN
        Publish->>CDN: Purge Request
        CDN->>CDN: Invalidate cache
    else TTL-based
        Note over CDN: Wait for TTL expiry
    end
```

### Multi-Layer Cache

```mermaid
flowchart LR
    subgraph Browser
        BC[Browser Cache]
    end

    subgraph Edge
        CDN[CDN Cache]
    end

    subgraph Origin
        DC[Dispatcher Cache]
        AEM[AEM Publish]
    end

    BC -->|Miss| CDN
    CDN -->|Miss| DC
    DC -->|Miss| AEM
    AEM -->|Response| DC
    DC -->|Response| CDN
    CDN -->|Response| BC

    style BC fill:#e3f2fd
    style CDN fill:#00796b,color:#fff
    style DC fill:#4a148c,color:#fff
```

---

## Sequence Diagrams

### API Integration Pattern

```mermaid
sequenceDiagram
    participant Browser
    participant CDN
    participant Dispatcher
    participant AEM
    participant Backend

    Browser->>CDN: GET /content/page.html
    CDN->>Dispatcher: Forward (cache miss)
    Dispatcher->>AEM: Forward (cache miss)

    AEM->>AEM: Resolve component
    AEM->>AEM: Initialize Sling Model

    AEM->>Backend: GET /api/products
    Note over AEM,Backend: With timeout + circuit breaker

    alt Success
        Backend-->>AEM: Product data
        AEM->>AEM: Render with data
    else Timeout/Error
        AEM->>AEM: Render with fallback
    end

    AEM-->>Dispatcher: HTML response
    Dispatcher->>Dispatcher: Store in cache
    Dispatcher-->>CDN: Response
    CDN->>CDN: Store in cache
    CDN-->>Browser: HTML
```

### Content Replication

```mermaid
sequenceDiagram
    participant Author as Author Instance
    participant Queue as Replication Queue
    participant Agent as Replication Agent
    participant Publish as Publish Instance
    participant Dispatcher

    Author->>Queue: Add to queue
    Queue->>Agent: Process item
    Agent->>Publish: POST /bin/receive
    Publish->>Publish: Store content
    Publish-->>Agent: 200 OK

    Agent->>Dispatcher: POST /dispatcher/invalidate
    Dispatcher->>Dispatcher: Touch .stat
    Dispatcher-->>Agent: 200 OK

    Agent-->>Author: Replication complete
```

---

## State Diagrams

### Content Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Draft: Create

    Draft --> InReview: Submit
    InReview --> Draft: Reject
    InReview --> Approved: Approve

    Approved --> Scheduled: Schedule
    Approved --> Published: Activate Now

    Scheduled --> Published: Time reached

    Published --> Modified: Edit
    Modified --> InReview: Submit changes

    Published --> Unpublished: Deactivate
    Unpublished --> Published: Reactivate
    Unpublished --> Archived: Archive

    Archived --> [*]
```

### Request State

```mermaid
stateDiagram-v2
    [*] --> Received

    Received --> CDN_Check: Route to CDN
    CDN_Check --> CDN_Hit: Cache valid
    CDN_Check --> Dispatcher_Check: Cache miss

    CDN_Hit --> [*]: Return cached

    Dispatcher_Check --> Dispatcher_Hit: Cache valid
    Dispatcher_Check --> AEM_Process: Cache miss

    Dispatcher_Hit --> [*]: Return cached

    AEM_Process --> Rendered: Success
    AEM_Process --> Error: Failure

    Rendered --> Cached: Store response
    Cached --> [*]: Return response

    Error --> [*]: Return error
```

---

## ASCII Patterns

### Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         PRESENTATION                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  HTL Templates  │  │   ClientLibs    │  │  Author Dialog  │  │
│  └────────┬────────┘  └────────┬────────┘  └─────────────────┘  │
├───────────┼────────────────────┼────────────────────────────────┤
│           │                    │               MODEL             │
│  ┌────────▼────────────────────▼──────────────────────────────┐ │
│  │                      Sling Models                           │ │
│  │  @ValueMapValue | @OSGiService | @ChildResource            │ │
│  └────────────────────────────┬───────────────────────────────┘ │
├───────────────────────────────┼─────────────────────────────────┤
│                               │                SERVICE           │
│  ┌────────────────────────────▼───────────────────────────────┐ │
│  │                      OSGi Services                          │ │
│  │  Business Logic | External APIs | Caching                  │ │
│  └────────────────────────────┬───────────────────────────────┘ │
├───────────────────────────────┼─────────────────────────────────┤
│                               │              REPOSITORY          │
│  ┌────────────────────────────▼───────────────────────────────┐ │
│  │                     JCR Repository                          │ │
│  │  Pages | Content Fragments | Assets                         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Request Flow (Simple)

```
┌─────────┐     ┌─────────┐     ┌────────────┐     ┌─────────────┐
│ Browser │ ──▶ │   CDN   │ ──▶ │ Dispatcher │ ──▶ │ AEM Publish │
└─────────┘     └────┬────┘     └─────┬──────┘     └──────┬──────┘
                     │                │                    │
                     │ Cache?         │ Cache?             │
                     │   │            │   │                │
                     │   ▼            │   ▼                ▼
                     │ ┌───┐          │ ┌───┐          ┌───────┐
                     │ │HIT│──────────┼─│HIT│──────────│Render │
                     │ └───┘          │ └───┘          └───────┘
                     │                │                    │
                     ◀────────────────┴────────────────────┘
                              Response
```

### Boundary Diagram

```
╔═══════════════════════════════════════════════════════════════════╗
║                           AEM BOUNDARY                             ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║   ┌──────────────────────┐        ┌──────────────────────┐       ║
║   │    AUTHORING         │        │      DELIVERY        │       ║
║   │    ─────────         │        │      ────────        │       ║
║   │  • Content creation  │   ──▶  │  • Content serving   │       ║
║   │  • Preview           │        │  • JSON APIs         │       ║
║   │  • Workflows         │        │  • Asset delivery    │       ║
║   │  • Personalization   │        │                      │       ║
║   │    setup             │        │                      │       ║
║   └──────────────────────┘        └──────────────────────┘       ║
║                                                                    ║
╠═══════════════════════════════════════════════════════════════════╣
║                    OUTSIDE AEM BOUNDARY                            ║
╠═══════════════════════════════════════════════════════════════════╣
║                                                                    ║
║   ┌──────────────────────┐        ┌──────────────────────┐       ║
║   │    EDGE/CDN          │        │    BACKEND           │       ║
║   │    ────────          │        │    ───────           │       ║
║   │  • Edge caching      │        │  • Business logic    │       ║
║   │  • SSL termination   │        │  • Data processing   │       ║
║   │  • Geographic routing│        │  • Integrations      │       ║
║   │  • DDoS protection   │        │  • Personalization   │       ║
║   │                      │        │    engine            │       ║
║   └──────────────────────┘        └──────────────────────┘       ║
║                                                                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

### Content Hierarchy

```
/content
└── mysite
    ├── us                          # Region
    │   ├── en                      # Language
    │   │   ├── home                # Page
    │   │   ├── products
    │   │   │   ├── category-a
    │   │   │   └── category-b
    │   │   └── about
    │   └── es                      # Spanish
    │       └── ...
    └── eu
        ├── en
        ├── de
        └── fr

/content/dam
└── mysite
    ├── shared                      # Global assets
    └── regional
        ├── us
        └── eu
```

### Cache TTL Matrix

```
┌────────────────────┬─────────────┬──────────────┬────────────────┐
│   Content Type     │   CDN TTL   │ Dispatcher   │ Invalidation   │
├────────────────────┼─────────────┼──────────────┼────────────────┤
│ HTML Pages         │   5 min     │   5 min      │ On publish     │
│ Static Assets      │   1 year    │   1 year     │ Hash-based     │
│ JSON APIs          │   1 min     │   1 min      │ On publish     │
│ Personalized       │   0 (none)  │   0 (none)   │ N/A            │
│ Experience Frags   │   5 min     │   5 min      │ On publish     │
└────────────────────┴─────────────┴──────────────┴────────────────┘
```

---

## Diagram Best Practices

1. **Use consistent colors**
   - AEM components: `#4a148c` (purple)
   - Caching layers: `#00796b` (teal)
   - Errors/warnings: `#c62828` (red)
   - Users/external: default

2. **Keep diagrams focused**
   - One concept per diagram
   - 5-10 elements maximum
   - Clear flow direction

3. **Add legends when needed**
   - Explain non-obvious symbols
   - Note color meanings

4. **Match detail to audience**
   - Context: Business stakeholders
   - Container: Technical leads
   - Component: Developers
