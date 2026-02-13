# AEM Code Snippets

Copy-paste ready code snippets for common AEM development patterns.

---

## Sling Models

### Basic Sling Model
```java
package com.myproject.core.models;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;

import javax.annotation.PostConstruct;

@Model(
    adaptables = SlingHttpServletRequest.class,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
public class MyModel {

    @ValueMapValue
    private String title;

    @ValueMapValue
    private String description;

    @ValueMapValue(name = "jcr:title")
    private String jcrTitle;

    @PostConstruct
    protected void init() {
        // Initialization logic
        if (title == null) {
            title = jcrTitle;
        }
    }

    public String getTitle() {
        return title;
    }

    public String getDescription() {
        return description;
    }
}
```

### Sling Model with All Injection Types
```java
package com.myproject.core.models;

import com.adobe.cq.export.json.ComponentExporter;
import com.adobe.cq.export.json.ExporterConstants;
import com.day.cq.wcm.api.Page;
import com.day.cq.wcm.api.designer.Style;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.*;
import org.apache.sling.models.annotations.injectorspecific.*;

import javax.annotation.PostConstruct;
import javax.inject.Inject;
import java.util.List;

@Model(
    adaptables = {SlingHttpServletRequest.class, Resource.class},
    adapters = {MyModel.class, ComponentExporter.class},
    resourceType = MyModel.RESOURCE_TYPE,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
@Exporter(
    name = ExporterConstants.SLING_MODEL_EXPORTER_NAME,
    extensions = ExporterConstants.SLING_MODEL_EXTENSION
)
public class MyModel implements ComponentExporter {

    public static final String RESOURCE_TYPE = "myproject/components/mycomponent";

    // === Injection Examples ===

    // Request object
    @Self
    private SlingHttpServletRequest request;

    // Current resource
    @SlingObject
    private Resource resource;

    // Child resource as model
    @ChildResource(name = "items")
    private List<ItemModel> items;

    // OSGi service
    @OSGiService
    private MyService myService;

    // Value from ValueMap
    @ValueMapValue
    private String title;

    // Value with default
    @ValueMapValue
    private String subtitle = "Default Subtitle";

    // Script binding (currentPage, etc.)
    @ScriptVariable
    private Page currentPage;

    // Design/Style value
    @ScriptVariable
    private Style currentStyle;

    // Request attribute (from sightly include)
    @RequestAttribute(name = "showHeader")
    private Boolean showHeader;

    // Inject with filter
    @Inject
    @Filter("(component.name=com.myproject.core.services.impl.SpecificServiceImpl)")
    private MyService specificService;

    @PostConstruct
    protected void init() {
        // Post-construction logic
    }

    // === Getters ===

    public String getTitle() {
        return title;
    }

    public List<ItemModel> getItems() {
        return items;
    }

    @Override
    public String getExportedType() {
        return RESOURCE_TYPE;
    }
}
```

### Java 21 Record as Model (DTO)
```java
package com.myproject.core.models;

import com.fasterxml.jackson.annotation.JsonInclude;
import java.time.ZonedDateTime;
import java.util.List;
import java.util.Map;
import java.util.Objects;

@JsonInclude(JsonInclude.Include.NON_NULL)
public record ContentCard(
    String id,
    String title,
    String description,
    String url,
    String imagePath,
    List<String> tags,
    ZonedDateTime publishDate,
    Integer priority,
    Map<String, Object> metadata
) {
    // Compact constructor with validation
    public ContentCard {
        Objects.requireNonNull(id, "ID required");
        Objects.requireNonNull(title, "Title required");
        tags = tags != null ? List.copyOf(tags) : List.of();
        metadata = metadata != null ? Map.copyOf(metadata) : Map.of();
        priority = priority != null ? priority : 0;
    }

    // Static builder
    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private String id, title, description, url, imagePath;
        private List<String> tags;
        private ZonedDateTime publishDate;
        private Integer priority;
        private Map<String, Object> metadata;

        public Builder id(String id) { this.id = id; return this; }
        public Builder title(String title) { this.title = title; return this; }
        public Builder description(String d) { this.description = d; return this; }
        public Builder url(String url) { this.url = url; return this; }
        public Builder imagePath(String p) { this.imagePath = p; return this; }
        public Builder tags(List<String> t) { this.tags = t; return this; }
        public Builder publishDate(ZonedDateTime d) { this.publishDate = d; return this; }
        public Builder priority(Integer p) { this.priority = p; return this; }
        public Builder metadata(Map<String, Object> m) { this.metadata = m; return this; }

        public ContentCard build() {
            return new ContentCard(id, title, description, url, imagePath,
                tags, publishDate, priority, metadata);
        }
    }
}
```

