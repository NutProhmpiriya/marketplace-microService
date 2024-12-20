# Database Diagrams for Marketplace Microservices
แผนภาพฐานข้อมูลสำหรับระบบมาร์เก็ตเพลส

## PostgreSQL Databases

### User Service Database
```mermaid
erDiagram
    users {
        uuid id PK
        string email
        string phone_number
        string password_hash
        string first_name
        string last_name
        string avatar_url
        enum status
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    user_roles {
        uuid id PK
        uuid user_id FK
        enum role_type "ADMIN|SELLER|BUYER"
        timestamp created_at
    }

    user_addresses {
        uuid id PK
        uuid user_id FK
        string address_line1
        string address_line2
        string city
        string state
        string postal_code
        string country
        boolean is_default
        timestamp created_at
        timestamp updated_at
    }

    users ||--o{ user_roles : has
    users ||--o{ user_addresses : has
```

### Store Service Database
```mermaid
erDiagram
    stores {
        uuid id PK
        uuid owner_id FK
        string name
        string description
        string logo_url
        string banner_url
        enum status
        float rating
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    store_addresses {
        uuid id PK
        uuid store_id FK
        string address_line1
        string address_line2
        string city
        string state
        string postal_code
        string country
        timestamp created_at
        timestamp updated_at
    }

    store_categories {
        uuid id PK
        uuid store_id FK
        string category_name
        timestamp created_at
    }

    store_operating_hours {
        uuid id PK
        uuid store_id FK
        enum day
        time open_time
        time close_time
        boolean is_closed
        timestamp created_at
        timestamp updated_at
    }

    stores ||--o{ store_addresses : has
    stores ||--o{ store_categories : has
    stores ||--o{ store_operating_hours : has
```

### Order Service Database
```mermaid
erDiagram
    orders {
        uuid id PK
        uuid buyer_id FK
        uuid store_id FK
        decimal total_amount
        decimal shipping_fee
        decimal tax_amount
        enum status
        timestamp created_at
        timestamp updated_at
    }

    order_items {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        integer quantity
        decimal unit_price
        decimal total_price
        timestamp created_at
    }

    order_shipping {
        uuid id PK
        uuid order_id FK
        uuid address_id FK
        string tracking_number
        enum shipping_status
        string carrier
        timestamp estimated_delivery
        timestamp actual_delivery
        timestamp created_at
        timestamp updated_at
    }

    order_payments {
        uuid id PK
        uuid order_id FK
        decimal amount
        enum payment_method
        enum status
        string transaction_id
        timestamp created_at
        timestamp updated_at
    }

    orders ||--|{ order_items : contains
    orders ||--|| order_shipping : has
    orders ||--|{ order_payments : has
```

### Payment Service Database
```mermaid
erDiagram
    payments {
        uuid id PK
        uuid user_id FK
        uuid order_id FK
        decimal amount
        enum payment_method
        enum status
        string transaction_id
        json payment_details
        timestamp created_at
        timestamp updated_at
    }

    payment_methods {
        uuid id PK
        uuid user_id FK
        enum method_type
        string card_last4
        string card_brand
        boolean is_default
        timestamp created_at
        timestamp updated_at
    }

    refunds {
        uuid id PK
        uuid payment_id FK
        decimal amount
        enum status
        string reason
        timestamp created_at
        timestamp updated_at
    }

    payments ||--o{ refunds : has
    payments ||--|| payment_methods : uses
```

### Shipping Service Database
```mermaid
erDiagram
    shipments {
        uuid id PK
        uuid order_id FK
        uuid store_id FK
        uuid address_id FK
        string tracking_number
        enum status
        string carrier
        decimal shipping_fee
        timestamp estimated_delivery
        timestamp actual_delivery
        timestamp created_at
        timestamp updated_at
    }

    shipping_rates {
        uuid id PK
        string carrier
        string service_type
        string origin_region
        string destination_region
        decimal base_rate
        decimal weight_rate
        timestamp created_at
        timestamp updated_at
    }

    shipments ||--|| shipping_rates : uses
```

## MongoDB Collections

### Product Service Collections
```mermaid
erDiagram
    products {
        uuid id PK
        uuid store_id FK
        string name
        string description
        array categories
        array images
        decimal price
        integer stock
        json attributes
        float rating
        enum status
        timestamp created_at
        timestamp updated_at
    }

    product_variants {
        uuid id PK
        uuid product_id FK
        string sku
        string name
        json options
        decimal price
        integer stock
        timestamp created_at
        timestamp updated_at
    }

    product_reviews {
        uuid id PK
        uuid product_id FK
        uuid user_id FK
        integer rating
        string comment
        array images
        timestamp created_at
        timestamp updated_at
    }

    products ||--o{ product_variants : has
    products ||--o{ product_reviews : has
```

### Chat Service Collections
```mermaid
erDiagram
    chat_rooms {
        uuid id PK
        uuid user_id FK
        uuid store_id FK
        timestamp last_message_at
        timestamp created_at
    }

    messages {
        uuid id PK
        uuid chat_room_id FK
        uuid sender_id FK
        enum sender_type "USER|STORE"
        string content
        array attachments
        boolean is_read
        timestamp created_at
    }

    chat_rooms ||--o{ messages : contains
```

### Notification Service Collections
```mermaid
erDiagram
    notifications {
        uuid id PK
        uuid user_id FK
        string title
        string message
        enum type
        json data
        boolean is_read
        timestamp created_at
    }

    notification_settings {
        uuid id PK
        uuid user_id FK
        boolean email_enabled
        boolean push_enabled
        boolean sms_enabled
        json preferences
        timestamp updated_at
    }

    notifications ||--|| notification_settings : configures
```

## Redis Data Structures

### Caching
```mermaid
graph TB
    subgraph "Session Cache"
        SC[Session Data]
        ST[Session Tokens]
    end

    subgraph "Product Cache"
        PC[Product Details]
        PSC[Product Search]
    end

    subgraph "User Cache"
        UC[User Profile]
        UAC[User Auth]
    end

    subgraph "Cart Cache"
        CC[Shopping Cart]
        CIC[Cart Items]
    end
```

### Rate Limiting
```mermaid
graph TB
    subgraph "API Rate Limits"
        RL[Request Counters]
        RT[Token Buckets]
    end

    subgraph "Security"
        BL[Blacklist]
        WL[Whitelist]
    end
```

## Elasticsearch Indices

### Search Service Indices
```mermaid
graph TB
    subgraph "Product Index"
        PI[products]
        PI -->|fields| PN[name]
        PI -->|fields| PD[description]
        PI -->|fields| PC[categories]
        PI -->|fields| PA[attributes]
    end

    subgraph "Store Index"
        SI[stores]
        SI -->|fields| SN[name]
        SI -->|fields| SD[description]
        SI -->|fields| SC[categories]
    end
```
