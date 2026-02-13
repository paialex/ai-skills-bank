# Sling Jobs for AEM as a Cloud Service

## Why Sling Jobs Instead of Scheduler API

In AEM as a Cloud Service, the traditional `Scheduler` API is **NOT recommended** for background tasks because:

| Scheduler API (❌ Avoid) | Sling Jobs (✅ Use) |
|--------------------------|---------------------|
| Not cluster-aware | Cluster-aware - runs on one instance |
| Lost on instance restart | Persistent - survives restarts |
| No guaranteed execution | Guaranteed at-least-once execution |
| No retry mechanism | Built-in retry with backoff |
| No job tracking | Job status and history tracking |

## Core Concepts

### Job Topics
Jobs are categorized by **topics** following OSGi event topic conventions:
```
com/mycompany/myproject/job/type
```

### Job Lifecycle
```
Created → Queued → Processing → Completed/Failed/Cancelled
              ↓
         Retried (if failed)
```

### Job Result Types
- `JobResult.OK` - Job processed successfully
- `JobResult.FAILED` - Processing failed, can be retried
- `JobResult.CANCEL` - Processing failed permanently, no retry

---

## Pattern 1: Job Consumer (Basic)

The simplest pattern for processing jobs.

```java
import org.apache.sling.event.jobs.Job;
import org.apache.sling.event.jobs.consumer.JobConsumer;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = JobConsumer.class,
    property = {
        JobConsumer.PROPERTY_TOPICS + "=com/myproject/jobs/sendemail"
    }
)
public class SendEmailJobConsumer implements JobConsumer {

    private static final Logger LOG = LoggerFactory.getLogger(SendEmailJobConsumer.class);

    @Override
    public JobResult process(Job job) {
        String recipient = job.getProperty("recipient", String.class);
        String subject = job.getProperty("subject", String.class);
        String body = job.getProperty("body", String.class);

        try {
            LOG.info("Processing email job: {}", job.getId());

            // Process the job
            sendEmail(recipient, subject, body);

            return JobResult.OK;

        } catch (TransientException e) {
            // Temporary failure - retry
            LOG.warn("Transient error, will retry: {}", e.getMessage());
            return JobResult.FAILED;

        } catch (PermanentException e) {
            // Permanent failure - don't retry
            LOG.error("Permanent error, cancelling job: {}", e.getMessage());
            return JobResult.CANCEL;
        }
    }

    private void sendEmail(String to, String subject, String body) {
        // Email sending logic
    }
}
```

---

## Pattern 2: Job Executor (Advanced)

Use `JobExecutor` when you need:
- Progress tracking
- Detailed logging
- Cancellation support

```java
import org.apache.sling.event.jobs.Job;
import org.apache.sling.event.jobs.consumer.JobExecutor;
import org.apache.sling.event.jobs.consumer.JobExecutionContext;
import org.apache.sling.event.jobs.consumer.JobExecutionResult;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = JobExecutor.class,
    property = {
        JobExecutor.PROPERTY_TOPICS + "=com/myproject/jobs/bulkimport"
    }
)
public class BulkImportJobExecutor implements JobExecutor {

    private static final Logger LOG = LoggerFactory.getLogger(BulkImportJobExecutor.class);

    @Override
    public JobExecutionResult process(Job job, JobExecutionContext context) {
        String[] paths = job.getProperty("paths", String[].class);
        int totalItems = paths != null ? paths.length : 0;

        // Initialize progress tracking
        context.initProgress(totalItems, -1);
        context.log("Starting bulk import of {} items", totalItems);

        int processed = 0;
        int failed = 0;

        for (String path : paths) {
            // Check if job was cancelled
            if (context.isStopped()) {
                context.log("Job cancelled after processing {} items", processed);
                return context.result()
                    .message("Cancelled after " + processed + " items")
                    .cancelled();
            }

            try {
                importItem(path);
                processed++;
                context.incrementProgressCount(1);
                context.log("Imported: {}", path);

            } catch (Exception e) {
                failed++;
                context.log("Failed to import {}: {}", path, e.getMessage());
            }
        }

        String message = String.format("Completed: %d processed, %d failed", processed, failed);
        context.log(message);

        if (failed > 0 && processed == 0) {
            return context.result().message(message).failed();
        }

        return context.result().message(message).succeeded();
    }

    private void importItem(String path) {
        // Import logic
    }
}
```

---

## Pattern 3: Adding Jobs (Job Producer)

