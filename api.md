# API Documentation

## API Overview

RESTful API for music genre classification using audio file analysis and CNN-based prediction.

**Base URL**: `https://api.musicgenre.ai/v1`  
**Authentication**: JWT Bearer Token or API Key

---

## API Endpoints

```mermaid
graph LR
    subgraph Public["Public Endpoints"]
        Health[GET /health]
        Genres[GET /genres]
        Docs[GET /docs]
    end
    
    subgraph Auth["Authentication Required"]
        Predict[POST /predict]
        Batch[POST /batch-predict]
        Status[GET /prediction/:id]
        History[GET /history]
        Stats[GET /stats]
    end
    
    subgraph Admin["Admin Only"]
        Models[GET /models]
        Deploy[POST /models/deploy]
        Metrics[GET /metrics]
    end
```

---

## Authentication Flow

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Auth as Auth Service
    participant DB as Database
    
    Client->>API: POST /auth/login<br/>(credentials)
    API->>Auth: Validate credentials
    Auth->>DB: Check user
    DB-->>Auth: User data
    Auth->>Auth: Generate JWT
    Auth-->>API: JWT token
    API-->>Client: 200 OK + token
    
    Note over Client: Store token
    
    Client->>API: POST /predict<br/>Authorization: Bearer {token}
    API->>Auth: Validate token
    Auth-->>API: Token valid
    API->>API: Process request
    API-->>Client: 200 OK + prediction
```

---

## Endpoint Specifications

### 1. Single File Prediction

**Endpoint**: `POST /api/v1/predict`

**Request**:
```http
POST /api/v1/predict HTTP/1.1
Host: api.musicgenre.ai
Authorization: Bearer {token}
Content-Type: multipart/form-data

file: [audio file]
```

**Response Flow**:
```mermaid
sequenceDiagram
    participant C as Client
    participant API
    participant Validator
    participant Processor
    participant Model
    participant Cache
    
    C->>API: Upload audio file
    API->>Validator: Validate file
    
    alt Invalid File
        Validator-->>API: Error
        API-->>C: 400 Bad Request
    end
    
    Validator->>Processor: Convert to spectrogram
    Processor->>Model: Predict genre
    Model->>Cache: Store result
    Model-->>API: Prediction + confidence
    API-->>C: 200 OK + JSON response
```

**Response Schema**:
```json
{
  "success": true,
  "data": {
    "prediction_id": "uuid-v4",
    "genre": "rock",
    "confidence": 0.89,
    "all_probabilities": {
      "rock": 0.89,
      "metal": 0.05,
      "blues": 0.03,
      "pop": 0.02,
      "jazz": 0.01
    },
    "processing_time_ms": 1234,
    "model_version": "v2.1.0",
    "spectrogram_url": "https://cdn.example.com/spectrograms/uuid.png"
  },
  "timestamp": "2025-12-01T19:20:00Z"
}
```

---

### 2. Batch Prediction

**Endpoint**: `POST /api/v1/batch-predict`

**Request**:
```http
POST /api/v1/batch-predict HTTP/1.1
Host: api.musicgenre.ai
Authorization: Bearer {token}
Content-Type: multipart/form-data

files[]: [audio file 1]
files[]: [audio file 2]
files[]: [audio file N]
```

**Processing Flow**:
```mermaid
flowchart TD
    Upload[Upload Multiple Files] --> Queue[Add to Processing Queue]
    Queue --> Job[Create Batch Job]
    Job --> Redis[Store in Redis]
    
    Redis --> Worker1[Worker 1]
    Redis --> Worker2[Worker 2]
    Redis --> Worker3[Worker 3]
    
    Worker1 --> Process1[Process Files 1-10]
    Worker2 --> Process2[Process Files 11-20]
    Worker3 --> Process3[Process Files 21-30]
    
    Process1 --> Results
    Process2 --> Results
    Process3 --> Results
    
    Results[Aggregate Results] --> Response[Return Batch ID]
