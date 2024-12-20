# Authentication Flow
การทำงานของระบบยืนยันตัวตน

## Firebase Authentication Flows

### Email/Password Authentication
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant FB as Firebase Auth
    participant B as Backend

    U->>F: Enter Email/Password
    F->>FB: Sign In
    FB-->>F: Return ID Token
    F->>B: Request with ID Token
    B->>FB: Verify Token
    FB-->>B: Token Valid
    B-->>F: Return Response
```

### Google Authentication
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant G as Google
    participant FB as Firebase Auth
    participant B as Backend

    U->>F: Click Google Sign In
    F->>G: Redirect to Google
    G-->>F: Return OAuth Token
    F->>FB: Sign In with Google
    FB-->>F: Return ID Token
    F->>B: Request with ID Token
    B->>FB: Verify Token
    FB-->>B: Token Valid
    B-->>F: Return Response
```

### LINE Authentication
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant L as LINE
    participant FB as Firebase Auth
    participant B as Backend

    U->>F: Click LINE Sign In
    F->>L: Redirect to LINE
    L-->>F: Return OAuth Token
    F->>FB: Sign In with Custom Token
    FB-->>F: Return ID Token
    F->>B: Request with ID Token
    B->>FB: Verify Token
    FB-->>B: Token Valid
    B-->>F: Return Response
```

### Apple Sign In
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant A as Apple
    participant FB as Firebase Auth
    participant B as Backend

    U->>F: Click Apple Sign In
    F->>A: Redirect to Apple
    A-->>F: Return OAuth Token
    F->>FB: Sign In with Apple
    FB-->>F: Return ID Token
    F->>B: Request with ID Token
    B->>FB: Verify Token
    FB-->>B: Token Valid
    B-->>F: Return Response
```

### Phone Authentication
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant FB as Firebase Auth
    participant B as Backend

    U->>F: Enter Phone Number
    F->>FB: Request OTP
    FB-->>U: Send SMS OTP
    U->>F: Enter OTP
    F->>FB: Verify OTP
    FB-->>F: Return ID Token
    F->>B: Request with ID Token
    B->>FB: Verify Token
    FB-->>B: Token Valid
    B-->>F: Return Response
```

## Token Management

### Token Structure
```json
{
    "iss": "https://securetoken.google.com/<project-id>",
    "aud": "<project-id>",
    "auth_time": 1234567890,
    "user_id": "firebase-user-id",
    "sub": "firebase-user-id",
    "iat": 1234567890,
    "exp": 1234567890,
    "email": "user@example.com",
    "email_verified": true,
    "phone_number": "+66812345678",
    "name": "User Name",
    "picture": "https://example.com/photo.jpg",
    "provider_id": "google.com"
}
```

### Token Refresh Flow
```mermaid
sequenceDiagram
    participant F as Frontend
    participant FB as Firebase Auth
    participant B as Backend

    F->>F: Token Expired
    F->>FB: Request New Token
    FB-->>F: Return New Token
    F->>B: Request with New Token
    B->>FB: Verify Token
    FB-->>B: Token Valid
    B-->>F: Return Response
```

## Security Considerations

### Token Validation
1. Verify token signature
2. Check token expiration
3. Validate issuer (iss)
4. Validate audience (aud)
5. Check token not revoked

### Session Management
1. Token expiration time
2. Refresh token rotation
3. Force logout capabilities
4. Multiple device sessions

### Security Headers
```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
```
