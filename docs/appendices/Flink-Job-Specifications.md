# Flink Job Specifications: е-Играч Stream Processing

**Document Version**: 1.0
**Date**: 2025-10-26
**Technology Stack**: Apache Flink (Java), PyTorch (via DJL - Deep Java Library)
**Purpose**: Specification for real-time stream processing jobs
**Architecture**: Event Sourcing + CQRS with Stream Processing

---

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Job 1: Read Model Updater](#job-1-read-model-updater)
4. [Job 2: Fraud Detection Engine](#job-2-fraud-detection-engine)
5. [Job 3: Problem Gambling Detection](#job-3-problem-gambling-detection)
6. [Job 4: Real-Time Analytics Aggregator](#job-4-real-time-analytics-aggregator)
7. [Job 5: Transaction Authorization Validator](#job-5-transaction-authorization-validator)
8. [Common Infrastructure](#common-infrastructure)
9. [Deployment Configuration](#deployment-configuration)
10. [Monitoring & Metrics](#monitoring--metrics)
11. [Error Handling & Recovery](#error-handling--recovery)

---

## Introduction

### Purpose

The е-Играч system uses Apache Flink for real-time stream processing of gambling-related events. Flink jobs consume events from Kafka, process them in real-time, and produce:

- **Read Models**: Optimized query models in MongoDB
- **Alerts**: Fraud and problem gambling detection
- **Analytics**: Real-time metrics and aggregations
- **Validations**: Transaction limit checks

### Event Flow

```
[MongoDB Event Store]
        ↓
    (append event)
        ↓
   [Apache Kafka]
        ↓
  [Flink Jobs (5)]
   ↓   ↓   ↓   ↓   ↓
   |   |   |   |   └─→ [Transaction Validator] → Limit checks
   |   |   |   └──────→ [Analytics] → Metrics & dashboards
   |   |   └──────────→ [Problem Gambling] → ML alerts
   |   └──────────────→ [Fraud Detection] → ML alerts
   └──────────────────→ [Read Model Updater] → MongoDB projections
```

### Technology Stack

- **Flink Version**: 1.18.x
- **Java Version**: 21 (LTS)
- **ML Framework**: Deep Java Library (DJL) with PyTorch backend
- **Message Source**: Apache Kafka
- **State Backend**: RocksDB (for large state)
- **Checkpointing**: Distributed snapshots to persistent storage

---

## Architecture Overview

### Event Schema

All events consumed from Kafka follow this structure:

```json
{
  "aggregateId": "player-uuid",
  "aggregateType": "Player",
  "eventType": "TransactionAuthorized",
  "eventVersion": 1,
  "schemaVersion": 2,
  "eventData": {
    "transactionId": "tx-uuid",
    "playerId": "player-uuid",
    "amount": 5000,
    "timestamp": "2025-10-26T14:30:00Z"
  },
  "metadata": {
    "timestamp": "2025-10-26T14:30:00Z",
    "correlationId": "trace-123",
    "userId": "player-uuid"
  }
}
```

### Kafka Topics

| Topic Name | Partition Key | Events | Consumers |
|------------|---------------|---------|-----------|
| `eigrac.events.player` | `playerId` | PlayerRegistered, PlayerSuspended, SelfExclusionActivated | Read Model Updater, Problem Gambling |
| `eigrac.events.transaction` | `playerId` | TransactionAuthorized, TransactionRejected | All jobs |
| `eigrac.events.limit` | `playerId` | LimitSet, LimitExceeded | Transaction Validator, Read Model Updater |
| `eigrac.events.session` | `playerId` | SessionStarted, SessionEnded | Problem Gambling, Analytics |
| `eigrac.events.alert` | `alertId` | FraudAlert, ProblemGamblingAlert | Operator notification system |

**Partitioning Strategy**: All topics partitioned by `playerId` to ensure ordering per player.

---

## Job 1: Read Model Updater

### Purpose

Consumes events from Kafka and updates MongoDB read models (projections) for query optimization. Implements CQRS by separating write (event store) from read (projections).

### Architecture

```
[Kafka Topics] → [Flink Job] → [MongoDB Read Models]
                     ↓
              [State: Keyed by aggregateId]
                     ↓
              [Windowing: None (per-event processing)]
```

### Input

**Kafka Topics**:
- `eigrac.events.player`
- `eigrac.events.transaction`
- `eigrac.events.limit`
- `eigrac.events.session`

### Output

**MongoDB Collections**:
- `players_read_model`: Player profile with current state
- `transactions_read_model`: Flattened transaction history
- `limits_read_model`: Current limits per player
- `sessions_read_model`: Active and historical sessions

### Implementation (Java)

```java
package rs.eigrac.flink.jobs;

import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.connector.kafka.source.KafkaSource;
import org.apache.flink.connector.mongodb.sink.MongoSink;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import com.mongodb.client.model.ReplaceOptions;
import org.bson.Document;

/**
 * Flink job that consumes events from Kafka and updates MongoDB read models.
 * Implements CQRS by maintaining separate read-optimized projections.
 */
public class ReadModelUpdaterJob {

    public static void main(String[] args) throws Exception {
        // 1. Create execution environment
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // Configure checkpointing (every 10 seconds)
        env.enableCheckpointing(10000);
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

        // 2. Create Kafka source for events
        KafkaSource<String> kafkaSource = KafkaSource.<String>builder()
            .setBootstrapServers("kafka-cluster:9092")
            .setTopics(
                "eigrac.events.player",
                "eigrac.events.transaction",
                "eigrac.events.limit",
                "eigrac.events.session"
            )
            .setGroupId("read-model-updater-group")
            .setStartingOffsets(OffsetsInitializer.latest())
            .setValueOnlyDeserializer(new SimpleStringSchema())
            .build();

        // 3. Create data stream
        DataStream<String> eventStream = env.fromSource(
            kafkaSource,
            WatermarkStrategy.noWatermarks(),
            "Kafka Event Source"
        );

        // 4. Parse events and route to appropriate read model updater
        DataStream<Document> readModelUpdates = eventStream
            .map(new EventParser())  // Parse JSON to Event POJO
            .keyBy(event -> event.getAggregateId())  // Key by aggregate ID
            .flatMap(new ReadModelProjector());  // Project to read model

        // 5. Write to MongoDB
        MongoSink<Document> mongoSink = MongoSink.<Document>builder()
            .setUri("mongodb://mongodb-cluster:27017")
            .setDatabase("eigrac_read_models")
            .setCollection("players_read_model")  // Dynamically set based on event type
            .setBatchSize(100)
            .setBatchIntervalMs(1000)
            .setMaxRetries(3)
            .setDeliveryGuarantee(DeliveryGuarantee.AT_LEAST_ONCE)
            .build();

        readModelUpdates.sinkTo(mongoSink);

        // 6. Execute job
        env.execute("Read Model Updater Job");
    }
}

/**
 * Parses JSON event strings into Event POJOs.
 */
class EventParser implements MapFunction<String, Event> {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public Event map(String jsonEvent) throws Exception {
        return objectMapper.readValue(jsonEvent, Event.class);
    }
}

/**
 * Projects events onto read models using keyed state.
 */
class ReadModelProjector extends RichFlatMapFunction<Event, Document> {

    // Keyed state: Current read model for this aggregate
    private transient ValueState<PlayerReadModel> playerState;

    @Override
    public void open(Configuration parameters) {
        ValueStateDescriptor<PlayerReadModel> descriptor = new ValueStateDescriptor<>(
            "player-read-model",
            TypeInformation.of(PlayerReadModel.class)
        );
        playerState = getRuntimeContext().getState(descriptor);
    }

    @Override
    public void flatMap(Event event, Collector<Document> out) throws Exception {
        // Get current state or initialize
        PlayerReadModel currentModel = playerState.value();
        if (currentModel == null) {
            currentModel = new PlayerReadModel(event.getAggregateId());
        }

        // Apply event to read model (event-specific logic)
        switch (event.getEventType()) {
            case "PlayerRegistered":
                handlePlayerRegistered(currentModel, event);
                break;
            case "TransactionAuthorized":
                handleTransactionAuthorized(currentModel, event);
                break;
            case "LimitSet":
                handleLimitSet(currentModel, event);
                break;
            case "SessionStarted":
                handleSessionStarted(currentModel, event);
                break;
            // ... other event types
        }

        // Update state
        playerState.update(currentModel);

        // Emit updated read model to MongoDB
        Document mongoDoc = currentModel.toMongoDocument();
        out.collect(mongoDoc);
    }

    private void handlePlayerRegistered(PlayerReadModel model, Event event) {
        PlayerRegisteredData data = event.getEventData(PlayerRegisteredData.class);
        model.setPlayerId(data.getPlayerId());
        model.setVerifiedName(data.getVerifiedName());
        model.setRegistrationDate(data.getRegistrationDate());
        model.setStatus("ACTIVE");
    }

    private void handleTransactionAuthorized(PlayerReadModel model, Event event) {
        TransactionAuthorizedData data = event.getEventData(TransactionAuthorizedData.class);

        // Update total transaction count and amount
        model.incrementTransactionCount();
        model.addToTotalSpent(data.getAmount());
        model.setLastTransactionDate(data.getTimestamp());

        // Update monthly/daily totals
        model.addToMonthlySpent(data.getAmount(), data.getTimestamp());
        model.addToDailySpent(data.getAmount(), data.getTimestamp());
    }

    private void handleLimitSet(PlayerReadModel model, Event event) {
        LimitSetData data = event.getEventData(LimitSetData.class);
        model.setMonthlyLimit(data.getMonthlyLimit());
        model.setDailyLimit(data.getDailyLimit());
    }

    private void handleSessionStarted(PlayerReadModel model, Event event) {
        SessionStartedData data = event.getEventData(SessionStartedData.class);
        model.setCurrentSessionId(data.getSessionId());
        model.setSessionStartTime(data.getStartTime());
        model.incrementSessionCount();
    }
}

/**
 * Read model for player projection (POJO).
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
class PlayerReadModel {
    private String playerId;
    private String verifiedName;
    private LocalDate registrationDate;
    private String status;

    // Transaction stats
    private long transactionCount;
    private long totalSpent;
    private LocalDateTime lastTransactionDate;

    // Limits
    private long monthlyLimit;
    private long dailyLimit;
    private long monthlySpent;
    private long dailySpent;

    // Session info
    private String currentSessionId;
    private LocalDateTime sessionStartTime;
    private long totalSessionCount;

    public PlayerReadModel(String playerId) {
        this.playerId = playerId;
        this.transactionCount = 0;
        this.totalSpent = 0;
        this.totalSessionCount = 0;
    }

    public Document toMongoDocument() {
        return new Document()
            .append("_id", playerId)
            .append("playerId", playerId)
            .append("verifiedName", verifiedName)
            .append("registrationDate", registrationDate)
            .append("status", status)
            .append("transactionStats", new Document()
                .append("count", transactionCount)
                .append("totalSpent", totalSpent)
                .append("lastTransactionDate", lastTransactionDate)
            )
            .append("limits", new Document()
                .append("monthly", monthlyLimit)
                .append("daily", dailyLimit)
                .append("monthlySpent", monthlySpent)
                .append("dailySpent", dailySpent)
            )
            .append("session", new Document()
                .append("currentSessionId", currentSessionId)
                .append("startTime", sessionStartTime)
                .append("totalSessions", totalSessionCount)
            );
    }

    public void incrementTransactionCount() {
        this.transactionCount++;
    }

    public void addToTotalSpent(long amount) {
        this.totalSpent += amount;
    }

    public void addToMonthlySpent(long amount, LocalDateTime timestamp) {
        // Reset if new month
        if (timestamp.getMonth() != lastTransactionDate.getMonth()) {
            this.monthlySpent = 0;
        }
        this.monthlySpent += amount;
    }

    public void addToDailySpent(long amount, LocalDateTime timestamp) {
        // Reset if new day
        if (!timestamp.toLocalDate().equals(lastTransactionDate.toLocalDate())) {
            this.dailySpent = 0;
        }
        this.dailySpent += amount;
    }

    public void incrementSessionCount() {
        this.totalSessionCount++;
    }
}
```

### Configuration

**Job Parallelism**: 8 (matches Kafka partition count)
**Checkpoint Interval**: 10 seconds
**State Backend**: RocksDB (for large player state)
**Memory**: 4GB per task manager

### Monitoring

**Key Metrics**:
- Events processed per second
- Kafka consumer lag (per partition)
- MongoDB write latency
- State size per key
- Checkpoint duration

---

## Job 2: Fraud Detection Engine

### Purpose

Detects fraudulent patterns in real-time using machine learning. Analyzes transaction patterns, session behavior, and player history to identify suspicious activity.

### Architecture

```
[Kafka: eigrac.events.transaction]
        ↓
  [Flink Job with PyTorch Model]
        ↓
  [Windowing: 1-hour tumbling]
        ↓
  [ML Inference: Fraud Score]
        ↓
  [Alert if score > threshold] → [Kafka: eigrac.events.alert]
```

### ML Model

**Framework**: PyTorch (via Deep Java Library)
**Model Type**: Gradient Boosted Trees or Neural Network
**Input Features**:
- Transaction amount
- Transaction frequency (last 1h, 24h, 7d)
- Session duration
- Time of day
- Device fingerprint changes
- Geolocation changes
- Velocity checks (transactions per minute)

**Output**: Fraud probability score (0.0 to 1.0)

### Implementation (Java + PyTorch)

```java
package rs.eigrac.flink.jobs;

import ai.djl.Model;
import ai.djl.inference.Predictor;
import ai.djl.ndarray.NDArray;
import ai.djl.ndarray.NDList;
import ai.djl.ndarray.NDManager;
import ai.djl.translate.Batchifier;
import ai.djl.translate.Translator;
import ai.djl.translate.TranslatorContext;

/**
 * Flink job for real-time fraud detection using PyTorch model.
 */
public class FraudDetectionJob {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(30000);  // Checkpoint every 30 seconds

        // Kafka source for transactions
        KafkaSource<String> kafkaSource = KafkaSource.<String>builder()
            .setBootstrapServers("kafka-cluster:9092")
            .setTopics("eigrac.events.transaction")
            .setGroupId("fraud-detection-group")
            .setStartingOffsets(OffsetsInitializer.latest())
            .setValueOnlyDeserializer(new SimpleStringSchema())
            .build();

        DataStream<String> transactionStream = env.fromSource(
            kafkaSource,
            WatermarkStrategy.noWatermarks(),
            "Transaction Event Source"
        );

        // Parse and enrich with player history
        DataStream<TransactionFeatures> enrichedStream = transactionStream
            .map(new EventParser())
            .keyBy(event -> event.getPlayerId())
            .flatMap(new FeatureEnricher());  // Add historical features from state

        // Apply ML model for fraud detection
        DataStream<FraudAlert> fraudAlerts = enrichedStream
            .keyBy(features -> features.getPlayerId())
            .process(new FraudDetector());

        // Filter alerts above threshold and send to Kafka
        fraudAlerts
            .filter(alert -> alert.getFraudScore() > 0.75)  // 75% threshold
            .map(alert -> alert.toJson())
            .sinkTo(createKafkaSink("eigrac.events.alert"));

        env.execute("Fraud Detection Job");
    }
}

/**
 * Enriches transaction events with historical features from keyed state.
 */
class FeatureEnricher extends RichFlatMapFunction<Event, TransactionFeatures> {

    private transient ValueState<PlayerHistory> historyState;

    @Override
    public void open(Configuration parameters) {
        ValueStateDescriptor<PlayerHistory> descriptor = new ValueStateDescriptor<>(
            "player-history",
            TypeInformation.of(PlayerHistory.class)
        );
        historyState = getRuntimeContext().getState(descriptor);
    }

    @Override
    public void flatMap(Event event, Collector<TransactionFeatures> out) throws Exception {
        PlayerHistory history = historyState.value();
        if (history == null) {
            history = new PlayerHistory(event.getPlayerId());
        }

        TransactionAuthorizedData txData = event.getEventData(TransactionAuthorizedData.class);

        // Extract features
        TransactionFeatures features = TransactionFeatures.builder()
            .playerId(event.getPlayerId())
            .transactionId(txData.getTransactionId())
            .amount(txData.getAmount())
            .timestamp(txData.getTimestamp())
            .hourOfDay(txData.getTimestamp().getHour())
            .dayOfWeek(txData.getTimestamp().getDayOfWeek().getValue())
            // Historical features
            .transactionCountLast1h(history.getTransactionCountLastHour())
            .transactionCountLast24h(history.getTransactionCountLast24Hours())
            .avgAmountLast24h(history.getAvgAmountLast24Hours())
            .deviceFingerprintChanged(history.isDeviceFingerprintChanged(txData.getDeviceId()))
            .locationChanged(history.isLocationChanged(txData.getLocation()))
            .timeSinceLastTransaction(history.getTimeSinceLastTransaction(txData.getTimestamp()))
            .build();

        // Update history
        history.addTransaction(txData);
        historyState.update(history);

        out.collect(features);
    }
}

/**
 * Applies PyTorch fraud detection model to transaction features.
 */
class FraudDetector extends KeyedProcessFunction<String, TransactionFeatures, FraudAlert> {

    private transient Predictor<float[], float[]> predictor;
    private transient Model model;

    @Override
    public void open(Configuration parameters) throws Exception {
        // Load PyTorch model (trained offline, deployed with job)
        model = Model.newInstance("fraud_detection_model");
        model.load(Paths.get("/models/fraud_detection.pt"));

        // Create predictor with custom translator
        predictor = model.newPredictor(new FraudModelTranslator());
    }

    @Override
    public void processElement(
        TransactionFeatures features,
        KeyedProcessFunction<String, TransactionFeatures, FraudAlert>.Context ctx,
        Collector<FraudAlert> out
    ) throws Exception {

        // Prepare input features for ML model
        float[] inputFeatures = features.toFeatureVector();

        // Run ML inference
        float[] output = predictor.predict(inputFeatures);
        float fraudScore = output[0];  // Probability of fraud

        // Create alert if fraud detected
        FraudAlert alert = new FraudAlert(
            UUID.randomUUID().toString(),
            features.getPlayerId(),
            features.getTransactionId(),
            fraudScore,
            LocalDateTime.now(),
            getRiskLevel(fraudScore),
            features
        );

        out.collect(alert);
    }

    private String getRiskLevel(float score) {
        if (score > 0.9) return "CRITICAL";
        if (score > 0.75) return "HIGH";
        if (score > 0.5) return "MEDIUM";
        return "LOW";
    }

    @Override
    public void close() throws Exception {
        if (predictor != null) {
            predictor.close();
        }
        if (model != null) {
            model.close();
        }
    }
}

/**
 * Translator for PyTorch fraud detection model.
 */
class FraudModelTranslator implements Translator<float[], float[]> {

    @Override
    public NDList processInput(TranslatorContext ctx, float[] input) {
        NDManager manager = ctx.getNDManager();
        NDArray array = manager.create(input);
        return new NDList(array);
    }

    @Override
    public float[] processOutput(TranslatorContext ctx, NDList list) {
        NDArray output = list.singletonOrThrow();
        return output.toFloatArray();
    }

    @Override
    public Batchifier getBatchifier() {
        return Batchifier.STACK;
    }
}

/**
 * Transaction features for ML model.
 */
@Data
@Builder
class TransactionFeatures {
    // Transaction data
    private String playerId;
    private String transactionId;
    private long amount;
    private LocalDateTime timestamp;
    private int hourOfDay;
    private int dayOfWeek;

    // Historical features
    private long transactionCountLast1h;
    private long transactionCountLast24h;
    private double avgAmountLast24h;
    private boolean deviceFingerprintChanged;
    private boolean locationChanged;
    private long timeSinceLastTransaction;  // milliseconds

    /**
     * Converts features to normalized vector for ML model input.
     */
    public float[] toFeatureVector() {
        return new float[] {
            (float) amount / 10000.0f,  // Normalize amount
            (float) hourOfDay / 24.0f,
            (float) dayOfWeek / 7.0f,
            (float) transactionCountLast1h,
            (float) transactionCountLast24h / 100.0f,
            (float) avgAmountLast24h / 10000.0f,
            deviceFingerprintChanged ? 1.0f : 0.0f,
            locationChanged ? 1.0f : 0.0f,
            (float) Math.log(timeSinceLastTransaction + 1) / 10.0f  // Log scale
        };
    }
}

/**
 * Fraud alert output.
 */
@Data
@AllArgsConstructor
class FraudAlert {
    private String alertId;
    private String playerId;
    private String transactionId;
    private float fraudScore;
    private LocalDateTime detectedAt;
    private String riskLevel;  // CRITICAL, HIGH, MEDIUM, LOW
    private TransactionFeatures features;

    public String toJson() {
        ObjectMapper mapper = new ObjectMapper();
        try {
            return mapper.writeValueAsString(this);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Failed to serialize fraud alert", e);
        }
    }
}

/**
 * Player history stored in keyed state for feature extraction.
 */
@Data
class PlayerHistory {
    private String playerId;
    private List<TransactionRecord> recentTransactions;
    private String lastDeviceId;
    private String lastLocation;

    public PlayerHistory(String playerId) {
        this.playerId = playerId;
        this.recentTransactions = new ArrayList<>();
    }

    public void addTransaction(TransactionAuthorizedData txData) {
        TransactionRecord record = new TransactionRecord(
            txData.getTransactionId(),
            txData.getAmount(),
            txData.getTimestamp(),
            txData.getDeviceId(),
            txData.getLocation()
        );
        recentTransactions.add(record);

        // Keep only last 24 hours
        LocalDateTime cutoff = LocalDateTime.now().minusHours(24);
        recentTransactions.removeIf(tx -> tx.getTimestamp().isBefore(cutoff));

        lastDeviceId = txData.getDeviceId();
        lastLocation = txData.getLocation();
    }

    public long getTransactionCountLastHour() {
        LocalDateTime cutoff = LocalDateTime.now().minusHours(1);
        return recentTransactions.stream()
            .filter(tx -> tx.getTimestamp().isAfter(cutoff))
            .count();
    }

    public long getTransactionCountLast24Hours() {
        return recentTransactions.size();
    }

    public double getAvgAmountLast24Hours() {
        if (recentTransactions.isEmpty()) return 0.0;
        return recentTransactions.stream()
            .mapToLong(TransactionRecord::getAmount)
            .average()
            .orElse(0.0);
    }

    public boolean isDeviceFingerprintChanged(String currentDeviceId) {
        return lastDeviceId != null && !lastDeviceId.equals(currentDeviceId);
    }

    public boolean isLocationChanged(String currentLocation) {
        return lastLocation != null && !lastLocation.equals(currentLocation);
    }

    public long getTimeSinceLastTransaction(LocalDateTime currentTime) {
        if (recentTransactions.isEmpty()) return Long.MAX_VALUE;
        TransactionRecord last = recentTransactions.get(recentTransactions.size() - 1);
        return ChronoUnit.MILLIS.between(last.getTimestamp(), currentTime);
    }
}

@Data
@AllArgsConstructor
class TransactionRecord {
    private String transactionId;
    private long amount;
    private LocalDateTime timestamp;
    private String deviceId;
    private String location;
}
```

### ML Model Training (PyTorch)

**Training Pipeline** (separate Python process, not part of Flink job):

```python
# fraud_detection_training.py
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
import pandas as pd

class FraudDetectionModel(nn.Module):
    """
    Neural network for fraud detection.
    Input: 9 features
    Output: Fraud probability (0.0 to 1.0)
    """
    def __init__(self, input_size=9):
        super(FraudDetectionModel, self).__init__()
        self.fc1 = nn.Linear(input_size, 64)
        self.fc2 = nn.Linear(64, 32)
        self.fc3 = nn.Linear(32, 16)
        self.fc4 = nn.Linear(16, 1)
        self.dropout = nn.Dropout(0.3)
        self.sigmoid = nn.Sigmoid()

    def forward(self, x):
        x = torch.relu(self.fc1(x))
        x = self.dropout(x)
        x = torch.relu(self.fc2(x))
        x = self.dropout(x)
        x = torch.relu(self.fc3(x))
        x = self.fc4(x)
        x = self.sigmoid(x)
        return x

# Load training data from MongoDB
def load_training_data():
    # Connect to MongoDB and fetch labeled fraud data
    # Features: amount, hour, day_of_week, tx_count_1h, tx_count_24h,
    #           avg_amount_24h, device_changed, location_changed, time_since_last
    # Label: is_fraud (0 or 1)
    df = pd.read_csv('fraud_training_data.csv')  # Or load from MongoDB

    X = df[['amount_norm', 'hour_norm', 'day_norm', 'tx_count_1h',
            'tx_count_24h', 'avg_amount_norm', 'device_changed',
            'location_changed', 'time_since_last_norm']].values
    y = df['is_fraud'].values

    return torch.FloatTensor(X), torch.FloatTensor(y)

# Train model
def train_model():
    X_train, y_train = load_training_data()

    dataset = TensorDataset(X_train, y_train.unsqueeze(1))
    dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

    model = FraudDetectionModel(input_size=9)
    criterion = nn.BCELoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)

    # Training loop
    for epoch in range(50):
        for batch_X, batch_y in dataloader:
            optimizer.zero_grad()
            outputs = model(batch_X)
            loss = criterion(outputs, batch_y)
            loss.backward()
            optimizer.step()

        print(f'Epoch {epoch+1}/50, Loss: {loss.item():.4f}')

    # Save model for Flink deployment
    torch.save(model.state_dict(), 'fraud_detection.pt')
    print("Model saved to fraud_detection.pt")

if __name__ == '__main__':
    train_model()
```

### Configuration

**Job Parallelism**: 4
**Checkpoint Interval**: 30 seconds
**Model Path**: `/models/fraud_detection.pt` (mounted as ConfigMap in K8s)
**Alert Threshold**: 0.75 (75% fraud probability)
**Memory**: 8GB per task manager (for model inference)

### Monitoring

**Key Metrics**:
- Fraud alerts per minute
- Model inference latency (p50, p95, p99)
- False positive rate (requires manual labeling feedback)
- Transactions processed per second

---

## Job 3: Problem Gambling Detection

### Purpose

Detects problem gambling behavior patterns using ML. Monitors session duration, loss chasing, spending velocity, and self-exclusion violations.

### ML Features

- Session duration (daily, weekly)
- Loss chasing patterns (increasing bets after losses)
- Spending velocity (rapid increases)
- Time of day patterns (late-night gambling)
- Frequency of limit increases
- Self-exclusion violations

### Implementation

*Similar structure to Fraud Detection job, using PyTorch model with different features.*

**Model Output**: Problem gambling risk score (0.0 to 1.0)
**Alert Threshold**: 0.80 (high risk)
**Action**: Trigger mandatory break, notify support team

### Configuration

**Job Parallelism**: 4
**Window**: 24-hour sliding window
**Memory**: 8GB per task manager

---

## Job 4: Real-Time Analytics Aggregator

### Purpose

Computes real-time analytics and metrics for dashboards. Aggregates transactions, sessions, and player activity.

### Aggregations

1. **Transaction Metrics** (1-minute tumbling windows):
   - Total transaction count
   - Total transaction amount
   - Average transaction size
   - Transactions per operator

2. **Player Metrics** (5-minute tumbling windows):
   - Active players count
   - New registrations
   - Self-exclusions activated
   - Suspended players

3. **Session Metrics** (1-minute tumbling windows):
   - Active sessions
   - Average session duration
   - Sessions per hour

### Implementation (Java)

```java
/**
 * Flink job for real-time analytics aggregation.
 */
public class RealTimeAnalyticsJob {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(60000);  // Checkpoint every 1 minute

        // Kafka source
        KafkaSource<String> kafkaSource = KafkaSource.<String>builder()
            .setBootstrapServers("kafka-cluster:9092")
            .setTopics("eigrac.events.transaction", "eigrac.events.session", "eigrac.events.player")
            .setGroupId("analytics-group")
            .setStartingOffsets(OffsetsInitializer.latest())
            .setValueOnlyDeserializer(new SimpleStringSchema())
            .build();

        DataStream<String> eventStream = env.fromSource(
            kafkaSource,
            WatermarkStrategy
                .<String>forBoundedOutOfOrderness(Duration.ofSeconds(10))
                .withTimestampAssigner((event, timestamp) -> extractEventTimestamp(event)),
            "Event Source"
        );

        DataStream<Event> parsedEvents = eventStream.map(new EventParser());

        // Aggregate 1: Transaction metrics (1-minute windows)
        DataStream<TransactionMetrics> transactionMetrics = parsedEvents
            .filter(event -> event.getEventType().equals("TransactionAuthorized"))
            .windowAll(TumblingEventTimeWindows.of(Time.minutes(1)))
            .aggregate(new TransactionAggregator());

        // Aggregate 2: Player metrics (5-minute windows)
        DataStream<PlayerMetrics> playerMetrics = parsedEvents
            .filter(event ->
                event.getEventType().equals("PlayerRegistered") ||
                event.getEventType().equals("SelfExclusionActivated") ||
                event.getEventType().equals("PlayerSuspended")
            )
            .windowAll(TumblingEventTimeWindows.of(Time.minutes(5)))
            .aggregate(new PlayerAggregator());

        // Aggregate 3: Session metrics (1-minute windows)
        DataStream<SessionMetrics> sessionMetrics = parsedEvents
            .filter(event ->
                event.getEventType().equals("SessionStarted") ||
                event.getEventType().equals("SessionEnded")
            )
            .keyBy(event -> event.getAggregateId())
            .window(TumblingEventTimeWindows.of(Time.minutes(1)))
            .aggregate(new SessionAggregator());

        // Write metrics to MongoDB (analytics_metrics collection)
        transactionMetrics
            .map(metrics -> metrics.toMongoDocument())
            .sinkTo(createMongoSink("analytics_metrics"));

        playerMetrics
            .map(metrics -> metrics.toMongoDocument())
            .sinkTo(createMongoSink("analytics_metrics"));

        sessionMetrics
            .map(metrics -> metrics.toMongoDocument())
            .sinkTo(createMongoSink("analytics_metrics"));

        env.execute("Real-Time Analytics Job");
    }
}

/**
 * Aggregates transaction events into 1-minute metrics.
 */
class TransactionAggregator implements AggregateFunction<Event, TransactionAccumulator, TransactionMetrics> {

    @Override
    public TransactionAccumulator createAccumulator() {
        return new TransactionAccumulator();
    }

    @Override
    public TransactionAccumulator add(Event event, TransactionAccumulator acc) {
        TransactionAuthorizedData data = event.getEventData(TransactionAuthorizedData.class);
        acc.count++;
        acc.totalAmount += data.getAmount();
        return acc;
    }

    @Override
    public TransactionMetrics getResult(TransactionAccumulator acc) {
        return new TransactionMetrics(
            LocalDateTime.now(),
            acc.count,
            acc.totalAmount,
            acc.count > 0 ? acc.totalAmount / acc.count : 0
        );
    }

    @Override
    public TransactionAccumulator merge(TransactionAccumulator a, TransactionAccumulator b) {
        a.count += b.count;
        a.totalAmount += b.totalAmount;
        return a;
    }
}

@Data
class TransactionAccumulator {
    long count = 0;
    long totalAmount = 0;
}

@Data
@AllArgsConstructor
class TransactionMetrics {
    private LocalDateTime windowEnd;
    private long transactionCount;
    private long totalAmount;
    private long avgAmount;

    public Document toMongoDocument() {
        return new Document()
            .append("type", "transaction_metrics")
            .append("windowEnd", windowEnd)
            .append("transactionCount", transactionCount)
            .append("totalAmount", totalAmount)
            .append("avgAmount", avgAmount);
    }
}

// Similar implementations for PlayerAggregator and SessionAggregator...
```

### Configuration

**Job Parallelism**: 2
**Window Size**: 1 minute (transactions), 5 minutes (players)
**Checkpoint Interval**: 1 minute
**Memory**: 2GB per task manager

### Monitoring

**Key Metrics**:
- Window processing latency
- Metrics output rate
- Late events dropped

---

## Job 5: Transaction Authorization Validator

### Purpose

Real-time validation of transaction requests against player limits. Enforces daily/monthly limits and detects violations.

### Architecture

```
[Kafka: eigrac.commands.authorizeTransaction]
        ↓
  [Flink Job with Keyed State]
        ↓
  [Check against limits in state]
        ↓
  [Accept or Reject] → [Kafka: eigrac.events.transaction]
```

### Implementation (Java)

```java
/**
 * Flink job for real-time transaction authorization validation.
 */
public class TransactionAuthorizationJob {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.enableCheckpointing(5000);  // Checkpoint every 5 seconds (critical for correctness)

        // Kafka source for transaction authorization commands
        KafkaSource<String> commandSource = KafkaSource.<String>builder()
            .setBootstrapServers("kafka-cluster:9092")
            .setTopics("eigrac.commands.authorizeTransaction")
            .setGroupId("transaction-validator-group")
            .setStartingOffsets(OffsetsInitializer.latest())
            .setValueOnlyDeserializer(new SimpleStringSchema())
            .build();

        DataStream<String> commandStream = env.fromSource(
            commandSource,
            WatermarkStrategy.noWatermarks(),
            "Transaction Command Source"
        );

        // Parse commands
        DataStream<AuthorizeTransactionCommand> commands = commandStream
            .map(new CommandParser());

        // Validate against limits (keyed by playerId)
        DataStream<TransactionResult> results = commands
            .keyBy(cmd -> cmd.getPlayerId())
            .process(new TransactionValidator());

        // Emit events based on validation result
        results
            .map(result -> result.toEvent())
            .sinkTo(createKafkaSink("eigrac.events.transaction"));

        env.execute("Transaction Authorization Validator Job");
    }
}

/**
 * Validates transaction commands against player limits using keyed state.
 */
class TransactionValidator extends KeyedProcessFunction<String, AuthorizeTransactionCommand, TransactionResult> {

    private transient ValueState<PlayerLimitState> limitState;

    @Override
    public void open(Configuration parameters) {
        ValueStateDescriptor<PlayerLimitState> descriptor = new ValueStateDescriptor<>(
            "player-limit-state",
            TypeInformation.of(PlayerLimitState.class)
        );
        limitState = getRuntimeContext().getState(descriptor);
    }

    @Override
    public void processElement(
        AuthorizeTransactionCommand cmd,
        KeyedProcessFunction<String, AuthorizeTransactionCommand, TransactionResult>.Context ctx,
        Collector<TransactionResult> out
    ) throws Exception {

        // Get or initialize player limit state
        PlayerLimitState state = limitState.value();
        if (state == null) {
            // Fetch limits from MongoDB (could be cached in state from limit events)
            state = fetchPlayerLimits(cmd.getPlayerId());
        }

        // Validate transaction against limits
        ValidationResult validation = validateTransaction(cmd, state);

        if (validation.isApproved()) {
            // Update state with new transaction
            state.addTransaction(cmd.getAmount(), cmd.getTimestamp());
            limitState.update(state);

            // Emit TransactionAuthorized event
            out.collect(TransactionResult.authorized(cmd));
        } else {
            // Emit TransactionRejected event
            out.collect(TransactionResult.rejected(cmd, validation.getReason()));
        }
    }

    private ValidationResult validateTransaction(AuthorizeTransactionCommand cmd, PlayerLimitState state) {
        // Check daily limit
        long dailySpent = state.getDailySpent(cmd.getTimestamp().toLocalDate());
        if (dailySpent + cmd.getAmount() > state.getDailyLimit()) {
            return ValidationResult.rejected("DAILY_LIMIT_EXCEEDED");
        }

        // Check monthly limit
        long monthlySpent = state.getMonthlySpent(cmd.getTimestamp().getYear(), cmd.getTimestamp().getMonthValue());
        if (monthlySpent + cmd.getAmount() > state.getMonthlyLimit()) {
            return ValidationResult.rejected("MONTHLY_LIMIT_EXCEEDED");
        }

        // Check self-exclusion
        if (state.isSelfExcluded()) {
            return ValidationResult.rejected("SELF_EXCLUDED");
        }

        // Check suspension
        if (state.isSuspended()) {
            return ValidationResult.rejected("ACCOUNT_SUSPENDED");
        }

        return ValidationResult.approved();
    }

    private PlayerLimitState fetchPlayerLimits(String playerId) {
        // Fetch from MongoDB read model or build from event history
        // For production, this should be cached in Flink state
        // Here we simulate with default limits
        return new PlayerLimitState(playerId, 50000L, 200000L);  // Daily: 50k, Monthly: 200k
    }
}

/**
 * Player limit state stored in Flink keyed state.
 */
@Data
@AllArgsConstructor
class PlayerLimitState {
    private String playerId;
    private long dailyLimit;
    private long monthlyLimit;
    private boolean selfExcluded;
    private boolean suspended;
    private Map<LocalDate, Long> dailySpent;  // Date -> Amount
    private Map<YearMonth, Long> monthlySpent;  // Month -> Amount

    public PlayerLimitState(String playerId, long dailyLimit, long monthlyLimit) {
        this.playerId = playerId;
        this.dailyLimit = dailyLimit;
        this.monthlyLimit = monthlyLimit;
        this.selfExcluded = false;
        this.suspended = false;
        this.dailySpent = new HashMap<>();
        this.monthlySpent = new HashMap<>();
    }

    public void addTransaction(long amount, LocalDateTime timestamp) {
        LocalDate date = timestamp.toLocalDate();
        YearMonth month = YearMonth.from(timestamp);

        dailySpent.merge(date, amount, Long::sum);
        monthlySpent.merge(month, amount, Long::sum);

        // Cleanup old entries (keep last 31 days for daily, last 13 months for monthly)
        cleanupOldEntries(date, month);
    }

    public long getDailySpent(LocalDate date) {
        return dailySpent.getOrDefault(date, 0L);
    }

    public long getMonthlySpent(int year, int month) {
        return monthlySpent.getOrDefault(YearMonth.of(year, month), 0L);
    }

    private void cleanupOldEntries(LocalDate currentDate, YearMonth currentMonth) {
        // Remove daily entries older than 31 days
        dailySpent.entrySet().removeIf(entry ->
            ChronoUnit.DAYS.between(entry.getKey(), currentDate) > 31
        );

        // Remove monthly entries older than 13 months
        monthlySpent.entrySet().removeIf(entry ->
            ChronoUnit.MONTHS.between(entry.getKey(), currentMonth) > 13
        );
    }
}

@Data
@AllArgsConstructor
class AuthorizeTransactionCommand {
    private String commandId;
    private String playerId;
    private String operatorId;
    private long amount;
    private LocalDateTime timestamp;
    private String deviceId;
    private String location;
}

@Data
class ValidationResult {
    private boolean approved;
    private String reason;

    public static ValidationResult approved() {
        ValidationResult result = new ValidationResult();
        result.approved = true;
        return result;
    }

    public static ValidationResult rejected(String reason) {
        ValidationResult result = new ValidationResult();
        result.approved = false;
        result.reason = reason;
        return result;
    }
}

@Data
class TransactionResult {
    private String transactionId;
    private String playerId;
    private long amount;
    private boolean authorized;
    private String rejectionReason;
    private LocalDateTime processedAt;

    public static TransactionResult authorized(AuthorizeTransactionCommand cmd) {
        TransactionResult result = new TransactionResult();
        result.transactionId = UUID.randomUUID().toString();
        result.playerId = cmd.getPlayerId();
        result.amount = cmd.getAmount();
        result.authorized = true;
        result.processedAt = LocalDateTime.now();
        return result;
    }

    public static TransactionResult rejected(AuthorizeTransactionCommand cmd, String reason) {
        TransactionResult result = new TransactionResult();
        result.transactionId = UUID.randomUUID().toString();
        result.playerId = cmd.getPlayerId();
        result.amount = cmd.getAmount();
        result.authorized = false;
        result.rejectionReason = reason;
        result.processedAt = LocalDateTime.now();
        return result;
    }

    public String toEvent() {
        String eventType = authorized ? "TransactionAuthorized" : "TransactionRejected";

        JSONObject event = new JSONObject()
            .put("aggregateId", playerId)
            .put("aggregateType", "Player")
            .put("eventType", eventType)
            .put("eventVersion", 1)
            .put("schemaVersion", 1)
            .put("eventData", new JSONObject()
                .put("transactionId", transactionId)
                .put("playerId", playerId)
                .put("amount", amount)
                .put("authorized", authorized)
                .put("rejectionReason", rejectionReason)
                .put("processedAt", processedAt.toString())
            )
            .put("metadata", new JSONObject()
                .put("timestamp", processedAt.toString())
                .put("userId", playerId)
            );

        return event.toString();
    }
}
```

### Configuration

**Job Parallelism**: 8
**Checkpoint Interval**: 5 seconds (critical for exactly-once semantics)
**State Backend**: RocksDB
**Memory**: 4GB per task manager

### Monitoring

**Key Metrics**:
- Authorization latency (p50, p95, p99)
- Approval rate
- Rejection rate by reason
- State size per player

---

## Common Infrastructure

### State Management

All jobs use **RocksDB** as the state backend for large keyed state:

```java
// Configure RocksDB state backend in each job
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

RocksDBStateBackend stateBackend = new RocksDBStateBackend("hdfs://namenode:9000/flink/checkpoints", true);
stateBackend.setDbStoragePath("/tmp/flink/rocksdb");
env.setStateBackend(stateBackend);
```

### Kafka Configuration

**Common Kafka Settings**:
- **Bootstrap Servers**: `kafka-cluster:9092`
- **Consumer Group ID**: Unique per job
- **Auto Offset Reset**: `latest` (production), `earliest` (testing)
- **Isolation Level**: `read_committed` (for exactly-once)

### Checkpointing

**Checkpoint Configuration** (common across all jobs):
```java
env.enableCheckpointing(interval);
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
env.getCheckpointConfig().setCheckpointTimeout(60000);
env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);
env.getCheckpointConfig().setExternalizedCheckpointCleanup(
    CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION
);
```

---

## Deployment Configuration

### Kubernetes Deployment

**Example Deployment YAML** (Read Model Updater):

```yaml
apiVersion: flink.apache.org/v1beta1
kind: FlinkDeployment
metadata:
  name: read-model-updater
  namespace: eigrac-streaming
spec:
  image: eigrac/flink-jobs:1.0.0
  flinkVersion: v1_18
  flinkConfiguration:
    taskmanager.numberOfTaskSlots: "2"
    state.backend: rocksdb
    state.checkpoints.dir: file:///flink-data/checkpoints
    state.savepoints.dir: file:///flink-data/savepoints
    execution.checkpointing.interval: "10000"
    execution.checkpointing.mode: EXACTLY_ONCE
  serviceAccount: flink
  jobManager:
    resource:
      memory: "2048m"
      cpu: 1
  taskManager:
    replicas: 4
    resource:
      memory: "4096m"
      cpu: 2
  job:
    jarURI: local:///opt/flink/jobs/read-model-updater.jar
    parallelism: 8
    upgradeMode: stateless
```

### Resource Requirements

| Job | Task Managers | Memory per TM | CPU per TM | Total Memory | Total CPU |
|-----|---------------|---------------|------------|--------------|-----------|
| Read Model Updater | 4 | 4GB | 2 | 16GB | 8 |
| Fraud Detection | 4 | 8GB | 2 | 32GB | 8 |
| Problem Gambling | 4 | 8GB | 2 | 32GB | 8 |
| Real-Time Analytics | 2 | 2GB | 1 | 4GB | 2 |
| Transaction Validator | 4 | 4GB | 2 | 16GB | 8 |
| **Total** | **18** | - | - | **100GB** | **34** |

**Kubernetes Nodes**: 5 worker nodes (64GB RAM, 16 cores each)

---

## Monitoring & Metrics

### Flink Metrics

**Key Metrics to Monitor**:
- `numRecordsIn`: Records read from Kafka
- `numRecordsOut`: Records written to sinks
- `currentInputWatermark`: Event time progress
- `lastCheckpointDuration`: Checkpoint performance
- `numFailedCheckpoints`: Checkpoint failures

### Prometheus Integration

**Example Prometheus Configuration**:
```yaml
- job_name: 'flink'
  static_configs:
    - targets: ['flink-jobmanager:9249']
  metric_relabel_configs:
    - source_labels: [job_name]
      target_label: flink_job
```

### Grafana Dashboards

**Dashboard Panels**:
1. **Throughput**: Records/sec per job
2. **Latency**: Processing latency (p95, p99)
3. **Kafka Lag**: Consumer lag per topic/partition
4. **Checkpointing**: Checkpoint duration and failures
5. **State Size**: State backend size per job
6. **Alerts**: Fraud/problem gambling alerts per hour

---

## Error Handling & Recovery

### Failure Scenarios

1. **Task Manager Failure**:
   - Flink automatically restarts failed tasks
   - State recovered from last checkpoint
   - Kafka offsets reset to checkpoint position

2. **Kafka Unavailability**:
   - Jobs automatically retry connecting to Kafka
   - Exponential backoff with max 5 minutes
   - Alert if unavailable > 5 minutes

3. **MongoDB Unavailability**:
   - Write failures buffered in Flink state
   - Retry with exponential backoff
   - Alert if failures > 1 minute

4. **ML Model Loading Failure**:
   - Job fails fast on startup if model not found
   - Alert sent to operations team
   - Manual intervention required

### Restart Strategy

```java
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
    3,  // Number of restart attempts
    Time.of(30, TimeUnit.SECONDS)  // Delay between restarts
));
```

### Alerting Rules

**Critical Alerts**:
- Job failure (3 restarts exhausted)
- Checkpoint failures > 3 consecutive
- Kafka lag > 100,000 messages
- ML model inference errors > 10/minute

---

## Document Control

**Version**: 1.0
**Author**: е-Играч Architecture Team
**Last Updated**: 2025-10-26
**Status**: Ready for Implementation
**Tech Stack**: Apache Flink (Java), PyTorch (via DJL)

**Related Documents**:
- [MongoDB Schema Design](./MongoDB-Schema-Design.md)
- [Event Versioning Playbook](./Event-Versioning-Playbook.md)
- [Event Catalog](./Event-Catalog.md) (to be created)

**Next Steps**:
1. Review with streaming team
2. Set up Flink cluster on Kubernetes
3. Implement and test Read Model Updater job first
4. Train and deploy ML models for fraud/problem gambling detection
5. Configure monitoring and alerting

---

**END OF DOCUMENT**
