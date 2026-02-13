---
name: aem-architect
description: Comprehensive AEM (Adobe Experience Manager) architecture design skill for enterprise implementations. Use this skill when building AEM components, designing content architecture, configuring Dispatcher caching, creating dialogs with Granite UI, implementing Sling Models, OSGi services, servlets, or ClientLibs. Covers component development, performance optimization, caching strategies, security best practices, and CI/CD with Cloud Manager. Supports both AEM 6.5 and AEM as a Cloud Service. Also creates architecture diagrams (Mermaid, ASCII) for AEM systems. Triggers on requests for AEM development guidance, component creation, dialog design, caching configuration, architectural decisions, or diagram/visualization requests for AEM projects.
---

# AEM Architect Skill

Enterprise-grade AEM solution design and implementation guidance.

---

## Workflow Routing

Route to the appropriate workflow based on the request:

| Request Type | Workflow | Use When |
|--------------|----------|----------|
| **Diagrams/Visualizations** | [workflows/AEMDiagrams.md](workflows/AEMDiagrams.md) | Architecture diagrams, flowcharts, sequence diagrams, ASCII art |
| **Component Development** | Main workflow below | Building new components, dialogs, HTL |
| **Service/Backend** | Main workflow below | OSGi services, Sling Models, servlets |
| **Infrastructure** | Main workflow below | Dispatcher, caching, CI/CD |

### Diagram Request Detection

Use the **AEMDiagrams** workflow when the user mentions:
- "diagram", "flowchart", "sequence diagram", "architecture diagram"
- "visualize", "draw", "show me", "illustrate"
- "mermaid", "ASCII art", "box diagram"
- "component structure", "request flow", "cache flow"
- "service dependencies", "class diagram", "state diagram"

---

## Main Workflow

Follow this structured workflow for every AEM implementation request:

### Phase 1: Requirements Gathering

**Step 1.1: Ask for User Preference**

First, ask the user:
> "Would you like me to ask you some discovery questions to provide a more precise and tailored solution?
> - **Yes** - I'll ask targeted questions about your requirements
> - **No** - I'll proceed autonomously based on best practices and available context"

**Step 1.2: Check for Attached Requirements**

If the user has attached a file (`.txt`, `.doc`, `.docx`, or `.md`):
1. Read and analyze the file for requirements
2. Extract answers to the discovery questions below from the document
3. If user accepted questions in Step 1.1, ask additional precision questions for any gaps or ambiguities found

**Step 1.3: Discovery Questions (if user accepted)**

Ask relevant questions from these categories:

**Component Architecture**
- Component type? (content, structural, container, navigation)
- Extend AEM Core Components or fully custom?
- Required dialog fields? (text, image, pathfield, multifield, nested)
- Target environment? (author, publish, or both)

**Performance & Caching**
- Expected traffic volume? (low, medium, high, enterprise-scale)
- Caching strategy? (aggressive, moderate, dynamic)
- CDN integration required?

**Integration**
- Headless/hybrid JSON APIs required?
- External service integrations?
- GraphQL for Content Fragments?
- SPA framework integration?

### Phase 2: Analysis & Planning

**Step 2.1: Present High-Level Overview**

After gathering requirements (or analyzing autonomously), present:

> **Understanding Summary**
>
> Based on my analysis, here's what I understand about your requirements:
> - [Bullet points summarizing the project scope]
> - [Target environment and architecture]
> - [Key components and features needed]
> - [Performance and caching considerations]
> - [Integration points]

**Step 2.2: Present TODO List**

Create and present a numbered task list:

> **Implementation Plan**
>
> I will implement the following tasks:
>
> 1. [ ] Task 1: Description
> 2. [ ] Task 2: Description
> 3. [ ] Task 3: Description
> ...
>
> **Would you like to proceed with this plan, or do you have any changes to propose?**

**Step 2.3: Wait for User Approval**

- If user approves → Proceed to Phase 3
- If user proposes changes → Update the plan and present again
- Do NOT start implementation until user confirms

### Phase 3: Implementation

Execute the approved TODO list:
1. Create component structure
2. Implement Sling Model with proper injectors
3. Design dialog using Granite UI patterns
4. Configure ClientLibs with bundling strategy
5. Set up Dispatcher rules
6. Provide testing guidance

### Phase 4: Validation

Present completed work with:
- Summary of what was implemented
- File locations and structure
- Testing recommendations
- Cloud Manager quality gate considerations

## Reference Documentation

Read the appropriate reference based on the task:

### Workflows
| Workflow | File | Use When |
|----------|------|----------|
| **AEM Diagrams** | [workflows/AEMDiagrams.md](workflows/AEMDiagrams.md) | Creating Mermaid or ASCII architecture diagrams |

### Core References (in this skill)
| Topic | File | Use When |
|-------|------|----------|
| **Architecture Patterns** | [references/architecture-deep-dive.md](references/architecture-deep-dive.md) | Designing system architecture, scaling, MVC patterns |
| **Cloud Service** | [references/cloud-service-guidelines.md](references/cloud-service-guidelines.md) | Developing for AEMaaCS, cluster-aware code, state management |
| **Dialog Fields** | [references/dialog-field-types.md](references/dialog-field-types.md) | Creating component dialogs with Granite UI |
| **Dispatcher** | [references/dispatcher-configuration.md](references/dispatcher-configuration.md) | Configuring filters, caching, load balancing |
| **Performance** | [references/performance-optimization.md](references/performance-optimization.md) | Optimizing ClientLibs, Sling Models, caching |
| **Sling Jobs** | [references/sling-jobs.md](references/sling-jobs.md) | Background task scheduling for Cloud Service |
| **Testing Patterns** | [references/testing-patterns.md](references/testing-patterns.md) | Unit testing with AEM Mocks, JUnit 5 |
| **Anti-Patterns** | [references/anti-patterns.md](references/anti-patterns.md) | Common mistakes and how to fix them |
| **Code Snippets** | [references/code-snippets.md](references/code-snippets.md) | Copy-paste ready code templates |

