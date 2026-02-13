# AEM Architectural Patterns

Common architectural patterns for AEM as a Cloud Service solutions.

---

## Table of Contents

1. [Delivery Patterns](#delivery-patterns)
2. [Content Patterns](#content-patterns)
3. [Integration Patterns](#integration-patterns)
4. [Caching Patterns](#caching-patterns)
5. [Multi-site Patterns](#multi-site-patterns)
6. [Security Patterns](#security-patterns)

---

## Delivery Patterns

### Pattern: Edge-First Delivery

**Context**: High-traffic public website with global audience.

**Problem**: AEM Publish cannot handle peak traffic; latency varies by geography.

**Solution**: Push all cacheable content to CDN edge nodes.

```
┌─────────┐     ┌─────────────────────────────────────┐     ┌─────────────┐
│  User   │ ──▶ │              CDN Edge               │ ──▶ │ AEM Publish │
│ (Global)│     │  (Regional PoPs, 95%+ cache hits)   │     │  (Origin)   │
└─────────┘     └─────────────────────────────────────┘     └─────────────┘
```

**Key decisions**:
- Long TTLs (5-15 min) for HTML
- Immutable assets with hash-based URLs
- Aggressive cache warming
- Client-side personalization only

**Trade-offs**:
- ✅ Low latency globally
- ✅ AEM protected from traffic spikes
- ❌ Content freshness delayed by TTL
- ❌ Limited real-time personalization

---

### Pattern: Headless Delivery

**Context**: Content consumed by multiple channels (web, mobile, IoT).

**Problem**: Different frontends need same content in different formats.

**Solution**: AEM as content API, separate frontend applications.

```
┌─────────────────────────────────────────────────────────────────┐
│                        Content Consumers                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│  │   Web   │  │  Mobile │  │   IoT   │  │  Kiosk  │            │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘            │
└───────┼────────────┼────────────┼────────────┼─────────────────┘
        │            │            │            │
        └────────────┴─────┬──────┴────────────┘
                           │
                    ┌──────▼──────┐
                    │  CDN/Cache  │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │     AEM     │
                    │  Content    │
                    │  Fragments  │
                    │  + GraphQL  │
                    └─────────────┘
```

**Key decisions**:
- Content Fragments as structured content
- GraphQL or Assets HTTP API for delivery
- Frontend framework-agnostic
- Separate caching per consumer

**Trade-offs**:
- ✅ Content reuse across channels
- ✅ Frontend flexibility
- ❌ Authoring complexity (no visual preview)
- ❌ More systems to manage

---

### Pattern: Hybrid Delivery

**Context**: Marketing site with both static pages and dynamic experiences.

**Problem**: Need rich authoring AND content APIs.

**Solution**: Traditional pages for marketing, headless for structured content.

```
                    ┌─────────────────────────────────────────┐
                    │                  AEM                     │
                    │  ┌───────────────┐  ┌─────────────────┐ │
                    │  │    Pages      │  │     Content     │ │
                    │  │   (HTML)      │  │   Fragments     │ │
                    │  │               │  │   (JSON/GraphQL)│ │
                    │  └───────┬───────┘  └────────┬────────┘ │
                    └──────────┼───────────────────┼──────────┘
                               │                   │
              ┌────────────────┼───────────────────┼────────────────┐
              │                │                   │                │
              ▼                ▼                   ▼                ▼
        ┌──────────┐    ┌──────────┐        ┌──────────┐    ┌──────────┐
        │ Website  │    │   SPA    │        │  Mobile  │    │  Partner │
        │  (SSR)   │    │  Widget  │        │   App    │    │   API    │
        └──────────┘    └──────────┘        └──────────┘    └──────────┘
```

**Key decisions**:
- Pages for full-page experiences
- Content Fragments for reusable structured data
- Experience Fragments for shared page sections
- Clear content type guidelines for authors

**Trade-offs**:
- ✅ Flexibility for different use cases
- ✅ Rich authoring for pages
- ❌ Complex content modeling decisions
- ❌ Training overhead for authors

---

## Content Patterns

### Pattern: Content Fragment Hub

**Context**: Structured content reused across many pages.

**Problem**: Same content (products, FAQs, etc.) duplicated across pages.

**Solution**: Centralize in Content Fragments, reference from pages.

```
┌─────────────────────────────────────────────────────────────┐
│                   Content Fragment Library                   │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  Products  │  │    FAQs    │  │   Bios     │            │
│  │  (1000+)   │  │   (200+)   │  │   (50+)    │            │
│  └─────┬──────┘  └──────┬─────┘  └──────┬─────┘            │
└────────┼────────────────┼───────────────┼──────────────────┘
         │                │               │
    ┌────┴────┬───────────┴───────────────┴────────┐
    │         │                                     │
    ▼         ▼                                     ▼
┌────────┐ ┌────────┐                          ┌────────┐
│  PDP   │ │  PLP   │                          │  About │
│  Page  │ │  Page  │                          │  Page  │
└────────┘ └────────┘                          └────────┘
```

**Key decisions**:
- Content Fragment Models as contracts
- Reference fields for relationships
- Variation support for context
- Workflow per fragment type

---

### Pattern: Experience Fragment Composition

**Context**: Shared page sections across multiple pages.

**Problem**: Header, footer, promos repeated and hard to maintain.

**Solution**: Experience Fragments for reusable composed blocks.

```
Page Structure:
┌───────────────────────────────────────────┐
│  ┌─────────────────────────────────────┐  │
│  │   Experience Fragment: Header       │  │ ◀── Shared
│  └─────────────────────────────────────┘  │
│  ┌─────────────────────────────────────┐  │
│  │          Page-Specific Content      │  │ ◀── Unique
│  │                                     │  │
│  └─────────────────────────────────────┘  │
│  ┌─────────────────────────────────────┐  │
│  │   Experience Fragment: Promo Banner │  │ ◀── Shared
│  └─────────────────────────────────────┘  │
│  ┌─────────────────────────────────────┐  │
│  │   Experience Fragment: Footer       │  │ ◀── Shared
│  └─────────────────────────────────────┘  │
└───────────────────────────────────────────┘
```

**Key decisions**:
- One Experience Fragment per reusable block
- Editable templates reference Experience Fragments
- Separate publishing workflow
- Clear ownership per fragment

---

## Integration Patterns

### Pattern: Build-Time Integration

**Context**: External data that changes infrequently.

**Problem**: Real-time API calls add latency and failure risk.

**Solution**: Import data at build/deploy time or via scheduled sync.

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────────────────┐
│  External   │     │   Scheduled │     │           AEM               │
│    API      │ ──▶ │   Importer  │ ──▶ │    Content Fragments        │
│             │     │  (Sling Job)│     │    (imported data)          │
└─────────────┘     └─────────────┘     └─────────────────────────────┘
                          │
                          │ Runs: Nightly / On-demand
                          ▼
                    ┌───────────┐
                    │  Publish  │ ──▶ Zero API calls at runtime
                    └───────────┘
```

**Key decisions**:
- Sling Jobs for reliable scheduling
- Content Fragments or JCR nodes for storage
- Conflict resolution strategy
- Full vs delta sync

**Trade-offs**:
- ✅ Zero runtime dependencies
- ✅ Maximum cache efficiency
- ❌ Data freshness limited by sync frequency
- ❌ Storage overhead in AEM

---

### Pattern: Edge-Side Personalization

**Context**: Personalized content without sacrificing caching.

**Problem**: Personalization typically breaks caching.

**Solution**: Cache generic content, personalize at edge or client.

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Request Flow                                │
│                                                                      │
│  ┌────────┐    ┌────────┐    ┌────────────┐    ┌──────────────────┐ │
│  │ Browser│ ─▶ │  CDN   │ ─▶ │ Edge Worker│ ─▶ │   AEM (cached)   │ │
│  │        │    │        │    │            │    │   Generic HTML   │ │
│  └────────┘    └────────┘    └─────┬──────┘    └──────────────────┘ │
│                                    │                                 │
│                                    ▼                                 │
│                          ┌─────────────────┐                        │
│                          │ Personalization │                        │
│                          │     Engine      │                        │
│                          │   (API call)    │                        │
│                          └─────────────────┘                        │
│                                    │                                 │
│                                    ▼                                 │
│                          ┌─────────────────┐                        │
│                          │ Personalized    │                        │
│                          │ Response        │                        │
│                          └─────────────────┘                        │
└─────────────────────────────────────────────────────────────────────┘
```

**Key decisions**:
- Generic content cached at CDN
- Edge worker or client-side JS for personalization
- Personalization data via separate API
- Graceful fallback to generic

---

### Pattern: Backend for Frontend (BFF)

**Context**: AEM needs data from multiple backend systems.

**Problem**: Multiple API calls from AEM add latency and complexity.

**Solution**: Single BFF aggregates and transforms backend data.

```
                    ┌─────────────────────────────────────────────────┐
                    │                   Backend Systems               │
                    │  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
                    │  │   CRM   │  │   PIM   │  │ Pricing │        │
                    │  └────┬────┘  └────┬────┘  └────┬────┘        │
                    └───────┼────────────┼────────────┼──────────────┘
                            │            │            │
                            └────────────┼────────────┘
                                         │
                                  ┌──────▼──────┐
                                  │     BFF     │
                                  │  (Unified   │
                                  │   API)      │
                                  └──────┬──────┘
                                         │
                                  ┌──────▼──────┐
                                  │     AEM     │
                                  │  (Single    │
                                  │   API call) │
                                  └─────────────┘
```

**Key decisions**:
- BFF owned by AEM team or shared
- Contract between AEM and BFF
- Caching at BFF layer
- Timeout and fallback strategy

---

## Caching Patterns

### Pattern: Stale-While-Revalidate

**Context**: Need fresh content without blocking requests.

**Problem**: Low TTLs cause cache misses; high TTLs cause stale content.

**Solution**: Serve stale while fetching fresh in background.

```
┌────────────────────────────────────────────────────────────────────┐
│                         Request Timeline                            │
│                                                                     │
│  Request 1: Cache MISS                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ CDN ──▶ Origin ──▶ Response ──▶ Cache + Serve                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Request 2: Cache HIT (within TTL)                                 │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ CDN ──▶ Serve from cache (fast)                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Request 3: Cache STALE (TTL expired, stale-while-revalidate)      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ CDN ──▶ Serve stale immediately                              │   │
│  │     └──▶ Background: Fetch from origin ──▶ Update cache      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

**Headers**:
```
Cache-Control: max-age=60, stale-while-revalidate=3600
```

---

### Pattern: Selective Invalidation

**Context**: Large site with frequent updates to some areas.

**Problem**: Full cache flush is expensive; targeted flush is complex.

**Solution**: Use statfileslevel + resource-based invalidation.

```
/content/site/
├── us/                    # statfileslevel = 2
│   ├── en/               # statfileslevel = 3
│   │   ├── home          # Invalidate: /content/site/us/en/.stat
│   │   ├── products/
│   │   │   └── widget    # Invalidate: /content/site/us/en/.stat
│   │   └── news/
│   │       └── article   # Invalidate: /content/site/us/en/.stat
│   └── es/               # Separate .stat file
└── eu/                    # Separate .stat file
```

**Key decisions**:
- statfileslevel based on update patterns
- Group frequently-updated content
- Separate rarely-updated content
- Consider language/region boundaries

---

## Multi-site Patterns

### Pattern: Blueprint + Live Copy

**Context**: Brand consistency across regional sites with local variations.

**Problem**: Global content must be consistent; local content must be flexible.

**Solution**: MSM (Multi-Site Manager) with inheritance.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Blueprint                                │
│                    (Global Master Site)                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Global Header | Core Pages | Shared Components             ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────┬───────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│   Live Copy   │     │   Live Copy   │     │   Live Copy   │
│    US Site    │     │    EU Site    │     │   APAC Site   │
│   ─────────   │     │   ─────────   │     │   ─────────   │
│ • Inherits    │     │ • Inherits    │     │ • Inherits    │
│   global      │     │   global      │     │   global      │
│ • Local news  │     │ • Local       │     │ • Local       │
│ • US pricing  │     │   compliance  │     │   languages   │
└───────────────┘     └───────────────┘     └───────────────┘
```

**Key decisions**:
- What inherits vs. what's local
- Rollout configuration (push vs. pull)
- Conflict resolution
- Governance for blueprint changes

---

### Pattern: Shared Content with Overrides

**Context**: Multiple brands with mostly shared content.

**Problem**: Duplicating content is expensive; customization is needed.

**Solution**: Content hierarchy with fallback resolution.

```
Content Resolution:

1. Check: /content/brand-a/us/en/page
   └── Not found? Fallback ↓

2. Check: /content/shared/us/en/page
   └── Not found? Fallback ↓

3. Check: /content/shared/global/page
   └── Not found? 404
```

**Implementation**:
- Resource resolver configuration
- Custom Sling Model resolvers
- Clear content ownership rules

---

## Security Patterns

### Pattern: Closed User Groups (CUG)

**Context**: Content restricted to specific user groups.

**Problem**: Some content is not public; need access control.

**Solution**: CUG policies at content paths.

```
/content/site/
├── public/              # No CUG - accessible to all
│   ├── home
│   └── about
├── members/             # CUG: "members" group
│   ├── exclusive-content
│   └── member-resources
└── partners/            # CUG: "partners" group
    └── partner-portal
```

**Key decisions**:
- CUG scope (paths)
- Group membership management
- Authentication integration
- Cache implications (private content)

---

### Pattern: Service User Principle

**Context**: Background operations need JCR access.

**Problem**: Admin session is deprecated; need controlled access.

**Solution**: Dedicated service users with minimal permissions.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Service User Mapping                          │
│                                                                  │
│  ┌──────────────────────┐    ┌────────────────────────────────┐ │
│  │  Content Importer    │ ─▶ │  service-user: content-writer  │ │
│  │  (Sling Job)         │    │  ACL: write /content/imports   │ │
│  └──────────────────────┘    └────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────────┐    ┌────────────────────────────────┐ │
│  │  Report Generator    │ ─▶ │  service-user: report-reader   │ │
│  │  (Scheduled Task)    │    │  ACL: read /content/reports    │ │
│  └──────────────────────┘    └────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────────┐    ┌────────────────────────────────┐ │
│  │  Cache Warmer        │ ─▶ │  service-user: cache-service   │ │
│  │  (Background)        │    │  ACL: read /content/*          │ │
│  └──────────────────────┘    └────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

**Key decisions**:
- One service user per use case
- Minimal required permissions
- Named subservices in code
- Regular permission audits