---

## OSGi Services

### Basic OSGi Service
```java
package com.myproject.core.services;

public interface MyService {
    String process(String input);
}
```

```java
package com.myproject.core.services.impl;

import com.myproject.core.services.MyService;
import org.osgi.service.component.annotations.*;
import org.osgi.service.metatype.annotations.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = MyService.class,
    configurationPolicy = ConfigurationPolicy.OPTIONAL
)
@Designate(ocd = MyServiceImpl.Config.class)
public class MyServiceImpl implements MyService {

    private static final Logger LOG = LoggerFactory.getLogger(MyServiceImpl.class);

    @ObjectClassDefinition(
        name = "My Service Configuration",
        description = "Configuration for My Service"
    )
    public @interface Config {

        @AttributeDefinition(
            name = "Enabled",
            description = "Enable/disable the service"
        )
        boolean enabled() default true;

        @AttributeDefinition(
            name = "Max Items",
            description = "Maximum items to process"
        )
        int maxItems() default 100;

        @AttributeDefinition(
            name = "Allowed Paths",
            description = "Paths to process"
        )
        String[] allowedPaths() default {"/content"};
    }

    private boolean enabled;
    private int maxItems;
    private String[] allowedPaths;

    @Activate
    @Modified
    protected void activate(Config config) {
        this.enabled = config.enabled();
        this.maxItems = config.maxItems();
        this.allowedPaths = config.allowedPaths();
        LOG.info("MyService activated: enabled={}, maxItems={}", enabled, maxItems);
    }

    @Deactivate
    protected void deactivate() {
        LOG.info("MyService deactivated");
    }

    @Override
    public String process(String input) {
        if (!enabled) {
            return input;
        }
        // Processing logic
        return input.toUpperCase();
    }
}
```

### Service with Dynamic Reference
```java
@Component(service = AggregatorService.class)
public class AggregatorServiceImpl implements AggregatorService {

    @Reference(
        cardinality = ReferenceCardinality.OPTIONAL,
        policy = ReferencePolicy.DYNAMIC,
        policyOption = ReferencePolicyOption.GREEDY
    )
    private volatile CacheService cacheService;

    @Reference(
        cardinality = ReferenceCardinality.MULTIPLE,
        policy = ReferencePolicy.DYNAMIC
    )
    private volatile List<ContentProvider> providers;

    public List<Content> aggregate() {
        List<Content> result = new ArrayList<>();
        for (ContentProvider provider : providers) {
            result.addAll(provider.getContent());
        }

        // Use cache if available
        if (cacheService != null) {
            cacheService.put("aggregated", result);
        }

        return result;
    }
}
```

---

## Servlets

### GET Servlet (JSON Response)
```java
package com.myproject.core.servlets;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;
import org.apache.sling.servlets.annotations.SlingServletResourceTypes;
import org.osgi.service.component.annotations.Component;

import javax.servlet.Servlet;
import java.io.IOException;
import java.util.Map;

@Component(service = Servlet.class)
@SlingServletResourceTypes(
    resourceTypes = "myproject/components/mycomponent",
    selectors = "data",
    extensions = "json",
    methods = "GET"
)
public class DataServlet extends SlingSafeMethodsServlet {

    private final ObjectMapper mapper = new ObjectMapper();

    @Override
    protected void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response)
            throws IOException {

        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");

        // Get parameters
        String param = request.getParameter("param");
        int offset = getIntParam(request, "offset", 0);
        int count = getIntParam(request, "count", 10);

        // Build response
        Map<String, Object> result = Map.of(
            "success", true,
            "data", getData(request.getResource(), offset, count),
            "offset", offset,
            "count", count
        );

        mapper.writeValue(response.getWriter(), result);
    }

    private int getIntParam(SlingHttpServletRequest request, String name, int defaultValue) {
        String value = request.getParameter(name);
        if (value == null) return defaultValue;
        try {
            return Integer.parseInt(value);
        } catch (NumberFormatException e) {
            return defaultValue;
        }
    }

    private Object getData(Resource resource, int offset, int count) {
        // Implementation
        return List.of();
    }
}
```

