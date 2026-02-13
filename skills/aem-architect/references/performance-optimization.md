# AEM Performance Optimization Reference

Comprehensive guide for optimizing AEM performance across all layers.

## Performance Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PERFORMANCE OPTIMIZATION LAYERS                   │
├─────────────────────────────────────────────────────────────────────┤
│ BROWSER        │ Asset optimization, lazy loading, critical CSS     │
│ CDN            │ Edge caching, compression, image optimization      │
│ DISPATCHER     │ Page caching, static asset caching, compression    │
│ AEM PUBLISH    │ Component rendering, Sling Models, OSGi services   │
│ REPOSITORY     │ Query optimization, indexes, content structure     │
└─────────────────────────────────────────────────────────────────────┘
```

## Browser-Level Optimization

### Critical CSS Inlining

```html
<!-- Inline critical CSS in head -->
<style data-sly-use.criticalCss="com.myproject.CriticalCssModel">
    ${criticalCss.content @ context='unsafe'}
</style>

<!-- Load full CSS asynchronously -->
<link rel="preload" href="/etc.clientlibs/myproject/clientlibs/site.css" as="style"
      onload="this.onload=null;this.rel='stylesheet'">
<noscript>
    <link rel="stylesheet" href="/etc.clientlibs/myproject/clientlibs/site.css">
</noscript>
```

### Image Optimization

```html
<!-- Responsive images with srcset -->
<picture>
    <source media="(min-width: 1200px)"
            srcset="${image.src}.coreimg.1600.webp 1x, ${image.src}.coreimg.3200.webp 2x"
            type="image/webp">
    <source media="(min-width: 768px)"
            srcset="${image.src}.coreimg.1024.webp 1x, ${image.src}.coreimg.2048.webp 2x"
            type="image/webp">
    <source srcset="${image.src}.coreimg.640.webp 1x, ${image.src}.coreimg.1280.webp 2x"
            type="image/webp">
    <img src="${image.src}.coreimg.640.jpeg"
         alt="${image.alt}"
         loading="lazy"
         decoding="async"
         width="${image.width}"
         height="${image.height}">
</picture>
```

### Lazy Loading Implementation

```javascript
// Intersection Observer for lazy loading
document.addEventListener('DOMContentLoaded', function() {
    const lazyElements = document.querySelectorAll('[data-lazy]');

    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const element = entry.target;
                // Load content/component
                loadLazyContent(element);
                observer.unobserve(element);
            }
        });
    }, {
        rootMargin: '100px'
    });

    lazyElements.forEach(el => observer.observe(el));
});
```

### Resource Hints

```html
<!-- Preconnect to critical origins -->
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
<link rel="dns-prefetch" href="https://analytics.example.com">

<!-- Preload critical resources -->
<link rel="preload" href="/etc.clientlibs/myproject/clientlibs/site.min.js" as="script">
<link rel="preload" href="/content/dam/myproject/hero.webp" as="image" type="image/webp">

<!-- Prefetch next page -->
<link rel="prefetch" href="/content/mysite/en/about.html">
```

## ClientLib Optimization

### Optimal Bundling Strategy

```
clientlibs/
├── site/                    # Core site library (loaded on all pages)
│   ├── css/
│   │   ├── base.less       # Reset, variables, typography
│   │   ├── layout.less     # Grid, containers
│   │   └── utilities.less  # Helper classes
│   └── js/
│       ├── polyfills.js    # Browser polyfills
│       └── core.js         # Core functionality
│
├── components/              # Component-specific (loaded per component)
│   ├── carousel/
│   │   ├── css/carousel.less
│   │   └── js/carousel.js
│   ├── accordion/
│   │   ├── css/accordion.less
│   │   └── js/accordion.js
│   └── video/
│       ├── css/video.less
│       └── js/video.js
│
└── vendor/                  # Third-party libraries
    ├── jquery/
    └── swiper/
```

### ClientLib Definition with Dependencies

```xml
<!-- Core site clientlib -->
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:primaryType="cq:ClientLibraryFolder"
    categories="[myproject.site]"
    embed="[myproject.vendor.core]"
    jsProcessor="[default:none,min:gcc]"
    cssProcessor="[default:none,min:yui]"/>

<!-- Component clientlib with dependency -->
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:primaryType="cq:ClientLibraryFolder"
    categories="[myproject.components.carousel]"
    dependencies="[myproject.site]"
    jsProcessor="[default:none,min:gcc]"
    cssProcessor="[default:none,min:yui]"/>
```

### Async/Defer Script Loading

```html
<!-- Core JS - blocking but minified -->
<sly data-sly-call="${clientlib.js @ categories='myproject.site'}"/>

<!-- Component JS - deferred -->
<script src="/etc.clientlibs/myproject/clientlibs/components/carousel.js" defer></script>

