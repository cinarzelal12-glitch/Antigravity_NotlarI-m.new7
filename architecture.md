# Music Genre Classifier - System Architecture

## Overview

A production-ready music genre classification system that converts audio files (MP3/WAV) into spectrograms and uses a Convolutional Neural Network (CNN) to classify them into genres (rock, jazz, classical, etc.).

---

## System Architecture

```mermaid
graph TB
    subgraph Client["Client Layer"]
        UI[Web Interface/API Client]
    end
    
    subgraph API["API Gateway Layer"]
        FastAPI[FastAPI Server]
        Auth[Authentication]
        RateLimit[Rate Limiter]
    end
    
    subgraph Processing["Audio Processing Layer"]
        Upload[File Upload Handler]
        Validator[Audio Validator]
        Converter[Audio Converter<br/>MP3/WAV → Spectrogram]
    end
    
    subgraph ML["Machine Learning Layer"]
        Preprocessor[Image Preprocessor]
        CNN[CNN Model<br/>Genre Classifier]
        PostProcessor[Result Post-processor]
    end
    
    subgraph Storage["Storage Layer"]
        FileStore[File Storage<br/>S3/Local]
        ModelStore[Model Registry<br/>Versioned Models]
        Cache[Redis Cache<br/>Recent Predictions]
    end
    
    subgraph Monitor["Monitoring & Logging"]
        Metrics[Prometheus Metrics]
        Logs[Centralized Logging]
        Alerts[Alert Manager]
    end
    
    UI --> FastAPI
FastAPI --> Auth
    Auth --> RateLimit
    RateLimit --> Upload
    Upload --> Validator
    Validator --> Converter
    Converter --> Preprocessor
    Preprocessor --> CNN
    CNN --> PostProcessor
    PostProcessor --> FastAPI
    
    Upload -.-> FileStore
    Converter -.-> FileStore
    CNN -.-> ModelStore
    PostProcessor -.-> Cache
    
    FastAPI -.-> Metrics
    Processing -.-> Logs
    ML -.-> Logs
    Metrics --> Alerts
```

---

## Data Flow Pipeline

```mermaid
sequenceDiagram
    participant Client
    participant API as FastAPI Server
    participant Validator as Audio Validator
    participant Converter as Spectrogram Generator
    participant Model as CNN Model
    participant Cache as Redis Cache
    participant Storage as File Storage
    
    Client->>API: POST /predict (audio file)
    API->>API: Validate request
    API->>Validator: Validate audio format
    
    alt Invalid format
        Validator-->>API: Error: Unsupported format
        API-->>Client: 400 Bad Request
    end
    
    Validator->>Storage: Save original file
    Validator->>Converter: Convert to spectrogram

    Converter->>Converter: Extract features<br/>(MFCC, Mel-spectrogram)
    Converter->>Converter: Normalize image
    Converter->>Storage: Save spectrogram
    
    Converter->>Model: Predict genre
    Model->>Model: CNN inference
    Model->>Model: Calculate confidence scores
    
    Model->>Cache: Cache result (TTL: 1h)
    Model-->>API: Return prediction + confidence
    
    API->>API: Format response
    API-->>Client: 200 OK + genre prediction
```

---

## CNN Model Architecture

```mermaid
graph LR
    subgraph Input["Input Layer"]
        Spec[Spectrogram Image<br/>128x128x3]
    end
    
    subgraph Conv1["Conv Block 1"]
        C1[Conv2D 32 filters<br/>3x3, ReLU]
        P1[MaxPooling 2x2]
        D1[Dropout 0.25]
    end
    
    subgraph Conv2["Conv Block 2"]
        C2[Conv2D 64 filters<br/>3x3, ReLU]
        P2[MaxPooling 2x2]
        D2[Dropout 0.25]
    end
    
    subgraph Conv3["Conv Block 3"]
        C3[Conv2D 128 filters<br/>3x3, ReLU]
        P3[MaxPooling 2x2]
        D3[Dropout 0.3]
    end
    
    subgraph Conv4["Conv Block 4"]
        C4[Conv2D 256 filters<br/>3x3, ReLU]
        P4[MaxPooling 2x2]
        D4[Dropout 0.3]
    end
    
    subgraph Dense["Fully Connected"]
        Flatten[Flatten]
        FC1[Dense 512<br/>ReLU]
        D5[Dropout 0.5]
        FC2[Dense 256<br/>ReLU]
        D6[Dropout 0.5]
    end
    
    subgraph Output["Output Layer"]
        Out[Dense N_genres<br/>Softmax]
    end
    
    Spec --> C1 --> P1 --> D1
    D1 --> C2 --> P2 --> D2
    D2 --> C3 --> P3 --> D3
    D3 --> C4 --> P4 --> D4
    D4 --> Flatten --> FC1 --> D5 --> FC2 --> D6 --> Out
```

