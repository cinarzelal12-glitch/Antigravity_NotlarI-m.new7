# Dataset Preparation Guide

## Supported Music Genres

```mermaid
mindmap
    root((Music Genres))
        Rock
            Classic Rock
            Hard Rock
            Alternative Rock
            Punk Rock
        Jazz
            Smooth Jazz
            Bebop
            Fusion
            Swing
        Classical
            Baroque
            Romantic
            Contemporary
        Electronic
            House
            Techno
            Dubstep
            Trance
        Hip Hop
            Old School
            Trap
            Conscious
        Pop
            Contemporary
            Synth Pop
            Indie Pop
        Country
            Traditional
            Contemporary
        Blues
            Delta Blues
            Chicago Blues
        Metal
            Heavy Metal
            Death Metal
            Progressive
        Reggae
            Roots
            Dancehall
```

---

## Dataset Structure

```mermaid
graph TB
    subgraph Root["Project Root"]
        Data[data/]
    end
    
    subgraph Raw["Raw Dataset"]
        RawDir[raw/]
        R1[rock/]
        R2[jazz/]
        R3[classical/]
        R4[electronic/]
        RN[... other genres]
    end
    
    subgraph Processed["Processed Dataset"]
        ProcDir[processed/]
        Train[train/]
        Val[validation/]
        Test[test/]
    end
    
    subgraph Spectrograms["Spectrograms"]
        SpecDir[spectrograms/]
        STrain[train/]
        SVal[validation/]
        STest[test/]
    end
    
    subgraph Models["Model Artifacts"]
        ModelDir[models/]
        Checkpoints[checkpoints/]
        Final[final/]
        Metrics[metrics/]
    end
    
    Data --> RawDir & ProcDir & SpecDir & ModelDir
    RawDir --> R1 & R2 & R3 & R4 & RN
    ProcDir --> Train & Val & Test
    SpecDir --> STrain & SVal & STest
    ModelDir --> Checkpoints & Final & Metrics
```

---

## Data Collection Strategy

```mermaid
flowchart TD
    Start([Start Data Collection]) --> Sources{Data Sources}
    
    Sources -->|Public| GTZAN[GTZAN Dataset<br/>1000 tracks, 10 genres]
    Sources -->|Public| FMA[Free Music Archive<br/>106,574 tracks]
    Sources -->|Public| MagnaTagATune[MagnaTagATune<br/>25,863 clips]
    Sources -->|Custom| WebScrape[Web Scraping<br/>w/ License Check]
    
    GTZAN --> Collect
    FMA --> Collect
    MagnaTagATune --> Collect
    WebScrape --> Collect
    
    Collect[Aggregate Data] --> Legal{License Check}
    Legal -->|Valid| Quality
    Legal -->|Invalid| Reject[Reject Sample]
    
    Quality[Quality Control] --> QC1{Duration OK?}
    QC1 -->|Yes| QC2{Audio Quality?}
    QC1 -->|No| Reject
    
    QC2 -->|Good| QC3{Labeled?}
    QC2 -->|Poor| Reject
    
    QC3 -->|Yes| Store
    QC3 -->|No| Manual[Manual Labeling]
    
    Manual --> Store[Store in Raw Folder]
    Store --> Done([Collection Complete])
```

---

## Data Preprocessing Pipeline

```mermaid
flowchart LR
    subgraph Input["Input Audio"]
        Raw[Raw Audio Files<br/>Various Formats]
    end
    
    subgraph Validation["Validation"]
        V1[Check Format]
        V2[Check Duration<br/>min: 5s, max: 5min]
        V3[Check Sample Rate]
        V4[Check Corruption]
    end
    
    subgraph Normalization["Normalization"]
        N1[Convert to Mono]
        N2[Resample to 22050 Hz]
        N3[Normalize Amplitude<br/>-1 to 1]
        N4[Trim Silence]
    end
    
    subgraph Segmentation["Segmentation"]
        S1[Extract 30s Segments]
        S2[Apply Windowing]
        S3[Overlap 50%]
    end
    
    subgraph Output["Output"]
        Out[Processed WAV Files<br/>Ready for Feature Extraction]
    end
    
    Raw --> V1 --> V2 --> V3 --> V4
    V4 --> N1 --> N2 --> N3 --> N4
    N4 --> S1 --> S2 --> S3
    S3 --> Out
```

---

## Feature Extraction

```mermaid
graph TB
    subgraph Audio["Audio Input"]
        WAV[Preprocessed WAV<br/>22050 Hz, 30s]
    end
    
    subgraph STFT["Short-Time Fourier Transform"]
        Window[Apply Hamming Window<br/>Size: 2048<br/>Hop: 512]
        FFT[Compute FFT]
    end
    
    subgraph Features["Feature Types"]
        Mel[Mel-Spectrogram<br/>128 mel bins]
        MFCC[MFCC<br/>40 coefficients]
        Chroma[Chromagram<br/>12 pitch classes]
        Contrast[Spectral Contrast<br/>7 bands]
    end
    
    subgraph Visualization["Visualization"]
        Plot[Generate Image<br/>128x128 pixels]
        Colormap[Apply Colormap<br/>viridis/magma]
        Save[Save as PNG]
    end
    
    WAV --> Window
    Window --> FFT
    FFT --> Mel & MFCC & Chroma & Contrast
    
    Mel --> Plot
    MFCC --> Plot
    Chroma --> Plot
    Contrast --> Plot
    
    Plot --> Colormap
    Colormap --> Save
```

---

## Data Augmentation Techniques

