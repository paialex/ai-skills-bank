# AEM as a Cloud Service Anti-Patterns

## Overview

This reference documents common anti-patterns that cause issues in AEM as a Cloud Service. Each section shows the **wrong approach** (❌) and the **correct approach** (✅).

---

## 1. Scheduler Anti-Pattern

### ❌ WRONG: Using Scheduler API Directly
```java
import org.apache.sling.commons.scheduler.Scheduler;

@Component(service = Runnable.class, property = {
    Scheduler.PROPERTY_SCHEDULER_EXPRESSION + "=0 0/15 * * * ?"
})
public class MyScheduledTask implements Runnable {
    @Override
    public void run() {
        // This runs on EVERY instance in the cluster!
        // Not recommended for Cloud Service
    }
}
```

**Problems:**
- Runs on every instance in the cluster simultaneously
- Lost if instance restarts before execution
- No retry mechanism
- No job tracking

### ✅ CORRECT: Using Sling Jobs
```java
import org.apache.sling.event.jobs.JobManager;
import org.apache.sling.event.jobs.consumer.JobConsumer;

// Scheduler component
@Component(immediate = true)
public class MyScheduler {
    @Reference
    private JobManager jobManager;

    @Activate
    protected void activate() {
        // Check if already scheduled
        if (jobManager.getScheduledJobs(TOPIC, 1, null).isEmpty()) {
            jobManager.createJob(TOPIC)
                .schedule()
                .cron("0 0/15 * * * ?")
                .add();
        }
    }
}

// Job consumer
@Component(service = JobConsumer.class, property = {
    JobConsumer.PROPERTY_TOPICS + "=my/job/topic"
})
public class MyJobConsumer implements JobConsumer {
    @Override
    public JobResult process(Job job) {
        // Runs on ONE instance only
        return JobResult.OK;
    }
}
```

---

## 2. Static ResourceResolver Anti-Pattern

### ❌ WRONG: Storing ResourceResolver in Static Field
```java
@Component(service = MyService.class)
public class MyServiceImpl implements MyService {

    // NEVER do this!
    private static ResourceResolver staticResolver;

    @Reference
    private ResourceResolverFactory factory;

    @Activate
    protected void activate() {
        staticResolver = factory.getAdministrativeResourceResolver(null);
    }

    public void doSomething() {
        // Using stale resolver - WRONG!
        Resource res = staticResolver.getResource("/content/page");
    }
}
```

**Problems:**
- ResourceResolver becomes stale/closed
- Memory leaks
- Session issues in cluster
- Administrative resolver deprecated

### ✅ CORRECT: Request-Scoped or Service User
```java
@Component(service = MyService.class)
public class MyServiceImpl implements MyService {

    @Reference
    private ResourceResolverFactory factory;

    private static final String SERVICE_USER = "my-service-user";

    public void doSomething() {
        // Option 1: Service user for background operations
        try (ResourceResolver resolver = factory.getServiceResourceResolver(
                Map.of(ResourceResolverFactory.SUBSERVICE, SERVICE_USER))) {

            Resource res = resolver.getResource("/content/page");
            // Use resolver
        } catch (LoginException e) {
            LOG.error("Cannot get service resolver", e);
        }
    }

    // Option 2: Accept resolver from caller (request-scoped)
    public void doSomething(ResourceResolver resolver) {
        Resource res = resolver.getResource("/content/page");
    }
}
```

---

## 3. Storing State in OSGi Services

### ❌ WRONG: Mutable State in Services
```java
@Component(service = CounterService.class)
public class CounterServiceImpl implements CounterService {

    // WRONG: State is not shared across cluster instances!
    private int counter = 0;
    private Map<String, Object> cache = new HashMap<>();

    public int increment() {
        return ++counter; // Different value on each instance!
    }

    public void cacheValue(String key, Object value) {
        cache.put(key, value); // Not visible to other instances!
    }
}
```

**Problems:**
- Each instance has different state
- State lost on instance restart
- Inconsistent behavior across cluster