---

## Audio Processing Pipeline

```mermaid
flowchart TD
    Start([Audio File Upload]) --> Check{File Format?}
    
    Check -->|MP3| DecodeMp3[Decode MP3<br/>librosa.load]
    Check -->|WAV| DecodeWav[Decode WAV<br/>librosa.load]
    Check -->|Other| Error([Return Error])
    
    DecodeMp3 --> Resample
    DecodeWav --> Resample
    
    Resample[Resample to 22050 Hz] --> Duration{Duration Check}
    
    Duration -->|Too Short| Pad[Zero Padding<br/>to 30 seconds]
    Duration -->|Too Long| Trim[Trim to 30 seconds]
    Duration -->|Perfect| Process
    
    Pad --> Process
    Trim --> Process
    
    Process[Extract Features] --> Features
    
    subgraph Features["Feature Extraction"]
        MFCC[Mel-frequency<br/>cepstral coefficients]
        MelSpec[Mel-spectrogram]
        Chroma[Chroma features]
    end
    
    Features --> Convert[Convert to Image<br/>Save as PNG]
    Convert --> Normalize[Normalize Pixel Values<br/>0-1 range]
    Normalize --> Resize[Resize to 128x128]
    Resize --> Ready([Ready for CNN])
```

---

## Training Pipeline

```mermaid
flowchart LR
    subgraph Data["Data Preparation"]
        Raw[Raw Audio Files<br/>Labeled by Genre]
        Split[Train/Val/Test Split<br/>70/15/15]
        Aug[Data Augmentation<br/>- Time Stretch<br/>- Pitch Shift<br/>- Add Noise]
    end
    
    subgraph Training["Model Training"]
        Init[Initialize Model]
        Train[Training Loop<br/>- Batch Size: 32<br/>- Epochs: 100<br/>- Optimizer: Adam]
        Val[Validation]
        Early[Early Stopping<br/>Patience: 10]
        Save[Save Best Model]
    end
    
    subgraph Eval["Evaluation"]
        Test[Test Set Evaluation]
        Metrics[Calculate Metrics<br/>- Accuracy<br/>- F1 Score<br/>- Confusion Matrix]
        Report[Generate Report]
    end
    
    Raw --> Split
    Split --> Aug
    Aug --> Init
    Init --> Train
    Train --> Val
    Val --> Early
    Early -->|Continue| Train
    Early -->|Stop| Save
    Save --> Test
    Test --> Metrics
    Metrics --> Report
```

---

## Deployment Architecture

```mermaid
graph TB
    subgraph Internet
        Users[Users/Clients]
    end
    
    subgraph LoadBalancer["Load Balancer"]
        LB[Nginx/ALB]
    end
    
    subgraph K8s["Kubernetes Cluster"]
        subgraph APIPath["API Services"]
            API1[API Pod 1]
            API2[API Pod 2]
            API3[API Pod 3]
        end
        
        subgraph MLPath["ML Inference Services"]
            ML1[Model Server 1<br/>TensorFlow Serving]
            ML2[Model Server 2<br/>TensorFlow Serving]
        end
        
        subgraph Storage["Persistence"]
            PVC[Persistent Volume<br/>Temp Files]
        end
    end
    
    subgraph External["External Services"]
        S3[S3/Object Storage<br/>Audio Files & Models]
        Redis[Redis Cluster<br/>Caching]
        Prom[Prometheus<br/>Monitoring]
        Graf[Grafana<br/>Dashboards]
    end
    
    Users --> LB
    LB --> API1 & API2 & API3
    API1 & API2 & API3 --> ML1 & ML2
    API1 & API2 & API3 -.-> PVC
    API1 & API2 & API3 --> S3
    API1 & API2 & API3 --> Redis
    ML1 & ML2 --> S3
    
    API1 & API2 & API3 -.-> Prom
    ML1 & ML2 -.-> Prom
    Prom --> Graf
```

