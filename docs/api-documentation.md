# API Documentation
เอกสารอธิบาย API ของระบบ

## API Standards
### Response Format
```json
{
    "status": "success|error",
    "data": {},
    "error": {
        "code": "ERROR_CODE",
        "message": "Error message"
    },
    "metadata": {
        "page": 1,
        "limit": 10,
        "total": 100
    }
}
```

### Authentication
```
Authorization: Bearer [token]
```

## APIs by Service

### User Service
#### POST /api/v1/users/register
```json
{
    "email": "string",
    "password": "string",
    "firstName": "string",
    "lastName": "string"
}
```

#### POST /api/v1/users/login
```json
{
    "email": "string",
    "password": "string"
}
```

### Store Service
#### POST /api/v1/stores
```json
{
    "name": "string",
    "description": "string",
    "categories": ["string"]
}
```

### Product Service
#### POST /api/v1/products
```json
{
    "name": "string",
    "description": "string",
    "price": number,
    "categories": ["string"],
    "attributes": {}
}
```

### Order Service
#### POST /api/v1/orders
```json
{
    "storeId": "string",
    "items": [{
        "productId": "string",
        "quantity": number
    }],
    "shippingAddressId": "string"
}
```

### GraphQL APIs

#### User Queries
```graphql
query GetUser($id: ID!) {
    user(id: $id) {
        id
        email
        firstName
        lastName
        stores {
            id
            name
        }
    }
}
```

#### Store Queries
```graphql
query GetStore($id: ID!) {
    store(id: $id) {
        id
        name
        products {
            id
            name
            price
        }
        orders {
            id
            status
        }
    }
}
```

### WebSocket Events

#### Chat Events
```json
// Send Message
{
    "event": "SEND_MESSAGE",
    "data": {
        "roomId": "string",
        "message": "string"
    }
}

// Receive Message
{
    "event": "MESSAGE_RECEIVED",
    "data": {
        "roomId": "string",
        "message": "string",
        "sender": {}
    }
}
```

#### Order Events
```json
// Order Status Update
{
    "event": "ORDER_STATUS_UPDATED",
    "data": {
        "orderId": "string",
        "status": "string",
        "timestamp": "string"
    }
}
```