### POST Servlet
```java
package com.myproject.core.servlets;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.servlets.annotations.SlingServletResourceTypes;
import org.osgi.service.component.annotations.Component;

import javax.servlet.Servlet;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component(service = Servlet.class)
@SlingServletResourceTypes(
    resourceTypes = "myproject/components/form",
    methods = "POST"
)
public class FormServlet extends SlingAllMethodsServlet {

    @Override
    protected void doPost(SlingHttpServletRequest request, SlingHttpServletResponse response)
            throws IOException {

        try {
            // Get form data
            String name = request.getParameter("name");
            String email = request.getParameter("email");

            // Validate
            if (name == null || email == null) {
                sendError(response, HttpServletResponse.SC_BAD_REQUEST, "Missing required fields");
                return;
            }

            // Process
            processForm(name, email);

            // Success response
            response.setStatus(HttpServletResponse.SC_OK);
            response.setContentType("application/json");
            response.getWriter().write("{\"success\": true}");

        } catch (Exception e) {
            sendError(response, HttpServletResponse.SC_INTERNAL_SERVER_ERROR, e.getMessage());
        }
    }

    private void sendError(SlingHttpServletResponse response, int status, String message)
            throws IOException {
        response.setStatus(status);
        response.setContentType("application/json");
        response.getWriter().write("{\"error\": \"" + message + "\"}");
    }

    private void processForm(String name, String email) {
        // Implementation
    }
}
```

---

## Sling Jobs

### Job Consumer
```java
package com.myproject.core.jobs;

import org.apache.sling.event.jobs.Job;
import org.apache.sling.event.jobs.consumer.JobConsumer;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = JobConsumer.class,
    property = {
        JobConsumer.PROPERTY_TOPICS + "=myproject/jobs/process"
    }
)
public class ProcessJobConsumer implements JobConsumer {

    private static final Logger LOG = LoggerFactory.getLogger(ProcessJobConsumer.class);

    @Override
    public JobResult process(Job job) {
        String path = job.getProperty("path", String.class);
        LOG.info("Processing job for path: {}", path);

        try {
            // Process logic
            doProcess(path);
            return JobResult.OK;

        } catch (TransientException e) {
            LOG.warn("Transient error, will retry: {}", e.getMessage());
            return JobResult.FAILED;

        } catch (Exception e) {
            LOG.error("Permanent error: {}", e.getMessage(), e);
            return JobResult.CANCEL;
        }
    }

    private void doProcess(String path) {
        // Implementation
    }
}
```

### Job Scheduler
```java
package com.myproject.core.schedulers;

import org.apache.sling.event.jobs.JobManager;
import org.apache.sling.event.jobs.ScheduledJobInfo;
import org.osgi.service.component.annotations.*;
import org.osgi.service.metatype.annotations.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;

@Component(immediate = true, configurationPolicy = ConfigurationPolicy.REQUIRE)
@Designate(ocd = MyScheduler.Config.class)
public class MyScheduler {

    private static final Logger LOG = LoggerFactory.getLogger(MyScheduler.class);
    public static final String JOB_TOPIC = "myproject/jobs/scheduled";

    @ObjectClassDefinition(name = "My Scheduler")
    public @interface Config {
        @AttributeDefinition(name = "Enabled")
        boolean enabled() default true;

        @AttributeDefinition(name = "Cron Expression")
        String cronExpression() default "0 0 * * * ?";
    }

    @Reference
    private JobManager jobManager;

    private boolean enabled;
    private String cronExpression;

    @Activate
    protected void activate(Config config) {
        this.enabled = config.enabled();
        this.cronExpression = config.cronExpression();

        if (enabled) {
            scheduleJob();
        }
    }

    @Deactivate
    protected void deactivate() {
        // Don't unschedule in cluster environment
    }

    private void scheduleJob() {
        // Check if already scheduled
        Collection<ScheduledJobInfo> existing = jobManager.getScheduledJobs(JOB_TOPIC, 1, null);
        if (!existing.isEmpty()) {
            LOG.debug("Job already scheduled");
            return;
        }

        ScheduledJobInfo info = jobManager.createJob(JOB_TOPIC)
            .properties(Map.of("scheduleName", "my-scheduled-job"))
            .schedule()
            .cron(cronExpression)
            .add();

        if (info != null) {
            LOG.info("Scheduled job with cron: {}", cronExpression);
        }
    }
}
```

---

## Sling Filter

