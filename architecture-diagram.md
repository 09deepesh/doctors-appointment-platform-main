# Doctor's Appointment Platform - Architecture Diagram

## System Architecture Overview

```mermaid
graph TB
    %% Client Layer
    subgraph "Client Layer"
        WEB[Web Browser]
        MOBILE[Mobile Browser]
    end

    %% CDN & Edge
    subgraph "Edge Layer"
        CDN[CDN/Edge Cache]
        STATIC[Static Assets]
    end

    %% Application Layer
    subgraph "Next.js Application Layer"
        subgraph "Frontend Components"
            UI[React Components]
            PAGES[Next.js Pages]
            LAYOUTS[Layout Components]
        end
        
        subgraph "API Layer"
            API[Next.js API Routes]
            ACTIONS[Server Actions]
            MIDDLEWARE[Middleware]
        end
        
        subgraph "Authentication"
            CLERK[Clerk Auth]
            SESSIONS[Session Management]
            ROLES[Role-based Access]
        end
    end

    %% External Services
    subgraph "External Services"
        VONAGE[Vonage Video API]
        PAYMENTS[Clerk Billing]
        EMAIL[Email Service]
    end

    %% Database Layer
    subgraph "Database Layer"
        DB[(PostgreSQL - Neon)]
        PRISMA[Prisma ORM]
        MIGRATIONS[Database Migrations]
    end

    %% Data Models
    subgraph "Data Models"
        USER[User Model]
        APPT[Appointment Model]
        CREDIT[CreditTransaction Model]
        PAYOUT[Payout Model]
        AVAIL[Availability Model]
    end

    %% Connections
    WEB --> CDN
    MOBILE --> CDN
    CDN --> UI
    STATIC --> CDN
    
    UI --> API
    UI --> ACTIONS
    PAGES --> LAYOUTS
    
    API --> CLERK
    ACTIONS --> CLERK
    MIDDLEWARE --> CLERK
    
    CLERK --> SESSIONS
    SESSIONS --> ROLES
    
    API --> PRISMA
    ACTIONS --> PRISMA
    PRISMA --> DB
    
    API --> VONAGE
    API --> PAYMENTS
    
    DB --> USER
    DB --> APPT
    DB --> CREDIT
    DB --> PAYOUT
    DB --> AVAIL
    
    %% Styling
    classDef clientLayer fill:#e1f5fe
    classDef appLayer fill:#f3e5f5
    classDef externalLayer fill:#fff3e0
    classDef dataLayer fill:#e8f5e8
    
    class WEB,MOBILE clientLayer
    class UI,PAGES,LAYOUTS,API,ACTIONS,MIDDLEWARE,CLERK,SESSIONS,ROLES appLayer
    class VONAGE,PAYMENTS,EMAIL externalLayer
    class DB,PRISMA,MIGRATIONS,USER,APPT,CREDIT,PAYOUT,AVAIL dataLayer
```

## Component Architecture

```mermaid
graph LR
    subgraph "User Interface Layer"
        A[Header Component]
        B[Doctor Cards]
        C[Appointment Form]
        D[Video Call UI]
        E[Admin Dashboard]
    end
    
    subgraph "Page Components"
        F[Doctors Listing]
        G[Appointment Booking]
        H[Video Call Page]
        I[Admin Panel]
        J[Patient Dashboard]
    end
    
    subgraph "Server Actions"
        K[bookAppointment]
        L[setAvailability]
        M[generateVideoToken]
        N[cancelAppointment]
        O[processPayouts]
    end
    
    subgraph "Data Layer"
        P[Prisma Client]
        Q[Database Queries]
        R[Transaction Management]
    end
    
    A --> F
    B --> F
    C --> G
    D --> H
    E --> I
    
    F --> K
    G --> K
    H --> M
    I --> O
    J --> N
    
    K --> P
    L --> P
    M --> P
    N --> P
    O --> P
    
    P --> Q
    Q --> R
```

## Data Flow Architecture

