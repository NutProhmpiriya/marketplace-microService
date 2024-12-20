# Service Consolidation Guide
คู่มือการรวม Services

## Overview
จากการวิเคราะห์โครงสร้างเดิมที่มี 17 microservices เราสามารถรวมให้เหลือ 8 services หลักเพื่อลดความซับซ้อนและเพิ่มประสิทธิภาพของระบบ

## Service Consolidation Map
แผนผังการรวม Services

### 1. User & Store Service
```mermaid
graph TB
    subgraph "User & Store Service"
        U[User Service] --> US[Combined Service]
        S[Store Service] --> US
    end

    subgraph "Database"
        US --> PG[PostgreSQL]
        US --> R[Redis Cache]
    end

    subgraph "Features"
        US --> F1[User Management]
        US --> F2[Store Management]
        US --> F3[Profile Management]
        US --> F4[Authorization]
    end
```

**เหตุผลในการรวม:**
- User และ Store มีความสัมพันธ์ใกล้ชิด (User เป็นเจ้าของ Store)
- ใช้ PostgreSQL เป็นฐานข้อมูลเดียวกัน
- ลดความซับซ้อนในการจัดการสิทธิ์และความสัมพันธ์

### 2. Product & Inventory Service
```mermaid
graph TB
    subgraph "Product & Inventory Service"
        P[Product Service] --> PS[Combined Service]
        I[Inventory Service] --> PS
        R[Rating & Review] --> PS
    end

    subgraph "Database"
        PS --> MDB[MongoDB]
        PS --> RD[Redis Cache]
    end

    subgraph "Features"
        PS --> F1[Product Management]
        PS --> F2[Inventory Control]
        PS --> F3[Rating System]
        PS --> F4[Review Management]
    end
```

**เหตุผลในการรวม:**
- ข้อมูลสินค้า สต็อก และรีวิวมีความเกี่ยวข้องกันโดยตรง
- ใช้ MongoDB เพื่อความยืดหยุ่นในการจัดเก็บข้อมูล
- ลดการสื่อสารระหว่าง services

### 3. Order & Payment Service
```mermaid
graph TB
    subgraph "Order & Payment Service"
        O[Order Service] --> OPS[Combined Service]
        P[Payment Service] --> OPS
        S[Shipping Service] --> OPS
    end

    subgraph "Database"
        OPS --> PG[PostgreSQL]
        OPS --> R[Redis Cache]
    end

    subgraph "Features"
        OPS --> F1[Order Management]
        OPS --> F2[Payment Processing]
        OPS --> F3[Shipping Management]
        OPS --> F4[Transaction Management]
    end
```

**เหตุผลในการรวม:**
- เป็นส่วนหนึ่งของ transaction เดียวกัน
- ใช้ PostgreSQL สำหรับความถูกต้องของข้อมูลการเงิน
- ลดความซับซ้อนในการจัดการ distributed transactions

### 4. Communication Service
```mermaid
graph TB
    subgraph "Communication Service"
        C[Chat Service] --> CS[Combined Service]
        N[Notification Service] --> CS
    end

    subgraph "Technology"
        CS --> MDB[MongoDB]
        CS --> SK[Socket.IO]
        CS --> KF[Kafka]
    end

    subgraph "Features"
        CS --> F1[Real-time Chat]
        CS --> F2[Push Notifications]
        CS --> F3[Email Notifications]
        CS --> F4[SMS Notifications]
    end
```

**เหตุผลในการรวม:**
- จัดการการสื่อสารทั้งระบบในที่เดียว
- ใช้ Socket.IO สำหรับการสื่อสารแบบ real-time
- ลดความซ้ำซ้อนในการจัดการการแจ้งเตือน

### 5. Search & Recommendation Service
```mermaid
graph TB
    subgraph "Search & Recommendation Service"
        S[Search Service] --> SRS[Combined Service]
        R[Recommendation Service] --> SRS
    end

    subgraph "Technology"
        SRS --> ES[Elasticsearch]
        SRS --> KF[Kafka]
    end

    subgraph "Features"
        SRS --> F1[Product Search]
        SRS --> F2[Store Search]
        SRS --> F3[Recommendations]
        SRS --> F4[Analytics Integration]
    end
```