### Simple Job Addition
```java
import org.apache.sling.event.jobs.Job;
import org.apache.sling.event.jobs.JobManager;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

import java.util.HashMap;
import java.util.Map;

@Component(service = EmailService.class)
public class EmailServiceImpl implements EmailService {

    private static final String JOB_TOPIC = "com/myproject/jobs/sendemail";

    @Reference
    private JobManager jobManager;

    public String sendEmailAsync(String recipient, String subject, String body) {
        Map<String, Object> properties = new HashMap<>();
        properties.put("recipient", recipient);
        properties.put("subject", subject);
        properties.put("body", body);

        Job job = jobManager.addJob(JOB_TOPIC, properties);

        if (job != null) {
            return job.getId();
        }
        return null;
    }
}
```

### Using JobBuilder (Fluent API)
```java
Job job = jobManager.createJob(JOB_TOPIC)
    .properties(Map.of(
        "recipient", "user@example.com",
        "subject", "Welcome!",
        "priority", "high"
    ))
    .add();
```

---

## Pattern 4: Scheduled Jobs (Cloud-Safe Cron)

**CRITICAL**: This is the correct pattern for scheduled tasks in AEMaaCS.

### Scheduler Component
```java
import org.apache.sling.event.jobs.JobManager;
import org.apache.sling.event.jobs.ScheduledJobInfo;
import org.osgi.service.component.annotations.*;
import org.osgi.service.metatype.annotations.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;

@Component(
    service = ContentSyncScheduler.class,
    immediate = true,
    configurationPolicy = ConfigurationPolicy.REQUIRE
)
@Designate(ocd = ContentSyncScheduler.Config.class)
public class ContentSyncScheduler {

    private static final Logger LOG = LoggerFactory.getLogger(ContentSyncScheduler.class);

    public static final String JOB_TOPIC = "com/myproject/jobs/contentsync";
    private static final String SCHEDULE_NAME = "content-sync-schedule";

    @ObjectClassDefinition(name = "Content Sync Scheduler")
    public @interface Config {

        @AttributeDefinition(name = "Enabled")
        boolean enabled() default true;

        @AttributeDefinition(
            name = "Cron Expression",
            description = "e.g., '0 0/15 * * * ?' for every 15 minutes"
        )
        String cronExpression() default "0 0 * * * ?"; // Every hour

        @AttributeDefinition(name = "Content Paths")
        String[] contentPaths() default {"/content/mysite"};
    }

    @Reference
    private JobManager jobManager;

    private boolean enabled;
    private String cronExpression;
    private String[] contentPaths;

    @Activate
    protected void activate(Config config) {
        this.enabled = config.enabled();
        this.cronExpression = config.cronExpression();
        this.contentPaths = config.contentPaths();

        if (enabled) {
            scheduleJob();
        }
        LOG.info("ContentSyncScheduler activated: enabled={}", enabled);
    }

    @Modified
    protected void modified(Config config) {
        unscheduleJob();
        activate(config);
    }

    @Deactivate
    protected void deactivate() {
        // NOTE: Do NOT unschedule in deactivate for clustered environments
        // The job should persist across instance restarts
        LOG.info("ContentSyncScheduler deactivated");
    }

    private void scheduleJob() {
        // Check if already scheduled to avoid duplicates
        Collection<ScheduledJobInfo> existingJobs =
            jobManager.getScheduledJobs(JOB_TOPIC, 10, null);

        for (ScheduledJobInfo info : existingJobs) {
            if (SCHEDULE_NAME.equals(info.getJobProperties().get("scheduleName"))) {
                LOG.debug("Job already scheduled, skipping");
                return;
            }
        }

        // Create scheduled job
        Map<String, Object> properties = new HashMap<>();
        properties.put("scheduleName", SCHEDULE_NAME);
        properties.put("contentPaths", contentPaths);

        ScheduledJobInfo jobInfo = jobManager.createJob(JOB_TOPIC)
            .properties(properties)
            .schedule()
            .cron(cronExpression)
            .add();

        if (jobInfo != null) {
            LOG.info("Scheduled content sync job with cron: {}", cronExpression);
        } else {
            LOG.error("Failed to schedule content sync job");
        }
    }

    private void unscheduleJob() {
        Collection<ScheduledJobInfo> jobs =
            jobManager.getScheduledJobs(JOB_TOPIC, 10, null);

        for (ScheduledJobInfo info : jobs) {
            if (SCHEDULE_NAME.equals(info.getJobProperties().get("scheduleName"))) {
                info.unschedule();
                LOG.info("Unscheduled content sync job");
            }
        }
    }

    /**
     * Trigger job immediately (for manual invocation).
     */
    public String triggerNow() {
        Map<String, Object> properties = new HashMap<>();
        properties.put("contentPaths", contentPaths);
        properties.put("triggeredManually", true);

        Job job = jobManager.addJob(JOB_TOPIC, properties);
        return job != null ? job.getId() : null;
    }
}
```

