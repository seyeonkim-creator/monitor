# Neo-bank 모니터링 시스템 아키텍처

## 1. Neo-bank Monitor (Meta Ads + Website)

```mermaid
graph TB
    Start([시작]) --> Config[Config<br/>설정 관리]
    Config --> Orchestrator[AdCollectionOrchestrator<br/>오케스트레이터]
    
    Orchestrator --> MetaCollector[MetaAdCollector<br/>Meta Ad Library 수집]
    Orchestrator --> WebsiteCollector[WebsiteCollector<br/>웹사이트 수집]
    
    MetaCollector --> |광고 데이터| DataStorage[DataStorage<br/>데이터 저장]
    WebsiteCollector --> |웹사이트 데이터| DataStorage
    
    DataStorage --> |SQLite 저장| SQLite[(SQLite DB)]
    DataStorage --> |Google Sheets 업로드| GoogleSheets[(Google Sheets)]
    
    DataStorage --> Validator[DataValidator<br/>데이터 검증]
    Validator --> |검증 통과| USPAnalyzer[USPAnalyzer<br/>USP 분석]
    
    USPAnalyzer --> |Gemini API| Gemini[Gemini AI<br/>인사이트 생성]
    Gemini --> |분석 결과| Report[리포트 생성]
    
    Report --> SlackSender[SlackSender<br/>Slack 전송]
    SlackSender --> |Webhook| Slack[Slack 채널]
    
    Slack --> End([완료])
    
    style Config fill:#e1f5ff
    style Orchestrator fill:#fff4e1
    style MetaCollector fill:#ffe1f5
    style WebsiteCollector fill:#ffe1f5
    style DataStorage fill:#e1ffe1
    style Validator fill:#ffe1e1
    style USPAnalyzer fill:#f5e1ff
    style SlackSender fill:#e1f5e1
```

### 주요 클래스 구조

```mermaid
classDiagram
    class Config {
        +config: dict
        +env: dict
        +_load_config()
        +_load_env()
    }
    
    class MetaAdCollector {
        +collect_meta_ads()
        +_crawl_meta_ad_library()
        +_extract_ad_data()
    }
    
    class WebsiteCollector {
        +collect_website_data()
        +_extract_website_info()
    }
    
    class DataStorage {
        +save_to_database()
        +upload_to_sheets()
        +_init_google_sheets()
    }
    
    class DataValidator {
        +validate_data()
        +check_minimum_ads()
        +check_duplicates()
    }
    
    class USPAnalyzer {
        +analyze_usp_trends()
        +_generate_insights()
        +_call_gemini_api()
    }
    
    class SlackSender {
        +send_report()
        +_format_slack_message()
    }
    
    class AdCollectionOrchestrator {
        +run_collection()
        -collector: MetaAdCollector
        -website_collector: WebsiteCollector
        -storage: DataStorage
        -validator: DataValidator
        -analyzer: USPAnalyzer
        -slack_sender: SlackSender
    }
    
    Config --> AdCollectionOrchestrator
    AdCollectionOrchestrator --> MetaAdCollector
    AdCollectionOrchestrator --> WebsiteCollector
    AdCollectionOrchestrator --> DataStorage
    AdCollectionOrchestrator --> DataValidator
    AdCollectionOrchestrator --> USPAnalyzer
    AdCollectionOrchestrator --> SlackSender
    MetaAdCollector --> DataStorage
    WebsiteCollector --> DataStorage
    DataStorage --> DataValidator
    DataValidator --> USPAnalyzer
    USPAnalyzer --> SlackSender
```

## 2. Reddit Monitor (Reddit RSS)

```mermaid
graph TB
    Start([시작]) --> Config[Config<br/>설정 관리]
    Config --> Orchestrator[RedditMonitorOrchestrator<br/>오케스트레이터]
    
    Orchestrator --> RedditCollector[RedditCollector<br/>Reddit RSS 수집]
    
    RedditCollector --> |하이브리드 전략| SubredditRSS[서브레딧 RSS<br/>최신 게시물]
    RedditCollector --> |하이브리드 전략| SearchRSS[검색 RSS<br/>브랜드별 검색]
    
    SubredditRSS --> |게시물| Filter[브랜드 필터링<br/>관련성 강화]
    SearchRSS --> |게시물| Filter
    
    Filter --> |필터링된 데이터| RedditStorage[RedditDataStorage<br/>데이터 저장]
    
    RedditStorage --> |Google Sheets 업로드| GoogleSheets[(Google Sheets<br/>Reddit Data 시트)]
    
    RedditCollector --> |인사이트 추출| Insights[인사이트 분석<br/>브랜드별 차별화]
    
    Insights --> |패턴 분석| PatternAnalysis[패턴 분석<br/>- 고유 키워드<br/>- 구체적 언급<br/>- 경쟁사 비교]
    Insights --> |마케팅 신호| MarketingSignals[마케팅 신호<br/>- 차별화 USP<br/>- 특화 기능<br/>- 우려사항]
    
    PatternAnalysis --> SlackSender[RedditSlackSender<br/>Slack 전송]
    MarketingSignals --> SlackSender
    
    SlackSender --> |Webhook| Slack[Slack 채널]
    
    Slack --> End([완료])
    
    style Config fill:#e1f5ff
    style Orchestrator fill:#fff4e1
    style RedditCollector fill:#ffe1f5
    style RedditStorage fill:#e1ffe1
    style Insights fill:#f5e1ff
    style PatternAnalysis fill:#f5e1ff
    style MarketingSignals fill:#f5e1ff
    style SlackSender fill:#e1f5e1
```