---

## API Design

```mermaid
graph LR
    subgraph Endpoints["REST API Endpoints"]
        E1[POST /api/v1/predict]
        E2[GET /api/v1/genres]
        E3[GET /api/v1/health]
        E4[POST /api/v1/batch-predict]
        E5[GET /api/v1/prediction/:id]
    end
    
    subgraph Auth["Authentication"]
        JWT[JWT Token<br/>or API Key]
    end
    
    subgraph Response["Response Format"]
        JSON[JSON Response<br/>- genre<br/>- confidence<br/>- all_probabilities<br/>- processing_time]
    end
    
    E1 --> Auth
    E2 --> Auth
    E3 --> NoAuth[Public]
    E4 --> Auth
    E5 --> Auth
    
    Auth --> JSON
    NoAuth --> JSON
```

---

## Database Schema

```mermaid
erDiagram
    PREDICTIONS ||--o{ PREDICTION_RESULTS : contains
    AUDIO_FILES ||--o{ PREDICTIONS : has
    MODELS ||--o{ PREDICTIONS : uses
    
    AUDIO_FILES {
        uuid id PK
        string filename
        string original_path
        string spectrogram_path
        int duration_ms
        string format
        timestamp uploaded_at
        string checksum
    }
    
    PREDICTIONS {
        uuid id PK
        uuid audio_file_id FK
        uuid model_id FK
        timestamp predicted_at
        int processing_time_ms
        string status
    }
    
    PREDICTION_RESULTS {
        uuid id PK
        uuid prediction_id FK
        string genre
        float confidence
        json all_probabilities
    }
    
    MODELS {
        uuid id PK
        string version
        string architecture
        float accuracy
        timestamp trained_at
        string storage_path
        json metrics
        boolean is_active
    }
```

---

## Technology Stack

```mermaid
mindmap
    root((Music Genre<br/>Classifier))
        Backend
            FastAPI
            Python 3.10+
            Uvicorn
        ML/AI
            TensorFlow/Keras
            Librosa
            NumPy
            Matplotlib
        Audio Processing
            librosa
            pydub
            soundfile
        Storage
            AWS S3
            Redis
            PostgreSQL
        Infrastructure
            Docker
            Kubernetes
            Nginx
        Monitoring
            Prometheus
            Grafana
            ELK Stack
        Testing
            pytest
            pytest-cov
            locust
```

---

## Security Architecture

```mermaid
flowchart TD
    Client[Client Request] --> WAF[Web Application Firewall]
    WAF --> RateLimit[Rate Limiting<br/>100 req/min per IP]
    RateLimit --> Auth{Authenticated?}
    
    Auth -->|No| Reject[401 Unauthorized]
    Auth -->|Yes| ValidateJWT[Validate JWT Token]
    
    ValidateJWT --> FileCheck{File Security}
    FileCheck --> SizeCheck[Check File Size<br/>Max 50MB]
    SizeCheck --> TypeCheck[Verify MIME Type]
    TypeCheck --> VirusScan[Virus Scan<br/>ClamAV]
    
    VirusScan -->|Infected| Quarantine[Quarantine & Alert]
    VirusScan -->|Clean| Sanitize[Sanitize Filename]
    
    Sanitize --> Process[Process Request]
    Process --> Audit[Audit Log]
    Audit --> Response[Send Response]
    
    Response --> Encrypt[Encrypt Response<br/>HTTPS/TLS 1.3]
```

---

## Monitoring & Observability

