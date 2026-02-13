# AEM Architecture Deep Dive

Comprehensive architectural patterns and best practices for enterprise AEM implementations.

## TarMK Architecture and Scaling

### Author-Publish Separation

AEM uses distinct **Author** and **Publish** environments that scale independently:

- **Author**: Content creation and management (single active + cold standby)
- **Publish**: Serves published content (horizontally scalable)

```
Traffic increases → Add publish instances (no author changes needed)
```

### TarMK Storage Characteristics

| Feature | Description |
|---------|-------------|
| Storage Type | File-based TAR storage |
| External Dependencies | None (no database required) |
| Read/Write Throughput | High |
| Backup | File system based (MVCC consistency) |
| Hot Backups | Supported |
| Multi-master Clustering | Not supported (use MongoMK for that) |

### Scalability Pattern

```
                    ┌─────────────────┐
                    │   CDN Layer     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
        ┌─────┴─────┐  ┌─────┴─────┐  ┌─────┴─────┐
        │ Dispatch 1 │  │ Dispatch 2 │  │ Dispatch 3 │
        └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
              │              │              │
        ┌─────┴─────┐  ┌─────┴─────┐  ┌─────┴─────┐
        │ Publish 1  │  │ Publish 2  │  │ Publish N  │
        │  (TarMK)   │  │  (TarMK)   │  │  (TarMK)   │
        └────────────┘  └────────────┘  └────────────┘
                             ▲
                             │ Replication
                    ┌────────┴────────┐
                    │     Author      │
                    │    (TarMK)      │
                    │    + Standby    │
                    └─────────────────┘
```

### Design Best Practices

1. **Loose coupling** between features
2. **Partition content logically** (by sites or regions)
3. **Offload user-generated data** to separate persistence
4. **Use asynchronous processing** to keep publish tier stateless

## MVC Pattern in AEM

```
┌──────────────────────────────────────────────────────────────┐
│                        AEM MVC                                │
├──────────────────────────────────────────────────────────────┤
│  MODEL                                                        │
│  ├── Sling Models: Business logic, data access               │
│  └── OSGi Services: Reusable logic, integrations             │
│                                                               │
│  VIEW                                                         │
│  ├── HTL Scripts: Markup rendering                           │
│  └── ClientLibs: CSS/JS presentation                         │
│                                                               │
│  CONTROLLER                                                   │
│  ├── Components: Request handling                            │
│  └── Servlets: API endpoints, form handlers                  │
└──────────────────────────────────────────────────────────────┘
```

## Design Patterns

### Composite Pattern (Content Structure)
Pages composed of reusable component containers:

```
Page
├── Header (Experience Fragment)
├── Main Content (Parsys)
│   ├── Hero Component
│   ├── Card Container
│   │   ├── Card 1
│   │   ├── Card 2
│   │   └── Card 3
│   └── Text Component
└── Footer (Experience Fragment)
```

### Decorator Pattern (Component Behavior)
Extend component behavior without duplicating code:

```java
// Base component
@Model(adaptables = Resource.class)
public class BaseCard {
    @ValueMapValue
    protected String title;
}

// Extended component
@Model(adaptables = Resource.class,
       resourceType = "myproject/components/productcard")
public class ProductCard extends BaseCard {
    @ValueMapValue
    private BigDecimal price;

    public String getFormattedPrice() {
        return currencyFormatter.format(price);
    }
}
```

## OSGi Mastery

### Component vs Service

| Type | Registration | Purpose |
|------|--------------|---------|
| OSGi Component | `@Component` | Basic OSGi managed class |
| OSGi Service | `@Component(service=X.class)` | Globally reusable, injectable |

**Rule**: Every OSGi service is a component, but not all components are services.

### Service Pattern

```java
// 1. Define interface
public interface ContentSearchService {
    List<SearchResult> search(String query, int limit);
}

// 2. Implement with @Component(service=...)
@Component(service = ContentSearchService.class)
public class ContentSearchServiceImpl implements ContentSearchService {

    @Reference
    private QueryBuilder queryBuilder;

    @Override
    public List<SearchResult> search(String query, int limit) {
        // Implementation
    }
}

// 3. Inject via @Reference or @OSGiService
@Model(adaptables = SlingHttpServletRequest.class)
public class SearchComponent {

    @OSGiService
    private ContentSearchService searchService;
}
```

### Servlet Best Practices

**Use modern annotations:**

```java
// GOOD: OSGi R7 annotations
@Component(service = Servlet.class)
@SlingServletResourceTypes(
    resourceTypes = "myproject/components/search",
    methods = HttpConstants.METHOD_GET,
    extensions = "json"
)
public class SearchServlet extends SlingSafeMethodsServlet { }

// AVOID: Deprecated Felix annotations
@SlingServlet(...)  // Don't use
```

**Path-based servlets:**

```java
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myproject/api",
        "sling.servlet.methods=GET"
    },
    configurationPolicy = ConfigurationPolicy.REQUIRE  // Important!
)
```

## Sling Models Excellence

### Recommended Pattern

```java
@Model(
    adaptables = {Resource.class, SlingHttpServletRequest.class},
    adapters = {MyComponent.class, ComponentExporter.class},
    resourceType = MyComponentImpl.RESOURCE_TYPE,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
@Exporter(name = ExporterConstants.SLING_MODEL_EXPORTER_NAME,
          extensions = ExporterConstants.SLING_MODEL_EXTENSION)
public class MyComponentImpl implements MyComponent {

    static final String RESOURCE_TYPE = "myproject/components/mycomponent";

    @ValueMapValue
    private String title;

    @OSGiService
    private MyService myService;

    @ChildResource
    private List<Resource> items;

    @PostConstruct
    protected void init() {
        // Initialization logic
    }
}
```