### 주요 클래스 구조

```mermaid
classDiagram
    class Config {
        +env: dict
        +_load_env()
        +_create_directories()
    }
    
    class RedditCollector {
        +collect_reddit_data()
        +collect_from_subreddit_rss()
        +collect_from_search_rss()
        +filter_by_brand()
        +extract_insights()
        +_analyze_brand_patterns()
        +_extract_unique_keywords()
        +_extract_specific_mentions()
        +_extract_brand_specific_issues()
    }
    
    class RedditDataStorage {
        +save_to_sheets()
        +_init_google_sheets()
    }
    
    class RedditSlackSender {
        +send_report()
        +_format_slack_message()
    }
    
    class RedditMonitorOrchestrator {
        +run_monitoring()
        -collector: RedditCollector
        -storage: RedditDataStorage
        -slack_sender: RedditSlackSender
    }
    
    Config --> RedditMonitorOrchestrator
    RedditMonitorOrchestrator --> RedditCollector
    RedditMonitorOrchestrator --> RedditDataStorage
    RedditMonitorOrchestrator --> RedditSlackSender
    RedditCollector --> RedditDataStorage
    RedditCollector --> RedditSlackSender
```

## 3. 데이터 흐름

### Neo-bank Monitor 데이터 흐름

```mermaid
sequenceDiagram
    participant Main
    participant Orchestrator
    participant MetaCollector
    participant WebsiteCollector
    participant Storage
    participant Validator
    participant Analyzer
    participant Slack
    
    Main->>Orchestrator: run_collection()
    Orchestrator->>MetaCollector: collect_meta_ads()
    MetaCollector-->>Orchestrator: Meta 광고 데이터
    Orchestrator->>WebsiteCollector: collect_website_data()
    WebsiteCollector-->>Orchestrator: 웹사이트 데이터
    Orchestrator->>Storage: save_to_database()
    Storage->>Storage: SQLite 저장
    Orchestrator->>Storage: upload_to_sheets()
    Storage->>Storage: Google Sheets 업로드
    Orchestrator->>Validator: validate_data()
    Validator-->>Orchestrator: 검증 결과
    Orchestrator->>Analyzer: analyze_usp_trends()
    Analyzer->>Analyzer: Gemini API 호출
    Analyzer-->>Orchestrator: 분석 결과
    Orchestrator->>Slack: send_report()
    Slack-->>Orchestrator: 전송 완료
```

### Reddit Monitor 데이터 흐름

```mermaid
sequenceDiagram
    participant Main
    participant Orchestrator
    participant Collector
    participant Storage
    participant Slack
    
    Main->>Orchestrator: run_monitoring()
    Orchestrator->>Collector: collect_reddit_data()
    Collector->>Collector: 서브레딧 RSS 수집
    Collector->>Collector: 검색 RSS 수집
    Collector->>Collector: 브랜드 필터링
    Collector-->>Orchestrator: Reddit 게시물 데이터
    Orchestrator->>Collector: extract_insights()
    Collector->>Collector: 브랜드별 패턴 분석
    Collector->>Collector: 마케팅 신호 감지
    Collector-->>Orchestrator: 인사이트
    Orchestrator->>Storage: save_to_sheets()
    Storage->>Storage: Google Sheets 업로드
    Orchestrator->>Slack: send_report()
    Slack-->>Orchestrator: 전송 완료
```

## 4. 시스템 통합 구조

```mermaid
graph LR
    subgraph "Neo-bank Monitor"
        A1[Config] --> A2[Orchestrator]
        A2 --> A3[MetaAdCollector]
        A2 --> A4[WebsiteCollector]
        A3 --> A5[DataStorage]
        A4 --> A5
        A5 --> A6[USPAnalyzer]
        A6 --> A7[SlackSender]
    end
    
    subgraph "Reddit Monitor"
        B1[Config] --> B2[Orchestrator]
        B2 --> B3[RedditCollector]
        B3 --> B4[RedditDataStorage]
        B3 --> B5[RedditSlackSender]
    end
    
    subgraph "공통 저장소"
        A5 --> C1[(Google Sheets)]
        B4 --> C1
        A5 --> C2[(SQLite)]
    end
    
    subgraph "외부 서비스"
        A3 --> D1[Meta Ad Library]
        A4 --> D2[웹사이트]
        B3 --> D3[Reddit RSS]
        A6 --> D4[Gemini API]
        A7 --> D5[Slack]
        B5 --> D5
    end
    
    style A2 fill:#fff4e1
    style B2 fill:#fff4e1
    style C1 fill:#e1ffe1
    style C2 fill:#e1ffe1
    style D5 fill:#e1f5e1
```