### ✅ CORRECT: External State Storage
```java
@Component(service = CounterService.class)
public class CounterServiceImpl implements CounterService {

    @Reference
    private ResourceResolverFactory factory;

    public int increment() {
        // Store in JCR (shared across cluster)
        try (ResourceResolver resolver = getServiceResolver()) {
            Resource counterRes = resolver.getResource("/var/myapp/counter");
            ModifiableValueMap props = counterRes.adaptTo(ModifiableValueMap.class);
            int current = props.get("value", 0);
            props.put("value", current + 1);
            resolver.commit();
            return current + 1;
        }
    }
}

// Or use Context-Aware Configuration for settings
// Or use external cache (Redis, etc.) for high-performance caching
```

---

## 4. Long-Running Requests

### ❌ WRONG: Blocking Request Thread
```java
@Model(adaptables = SlingHttpServletRequest.class)
public class SlowModel {

    @PostConstruct
    protected void init() {
        // WRONG: Blocks request for 30 seconds!
        HttpClient client = HttpClient.newHttpClient();
        HttpResponse<String> response = client.send(
            HttpRequest.newBuilder()
                .uri(URI.create("https://slow-api.example.com/data"))
                .build(),
            HttpResponse.BodyHandlers.ofString()
        );
        // Timeout might cause request failure
    }
}
```

**Problems:**
- Request threads exhausted
- Gateway timeouts (Cloud Service has strict limits)
- Poor user experience

### ✅ CORRECT: Async Processing with Timeouts
```java
@Model(adaptables = SlingHttpServletRequest.class)
public class FastModel {

    @OSGiService
    private ExternalDataService dataService;

    private ExternalData data;

    @PostConstruct
    protected void init() {
        // Option 1: Use cached data with async refresh
        data = dataService.getCachedData();

        // Option 2: Use background job for long operations
        // and show "loading" state to user

        // Option 3: If API call needed, use strict timeout
        // Cloud Service allows max 60 seconds for requests
    }
}

@Component(service = ExternalDataService.class)
public class ExternalDataServiceImpl implements ExternalDataService {

    private static final int TIMEOUT_SECONDS = 5;

    public ExternalData fetchData() {
        HttpClient client = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(TIMEOUT_SECONDS))
            .build();

        try {
            HttpResponse<String> response = client.send(
                HttpRequest.newBuilder()
                    .uri(URI.create("https://api.example.com/data"))
                    .timeout(Duration.ofSeconds(TIMEOUT_SECONDS))
                    .build(),
                HttpResponse.BodyHandlers.ofString()
            );
            // Process response
        } catch (Exception e) {
            // Return cached/default data on failure
            return getCachedData();
        }
    }
}
```

---

## 5. Deprecated APIs

### ❌ WRONG: Using Deprecated/Removed APIs
```java
// Administrative resolver - REMOVED in Cloud Service
ResourceResolver resolver = factory.getAdministrativeResourceResolver(null);

// Deprecated Replicator pattern
replicator.replicate(session, ReplicationActionType.ACTIVATE, path);

// JCR Session directly (prefer Sling APIs)
Session session = resolver.adaptTo(Session.class);
Node node = session.getNode("/content/page");

// Old OSGi annotations
@Component(immediate = true, metatype = true)
@Service
public class MyService { }

// Deprecated SCR annotations
@Property(name = "prop", value = "val")
```

### ✅ CORRECT: Modern APIs
```java
// Service user
ResourceResolver resolver = factory.getServiceResourceResolver(
    Map.of(ResourceResolverFactory.SUBSERVICE, "my-service"));

// Sling APIs for content access
Resource resource = resolver.getResource("/content/page");
ModifiableValueMap props = resource.adaptTo(ModifiableValueMap.class);

// OSGi DS R7 annotations
@Component(
    service = MyService.class,
    configurationPolicy = ConfigurationPolicy.OPTIONAL
)
@Designate(ocd = MyService.Config.class)
public class MyServiceImpl implements MyService {

    @ObjectClassDefinition(name = "My Service Configuration")
    public @interface Config {
        @AttributeDefinition(name = "Property")
        String prop() default "val";
    }
}
```

---

## 6. Hardcoded Paths and Configurations

