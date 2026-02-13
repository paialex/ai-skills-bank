# AEM as a Cloud Service Development Guidelines

Critical considerations for developing on AEM as a Cloud Service (AEMaaCS).

## Cluster-Aware Code

Multiple instances run concurrently in AEMaaCS. Code must be resilient:

- Instances can be stopped at any time
- During updates, old and new code run in parallel
- New code must handle content created by older versions
- Use Apache Sling Discovery API to identify primary instance if needed

```java
@Reference
private DiscoveryService discoveryService;

public boolean isPrimaryInstance() {
    TopologyView view = discoveryService.getTopology();
    InstanceDescription local = view.getLocalInstance();
    return local.isLeader();
}
```

## State Management

**Never persist state in memory or local file system:**

- Disk is ephemeral - instances are recycled
- Any locally stored state will be lost
- Persist state in the repository or external store (Redis, database)

```java
// BAD: Local file storage
File tempFile = new File("/tmp/mydata.txt");

// GOOD: Repository storage
try (ResourceResolver resolver = getServiceResolver()) {
    Resource data = resolver.getResource("/var/myproject/data");
    // Read/write to repository
}
```

**Limited temporary file use:**
- Allowed for request processing only
- Avoid large files
- Clean up immediately after use

## Event Observation

JCR and Sling resource observation events are not guaranteed:

- Events may not execute locally
- Instances may be recycled before event fires
- Code must tolerate missing or delayed events

```java
// Design for resilience
@Component(service = EventHandler.class,
    property = EventConstants.EVENT_TOPIC + "=" + SlingConstants.TOPIC_RESOURCE_CHANGED)
public class ResilientEventHandler implements EventHandler {

    @Override
    public void handleEvent(Event event) {
        // Implement idempotent operations
        // Don't rely on this always firing
        // Use as optimization, not guarantee
    }
}
```

## Background Tasks and Long-Running Jobs

Background tasks must be resilient and resumable:

- Instance could terminate mid-execution
- Use Sling Jobs for at-least-once execution guarantee
- Avoid Sling Commons Scheduler in Cloud Service

```java
// GOOD: Sling Jobs for reliable execution
@Component(service = JobConsumer.class,
    property = JobConsumer.PROPERTY_TOPICS + "=" + "myproject/jobs/process")
public class ProcessJobConsumer implements JobConsumer {

    @Override
    public JobResult process(Job job) {
        try {
            // Implement resumable logic
            String checkpoint = job.getProperty("checkpoint", String.class);
            // Process from checkpoint
            return JobResult.OK;
        } catch (Exception e) {
            return JobResult.FAILED; // Will retry
        }
    }
}
```

## HTTP Connections

Always set timeouts for outgoing HTTP connections:

| Timeout Type | Recommended | Cloud Default |
|--------------|-------------|---------------|
| Connect | 1 second | 10 seconds |
| Read | 5 seconds | 60 seconds |

```java
// Apache HttpClient 4.x
RequestConfig config = RequestConfig.custom()
    .setConnectTimeout(1000)      // 1 second
    .setSocketTimeout(5000)       // 5 seconds
    .setConnectionRequestTimeout(1000)
    .build();

CloseableHttpClient client = HttpClients.custom()
    .setDefaultRequestConfig(config)
    .build();
```

## Rate Limiting

Handle HTTP 429 (Too Many Requests) responses:

```java
public Response fetchWithRetry(String url, int maxRetries) {
    int retries = 0;
    long delay = 1000; // Start with 1 second

    while (retries < maxRetries) {
        Response response = httpClient.get(url);

        if (response.getStatus() == 429) {
            retries++;
            try {
                Thread.sleep(delay);
                delay *= 2; // Exponential backoff
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException(e);
            }
        } else {
            return response;
        }
    }
    throw new TooManyRetriesException();
}
```

## Touch UI Only

AEMaaCS supports only Touch UI:
- Classic UI customizations are not available
- All dialogs must use Granite/Coral UI
- No Classic widgets or extjs

## No Native Binaries

Restrictions on binaries:
- Do not deploy native binaries or libraries
- Do not download binaries at runtime
- Binaries must be delivered via CDN
- Avoid streaming binaries through AEM

```java
// BAD: Streaming binaries
InputStream stream = asset.getOriginal().getStream();

// GOOD: Use CDN URL
String cdnUrl = externalizer.publishLink(resolver, asset.getPath());
```

## Replication Limitations

- **No reverse replication** from Publish to Author
- **No custom replication agents**
- Content replicates via pub-sub mechanism
- Use Cloud Manager for content transfer

## Environment Sizing

| Environment | Purpose | Capacity |
|-------------|---------|----------|
| Development | Dev/testing | Limited |
| RDE (Rapid Dev) | Quick iteration | Limited |
| Stage | Pre-production testing | Production-like |
| Production | Live site | Full capacity |

**Important:**
- Don't process large content on Dev/RDE environments
- Run indexing and heavy tests on Stage
- Performance testing must be on Stage

## Immutable Repository

In Cloud environments:
- `/libs` and `/apps` are immutable
- Changes must deploy through Cloud Manager pipelines
- CRXDE Lite has limited access
- Local SDK allows modification for development

## OSGi Configuration Deployment

All configurations must be deployed via pipeline:

```
/apps/myproject/osgiconfig/
├── config/                      # All environments
├── config.author/               # Author only
├── config.publish/              # Publish only
├── config.dev/                  # Development
├── config.stage/                # Staging
└── config.prod/                 # Production
```

## Content Package Filters

Define proper content package filters:

```xml
<filter root="/apps/myproject"/>
<filter root="/content/myproject">
    <exclude pattern="/content/myproject/sample-content(/.*)?"/>
</filter>
<filter root="/conf/myproject"/>
```

## Collaboration Best Practices

### Repository Management
- Keep repositories public when possible for community contributions
- Use pull requests instead of direct commits to main
- Require at least one approval before merging
- Enable branch protection

### Code Quality
- Enforce ESLint and Stylelint rules
- Fix linting errors before submitting PR
- Aim for green Lighthouse scores
- Use scaled trunk-based development (small, frequent PRs)

### CSS Best Practices for Cloud
- Write block-scoped CSS selectors
- Use intuitive, concise class names
- Avoid complex selectors
- Maintain consistent indentation
- Prefix private selectors with block name