```mermaid
sequenceDiagram
    participant P as Patient
    participant UI as Frontend
    participant API as Server Actions
    participant AUTH as Clerk Auth
    participant DB as Database
    participant V as Vonage Video
    participant D as Doctor
    
    P->>UI: Browse Doctors
    UI->>API: getAvailableDoctors()
    API->>AUTH: Verify Session
    AUTH-->>API: User Authenticated
    API->>DB: Query Doctors
    DB-->>API: Doctor List
    API-->>UI: Return Doctors
    UI-->>P: Display Doctors
    
    P->>UI: Select Time Slot
    UI->>API: bookAppointment()
    API->>AUTH: Verify Patient Role
    API->>DB: Check Credits & Availability
    DB-->>API: Validation Success
    API->>V: Create Video Session
    V-->>API: Session ID
    API->>DB: Create Appointment + Deduct Credits
    DB-->>API: Appointment Created
    API-->>UI: Booking Confirmed
    UI-->>P: Appointment Scheduled
    
    Note over P,D: At Appointment Time
    P->>UI: Join Video Call
    UI->>API: generateVideoToken()
    API->>AUTH: Verify Authorization
    API->>DB: Validate Appointment
    API->>V: Generate Token
    V-->>API: Video Token
    API-->>UI: Token + Session ID
    UI->>V: Connect to Video Session
    V->>D: Notify Doctor
    D->>V: Join Video Session
    V-->>P: Video Connection Established
    V-->>D: Video Connection Established
    
    Note over P,D: After Consultation
    D->>UI: End Call
    UI->>API: markCompleted()
    API->>DB: Update Status
    DB-->>API: Status Updated
    API-->>UI: Call Ended
```

## Security Architecture

```mermaid
graph TB
    subgraph "Authentication Layer"
        CLERK_AUTH[Clerk Authentication]
        SESSION_MGMT[Session Management]
        JWT_TOKENS[JWT Tokens]
    end
    
    subgraph "Authorization Layer"
        ROLE_CHECK[Role Verification]
        PERMISSION_MATRIX[Permission Matrix]
        ACCESS_CONTROL[Access Control]
    end
    
    subgraph "API Security"
        MIDDLEWARE_SEC[Middleware Security]
        RATE_LIMITING[Rate Limiting]
        INPUT_VALIDATION[Input Validation]
    end
    
    subgraph "Data Security"
        ENCRYPTION[Data Encryption]
        SQL_INJECTION[SQL Injection Prevention]
        XSS_PROTECTION[XSS Protection]
    end
    
    CLERK_AUTH --> SESSION_MGMT
    SESSION_MGMT --> JWT_TOKENS
    JWT_TOKENS --> ROLE_CHECK
    ROLE_CHECK --> PERMISSION_MATRIX
    PERMISSION_MATRIX --> ACCESS_CONTROL
    
    ACCESS_CONTROL --> MIDDLEWARE_SEC
    MIDDLEWARE_SEC --> RATE_LIMITING
    RATE_LIMITING --> INPUT_VALIDATION
    
    INPUT_VALIDATION --> ENCRYPTION
    ENCRYPTION --> SQL_INJECTION
    SQL_INJECTION --> XSS_PROTECTION
```

## Deployment Architecture

```mermaid
graph TB
    subgraph "Production Environment"
        subgraph "Frontend Deployment"
            VERCEL[Vercel Platform]
            EDGE[Edge Functions]
            CDN_GLOBAL[Global CDN]
        end
        
        subgraph "Database"
            NEON_DB[Neon PostgreSQL]
            CONNECTION_POOL[Connection Pooling]
            BACKUP[Automated Backups]
        end
        
        subgraph "External Services"
            VONAGE_PROD[Vonage Video API]
            CLERK_PROD[Clerk Production]
            PAYMENT_PROD[Clerk Billing]
        end
        
        subgraph "Monitoring"
            ANALYTICS[Analytics]
            LOGS[Application Logs]
            METRICS[Performance Metrics]
        end
    end
    
    VERCEL --> EDGE
    EDGE --> CDN_GLOBAL
    VERCEL --> NEON_DB
    NEON_DB --> CONNECTION_POOL
    NEON_DB --> BACKUP
    
    VERCEL --> VONAGE_PROD
    VERCEL --> CLERK_PROD
    VERCEL --> PAYMENT_PROD
    
    VERCEL --> ANALYTICS
    VERCEL --> LOGS
    VERCEL --> METRICS
```

## Key Architecture Decisions

### 1. **Next.js App Router**
- **Server Components**: For better performance and SEO
- **Server Actions**: For form handling without API routes
- **Middleware**: For authentication and route protection

### 2. **Database Design**
- **Prisma ORM**: Type-safe database operations
- **PostgreSQL**: ACID compliance for financial transactions
- **Connection Pooling**: Efficient database connections

### 3. **Authentication Strategy**
- **Clerk**: Complete auth solution with role management
- **JWT Tokens**: Stateless authentication
- **Role-based Access**: Granular permissions

### 4. **Video Integration**
- **Vonage Video API**: Enterprise-grade video calling
- **WebRTC**: Low-latency communication
- **Session Management**: Secure token-based access

### 5. **Payment Processing**
- **Clerk Billing**: Integrated subscription management
- **Credit System**: Internal currency for appointments
- **Transaction Logging**: Complete audit trail

This architecture ensures scalability, security, and maintainability while providing a seamless user experience for both patients and doctors.