```java
package com.myproject.core.filters;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.servlets.annotations.SlingServletFilter;
import org.apache.sling.servlets.annotations.SlingServletFilterScope;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.*;
import java.io.IOException;

@Component(service = Filter.class)
@SlingServletFilter(
    scope = SlingServletFilterScope.REQUEST,
    pattern = "/content/mysite/.*",
    order = -500
)
public class MyFilter implements Filter {

    private static final Logger LOG = LoggerFactory.getLogger(MyFilter.class);

    @Override
    public void init(FilterConfig filterConfig) {
        // Initialization
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        if (!(request instanceof SlingHttpServletRequest)) {
            chain.doFilter(request, response);
            return;
        }

        SlingHttpServletRequest slingRequest = (SlingHttpServletRequest) request;
        SlingHttpServletResponse slingResponse = (SlingHttpServletResponse) response;

        // Pre-processing
        LOG.debug("Request to: {}", slingRequest.getRequestURI());
        slingRequest.setAttribute("myAttribute", "value");

        // Continue filter chain
        chain.doFilter(request, response);

        // Post-processing
        slingResponse.setHeader("X-Custom-Header", "value");
    }

    @Override
    public void destroy() {
        // Cleanup
    }
}
```

---

## Context-Aware Configuration

### CA Config Interface
```java
package com.myproject.core.caconfig;

import org.apache.sling.caconfig.annotation.Configuration;
import org.apache.sling.caconfig.annotation.Property;

@Configuration(
    label = "My Site Configuration",
    description = "Site-specific settings"
)
public @interface SiteConfig {

    @Property(label = "API Endpoint")
    String apiEndpoint() default "";

    @Property(label = "Cache TTL (seconds)")
    int cacheTtlSeconds() default 300;

    @Property(label = "Features Enabled")
    String[] enabledFeatures() default {};

    @Property(label = "Enable Analytics")
    boolean analyticsEnabled() default true;
}
```

### Using CA Config in Model
```java
@Model(adaptables = Resource.class)
public class MyModel {

    @Self
    private Resource resource;

    private SiteConfig config;

    @PostConstruct
    protected void init() {
        ConfigurationBuilder builder = resource.adaptTo(ConfigurationBuilder.class);
        if (builder != null) {
            config = builder.as(SiteConfig.class);
        }
    }

    public String getApiEndpoint() {
        return config != null ? config.apiEndpoint() : "";
    }

    public boolean isAnalyticsEnabled() {
        return config != null && config.analyticsEnabled();
    }
}
```

---

## Workflow Process Step

```java
package com.myproject.core.workflow;

import com.adobe.granite.workflow.WorkflowException;
import com.adobe.granite.workflow.WorkflowSession;
import com.adobe.granite.workflow.exec.WorkItem;
import com.adobe.granite.workflow.exec.WorkflowProcess;
import com.adobe.granite.workflow.metadata.MetaDataMap;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = WorkflowProcess.class,
    property = {"process.label=My Custom Process"}
)
public class MyWorkflowProcess implements WorkflowProcess {

    private static final Logger LOG = LoggerFactory.getLogger(MyWorkflowProcess.class);

    @Override
    public void execute(WorkItem workItem, WorkflowSession session, MetaDataMap args)
            throws WorkflowException {

        String payload = workItem.getWorkflowData().getPayload().toString();
        LOG.info("Processing workflow for: {}", payload);

        // Get process arguments
        String action = args.get("PROCESS_ARGS", "default");

        try {
            // Process logic
            processPayload(payload, action, session);

        } catch (Exception e) {
            LOG.error("Workflow processing failed", e);
            throw new WorkflowException("Processing failed", e);
        }
    }

    private void processPayload(String payload, String action, WorkflowSession session) {
        // Implementation
    }
}
```

---

## HTL Patterns

### Basic Component
```html
<sly data-sly-use.model="com.myproject.core.models.MyModel"/>

<div class="cmp-mycomponent" data-cmp-is="mycomponent">
    <sly data-sly-test="${model.title}">
        <h2 class="cmp-mycomponent__title">${model.title}</h2>
    </sly>

    <sly data-sly-test="${model.description}">
        <p class="cmp-mycomponent__description">${model.description @ context='html'}</p>
    </sly>
</div>
```

### List Iteration
```html
<sly data-sly-use.model="com.myproject.core.models.ListModel"/>

<ul class="cmp-list" data-sly-test="${model.items.size > 0}">
    <sly data-sly-list.item="${model.items}">
        <li class="cmp-list__item" data-index="${itemList.index}">
            <a href="${item.url @ extension='html'}" class="cmp-list__link">
                ${item.title}
            </a>
            <sly data-sly-test="${itemList.first}"><!-- First item --></sly>
            <sly data-sly-test="${itemList.last}"><!-- Last item --></sly>
        </li>
    </sly>
</ul>

<div data-sly-test="${model.items.size == 0}" class="cmp-list--empty">
    ${'No items available' @ i18n}
</div>
```