### ❌ WRONG: Hardcoded Values
```java
@Model(adaptables = Resource.class)
public class MyModel {

    // WRONG: Hardcoded paths
    private static final String API_URL = "https://prod-api.example.com";
    private static final String DAM_ROOT = "/content/dam/mysite";

    public String getData() {
        // Works in prod, fails in dev/stage
        return fetch(API_URL + "/endpoint");
    }
}
```

### ✅ CORRECT: Context-Aware Configuration
```java
// CA Config interface
@Configuration(label = "My Component Configuration")
public @interface MyConfig {
    @Property(label = "API URL")
    String apiUrl() default "";

    @Property(label = "DAM Root")
    String damRoot() default "/content/dam";
}

// Model using CA Config
@Model(adaptables = Resource.class)
public class MyModel {

    @Self
    private Resource resource;

    private MyConfig config;

    @PostConstruct
    protected void init() {
        ConfigurationBuilder builder = resource.adaptTo(ConfigurationBuilder.class);
        config = builder.as(MyConfig.class);
    }

    public String getData() {
        // Uses environment-specific config
        return fetch(config.apiUrl() + "/endpoint");
    }
}
```

---

## 7. Missing Null Checks

### ❌ WRONG: Assuming Resources Exist
```java
@Model(adaptables = Resource.class)
public class UnsafeModel {

    @Inject
    private Resource resource;

    public String getChildTitle() {
        // NullPointerException if child doesn't exist!
        return resource.getChild("header")
                      .getValueMap()
                      .get("title", String.class);
    }
}
```

### ✅ CORRECT: Defensive Coding
```java
@Model(adaptables = Resource.class)
public class SafeModel {

    @Inject
    private Resource resource;

    public String getChildTitle() {
        return Optional.ofNullable(resource.getChild("header"))
            .map(Resource::getValueMap)
            .map(vm -> vm.get("title", String.class))
            .orElse("");
    }

    // Or explicit null checks
    public String getChildTitleExplicit() {
        Resource header = resource.getChild("header");
        if (header == null) {
            return "";
        }
        return header.getValueMap().get("title", "");
    }
}
```

---

## 8. Large Binary Operations in Memory

### ❌ WRONG: Loading Large Files into Memory
```java
public byte[] processLargeFile(String assetPath) {
    Resource assetResource = resolver.getResource(assetPath);
    Asset asset = assetResource.adaptTo(Asset.class);

    // WRONG: Loads entire file into memory!
    InputStream is = asset.getOriginal().getStream();
    return IOUtils.toByteArray(is); // OutOfMemoryError for large files!
}
```

### ✅ CORRECT: Streaming Processing
```java
public void processLargeFile(String assetPath, OutputStream output) {
    Resource assetResource = resolver.getResource(assetPath);
    Asset asset = assetResource.adaptTo(Asset.class);

    try (InputStream is = asset.getOriginal().getStream()) {
        // Stream directly without loading into memory
        byte[] buffer = new byte[8192];
        int bytesRead;
        while ((bytesRead = is.read(buffer)) != -1) {
            output.write(buffer, 0, bytesRead);
        }
    }
}
```

---

## 9. Ignoring Transaction Boundaries

### ❌ WRONG: No Transaction Management
```java
public void updateMultipleResources() {
    try (ResourceResolver resolver = getServiceResolver()) {
        // Modify resource 1
        Resource res1 = resolver.getResource("/content/page1");
        res1.adaptTo(ModifiableValueMap.class).put("prop", "val1");
        resolver.commit(); // Commit after each!

        // Modify resource 2 - if this fails, res1 is already committed!
        Resource res2 = resolver.getResource("/content/page2");
        res2.adaptTo(ModifiableValueMap.class).put("prop", "val2");
        resolver.commit();
    }
}
```

### ✅ CORRECT: Atomic Transactions
```java
public void updateMultipleResources() {
    try (ResourceResolver resolver = getServiceResolver()) {
        // Modify all resources
        Resource res1 = resolver.getResource("/content/page1");
        res1.adaptTo(ModifiableValueMap.class).put("prop", "val1");

        Resource res2 = resolver.getResource("/content/page2");
        res2.adaptTo(ModifiableValueMap.class).put("prop", "val2");

        // Single commit - atomic operation
        resolver.commit();

    } catch (PersistenceException e) {
        // Both changes rolled back automatically
        LOG.error("Failed to update resources", e);
        throw new ServiceException("Update failed", e);
    }
}
```

