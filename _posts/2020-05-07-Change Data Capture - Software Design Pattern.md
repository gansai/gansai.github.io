---
layout: post
title: Change Data Capture - Software Design Pattern 
published: true
disqus_identifier: Change Data Capture - Software Design Pattern 
comments: true
---

# Change Data Capture - Software Design Pattern

Change Data Capture or CDC is an approach to data integration, generally used in enterprise applications. 

### What is Change Data Capture ?

There are 3 things which happens within CDC products:

1. Identification of data
2. Capture of changes
3. Delivery of changes



**Debezium** is one of the CDC products, which is 

- open-source
- distributed platform

In this post, I wanted to take a new approach to writing blog posts. Code commentaries.

Basically there are 2 motivations:

1. I want to read more code and understand code
2. This is something I have not done yet, in my blogging efforts.

Code commentary:

Language: Java

Repository: Debezium

Package: package io.debezium.pipeline;

Class: ChangeEventSourceCoordinator

Github Path: https://github.com/debezium/debezium/blob/master/debezium-core/src/main/java/io/debezium/pipeline/ChangeEventSourceCoordinator.java

```java
/**
 * Coordinates one or more {@link ChangeEventSource}s and executes them in order.
 *
 * @author Gunnar Morling
 */
@ThreadSafe
public class ChangeEventSourceCoordinator {

    private static final Logger LOGGER = LoggerFactory.getLogger(ChangeEventSourceCoordinator.class);

    private static final Duration SHUTDOWN_WAIT_TIMEOUT = Duration.ofSeconds(90);

    private final OffsetContext previousOffset;
    private final ErrorHandler errorHandler;
    private final ChangeEventSourceFactory changeEventSourceFactory;
    private final ChangeEventSourceMetricsFactory changeEventSourceMetricsFactory;
    private final ExecutorService executor;
    private final EventDispatcher<?> eventDispatcher;
    private final DatabaseSchema<?> schema;

    private volatile boolean running;
    private volatile StreamingChangeEventSource streamingSource;

    private SnapshotChangeEventSourceMetrics snapshotMetrics;
    private StreamingChangeEventSourceMetrics streamingMetrics;

    public ChangeEventSourceCoordinator(OffsetContext previousOffset, ErrorHandler errorHandler, Class<? extends SourceConnector> connectorType,
                                        CommonConnectorConfig connectorConfig,
                                        ChangeEventSourceFactory changeEventSourceFactory,
                                        ChangeEventSourceMetricsFactory changeEventSourceMetricsFactory, EventDispatcher<?> eventDispatcher, DatabaseSchema<?> schema) {
        this.previousOffset = previousOffset;
        this.errorHandler = errorHandler;
        this.changeEventSourceFactory = changeEventSourceFactory;
        this.changeEventSourceMetricsFactory = changeEventSourceMetricsFactory;
        this.executor = Threads.newSingleThreadExecutor(connectorType, connectorConfig.getLogicalName(), "change-event-source-coordinator");
        this.eventDispatcher = eventDispatcher;
        this.schema = schema;
    }

    public synchronized <T extends CdcSourceTaskContext> void start(T taskContext, ChangeEventQueueMetrics changeEventQueueMetrics,
                                                                    EventMetadataProvider metadataProvider) {
        this.snapshotMetrics = changeEventSourceMetricsFactory.getSnapshotMetrics(taskContext, changeEventQueueMetrics, metadataProvider);
        this.streamingMetrics = changeEventSourceMetricsFactory.getStreamingMetrics(taskContext, changeEventQueueMetrics, metadataProvider);
        running = true;

        // run the snapshot source on a separate thread so start() won't block
        executor.submit(() -> {
            try {
                snapshotMetrics.register(LOGGER);
                streamingMetrics.register(LOGGER);
                LOGGER.info("Metrics registered");

                ChangeEventSourceContext context = new ChangeEventSourceContextImpl();
                LOGGER.info("Context created");

                SnapshotChangeEventSource snapshotSource = changeEventSourceFactory.getSnapshotChangeEventSource(previousOffset, snapshotMetrics);
                eventDispatcher.setEventListener(snapshotMetrics);
                SnapshotResult snapshotResult = snapshotSource.execute(context);
                LOGGER.info("Snapshot ended with {}", snapshotResult);

                if (snapshotResult.getStatus() == SnapshotResultStatus.COMPLETED || schema.tableInformationComplete()) {
                    schema.assureNonEmptySchema();
                }

                if (running && snapshotResult.isCompletedOrSkipped()) {
                    streamingSource = changeEventSourceFactory.getStreamingChangeEventSource(snapshotResult.getOffset());
                    eventDispatcher.setEventListener(streamingMetrics);
                    streamingMetrics.connected(true);
                    LOGGER.info("Starting streaming");
                    streamingSource.execute(context);
                    LOGGER.info("Finished streaming");
                }
            }
            catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                LOGGER.warn("Change event source executor was interrupted", e);
            }
            catch (Throwable e) {
                errorHandler.setProducerThrowable(e);
            }
            finally {
                streamingMetrics.connected(false);
            }
        });
    }

    public void commitOffset(Map<String, ?> offset) {
        if (streamingSource != null && offset != null) {
            streamingSource.commitOffset(offset);
        }
    }

    /**
     * Stops this coordinator.
     */
    public synchronized void stop() throws InterruptedException {
        running = false;

        try {
            // Clear interrupt flag so the graceful termination is always attempted
            Thread.interrupted();
            executor.shutdown();
            boolean isShutdown = executor.awaitTermination(SHUTDOWN_WAIT_TIMEOUT.toMillis(), TimeUnit.MILLISECONDS);

            if (!isShutdown) {
                LOGGER.warn("Coordinator didn't stop in the expected time, shutting down executor now");

                // Clear interrupt flag so the forced termination is always attempted
                Thread.interrupted();
                executor.shutdownNow();
                executor.awaitTermination(SHUTDOWN_WAIT_TIMEOUT.toMillis(), TimeUnit.MILLISECONDS);
            }
        }
        finally {
            snapshotMetrics.unregister(LOGGER);
            streamingMetrics.unregister(LOGGER);
        }
    }

    private class ChangeEventSourceContextImpl implements ChangeEventSourceContext {

        @Override
        public boolean isRunning() {
            return running;
        }
    }
}
```

### Commentary:

1. At line: 41, *@ThreadSafe annotation* , with this annotation. This annotation generally captures design intent. Basically this annotation is used to document thread safety, denoting that, this class: ```ChangeEventSourceCoordinator``` is thread-safe.  You can check, that one of the recommendations laid out by [SEI CERT Oracle Coding Standard for Java](https://wiki.sei.cmu.edu/confluence/display/java/CON52-J.+Document+thread-safety+and+use+annotations+where+applicable), is to document thread safety. This can be added to our kitty of best practices for coding.

2. At line: 46, Usage of Duration class: 

   1. ```java
      private static final Duration SHUTDOWN_WAIT_TIMEOUT = Duration.ofSeconds(90);
      ```

      Duration class was introduced as part of Java 8. This is later used within stop() method, as a parameter to ```awaitTermination()``` for shutting down executor. Generally the old way of coding is to use a ```Long``` to have some value of milliseconds. But, using Duration is a cleaner approach and improves readability too.

3. Then, we can see the organization of import statements. 

   1. Firstly, imports related to java jdk
   2. Secondly, imports to external 3rd party, for example: sl4j for logger component, kafka for kafka-connect component
   3. Thirdly, imports from own component ( from self )