```

**Response**:
```json
{
  "success": true,
  "data": {
    "batch_id": "batch-uuid",
    "status": "processing",
    "total_files": 25,
    "estimated_time_seconds": 180,
    "status_url": "/api/v1/batch/batch-uuid"
  }
}
```

---

### 3. Get Prediction Status

**Endpoint**: `GET /api/v1/prediction/:id`

```mermaid
stateDiagram-v2
    [*] --> Queued: Request received
    Queued --> Processing: Worker picked up
    Processing --> Converting: Audio validated
    Converting --> Predicting: Spectrogram ready
    Predicting --> Completed: Success
    Predicting --> Failed: Error occurred
    Completed --> [*]
    Failed --> [*]
    
    note right of Processing
        Retry up to 3 times
        on transient failures
    end note
```

**Response**:
```json
{
  "success": true,
  "data": {
    "prediction_id": "uuid",
    "status": "completed",
    "genre": "jazz",
    "confidence": 0.92,
    "created_at": "2025-12-01T19:15:00Z",
    "completed_at": "2025-12-01T19:15:03Z"
  }
}
```

---

### 4. Get Supported Genres

**Endpoint**: `GET /api/v1/genres`

**Response**:
```json
{
  "success": true,
  "data": {
    "genres": [
      {
        "id": "rock",
        "name": "Rock",
        "description": "Rock music genre",
        "sub_genres": ["classic-rock", "hard-rock", "alternative"]
      },
      {
        "id": "jazz",
        "name": "Jazz",
        "description": "Jazz music genre",
        "sub_genres": ["smooth-jazz", "bebop", "fusion"]
      }
    ],
    "total": 10
  }
}
```

---

### 5. Health Check

**Endpoint**: `GET /api/v1/health`

```mermaid
flowchart LR
    Request[Health Check] --> API[Check API]
    API --> DB[Check Database]
    DB --> Redis[Check Redis]
    Redis --> Model[Check Model]
    Model --> Storage[Check Storage]
    
    API -->|OK| Status
    DB -->|OK| Status
    Redis -->|OK| Status
    Model -->|OK| Status
    Storage -->|OK| Status
    
    Status[Aggregate Status] --> Response{All OK?}
    Response -->|Yes| Healthy[200 Healthy]
    Response -->|No| Degraded[503 Degraded]
```

**Response**:
```json
{
  "status": "healthy",
  "version": "2.1.0",
  "components": {
    "api": "up",
    "database": "up",
    "redis": "up",
    "model": "up",
    "storage": "up"
  },
  "uptime_seconds": 86400,
  "timestamp": "2025-12-01T19:20:00Z"
}
```

---

## Error Handling

```mermaid
graph TB
    Error[Error Occurred] --> Type{Error Type}
    
    Type -->|400| Client[Client Error]
    Type -->|401| Auth[Authentication Error]
    Type -->|403| Forbidden[Forbidden]
    Type -->|429| RateLimit[Rate Limit]
    Type -->|500| Server[Server Error]
    
    Client --> ClientReasons
    Auth --> AuthReasons
    Forbidden --> ForbidReasons
    RateLimit --> RateLimitReasons
    Server --> ServerReasons
    
    subgraph ClientReasons["400 Bad Request"]
        CR1[Invalid file format]
        CR2[File too large]
        CR3[Corrupt audio]
        CR4[Missing parameters]
    end
    
    subgraph AuthReasons["401 Unauthorized"]
        AR1[Missing token]
        AR2[Invalid token]
        AR3[Expired token]
    end
    
    subgraph ForbidReasons["403 Forbidden"]
        FR1[Insufficient permissions]
        FR2[Resource access denied]
    end
    
    subgraph RateLimitReasons["429 Too Many Requests"]
        RL1[Exceeded rate limit]
        RL2[Try again later]
    end
    
    subgraph ServerReasons["500 Internal Server Error"]
        SR1[Model inference failed]
        SR2[Database error]
        SR3[Storage error]
    end