### Template Include
```html
<!-- Main component -->
<sly data-sly-use.templates="templates/card.html"/>

<div class="cmp-cards">
    <sly data-sly-list.card="${model.cards}">
        <sly data-sly-call="${templates.card @ card=card, index=cardList.index}"/>
    </sly>
</div>

<!-- templates/card.html -->
<template data-sly-template.card="${@ card, index}">
    <article class="cmp-card" style="--index: ${index}">
        <h3 class="cmp-card__title">${card.title}</h3>
        <p class="cmp-card__description">${card.description}</p>
    </article>
</template>
```

### Resource Include
```html
<!-- Include child resource -->
<sly data-sly-resource="${'header' @ resourceType='myproject/components/header'}"/>

<!-- Include with decoration -->
<sly data-sly-resource="${'content' @ decorationTagName='section', cssClassName='content-section'}"/>

<!-- Include with wrapper -->
<div class="component-wrapper">
    <sly data-sly-resource="${@ path='parsys', resourceType='wcm/foundation/components/parsys'}"/>
</div>
```

### Data Attributes from Model
```html
<sly data-sly-use.model="com.myproject.core.models.MyModel"/>

<div class="${model.cssClasses}"
     id="${model.componentId}"
     style="${model.inlineStyles}"
     data-sly-attribute="${model.dataAttributes}">
    <!-- Content -->
</div>
```

---

## Dialog Snippets

### Basic Dialog Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="My Component"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <tabs jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/tabs"
                maximized="{Boolean}true">
                <items jcr:primaryType="nt:unstructured">
                    <!-- Tab 1 -->
                    <general jcr:primaryType="nt:unstructured"
                        jcr:title="General"
                        sling:resourceType="granite/ui/components/coral/foundation/container"
                        margin="{Boolean}true">
                        <items jcr:primaryType="nt:unstructured">
                            <!-- Fields here -->
                        </items>
                    </general>
                </items>
            </tabs>
        </items>
    </content>
</jcr:root>
```

### Common Dialog Fields
```xml
<!-- Textfield -->
<title jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Title"
    fieldDescription="Enter the title"
    name="./title"
    required="{Boolean}true"
    maxlength="100"/>

<!-- Textarea -->
<description jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
    fieldLabel="Description"
    name="./description"
    rows="4"/>

<!-- Pathfield -->
<link jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
    fieldLabel="Link"
    name="./linkPath"
    rootPath="/content"/>

<!-- Select -->
<variant jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/select"
    fieldLabel="Variant"
    name="./variant">
    <items jcr:primaryType="nt:unstructured">
        <default jcr:primaryType="nt:unstructured"
            text="Default"
            value="default"
            selected="{Boolean}true"/>
        <outlined jcr:primaryType="nt:unstructured"
            text="Outlined"
            value="outlined"/>
    </items>
</variant>

<!-- Checkbox -->
<enabled jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/checkbox"
    text="Enable feature"
    name="./enabled"
    value="{Boolean}true"
    checked="{Boolean}true"/>

<!-- Multifield -->
<items jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
    composite="{Boolean}true"
    fieldLabel="Items">
    <field jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container"
        name="./items">
        <items jcr:primaryType="nt:unstructured">
            <title jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                fieldLabel="Title"
                name="./title"/>
            <path jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                fieldLabel="Path"
                name="./path"/>
        </items>
    </field>
</items>

<!-- ColorField -->
<backgroundColor jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/colorfield"
    fieldLabel="Background Color"
    name="./backgroundColor"/>

<!-- NumberField -->
<count jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/numberfield"
    fieldLabel="Count"
    name="./count"
    min="1"
    max="100"
    value="10"/>

<!-- Tag Field -->
<tags jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/coral/common/form/tagfield"
    fieldLabel="Tags"
    name="./cq:tags"
    multiple="{Boolean}true"/>
```

---

## Service User Mapping

**OSGi Config: `org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended-myproject.cfg.json`**
```json
{
    "user.mapping": [
        "com.myproject.core:content-reader=[content-reader-service]",
        "com.myproject.core:content-writer=[content-writer-service]"
    ]
}
```

**Usage in Service:**
```java
private ResourceResolver getServiceResolver(String subService) {
    try {
        return resolverFactory.getServiceResourceResolver(
            Map.of(ResourceResolverFactory.SUBSERVICE, subService)
        );
    } catch (LoginException e) {
        LOG.error("Cannot get service resolver for: {}", subService, e);
        return null;
    }
}
```

---

## ClientLib Definition

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:primaryType="cq:ClientLibraryFolder"
    categories="[myproject.components.mycomponent]"
    dependencies="[myproject.base]"
    allowProxy="{Boolean}true"/>
```

---

This reference provides copy-paste ready snippets for the most common AEM development tasks. Always adapt to your specific project conventions and requirements.