```mermaid
mindmap
    root((Data<br/>Augmentation))
        Time Domain
            Time Stretching
                0.8x - 1.2x speed
            Pitch Shifting
                -2 to +2 semitones
            Time Shifting
                Random offset
            Background Noise
                SNR: 15-30 dB
        Frequency Domain
            Frequency Masking
                Random freq bands
            Time Masking
                Random time segments
        Mixing
            Mixup
                Alpha: 0.2
            CutMix
                Mix spectrograms
        Volume
            Gain Adjustment
                -3 to +3 dB
            Dynamic Range
                Compression
```

---

## Dataset Statistics & Distribution

```mermaid
graph TB
    subgraph Stats["Dataset Statistics"]
        Total[Total Samples: 10,000]
        Train[Training: 7,000<br/>70%]
        Val[Validation: 1,500<br/>15%]
        Test[Testing: 1,500<br/>15%]
    end
    
    subgraph Distribution["Genre Distribution"]
        Rock[Rock: 1,000]
        Jazz[Jazz: 1,000]
        Classical[Classical: 1,000]
        Electronic[Electronic: 1,000]
        HipHop[Hip Hop: 1,000]
        Pop[Pop: 1,000]
        Country[Country: 1,000]
        Blues[Blues: 1,000]
        Metal[Metal: 1,000]
        Reggae[Reggae: 1,000]
    end
    
    Total --> Train & Val & Test
    
    Train -.-> Rock & Jazz & Classical & Electronic & HipHop
    Train -.-> Pop & Country & Blues & Metal & Reggae
```

---

## Data Quality Metrics

```mermaid
flowchart TD
    Start([Audio Sample]) --> Q1{Signal-to-Noise<br/>Ratio > 20dB?}
    
    Q1 -->|No| Reject[Reject Sample]
    Q1 -->|Yes| Q2{Dynamic Range<br/>> 10dB?}
    
    Q2 -->|No| Reject
    Q2 -->|Yes| Q3{Clipping<br/>Detected?}
    
    Q3 -->|Yes| Reject
    Q3 -->|No| Q4{Silence Ratio<br/>< 20%?}
    
    Q4 -->|No| Reject
    Q4 -->|Yes| Q5{Sample Rate<br/>Consistent?}
    
    Q5 -->|No| Reject
    Q5 -->|Yes| Q6{Valid Genre<br/>Label?}
    
    Q6 -->|No| Reject
    Q6 -->|Yes| Accept[Accept Sample<br/>Add to Dataset]
    
    Accept --> Log[Log Quality Metrics]
    Reject --> LogReject[Log Rejection Reason]
```

---

## Metadata Schema

```mermaid
erDiagram
    AUDIO_METADATA {
        string file_id PK
        string filename
        string genre
        float duration_seconds
        int sample_rate
        int bit_depth
        string format
        string source_dataset
        timestamp collected_at
        string license
    }
    
    AUDIO_FEATURES {
        string feature_id PK
        string file_id FK
        float tempo_bpm
        string key
        string mode
        float loudness_db
        json mfcc_mean
        json spectral_centroid
        json spectral_rolloff
        json zero_crossing_rate
    }
    
    SPECTROGRAM_METADATA {
        string spec_id PK
        string file_id FK
        int width
        int height
        string colormap
        string feature_type
        string image_path
        timestamp generated_at
    }
    
    QUALITY_METRICS {
        string metric_id PK
        string file_id FK
        float snr_db
        float dynamic_range_db
        float silence_ratio
        boolean has_clipping
        boolean passed_qc
        timestamp checked_at
    }
    
    AUDIO_METADATA ||--o{ AUDIO_FEATURES : has
    AUDIO_METADATA ||--o{ SPECTROGRAM_METADATA : generates
    AUDIO_METADATA ||--|| QUALITY_METRICS : validates
```

---

## Data Version Control

```mermaid
gitGraph
    commit id: "Initial Dataset v1.0"
    commit id: "Add GTZAN 1000 samples"
    branch preprocessing
    checkout preprocessing
    commit id: "Normalize audio"
    commit id: "Extract spectrograms"
    checkout main
    merge preprocessing
    commit id: "Dataset v1.1"
    
    branch augmentation
    checkout augmentation
    commit id: "Add time stretch"
    commit id: "Add pitch shift"
    commit id: "Add noise injection"
    checkout main
    merge augmentation
    commit id: "Dataset v2.0"
    
    branch expansion
    checkout expansion
    commit id: "Add FMA samples"
    commit id: "Add new genres"
    checkout main
    merge expansion
    commit id: "Dataset v3.0 (Current)"
```

---

## Dataset Maintenance Workflow

```mermaid
stateDiagram-v2
    [*] --> Monitoring: Continuous Monitoring
    
    Monitoring --> Analysis: Weekly Analysis
    Analysis --> IssueDetection: Scan for Issues
    
    IssueDetection --> NoIssues: All OK
    IssueDetection --> QualityIssue: Quality Problem
    IssueDetection --> ImbalanceIssue: Class Imbalance
    IssueDetection --> CorruptionIssue: Data Corruption
    
    QualityIssue --> Cleanup: Remove Bad Samples
    ImbalanceIssue --> Collection: Collect More Data
    CorruptionIssue --> Recovery: Restore from Backup
    
    Cleanup --> Validation
    Collection --> Validation
Recovery --> Validation
    
    Validation --> Retraining: Update Dataset
    NoIssues --> Monitoring: Continue Monitoring
    
    Retraining --> [*]: Version Bump
```