```

**Error Response Format**:
```json
{
  "success": false,
  "error": {
    "code": "INVALID_FILE_FORMAT",
    "message": "The uploaded file format is not supported",
    "details": "Supported formats: MP3, WAV",
    "timestamp": "2025-12-01T19:20:00Z",
    "request_id": "req-uuid"
  }
}
```

---

## Rate Limiting

```mermaid
sequenceDiagram
    participant C as Client
    participant RL as Rate Limiter
    participant API
    participant Redis as Redis Cache
    
    C->>RL: Request
    RL->>Redis: Get request count for IP
    Redis-->>RL: Current count: 95
    
    RL->>RL: Check limit (100/min)
    
    alt Under Limit
        RL->>Redis: Increment counter
        RL->>API: Forward request
        API-->>C: 200 OK<br/>X-RateLimit-Remaining: 4
    else Over Limit
        RL-->>C: 429 Too Many Requests<br/>Retry-After: 60
    end
```

**Rate Limits**:
- Free Tier: 100 requests/hour
- Basic Tier: 1,000 requests/hour
- Pro Tier: 10,000 requests/hour
- Enterprise: Custom

**Response Headers**:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1638360000
```

---

## Request/Response Examples

### Successful Prediction

```bash
curl -X POST "https://api.musicgenre.ai/v1/predict" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "file=@song.mp3"
```

### Response

```json
{
  "success": true,
  "data": {
    "prediction_id": "pred-123abc",
    "genre": "electronic",
    "confidence": 0.94,
    "all_probabilities": {
      "electronic": 0.94,
      "pop": 0.03,
      "rock": 0.02,
      "jazz": 0.01
    },
    "processing_time_ms": 987,
    "model_version": "v2.1.0"
  },
  "timestamp": "2025-12-01T19:20:00Z"
}
```

---

## Webhook Support

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Queue
    participant Worker
    participant Webhook as Client Webhook
    
    Client->>API: POST /batch-predict<br/>webhook_url provided
    API->>Queue: Add to queue
    API-->>Client: 202 Accepted<br/>batch_id
    
    Queue->>Worker: Process batch
    Worker->>Worker: Process all files
    Worker->>Webhook: POST results
    
    alt Webhook Success
        Webhook-->>Worker: 200 OK
        Worker->>API: Mark complete
    else Webhook Failed
        Webhook-->>Worker: Error
        Worker->>Worker: Retry 3 times
        Worker->>API: Log failure
    end
```

**Webhook Payload**:
```json
{
  "event": "batch.completed",
  "batch_id": "batch-uuid",
  "status": "completed",
  "results": [
    {
      "file": "song1.mp3",
      "genre": "rock",
      "confidence": 0.89
    }
  ],
  "completed_at": "2025-12-01T19:25:00Z"
}
```

---

## SDK Examples

### Python SDK

```python
from musicgenre import MusicGenreClient

client = MusicGenreClient(api_key="your-api-key")

# Single prediction
result = client.predict("path/to/song.mp3")
print(f"Genre: {result.genre}, Confidence: {result.confidence}")

# Batch prediction
batch = client.batch_predict(["song1.mp3", "song2.mp3"])
print(f"Batch ID: {batch.id}")

# Check status
status = client.get_status(batch.id)
print(f"Status: {status.status}")
```

### JavaScript SDK

```javascript
import { MusicGenreClient } from '@musicgenre/sdk';

const client = new MusicGenreClient({ apiKey: 'your-api-key' });

// Single prediction
const result = await client.predict('path/to/song.mp3');
console.log(`Genre: ${result.genre}, Confidence: ${result.confidence}`);

// Batch prediction with webhook
const batch = await client.batchPredict(
  ['song1.mp3', 'song2.mp3'],
  { webhookUrl: 'https://your-app.com/webhook' }
);
```

---

## API Versioning Strategy

```mermaid
timeline
    title API Version Timeline
    section v1.0
        Initial Release : Basic prediction
                       : 10 genres
    section v1.5
        Added Features : Batch prediction
                       : Webhooks
    section v2.0
        Major Update : 20 genres
                     : Improved accuracy
                     : Faster processing
    section v2.1 (Current)
        Improvements : Better error handling
                     : Extended metadata
                     : Performance boost
    section v3.0 (Planned)
        Future : Real-time streaming
               : Multi-label classification
               : Custom model training
```

