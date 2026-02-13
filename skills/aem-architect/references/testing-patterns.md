# AEM Unit Testing Patterns

## Overview

This reference covers unit testing patterns for AEM components using:
- **JUnit 5** (Jupiter) - Modern Java testing framework
- **AEM Mocks** (wcm.io) - Mock framework for AEM/Sling
- **Mockito** - Mocking framework for dependencies

Patterns are based on [AEM Core WCM Components](https://github.com/adobe/aem-core-wcm-components) testing approach.

---

## Maven Dependencies

```xml
<!-- JUnit 5 -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- AEM Mocks (wcm.io) -->
<dependency>
    <groupId>io.wcm</groupId>
    <artifactId>io.wcm.testing.aem-mock.junit5</artifactId>
    <scope>test</scope>
</dependency>

<!-- Mockito -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- Hamcrest (optional, for matchers) -->
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <scope>test</scope>
</dependency>
```

---

## Pattern 1: Basic Sling Model Test

The simplest pattern for testing a Sling Model.

```java
package com.myproject.core.models;

import io.wcm.testing.mock.aem.junit5.AemContext;
import io.wcm.testing.mock.aem.junit5.AemContextExtension;
import org.apache.sling.testing.mock.sling.ResourceResolverType;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(AemContextExtension.class)
class TitleModelTest {

    private final AemContext context = new AemContext(ResourceResolverType.JCR_MOCK);

    private static final String CONTENT_ROOT = "/content/mysite";
    private static final String TITLE_PATH = CONTENT_ROOT + "/jcr:content/root/title";

    @BeforeEach
    void setUp() {
        // Load test content from JSON
        context.load().json("/title/test-content.json", CONTENT_ROOT);

        // Register the Sling Model
        context.addModelsForClasses(TitleModel.class);
    }

    @Test
    void testTitle() {
        // Set current resource
        context.currentResource(TITLE_PATH);

        // Adapt to model
        TitleModel model = context.request().adaptTo(TitleModel.class);

        // Assert
        assertNotNull(model);
        assertEquals("Hello World", model.getText());
        assertEquals("h2", model.getType());
    }

    @Test
    void testEmptyTitle() {
        context.currentResource(CONTENT_ROOT + "/jcr:content/root/empty-title");

        TitleModel model = context.request().adaptTo(TitleModel.class);

        assertNotNull(model);
        assertNull(model.getText());
    }
}
```

### Test Content JSON (`/src/test/resources/title/test-content.json`)
```json
{
    "jcr:content": {
        "root": {
            "title": {
                "jcr:primaryType": "nt:unstructured",
                "sling:resourceType": "myproject/components/title",
                "jcr:title": "Hello World",
                "type": "h2"
            },
            "empty-title": {
                "jcr:primaryType": "nt:unstructured",
                "sling:resourceType": "myproject/components/title"
            }
        }
    }
}
```

---

## Pattern 2: Test Context Factory (Core Components Pattern)

Create a reusable test context with common setup.

```java
package com.myproject.core.testcontext;

import com.google.common.collect.ImmutableMap;
import io.wcm.testing.mock.aem.junit5.AemContext;
import io.wcm.testing.mock.aem.junit5.AemContextBuilder;
import org.apache.commons.lang3.ArrayUtils;
import org.apache.sling.testing.mock.sling.ResourceResolverType;

import static org.apache.sling.testing.mock.caconfig.ContextPlugins.CACONFIG;

public final class MyProjectTestContext {

    public static final String TEST_CONTENT_JSON = "/test-content.json";
    public static final String TEST_APPS_JSON = "/test-apps.json";

    private MyProjectTestContext() {
        // Static only
    }

    private static final ImmutableMap<String, Object> PROPERTIES =
        ImmutableMap.of(
            "resource.resolver.mapping", ArrayUtils.toArray("/:/")
        );

    /**
     * Create a new AemContext with standard configuration.
     */
    public static AemContext newAemContext() {
        return new AemContextBuilder()
            .plugin(CACONFIG)  // Context-Aware Config support
            .resourceResolverType(ResourceResolverType.JCR_MOCK)
            .resourceResolverFactoryActivatorProps(PROPERTIES)
            .<AemContext>afterSetUp(context -> {
                // Register common services
                registerCommonServices(context);
            })
            .build();
    }

    private static void registerCommonServices(AemContext context) {
        // Register mock services needed by most tests
        // Example: context.registerService(MyService.class, new MockMyService());
    }

    /**
     * Load standard test content.
     */
    public static void loadTestContent(AemContext context, String testBase) {
        context.load().json(testBase + TEST_CONTENT_JSON, "/content");
        context.load().json(testBase + TEST_APPS_JSON, "/apps");
    }
}
```

### Using the Context Factory
```java
@ExtendWith(AemContextExtension.class)
class MyComponentTest {

    private final AemContext context = MyProjectTestContext.newAemContext();

    @BeforeEach
    void setUp() {
        MyProjectTestContext.loadTestContent(context, "/mycomponent");
        context.addModelsForClasses(MyModel.class);
    }
}
```

---

## Pattern 3: Testing with Mock OSGi Services

```java
@ExtendWith({AemContextExtension.class, MockitoExtension.class})
class ContentHubModelTest {

    private final AemContext context = new AemContext(ResourceResolverType.JCR_MOCK);

    @Mock
    private ContentAggregationService aggregationService;

    @Mock
    private PersonalizationService personalizationService;

    @BeforeEach
    void setUp() {
        // Register mock services BEFORE loading models
        context.registerService(ContentAggregationService.class, aggregationService);
        context.registerService(PersonalizationService.class, personalizationService);

        // Load content and models
        context.load().json("/contenthub/test-content.json", "/content");
        context.addModelsForClasses(ContentHubModel.class);
    }

    @Test
    void testContentLoading() {
        // Configure mock behavior
        List<ContentCard> mockCards = List.of(
            new ContentCard("1", "Card 1", null, null, null, null, null, null, null, null, null, null, null),
            new ContentCard("2", "Card 2", null, null, null, null, null, null, null, null, null, null, null)
        );

        when(aggregationService.aggregateContent(any(), anyInt(), any()))
            .thenReturn(mockCards);

        // Test
        context.currentResource("/content/page/jcr:content/root/contenthub");
        ContentHubModel model = context.request().adaptTo(ContentHubModel.class);

        // Verify
        assertNotNull(model);
        assertEquals(2, model.getContentCards().size());
        verify(aggregationService).aggregateContent(any(), anyInt(), any());
    }
}
```

---

## Pattern 4: Testing with @InjectMocks

For services with many dependencies:

```java
@ExtendWith(MockitoExtension.class)
class ContentAggregationServiceTest {

    @Mock
    private QueryBuilder queryBuilder;

    @Mock
    private CacheService cacheService;

    @InjectMocks
    private ContentAggregationServiceImpl service;

    @Mock
    private ResourceResolver resolver;

    @Mock
    private PageManager pageManager;

    @BeforeEach
    void setUp() {
        when(resolver.adaptTo(PageManager.class)).thenReturn(pageManager);
    }

    @Test
    void testAggregateContent() {
        // Setup
        Page mockPage = mock(Page.class);
        when(mockPage.getTitle()).thenReturn("Test Page");
        when(mockPage.getPath()).thenReturn("/content/test");

        when(pageManager.getPage("/content/articles"))
            .thenReturn(mockPage);

        ContentSource source = mock(ContentSource.class);
        when(source.getSourceType()).thenReturn(ContentSource.SourceType.PAGES);
        when(source.getSourcePath()).thenReturn("/content/articles");
        when(source.isValid()).thenReturn(true);

        // Execute
        List<ContentCard> result = service.aggregateContent(
            List.of(source), 10, resolver);

        // Verify
        assertNotNull(result);
    }
}
```

---

## Pattern 5: Nested Test Classes

Organize tests using JUnit 5's `@Nested`:

```java
@ExtendWith(AemContextExtension.class)
class ImageModelTest {

    private final AemContext context = new AemContext();

    @BeforeEach
    void setUp() {
        context.load().json("/image/test-content.json", "/content");
        context.addModelsForClasses(ImageModel.class);
    }

    @Nested
    @DisplayName("Basic Image Tests")
    class BasicTests {

        @Test
        @DisplayName("Should return image source")
        void testImageSrc() {
            context.currentResource("/content/page/jcr:content/image");
            ImageModel model = context.request().adaptTo(ImageModel.class);

            assertNotNull(model.getSrc());
            assertTrue(model.getSrc().contains(".img."));
        }

        @Test
        @DisplayName("Should return alt text")
        void testAltText() {
            context.currentResource("/content/page/jcr:content/image");
            ImageModel model = context.request().adaptTo(ImageModel.class);

            assertEquals("Test Image", model.getAlt());
        }
    }

    @Nested
    @DisplayName("Lazy Loading Tests")
    class LazyLoadingTests {

        @Test
        void testLazyEnabled() {
            context.currentResource("/content/page/jcr:content/lazy-image");
            ImageModel model = context.request().adaptTo(ImageModel.class);

            assertTrue(model.isLazyEnabled());
        }
    }

    @Nested
    @DisplayName("Edge Cases")
    class EdgeCases {

        @Test
        void testMissingImage() {
            context.currentResource("/content/page/jcr:content/no-image");
            ImageModel model = context.request().adaptTo(ImageModel.class);

            assertNull(model.getSrc());
        }
    }
}
```

---

## Pattern 6: Testing Servlets

```java
@ExtendWith(AemContextExtension.class)
class LoadMoreServletTest {

    private final AemContext context = new AemContext();

    @Mock
    private ContentAggregationService aggregationService;

    private LoadMoreServlet servlet;

    @BeforeEach
    void setUp() {
        context.registerService(ContentAggregationService.class, aggregationService);

        servlet = context.registerInjectActivateService(new LoadMoreServlet());

        context.load().json("/servlet/test-content.json", "/content");
    }

    @Test
    void testDoGet() throws Exception {
        // Setup mock response
        when(aggregationService.loadMore(any(), anyInt(), anyInt(), any()))
            .thenReturn(List.of(
                createMockCard("1", "Card 1"),
                createMockCard("2", "Card 2")
            ));

        // Configure request
        context.currentResource("/content/page/jcr:content/hub");
        context.request().setQueryString("offset=12&count=6");

        // Execute
        servlet.doGet(context.request(), context.response());

        // Verify
        assertEquals(200, context.response().getStatus());
        assertEquals("application/json", context.response().getContentType());

        String json = context.response().getOutputAsString();
        assertTrue(json.contains("\"cards\""));
        assertTrue(json.contains("Card 1"));
    }

    @Test
    void testDoGetWithInvalidParams() throws Exception {
        context.currentResource("/content/page/jcr:content/hub");
        context.request().setQueryString("offset=invalid");

        servlet.doGet(context.request(), context.response());

        // Should use default offset of 0
        assertEquals(200, context.response().getStatus());
    }

    private ContentCard createMockCard(String id, String title) {
        return ContentCard.builder().id(id).title(title).build();
    }
}
```

---

## Pattern 7: Testing with Content Policies

```java
@ExtendWith(AemContextExtension.class)
class ComponentWithPolicyTest {

    private final AemContext context = new AemContext();

    @BeforeEach
    void setUp() {
        context.load().json("/component/test-content.json", "/content");
        context.load().json("/component/test-conf.json", "/conf");
        context.addModelsForClasses(MyModel.class);
    }

    @Test
    void testWithContentPolicy() {
        // Map content policy to resource type
        context.contentPolicyMapping(
            "myproject/components/mycomponent",
            "allowedColors", new String[]{"red", "blue", "green"},
            "defaultSize", "medium"
        );

        context.currentResource("/content/page/jcr:content/component");
        MyModel model = context.request().adaptTo(MyModel.class);

        assertEquals("medium", model.getDefaultSize());
        assertArrayEquals(
            new String[]{"red", "blue", "green"},
            model.getAllowedColors()
        );
    }
}
```

---

## Pattern 8: Testing JSON Exporter

```java
@Test
void testJsonExport() throws Exception {
    context.currentResource("/content/page/jcr:content/component");
    MyModel model = context.request().adaptTo(MyModel.class);

    // Use utility to test JSON export
    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(model);

    // Verify JSON structure
    JsonNode root = mapper.readTree(json);
    assertEquals("expected-value", root.get("propertyName").asText());
    assertTrue(root.has(":type")); // Sling Model Exporter adds this
}

// Or use Core Components utility pattern
@Test
void testJSONExport() {
    context.currentResource("/content/page/jcr:content/component");
    MyModel model = context.request().adaptTo(MyModel.class);

    // Compare against expected JSON file
    Utils.testJSONExport(model, "/component/expected-output.json");
}
```

---

## Pattern 9: Testing OSGi Services with Activate/Modified

```java
@ExtendWith({AemContextExtension.class, MockitoExtension.class})
class CacheServiceTest {

    private final AemContext context = new AemContext();

    @Test
    void testServiceActivation() {
        // Register and activate service with config
        CacheServiceImpl service = context.registerInjectActivateService(
            new CacheServiceImpl(),
            "maxSize", 500,
            "defaultTtlSeconds", 600,
            "enabled", true
        );

        // Verify service is properly configured
        Map<String, Object> stats = service.getStats();
        assertEquals(500, stats.get("maxSize"));
        assertTrue((boolean) stats.get("enabled"));
    }

    @Test
    void testServiceModification() {
        CacheServiceImpl service = context.registerInjectActivateService(
            new CacheServiceImpl(),
            "enabled", true
        );

        // Verify initial state
        assertTrue((boolean) service.getStats().get("enabled"));

        // Note: Testing @Modified requires re-registration or reflection
    }
}
```

---

## Pattern 10: Loading Binary Files (Images, etc.)

```java
@BeforeEach
void setUp() {
    // Load JSON content
    context.load().json("/image/test-content.json", "/content");
    context.load().json("/image/test-dam.json", "/content/dam");

    // Load binary files
    context.load().binaryFile(
        "/image/test-image.png",
        "/content/dam/images/test.png/jcr:content/renditions/original"
    );

    context.load().binaryFile(
        "/image/test-image.png",
        "/content/page/jcr:content/image/file",
        "image/png"  // MIME type
    );
}
```

---

## Assertion Best Practices

### Use Specific Assertions
```java
// ✅ Good - specific assertions
assertEquals("Expected Title", model.getTitle());
assertTrue(model.isEnabled());
assertNotNull(model.getItems());
assertFalse(model.getItems().isEmpty());

// ❌ Avoid - generic assertions
assertTrue(model.getTitle().equals("Expected Title"));
assertTrue(model.isEnabled() == true);
```

### Use AssertAll for Multiple Assertions
```java
@Test
void testMultipleProperties() {
    MyModel model = context.request().adaptTo(MyModel.class);

    assertAll("Model properties",
        () -> assertEquals("Title", model.getTitle()),
        () -> assertEquals("Subtitle", model.getSubtitle()),
        () -> assertTrue(model.isEnabled()),
        () -> assertNotNull(model.getItems())
    );
}
```

### Use Descriptive Test Names
```java
@Test
@DisplayName("Should return empty list when no content sources configured")
void shouldReturnEmptyListWhenNoContentSourcesConfigured() {
    // ...
}
```

---

## Directory Structure

```
src/test/
├── java/
│   └── com/myproject/core/
│       ├── models/
│       │   ├── TitleModelTest.java
│       │   └── ImageModelTest.java
│       ├── services/
│       │   └── impl/
│       │       └── CacheServiceTest.java
│       ├── servlets/
│       │   └── LoadMoreServletTest.java
│       └── testcontext/
│           └── MyProjectTestContext.java
└── resources/
    ├── title/
    │   └── test-content.json
    ├── image/
    │   ├── test-content.json
    │   ├── test-dam.json
    │   └── test-image.png
    └── component/
        ├── test-content.json
        └── expected-output.json
```

---

## References

- [wcm.io AEM Mocks](https://wcm.io/testing/aem-mock/)
- [AEM Core WCM Components Tests](https://github.com/adobe/aem-core-wcm-components/tree/main/bundles/core/src/test)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://site.mockito.org/)