---

## 10. Client-Side Resource Types

### ❌ WRONG: Using Absolute Resource Types in HTL
```html
<!-- WRONG: Hardcoded absolute path -->
<sly data-sly-resource="${'header' @ resourceType='/apps/mysite/components/header'}"/>

<!-- WRONG: Missing @ context for user input -->
<div>${userInput}</div>
```

### ✅ CORRECT: Relative Resource Types and Context
```html
<!-- CORRECT: Relative resource type -->
<sly data-sly-resource="${'header' @ resourceType='mysite/components/header'}"/>

<!-- CORRECT: Explicit context for security -->
<div>${userInput @ context='html'}</div>
<a href="${linkUrl @ context='uri'}">${linkText @ context='text'}</a>

<!-- CORRECT: Using model for resource type -->
<sly data-sly-use.model="com.mysite.core.models.HeaderModel"/>
```

---

## 11. Servlet Without Proper Annotations

### ❌ WRONG: Old Servlet Registration
```java
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myservlet",      // Path-bound - security risk!
        "sling.servlet.methods=GET"
    }
)
public class UnsafeServlet extends SlingAllMethodsServlet {
    // Path-bound servlets bypass resource-level permissions
}
```

### ✅ CORRECT: Resource Type Bound Servlet
```java
@Component(service = Servlet.class)
@SlingServletResourceTypes(
    resourceTypes = "mysite/components/mycomponent",
    selectors = "data",
    extensions = "json",
    methods = "GET"
)
public class SafeServlet extends SlingSafeMethodsServlet {
    // Resource-bound respects ACLs
    // Accessed via: /content/page/jcr:content/component.data.json
}
```

---

## 12. Ignoring Dispatcher Cache

### ❌ WRONG: Not Considering Cache Invalidation
```java
// Component renders user-specific content without cache consideration
@Model(adaptables = SlingHttpServletRequest.class)
public class PersonalizedModel {

    @ScriptVariable
    private SlingHttpServletRequest request;

    public String getPersonalizedContent() {
        // This will be cached by Dispatcher and served to ALL users!
        String userId = request.getCookies()...;
        return "Hello, " + getUserName(userId);
    }
}
```

### ✅ CORRECT: Cache-Aware Personalization
```java
@Model(adaptables = SlingHttpServletRequest.class)
public class PersonalizedModel {

    // Option 1: Client-side personalization
    // Render placeholder, fetch personalized content via AJAX

    // Option 2: Edge-side includes (ESI)
    public String getEsiInclude() {
        return "<esi:include src=\"/content/page.personalized.html\"/>";
    }

    // Option 3: Set cache headers to prevent caching
    @PostConstruct
    protected void init() {
        response.setHeader("Cache-Control", "private, no-cache");
        response.setHeader("Dispatcher", "no-cache");
    }
}
```

---

## Quick Reference

| Anti-Pattern | Impact | Solution |
|--------------|--------|----------|
| Scheduler API | Runs on all instances | Sling Jobs |
| Static ResourceResolver | Stale sessions | Request-scoped / Service user |
| Mutable service state | Inconsistent cluster | JCR / External storage |
| Long-running requests | Timeouts | Async / Caching / Timeouts |
| Deprecated APIs | Build failures | DS R7 / Modern APIs |
| Hardcoded paths | Environment issues | CA Config |
| Missing null checks | NPE in production | Optional / Defensive coding |
| Large binaries in memory | OOM | Streaming |
| No transaction boundaries | Partial updates | Single commit |
| Path-bound servlets | Security risks | Resource-type binding |
| Ignoring Dispatcher | Wrong content served | Cache headers / AJAX |

---

## References

- [AEM as a Cloud Service Development Guidelines](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/content/implementing/developing/development-guidelines.html)
- [AEM Best Practices Analyzer](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/content/migration-journey/cloud-migration/best-practices-analyzer/overview-best-practices-analyzer.html)
- [OSGi DS R7 Annotations](https://docs.osgi.org/specification/osgi.cmpn/7.0.0/service.component.html)