**เหตุผลในการรวม:**
- ใช้ Elasticsearch ร่วมกัน
- มีการใช้ข้อมูลร่วมกันในการวิเคราะห์
- เพิ่มประสิทธิภาพในการค้นหาและแนะนำ

### 6. Analytics & Monitoring Service
```mermaid
graph TB
    subgraph "Analytics & Monitoring Service"
        A[Analytics Service] --> AMS[Combined Service]
        M[Monitoring Service] --> AMS
    end

    subgraph "Technology"
        AMS --> P[Prometheus]
        AMS --> G[Grafana]
        AMS --> L[Loki]
    end

    subgraph "Features"
        AMS --> F1[Business Analytics]
        AMS --> F2[System Monitoring]
        AMS --> F3[Log Management]
        AMS --> F4[Alert Management]
    end
```

**เหตุผลในการรวม:**
- รวมการจัดการ metrics และ logs
- ใช้ stack เดียวกัน (Prometheus, Grafana, Loki)
- เพิ่มประสิทธิภาพในการวิเคราะห์และติดตาม

### 7. Media Service (Standalone)
```mermaid
graph TB
    subgraph "Media Service"
        MS[Media Service]
    end

    subgraph "Technology"
        MS --> CS[Cloud Storage]
        MS --> CDN[Content Delivery Network]
    end

    subgraph "Features"
        MS --> F1[File Upload]
        MS --> F2[Image Processing]
        MS --> F3[Video Processing]
        MS --> F4[CDN Management]
    end
```

**เหตุผลที่แยก:**
- จัดการไฟล์มีเดียโดยเฉพาะ
- ต้องการ scalability แยกต่างหาก
- มีการใช้ Cloud Storage และ CDN

### 8. API Gateway & Auth Service
```mermaid
graph TB
    subgraph "API Gateway & Auth Service"
        K[Kong API Gateway] --> GW[Combined Service]
        A[Authentication Service] --> GW
    end

    subgraph "Technology"
        GW --> FB[Firebase Auth]
        GW --> RD[Redis]
    end

    subgraph "Features"
        GW --> F1[Route Management]
        GW --> F2[Authentication]
        GW --> F3[Rate Limiting]
        GW --> F4[Service Discovery]
    end
```

**เหตุผลในการรวม:**
- เป็น entry point เดียวของระบบ
- จัดการการยืนยันตัวตนและการเข้าถึง API
- ลดความซับซ้อนในการจัดการ security

## ข้อดีของการรวม Services

1. **ลดความซับซ้อน**
   - การ deploy และดูแลระบบง่ายขึ้น
   - ลดจำนวน services ที่ต้องจัดการ

2. **เพิ่มประสิทธิภาพ**
   - ลดการสื่อสารระหว่าง services
   - ลดการใช้ทรัพยากร

3. **ลดค่าใช้จ่าย**
   - ใช้ทรัพยากรน้อยลง
   - ลดค่า maintenance

4. **จัดการ Transaction ง่ายขึ้น**
   - ลดความซับซ้อนของ distributed transactions
   - เพิ่มความน่าเชื่อถือของข้อมูล

## ข้อควรระวัง

1. **Domain Boundaries**
   - ต้องออกแบบขอบเขตให้ชัดเจน
   - แยก business logic ให้เหมาะสม

2. **Scalability**
   - ต้องออกแบบให้ scale ได้อย่างอิสระ
   - เตรียมรับมือ load ที่เพิ่มขึ้น

3. **Database Design**
   - แยก schema ให้ชัดเจน
   - ออกแบบ indexes ให้เหมาะสม

4. **Error Handling**
   - จัดการ errors ให้ครอบคลุม
   - มี fallback mechanisms

## การ Monitor และ Maintenance

1. **Performance Monitoring**
   - ติดตาม metrics ของแต่ละ service
   - วิเคราะห์ bottlenecks

2. **Log Management**
   - รวบรวม logs จากทุกส่วน
   - วิเคราะห์ปัญหาที่เกิดขึ้น

3. **Scaling Strategy**
   - กำหนด scaling rules
   - monitor resource usage

4. **Backup Strategy**
   - สำรองข้อมูลแต่ละ service
   - ทดสอบการกู้คืนข้อมูล