```mermaid
graph TB
    subgraph Metrics["Metrics Collection"]
        M1[API Latency]
        M2[Model Inference Time]
        M3[Request Rate]
        M4[Error Rate]
        M5[Cache Hit Rate]
        M6[Model Accuracy]
    end
    
    subgraph Logging["Logging"]
        L1[Application Logs]
        L2[Access Logs]
        L3[Error Logs]
        L4[Audit Logs]
    end
    
    subgraph Tracing["Distributed Tracing"]
        T1[Request Tracing<br/>Jaeger/Zipkin]
    end
    
    subgraph Alerting["Alerting Rules"]
        A1[High Error Rate > 5%]
        A2[Slow Response > 2s]
        A3[Model Accuracy Drop]
        A4[Disk Usage > 80%]
    end
    
    Metrics --> Prom[Prometheus]
    Logging --> ELK[ELK Stack]
    Tracing --> Jaeger[Jaeger]
    
    Prom --> Grafana[Grafana Dashboards]
    Prom --> AlertMgr[Alert Manager]
    
    AlertMgr --> A1 & A2 & A3 & A4
    A1 & A2 & A3 & A4 --> Notify[Notifications<br/>Slack/PagerDuty]
```

---

## CI/CD Pipeline

```mermaid
flowchart LR
    subgraph Dev["Development"]
        Code[Write Code] --> Commit[Git Commit]
        Commit --> Push[Git Push]
    end
    
    subgraph CI["Continuous Integration"]
        Push --> Trigger[GitHub Actions]
        Trigger --> Lint[Linting & Format<br/>flake8, black]
        Lint --> Test[Run Tests<br/>pytest]
        Test --> Coverage[Coverage Report<br/>>80%]
        Coverage --> Build[Build Docker Image]
    end
    
    subgraph CD["Continuous Deployment"]
        Build --> Push2[Push to Registry<br/>Docker Hub/ECR]
        Push2 --> Deploy{Environment}
        
        Deploy -->|Dev| DevK8s[Deploy to Dev<br/>Kubernetes]
        Deploy -->|Staging| StageK8s[Deploy to Staging]
        Deploy -->|Prod| Manual[Manual Approval]
        
        Manual --> ProdK8s[Deploy to Production<br/>Blue-Green]
    end
    
    subgraph Verify["Verification"]
        DevK8s & StageK8s & ProdK8s --> Health[Health Checks]
        Health --> Smoke[Smoke Tests]
        Smoke --> Monitor[Monitor Metrics]
    end
```

---

## Performance Optimization

```mermaid
graph TB
    subgraph Optimization["Performance Strategies"]
        O1[Model Optimization]
        O2[Caching Strategy]
        O3[Batch Processing]
        O4[Async Processing]
    end
    
    subgraph ModelOpt["Model Optimization"]
        MO1[Model Quantization<br/>TF-Lite]
        MO2[Model Pruning]
        MO3[ONNX Runtime]
    end
    
    subgraph CacheOpt["Caching"]
        C1[Redis Cache<br/>Predictions]
        C2[CDN for Static<br/>Spectrograms]
        C3[Model Cache<br/>In-Memory]
    end
    
    subgraph BatchOpt["Batch Processing"]
        B1[Queue System<br/>Celery/RabbitMQ]
        B2[Batch Inference<br/>Multiple Files]
    end
    
    subgraph AsyncOpt["Async Processing"]
        A1[FastAPI Async<br/>Endpoints]
        A2[Background Tasks]
        A3[WebSocket Support]
    end
    
    O1 --> MO1 & MO2 & MO3
    O2 --> C1 & C2 & C3
    O3 --> B1 & B2
    O4 --> A1 & A2 & A3
```

---

## Error Handling & Recovery

```mermaid
stateDiagram-v2
    [*] --> Received: Request Received
    
    Received --> Validating: Validate Input
    Validating --> Processing: Valid
    Validating --> ClientError: Invalid Format
    
    Processing --> Converting: Audio OK
    Processing --> ClientError: Corrupt File
    
    Converting --> Predicting: Conversion Success
    Converting --> ServerError: Conversion Failed
    
    Predicting --> Success: Prediction OK
    Predicting --> ServerError: Model Error
    Predicting --> Retry: Timeout
    
    Retry --> Predicting: Attempt < 3
    Retry --> ServerError: Max Retries
    
    Success --> Caching: Cache Result
    Caching --> [*]: Return 200
    
    ClientError --> Logging: Log Error
    ServerError --> Logging: Log Error
    Logging --> Alert: Notify Team
    Alert --> [*]: Return Error Response
```