<!-- Analytics - async -->
<script src="/etc.clientlibs/myproject/clientlibs/analytics.js" async></script>
```

## Sling Model Optimization

### Efficient Model Design

```java
@Model(
    adaptables = SlingHttpServletRequest.class,
    adapters = {OptimizedComponent.class, ComponentExporter.class},
    resourceType = OptimizedComponentImpl.RESOURCE_TYPE,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
public class OptimizedComponentImpl implements OptimizedComponent {

    private static final Logger LOG = LoggerFactory.getLogger(OptimizedComponentImpl.class);

    @Self
    private SlingHttpServletRequest request;

    @ScriptVariable
    private Page currentPage;

    @ValueMapValue
    private String title;

    // Lazy initialization - computed only when needed
    private List<CardItem> cards;
    private NavigationTree navigation;

    @PostConstruct
    protected void init() {
        // Only perform essential initialization here
        // Defer expensive operations to getters
    }

    @Override
    public List<CardItem> getCards() {
        if (cards == null) {
            cards = computeCards();
        }
        return cards;
    }

    private List<CardItem> computeCards() {
        // Expensive operation performed only when needed
        Resource cardsResource = request.getResource().getChild("cards");
        if (cardsResource == null) {
            return Collections.emptyList();
        }
        return StreamSupport.stream(cardsResource.getChildren().spliterator(), false)
            .map(r -> r.adaptTo(CardItem.class))
            .filter(Objects::nonNull)
            .collect(Collectors.toList());
    }

    @Override
    public NavigationTree getNavigation() {
        if (navigation == null) {
            // Use service for potentially cached results
            navigation = navigationService.getNavigationTree(currentPage);
        }
        return navigation;
    }
}
```

### Avoid N+1 Query Pattern

```java
// BAD: N+1 queries
public List<ArticleInfo> getArticles() {
    return StreamSupport.stream(articlesResource.getChildren().spliterator(), false)
        .map(resource -> {
            // Each iteration makes a new query
            String authorPath = resource.getValueMap().get("authorPath", String.class);
            Resource authorResource = resolver.getResource(authorPath);  // Query!
            return new ArticleInfo(resource, authorResource);
        })
        .collect(Collectors.toList());
}

// GOOD: Batch loading
public List<ArticleInfo> getArticles() {
    List<Resource> articleResources = StreamSupport.stream(
        articlesResource.getChildren().spliterator(), false)
        .collect(Collectors.toList());

    // Collect all author paths
    Set<String> authorPaths = articleResources.stream()
        .map(r -> r.getValueMap().get("authorPath", String.class))
        .filter(Objects::nonNull)
        .collect(Collectors.toSet());

    // Single batch load
    Map<String, Resource> authorResources = authorPaths.stream()
        .collect(Collectors.toMap(
            path -> path,
            path -> resolver.getResource(path),
            (a, b) -> a
        ));

    return articleResources.stream()
        .map(resource -> {
            String authorPath = resource.getValueMap().get("authorPath", String.class);
            return new ArticleInfo(resource, authorResources.get(authorPath));
        })
        .collect(Collectors.toList());
}
```

## OSGi Service Caching

### Service with In-Memory Cache

```java
@Component(service = ContentService.class)
public class ContentServiceImpl implements ContentService {

    private static final long CACHE_DURATION_MINUTES = 5;
    private static final int CACHE_MAX_SIZE = 1000;

    // Guava cache with expiration
    private final Cache<String, CachedContent> cache = CacheBuilder.newBuilder()
        .maximumSize(CACHE_MAX_SIZE)
        .expireAfterWrite(CACHE_DURATION_MINUTES, TimeUnit.MINUTES)
        .recordStats()
        .build();

    @Reference
    private ResourceResolverFactory resolverFactory;

    @Override
    public ContentData getContent(String path) {
        try {
            return cache.get(path, () -> loadContent(path));
        } catch (ExecutionException e) {
            LOG.error("Error loading content: {}", path, e);
            return ContentData.empty();
        }
    }

    private ContentData loadContent(String path) {
        // Expensive operation cached for 5 minutes
        try (ResourceResolver resolver = getServiceResolver()) {
            Resource resource = resolver.getResource(path);
            return resource != null ? parseContent(resource) : ContentData.empty();
        }
    }

    // Invalidation hook for content changes
    public void invalidate(String path) {
        cache.invalidate(path);
    }

    public void invalidateAll() {
        cache.invalidateAll();
    }
}
```

### Event-Based Cache Invalidation

```java
@Component(
    service = EventHandler.class,
    property = {
        EventConstants.EVENT_TOPIC + "=" + PageEvent.EVENT_TOPIC,
        EventConstants.EVENT_TOPIC + "=" + ReplicationAction.EVENT_TOPIC
    }
)
public class CacheInvalidationHandler implements EventHandler {

    @Reference
    private ContentService contentService;

    @Override
    public void handleEvent(Event event) {
        String path = (String) event.getProperty("path");
        if (path != null && path.startsWith("/content/mysite")) {
            contentService.invalidate(path);
            LOG.debug("Cache invalidated for: {}", path);
        }
    }
}
```

## Query Optimization

### Avoid Queries in Components

```java
// BAD: Query in component
@Model(adaptables = Resource.class)
public class BadSearchComponent {

    @Self
    private Resource resource;

    public List<Page> getRelatedPages() {
        ResourceResolver resolver = resource.getResourceResolver();
        String query = "SELECT * FROM [cq:Page] WHERE [jcr:content/category] = 'news'";
        // Query executed on every page render!
        return executeQuery(resolver, query);
    }
}

// GOOD: Use content structure
@Model(adaptables = Resource.class)
public class GoodSearchComponent {

    @ValueMapValue
    private String listRoot;

    public List<Page> getRelatedPages() {
        if (listRoot == null) return Collections.emptyList();

        Resource rootResource = resource.getResourceResolver().getResource(listRoot);
        if (rootResource == null) return Collections.emptyList();

        // Traverse content structure instead of querying
        return StreamSupport.stream(rootResource.getChildren().spliterator(), false)
            .map(r -> r.adaptTo(Page.class))
            .filter(Objects::nonNull)
            .limit(10)
            .collect(Collectors.toList());
    }
}
```

### Custom Oak Index for Required Queries

```xml
<!-- /oak:index/myprojectCategoryIndex -->
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    xmlns:oak="http://jackrabbit.apache.org/oak/ns/1.0"
    jcr:primaryType="oak:QueryIndexDefinition"
    type="lucene"
    async="async"
    compatVersion="{Long}2"
    evaluatePathRestrictions="{Boolean}true"
    includedPaths="[/content/mysite]">

    <indexRules jcr:primaryType="nt:unstructured">
        <cq:PageContent jcr:primaryType="nt:unstructured">
            <properties jcr:primaryType="nt:unstructured">
                <category jcr:primaryType="nt:unstructured"
                    name="category"
                    propertyIndex="{Boolean}true"/>
                <publishDate jcr:primaryType="nt:unstructured"
                    name="publishDate"
                    propertyIndex="{Boolean}true"
                    ordered="{Boolean}true"
                    type="Date"/>
            </properties>
        </cq:PageContent>
    </indexRules>
</jcr:root>
```

## Caching Headers Configuration

### Static Assets (Long Cache)

```apache
# Apache configuration for static assets
<LocationMatch "^/etc\.clientlibs/.*\.(css|js)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
</LocationMatch>

<LocationMatch "^/content/dam/.*\.(jpg|jpeg|png|gif|webp|svg|ico)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
</LocationMatch>
```

### HTML Pages (Short Cache with Revalidation)

```apache
<LocationMatch "^/content/mysite/.*\.html$">
    Header set Cache-Control "public, max-age=300, must-revalidate"
    Header set Surrogate-Control "max-age=3600"
</LocationMatch>
```

### API/JSON Endpoints

```apache
# Cacheable API responses
<LocationMatch "^/content/mysite/.*\.model\.json$">
    Header set Cache-Control "public, max-age=60"
</LocationMatch>

# Non-cacheable API responses
<LocationMatch "^/api/user/.*$">
    Header set Cache-Control "private, no-cache, no-store"
</LocationMatch>
```

## Performance Monitoring

### Key Metrics to Track

| Metric | Target | Tool |
|--------|--------|------|
| First Contentful Paint (FCP) | < 1.8s | Lighthouse |
| Largest Contentful Paint (LCP) | < 2.5s | Lighthouse |
| Time to Interactive (TTI) | < 3.8s | Lighthouse |
| Cumulative Layout Shift (CLS) | < 0.1 | Lighthouse |
| Total Blocking Time (TBT) | < 200ms | Lighthouse |
| Dispatcher Cache Hit Ratio | > 95% | Dispatcher logs |
| CDN Cache Hit Ratio | > 90% | CDN analytics |
| Average Response Time | < 200ms | APM |

### Request Tracing

```java
@Component(service = Filter.class)
@SlingServletFilter(scope = SlingServletFilterScope.REQUEST)
public class PerformanceTracingFilter implements Filter {

    private static final String TRACE_HEADER = "X-Request-Id";

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        long startTime = System.currentTimeMillis();
        String requestId = UUID.randomUUID().toString();

        try {
            MDC.put("requestId", requestId);
            ((HttpServletResponse) response).setHeader(TRACE_HEADER, requestId);
            chain.doFilter(request, response);
        } finally {
            long duration = System.currentTimeMillis() - startTime;
            LOG.info("Request {} completed in {}ms", requestId, duration);
            MDC.clear();
        }
    }
}
```

## Performance Checklist by Phase

### Development
- [ ] Use Sling Models with lazy initialization
- [ ] Avoid JCR queries in render path
- [ ] Implement proper caching in services
- [ ] Optimize clientlib structure
- [ ] Test with production-like content volume

### Build/Deploy
- [ ] Enable CSS/JS minification
- [ ] Configure content-hash fingerprinting
- [ ] Set up proper Oak indexes
- [ ] Validate Dispatcher configuration

### Production
- [ ] Monitor cache hit ratios
- [ ] Set up alerting for slow responses
- [ ] Regular performance testing
- [ ] Review and optimize slow queries
- [ ] Analyze and act on Core Web Vitals data