### Injector Reference

| Annotation | Source | Example |
|------------|--------|---------|
| `@ValueMapValue` | JCR property | `private String title;` |
| `@ChildResource` | Child node | `private Resource image;` |
| `@OSGiService` | OSGi service | `private MyService svc;` |
| `@Self` | Current resource/request | `private Resource resource;` |
| `@ScriptVariable` | HTL binding | `private Page currentPage;` |
| `@RequestAttribute` | Request attribute | `private String param;` |

## Context-Aware Configuration

### Hierarchical Configuration

```
/conf/mysite/settings/myconfig
    ├── apiEndpoint: "https://api.prod.com"
    └── cacheTtl: 300

/conf/mysite/de/settings/myconfig
    └── apiEndpoint: "https://api.de.com"  # Override for German site
```

### Configuration Annotation

```java
@Configuration(label = "Site Configuration")
public @interface SiteConfiguration {

    @Property(label = "API Endpoint")
    String apiEndpoint() default "";

    @Property(label = "Cache TTL")
    int cacheTtl() default 300;

    @Property(label = "Feature Flags")
    String[] featureFlags() default {};
}
```

### Reading in Sling Model

```java
@PostConstruct
protected void init() {
    Resource resource = request.getResource();
    ConfigurationBuilder builder = resource.adaptTo(ConfigurationBuilder.class);
    SiteConfiguration config = builder.as(SiteConfiguration.class);

    // Access with fallback inheritance
    String endpoint = config.apiEndpoint();
}
```

## Content Modeling

### Performance-Oriented Hierarchy

```
GOOD: Intermediate hierarchy for large sets
/content/mysite/articles/2024/01/article-1
/content/mysite/articles/2024/01/article-2
/content/mysite/articles/2024/02/article-1

BAD: Flat structure with thousands of children
/content/mysite/articles/article-1
/content/mysite/articles/article-2
... (10,000 items)
```

**Rule**: Keep under ~1000 child nodes per parent.

### Template Strategy

**Fewer templates, more flexible components:**

| Approach | Templates | Components |
|----------|-----------|------------|
| BAD | Many one-off templates | Limited flexibility |
| GOOD | Few base templates | Configurable, reusable |

### Content Fragments vs Experience Fragments

| Type | Use Case | Delivery |
|------|----------|----------|
| Content Fragment | Structured data (articles, products) | Headless/GraphQL |
| Experience Fragment | Reusable content blocks | Multi-channel (web, email) |

## Headless Integration (React)

### JSON APIs

**Sling Model Exporter:**

```java
@Exporter(name = "jackson", extensions = "json")
public class ArticleModel {

    @ValueMapValue
    private String title;

    @ValueMapValue
    private String body;

    // Accessible via /content/mysite/article.model.json
}
```

**GraphQL for Content Fragments:**

```graphql
{
  articleList(
    filter: { category: { _expressions: [{ value: "news" }] } }
    orderBy: { publishDate: DESC }
  ) {
    items {
      _path
      title
      summary
      publishDate
    }
  }
}
```

### Hybrid Integration

```
AEM Page (server-rendered shell)
└── React Container Component
    ├── Fetches data from AEM JSON APIs
    └── Renders interactive UI
```

**Benefits:**
- SEO-friendly server-side HTML
- Interactive client-side experience
- AEM manages content, React handles UX

## Cache Invalidation Strategy

### Statfileslevel Configuration

| Level | .stat Location | Invalidation Scope |
|-------|----------------|-------------------|
| 0 | / | Entire cache |
| 2 | /content, /content/site | Per-site |
| 3 | + /content/site/language | Per-language |
| 4 | + /content/site/region/country | Per-country |

**Recommendation**: Level 2-3 for most multi-site setups.

### Flush Process

```
Author publishes page
    ↓
Publish receives content
    ↓
Flush agent sends invalidation request
    ↓
Dispatcher updates .stat files
    ↓
Next request sees stale cache
    ↓
Dispatcher fetches fresh content
```

## Security Hardening

### Default Deny Filter

```apache
/filter {
    /0001 { /glob "*" /type "deny" }

    # Allow only what's needed
    /0100 { /type "allow" /method "GET" /url "/content/mysite/*" }
    /0200 { /type "allow" /method "GET" /url "/etc.clientlibs/*" }
}
```

### Block Administrative URLs

```apache
/0900 { /type "deny" /url "/system/console/*" }
/0901 { /type "deny" /url "/crx/*" }
/0902 { /type "deny" /url "/libs/cq/testing/*" }
/0903 { /type "deny" /url "*.infinity.json" }
/0904 { /type "deny" /url "*.json" /selectors "(1|2|3|4|5|6|7|8|9)" }
```

## CI/CD with Cloud Manager

### Pipeline Flow

```
Code Push → Build → Unit Tests → Static Analysis
    ↓
Quality Gates (Code Quality, Security, Performance)
    ↓
Deploy to Stage → Integration Tests
    ↓
Approval → Deploy to Production
```

### Quality Gates

| Gate | Checks | Failure Action |
|------|--------|----------------|
| Code Quality | SonarQube, code smells | Fix before merge |
| Security | Vulnerability scan | Immediate fix |
| Performance | Load test, response times | Pipeline pauses |

**Critical issues** → Pipeline fails immediately
**Important issues** → Human approval required