### Job Consumer for Scheduled Job
```java
@Component(
    service = JobConsumer.class,
    property = {
        JobConsumer.PROPERTY_TOPICS + "=" + ContentSyncScheduler.JOB_TOPIC
    }
)
public class ContentSyncJobConsumer implements JobConsumer {

    private static final Logger LOG = LoggerFactory.getLogger(ContentSyncJobConsumer.class);

    @Reference
    private ResourceResolverFactory resolverFactory;

    @Override
    public JobResult process(Job job) {
        String[] paths = job.getProperty("contentPaths", String[].class);
        boolean manual = job.getProperty("triggeredManually", false);

        LOG.info("Processing content sync job: paths={}, manual={}",
            Arrays.toString(paths), manual);

        try (ResourceResolver resolver = getServiceResolver()) {
            if (resolver == null) {
                return JobResult.FAILED;
            }

            for (String path : paths) {
                syncContent(resolver, path);
            }

            return JobResult.OK;

        } catch (Exception e) {
            LOG.error("Content sync failed", e);
            return JobResult.FAILED;
        }
    }

    private ResourceResolver getServiceResolver() {
        try {
            return resolverFactory.getServiceResourceResolver(
                Map.of(ResourceResolverFactory.SUBSERVICE, "content-sync-service")
            );
        } catch (LoginException e) {
            LOG.error("Cannot get service resolver", e);
            return null;
        }
    }

    private void syncContent(ResourceResolver resolver, String path) {
        // Sync logic
    }
}
```

---

## Schedule Builder Methods

| Method | Description | Example |
|--------|-------------|---------|
| `.cron(expression)` | Standard cron expression | `"0 0/15 * * * ?"` |
| `.at(date)` | Run once at specific date | `new Date()` |
| `.hourly(minute)` | Run every hour at minute | `.hourly(30)` |
| `.daily(hour, minute)` | Run daily at time | `.daily(2, 0)` |
| `.weekly(day, hour, minute)` | Run weekly | `.weekly(1, 6, 0)` |

### Cron Expression Reference
```
┌─────────── second (0-59)
│ ┌───────── minute (0-59)
│ │ ┌─────── hour (0-23)
│ │ │ ┌───── day of month (1-31)
│ │ │ │ ┌─── month (1-12)
│ │ │ │ │ ┌─ day of week (0-7, 0=Sun)
│ │ │ │ │ │
* * * * * *

Examples:
0 0 * * * ?      = Every hour
0 0/15 * * * ?   = Every 15 minutes
0 0 2 * * ?      = Daily at 2 AM
0 0 0 * * SUN    = Weekly on Sunday midnight
```

---

## Job Queue Configuration

Configure via OSGi:

**`org.apache.sling.event.jobs.QueueConfiguration-myqueue.cfg.json`**
```json
{
    "queue.name": "My Custom Queue",
    "queue.topics": ["com/myproject/jobs/*"],
    "queue.type": "ORDERED",
    "queue.maxparallel": 5,
    "queue.retries": 3,
    "queue.retrydelay": 30000,
    "queue.priority": "NORM"
}
```

### Queue Types
| Type | Description |
|------|-------------|
| `ORDERED` | Jobs processed in order, one at a time |
| `UNORDERED` | Jobs processed in parallel |
| `TOPIC_ROUND_ROBIN` | Round-robin across topics |

---

## Service User Mapping

Jobs running in background need service users:

**`org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended-myproject.cfg.json`**
```json
{
    "user.mapping": [
        "com.myproject.core:content-sync-service=[content-writer-service]"
    ]
}
```

---

## Best Practices

### ✅ Do
1. **Always check for existing schedules** before creating new ones
2. **Use service users** for background resource access
3. **Handle transient vs permanent failures** appropriately
4. **Log job progress** for monitoring
5. **Use meaningful job topics** with consistent naming

### ❌ Don't
1. **Don't unschedule in @Deactivate** in clustered environments
2. **Don't store large data in job properties** - use paths instead
3. **Don't hold ResourceResolver** across job boundary
4. **Don't use Scheduler API** for Cloud Service

---

## Monitoring Jobs

### JMX MBeans
Jobs expose JMX MBeans for monitoring at:
```
org.apache.sling:type=queues,name=*
```

### Job Console
Access at: `/system/console/slingevent`

### Programmatic Status Check
```java
Job job = jobManager.getJobById(jobId);
if (job != null) {
    Job.JobState state = job.getJobState();
    // QUEUED, ACTIVE, SUCCEEDED, STOPPED, GIVEN_UP, ERROR, DROPPED
}
```

---

## References

- [Apache Sling Jobs Documentation](https://sling.apache.org/documentation/bundles/apache-sling-eventing-and-job-handling.html)
- [AEM as a Cloud Service Best Practices](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/content/implementing/developing/aem-project-content-package-structure.html)
