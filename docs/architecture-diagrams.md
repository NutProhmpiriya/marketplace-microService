# Marketplace Microservices Architecture Diagrams
แผนภาพสถาปัตยกรรมไมโครเซอร์วิสสำหรับระบบมาร์เก็ตเพลส

## High-Level Architecture Overview
ภาพรวมสถาปัตยกรรมระดับสูง

```mermaid
graph TB
    subgraph "Client Layer"
        Web["Web Application<br/>(Next.js/TypeScript)"]
        Mobile["Mobile Apps"]
    end

    subgraph "API Gateway Layer"
        Kong["Kong API Gateway"]
    end

    subgraph "Authentication"
        Firebase["Firebase Auth<br/>Gmail/Apple/Line/<br/>Email/Phone"]
    end

    subgraph "Core Services"
        User["User Service<br/>Go/PostgreSQL"]
        Store["Store Service<br/>Go/PostgreSQL"]
        Product["Product Service<br/>Go/MongoDB"]
        Order["Order Service<br/>Go/PostgreSQL"]
        Payment["Payment Service<br/>Go/PostgreSQL"]
        Chat["Chat Service<br/>Go/MongoDB"]
        Search["Search Service<br/>Elasticsearch"]
    end

    subgraph "Supporting Services"
        Media["Media Service"]
        Notification["Notification Service"]
        Analytics["Analytics Service"]
        Shipping["Shipping Service"]
    end

    subgraph "Data Storage"
        PostgreSQL[(PostgreSQL)]
        MongoDB[(MongoDB)]
        Redis[(Redis)]
        ES[(Elasticsearch)]
    end

    Web --> Kong
    Mobile --> Kong
    Kong --> Firebase
    Firebase --> Kong
    Kong --> User
    Kong --> Store
    Kong --> Product
    Kong --> Order
    Kong --> Payment
    Kong --> Chat
    Kong --> Search

    User --> PostgreSQL
    Store --> PostgreSQL
    Product --> MongoDB
    Order --> PostgreSQL
    Payment --> PostgreSQL
    Chat --> MongoDB
    Search --> ES

    User --> Redis
    Store --> Redis
    Product --> Redis
```

## Service Communication Flow
การไหลของการสื่อสารระหว่างเซอร์วิส

```mermaid
sequenceDiagram
    participant C as Client
    participant G as API Gateway
    participant A as Auth Service
    participant U as User Service
    participant S as Store Service
    participant P as Product Service
    participant O as Order Service
    participant PM as Payment Service
    participant N as Notification Service

    C->>G: Request
    G->>A: Validate Token
    A->>G: Token Valid
    
    alt Create Store
        G->>U: Validate User
        U->>S: Create Store
        S->>N: Notify Creation
        N->>C: Store Created
    end

    alt Create Order
        G->>O: Create Order
        O->>P: Check Stock
        P->>O: Stock Available
        O->>PM: Process Payment
        PM->>O: Payment Success
        O->>N: Notify Seller
        N->>C: Order Confirmed
    end
```

## Event-Driven Architecture
สถาปัตยกรรมแบบขับเคลื่อนด้วยเหตุการณ์

```mermaid
graph LR
    subgraph "Kafka Event Bus"
        Kafka[("Apache Kafka")]
    end

    subgraph "Publishers"
        Order["Order Service"]
        Payment["Payment Service"]
        User["User Service"]
    end

    subgraph "Subscribers"
        Notification["Notification Service"]
        Analytics["Analytics Service"]
        Inventory["Inventory Service"]
    end

    Order -->|OrderCreated| Kafka
    Payment -->|PaymentProcessed| Kafka
    User -->|UserRegistered| Kafka

    Kafka -->|OrderCreated| Notification
    Kafka -->|PaymentProcessed| Analytics
    Kafka -->|OrderCreated| Inventory
```

## Monitoring & Logging Architecture
สถาปัตยกรรมการติดตามและบันทึก

```mermaid
graph TB
    subgraph "Services"
        S1["Service 1"]
        S2["Service 2"]
        S3["Service 3"]
    end

    subgraph "Monitoring Stack"
        P["Prometheus"]
        G["Grafana"]
        L["Loki"]
    end

    S1 -->|metrics| P
    S2 -->|metrics| P
    S3 -->|metrics| P
    
    S1 -->|logs| L
    S2 -->|logs| L
    S3 -->|logs| L
    
    P --> G
    L --> G
```

## Deployment Architecture
สถาปัตยกรรมการติดตั้ง

```mermaid
graph TB
    subgraph "Source Control"
        Git["GitHub"]
    end

    subgraph "CI/CD"
        GHA["GitHub Actions"]
        Argo["ArgoCD"]
    end

    subgraph "Infrastructure"
        subgraph "Google Cloud Platform"
            GKE["GKE<br/>Kubernetes"]
            SQL["Cloud SQL"]
            Storage["Cloud Storage"]
        end
    end

    Git -->|Push| GHA
    GHA -->|Deploy| Argo
    Argo -->|Sync| GKE
    GKE --- SQL
    GKE --- Storage
```

## Database Relations
ความสัมพันธ์ของฐานข้อมูล

```mermaid
erDiagram
    USER ||--o{ STORE : owns
    STORE ||--o{ PRODUCT : sells
    USER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "ordered in"
    STORE ||--o{ ORDER : receives
    USER ||--o{ CHAT : participates
    STORE ||--o{ CHAT : participates
```

## Security Architecture
สถาปัตยกรรมความปลอดภัย

```mermaid
graph TB
    subgraph "External"
        Client["Client"]
    end

    subgraph "Security Layer"
        WAF["Web Application Firewall"]
        Kong["Kong API Gateway"]
        Firebase["Firebase Auth"]
    end

    subgraph "Internal Services"
        Services["Microservices"]
    end

    Client -->|HTTPS| WAF
    WAF -->|Filtered| Kong
    Kong -->|Authenticate| Firebase
    Firebase -->|Token| Kong
    Kong -->|Authorized| Services
```

## Cache Strategy
กลยุทธ์การแคช

```mermaid
graph LR
    subgraph "Client"
        C["Client"]
    end

    subgraph "Cache Layers"
        CDN["CDN Cache"]
        API["API Cache<br/>Redis"]
        DB["Database Cache"]
    end

    C -->|1. Check| CDN
    CDN -->|2. Miss| API
    API -->|3. Miss| DB
    DB -->|4. Response| API
    API -->|5. Cache| CDN
```

## Service Scaling Strategy
กลยุทธ์การปรับขนาดเซอร์วิส

```mermaid
graph TB
    subgraph "Load Balancer"
        LB["Kong/GCP Load Balancer"]
    end

    subgraph "Service Instances"
        S1["Instance 1"]
        S2["Instance 2"]
        S3["Instance 3"]
    end

    subgraph "Autoscaling"
        HPA["Horizontal Pod Autoscaler"]
        Metrics["Metrics Server"]
    end

    LB --> S1
    LB --> S2
    LB --> S3
    
    Metrics -->|Monitor| HPA
    HPA -->|Scale| S1
    HPA -->|Scale| S2
    HPA -->|Scale| S3
```