### External Guides (in project root)
| Topic | File | Use When |
|-------|------|----------|
| Sling Models | [aem-sling-models-guide.md](../../../aem-sling-models-guide.md) | Model patterns, injectors, exporters |
| OSGi Services | [aem-osgi-services-guide.md](../../../aem-osgi-services-guide.md) | Service design, dependency injection |
| Servlets | [aem-servlets-guide.md](../../../aem-servlets-guide.md) | Endpoint patterns, security |
| Dialogs | [aem-dialog-creation-guide.md](../../../aem-dialog-creation-guide.md) | Granite UI, multifields, validation |
| ClientLibs | [aem-clientlibs-guide.md](../../../aem-clientlibs-guide.md) | JS/CSS bundling, lazy loading |
| Dispatcher | [aem-dispatcher-guide.md](../../../aem-dispatcher-guide.md) | Detailed cache configuration |
| Components | [component-development.md](../../../component-development.md) | HTL, component structure |
| Best Practices | [adobe-technical-practices.md](../../../adobe-technical-practices.md) | Adobe-recommended patterns |

## Quick Reference: Component Creation

### Component Structure
```
/apps/myproject/components/mycomponent/
├── .content.xml                    # Component definition
├── _cq_dialog/.content.xml         # Author dialog
├── _cq_editConfig.xml              # Edit configuration
├── mycomponent.html                # HTL template
└── clientlib/                      # Component CSS/JS
```

### Component Definition
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="My Component"
    jcr:description="Description"
    componentGroup="My Project - Content"
    sling:resourceSuperType="core/wcm/components/title/v3/title"/>
```

### Sling Model Pattern
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

    @Override
    public String getTitle() { return title; }
}
```

### Common Dialog Fields

| Field | Resource Type |
|-------|---------------|
| Text | `granite/ui/components/coral/foundation/form/textfield` |
| Textarea | `granite/ui/components/coral/foundation/form/textarea` |
| Rich Text | `cq/gui/components/authoring/dialog/richtext` |
| Checkbox | `granite/ui/components/coral/foundation/form/checkbox` |
| Select | `granite/ui/components/coral/foundation/form/select` |
| Path | `granite/ui/components/coral/foundation/form/pathfield` |
| Image | `cq/gui/components/authoring/dialog/fileupload` |
| Multifield | `granite/ui/components/coral/foundation/form/multifield` |

For complete field examples, see [references/dialog-field-types.md](references/dialog-field-types.md).

## Caching Decision Tree

```
Is content personalized?
├── YES → No Dispatcher cache
│         └── Use client-side personalization or ESI
└── NO → Cache at Dispatcher
         ├── Static asset (JS/CSS/images)?
         │   └── Long TTL (1 year) + content-hash
         ├── HTML content?
         │   └── Moderate TTL (5-60 min) + statfileslevel
         └── API/JSON?
             ├── Public → TTL based on freshness needs
             └── Private → No Dispatcher cache
```

### Statfileslevel Reference

| Structure | Level | Effect |
|-----------|-------|--------|
| `/content/site` | 2 | Per-site invalidation |
| `/content/site/language` | 3 | Per-language invalidation |
| `/content/site/region/country` | 4 | Per-country invalidation |

## Security Checklist

### Dispatcher Filters (Default Deny)
```apache
/filter {
    /0001 { /type "deny" /glob "*" }
    /0100 { /type "allow" /method "GET" /url "/content/mysite/*" }
    /0200 { /type "allow" /method "GET" /url "/etc.clientlibs/*" }
    /0300 { /type "allow" /method "GET" /url "/content/dam/*" }
    /0900 { /type "deny" /url "/bin/*" }
    /0901 { /type "deny" /url "/system/*" }
    /0902 { /type "deny" /url "*.infinity.json" }
}
```

### Service User Pattern
```java
try (ResourceResolver resolver = resolverFactory.getServiceResourceResolver(
        Map.of(ResourceResolverFactory.SUBSERVICE, "myproject-service-user"))) {
    // Minimal permissions operations
}
```

## Cloud Service Specifics

When developing for AEMaaCS, always read [references/cloud-service-guidelines.md](references/cloud-service-guidelines.md) for:

- **Cluster-aware code**: Instances can stop anytime
- **No local state**: Disk is ephemeral
- **Sling Jobs**: For reliable background tasks
- **HTTP timeouts**: Always set connect/read timeouts
- **Rate limiting**: Handle 429 with exponential backoff
- **Touch UI only**: No Classic UI support
- **Immutable repository**: `/libs` and `/apps` are read-only

## CI/CD Quality Gates

| Gate | Checks | Action |
|------|--------|--------|
| Code Quality | SonarQube, code smells | Fix before merge |
| Security | Vulnerability scan | Immediate fix |
| Performance | Load test, response times | Review if fails |

## Common Pitfalls

1. **JCR queries in render path** → Use content hierarchy traversal
2. **Hardcoded paths** → Use context-aware configuration
3. **Unclosed ResourceResolvers** → Use try-with-resources
4. **Deep multifield nesting** → Consider child nodes
5. **Missing Dispatcher filters** → Default-deny approach
6. **Admin sessions in code** → Use service users
7. **Untested cache invalidation** → Verify statfileslevel
8. **Large inline dependencies** → Lazy load heavy scripts
9. **Skipping Core Components** → Extend before building custom
10. **Ignoring Cloud Manager gates** → Fix locally first
