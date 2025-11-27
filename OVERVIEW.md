# Architecture - Shift Express

**Version**: 2.0.0
**Last updated**: 2025-11-12

---

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Database Architecture](#database-architecture)
- [Data Flows](#data-flows)
- [Multi-Tenant Security](#multi-tenant-security)
- [Deployment](#deployment)
- [Scalability](#scalability)

---

## Overview

Shift Express is a **B2B SaaS platform** for temporary shift management with employee gamification.

### Tech Stack

```mermaid
graph TB
    subgraph "Frontend"
        Mobile[Mobile App<br/>React Native + Expo]
        Web[Landing Page<br/>Next.js]
        CRM[CRM Admin<br/>Next.js]
    end

    subgraph "Backend"
        API[Custom API<br/>Express.js + TypeScript]
        Supabase[Supabase<br/>PostgreSQL + Auth + Storage]
        Redis[Redis Cache<br/>Upstash]
    end

    subgraph "Deployment"
        Vercel[Vercel<br/>API + Web + CRM]
        EAS[EAS Build<br/>Mobile iOS/Android]
    end

    Mobile --> API
    Web --> API
    CRM --> API
    API --> Supabase
    API --> Redis
    API --> Vercel
    Mobile --> EAS
    Web --> Vercel
    CRM --> Vercel

    style Mobile fill:#61dafb
    style Web fill:#61dafb
    style CRM fill:#61dafb
    style API fill:#68a063
    style Supabase fill:#3ecf8e
    style Redis fill:#dc382d
    style Vercel fill:#000000
    style EAS fill:#000020
```

### Key Versions and Dependencies

#### Mobile App (`apps/mobile`)

| Library | Version | Usage |
|---------|---------|-------|
| **React** | 19.1.0 | **React 19** (Compatible Expo 54+) |
| **React Native** | 0.81.4 | Mobile framework |
| **Expo** | 54.0.13 | Development platform |
| **Expo Router** | 6.0.12 | File-based navigation |
| **TypeScript** | 5.3.3 | Static typing |
| **Zustand** | 4.4.7 | State management |
| **TanStack Query** | 5.17.9 | Server state & cache |
| **Supabase JS** | 2.39.0 | Backend client |
| **i18next** | 25.5.3 | Internationalization (FR/EN) |
| **Sentry** | 7.2.0 | Error tracking |
| **Lucide Icons** | 0.544.0 | React Native icons |
| **Confetti Cannon** | 1.5.2 | Gamification animations |
| **Flash List** | 2.2.0 | High-performance lists |
| **date-fns** | 3.0.6 | Date manipulation |

**Important note:** The mobile app uses **React 19** (instead of React 18), which is compatible with Expo 54+. This version brings performance improvements and future-proofs the project.

#### Web App (`apps/web`)

| Library | Version | Usage |
|---------|---------|-------|
| **React** | 18.3.1 | UI library (stable) |
| **React DOM** | 18.3.1 | React for web |
| **Vite** | 5.0.11 | Ultra-fast build tool |
| **TypeScript** | 5.3.3 | Static typing |
| **React Router** | 6.21.1 | Routing |
| **Zustand** | 4.4.7 | State management |
| **TanStack Query** | 5.17.9 | Server state & cache |
| **TanStack Table** | 8.11.2 | Complex tables |
| **Supabase JS** | 2.39.0 | Backend client |
| **i18next** | 25.5.3 | Internationalization (FR/EN) |
| **Stripe JS** | 2.4.0 | Payments |
| **Framer Motion** | 12.23.24 | Animations |
| **Recharts** | 2.15.4 | Charts & analytics |
| **React Hook Form** | 7.63.0 | Optimized forms |
| **Zod** | 3.25.76 | Schema validation |
| **Tailwind CSS** | 3.4.0 | Utility-first styling |
| **Lucide React** | 0.303.0 | React icons |
| **html2canvas** | 1.4.1 | Screenshots & exports |
| **jsPDF** | 3.0.3 | PDF generation |
| **XLSX** | 0.18.5 | Excel export |
| **Sentry** | 10.24.0 | Error tracking |
| **Sonner** | 1.3.1 | Toast notifications |
| **next-themes** | 0.4.6 | Dark mode |

#### API Server (`apps/api`)

| Library | Version | Usage |
|---------|---------|-------|
| **Express** | 4.18.2 | Backend framework |
| **TypeScript** | 5.3.3 | Static typing |
| **Supabase JS** | 2.39.0 | Database & auth client |
| **Redis (ioredis)** | 5.3.2 | Caching & rate limiting |
| **Stripe** | 14.10.0 | Webhooks & payments |
| **Winston** | 3.11.0 | Structured logging |
| **Logtail** | 0.4.3 | Log aggregation |
| **Helmet** | 7.1.0 | Security headers |
| **CORS** | 2.8.5 | Cross-origin handling |
| **Zod** | 3.22.4 | API validation |
| **express-rate-limit** | 7.1.5 | Rate limiting |
| **rate-limit-redis** | 4.2.0 | Rate limit storage |

#### Database & Backend

| Service | Version | Usage |
|---------|---------|-------|
| **PostgreSQL** | 15.x | Main database |
| **Supabase** | Latest | BaaS (Auth, Storage, RLS) |
| **Upstash Redis** | Latest | Distributed cache |
| **Supabase Edge Functions** | Deno runtime | Webhooks & background jobs |

---

## System Architecture

### Monorepo Structure

```
shift-express/
├── apps/
│   ├── api/                # Express.js Backend
│   ├── web/                # Next.js Landing Page
│   ├── mobile/             # Expo Mobile App
│   └── crm-admin/          # Next.js CRM
├── packages/
│   ├── types/              # Shared TypeScript types
│   └── utils/              # Shared utilities
├── supabase/
│   ├── migrations/         # SQL migrations
│   ├── functions/          # Edge Functions (Deno)
│   └── seed.sql            # Initial data
└── docs/                   # Documentation
```

---

## Database Architecture

### Complete ERD Schema

```mermaid
erDiagram
    companies ||--o{ establishments : "has"
    companies ||--o{ users : "employs"
    companies ||--o{ positions : "defines"
    companies ||--o{ tickets : "creates"

    establishments ||--o{ employees : "hosts"
    establishments ||--o{ shifts : "publishes"
    establishments ||--o{ establishment_invitations : "generates"

    users ||--o{ employees : "is"
    users ||--o{ user_stats : "tracks"
    users ||--o{ user_cosmetics : "owns"
    users ||--o{ user_weekly_progress : "progresses"
    users ||--o{ team_invitations : "receives"

    positions ||--o{ employees : "fills"
    positions ||--o{ shifts : "requires"

    shifts ||--o{ shift_applications : "receives"

    employees ||--o{ shift_applications : "submits"
    employees ||--o{ employee_devices : "uses"

    cosmetic_items ||--o{ user_cosmetics : "unlocked_via"
    weekly_challenges ||--o{ user_weekly_progress : "tracks"

    companies {
        uuid id PK
        text name
        enum subscription_plan
        enum subscription_status
        text stripe_customer_id
        text stripe_subscription_id
    }

    establishments {
        uuid id PK
        uuid company_id FK
        text name
        text address
        enum status
    }

    users {
        uuid id PK
        text email
        enum role
        uuid company_id FK
        uuid assigned_establishment_id FK
        int points
        int level
    }

    shifts {
        uuid id PK
        uuid company_id FK
        uuid establishment_id FK
        uuid position_id FK
        timestamptz start_time
        timestamptz end_time
        enum status
        int required_employees
        int filled_employees
    }

    shift_applications {
        uuid id PK
        uuid shift_id FK
        uuid user_id FK
        enum status
        int response_time_seconds
        bool no_show
        timestamptz applied_at
    }

    user_stats {
        uuid user_id PK
        int total_points
        int weekly_points
        int current_streak_days
        int responses_under_2min
        int weekend_shifts_completed
        uuid equipped_avatar_id FK
        uuid equipped_badge_id FK
    }

    cosmetic_items {
        uuid id PK
        text type
        text name
        text rarity
        int points_cost
        jsonb unlock_requirement
        text preview_url
    }

    weekly_challenges {
        uuid id PK
        text challenge_key UNIQUE
        text name
        int points_reward
        int target_count
        bool is_active
    }

    user_weekly_progress {
        uuid id PK
        uuid user_id FK
        uuid challenge_id FK
        date week_start_date
        int current_count
        bool completed
        bool claimed
    }
```

---

## Data Flows

### 1. Authentication Flow

```mermaid
sequenceDiagram
    participant Mobile
    participant API
    participant Supabase
    participant Database

    Mobile->>API: POST /auth/login<br/>{email, password}
    API->>Supabase: supabase.auth.signInWithPassword()
    Supabase->>Database: Verify credentials + RLS
    Database-->>Supabase: User data
    Supabase-->>API: {user, session, access_token}
    API->>Database: Check user role
    Database-->>API: role = 'employee'
    API-->>Mobile: {user, token, role}
    Mobile->>Mobile: Store token in AsyncStorage

    Note over Mobile,Database: JWT expires after 1 hour
```

---

### 2. Shift Application Flow

```mermaid
sequenceDiagram
    participant Employee
    participant Mobile
    participant API
    participant Database
    participant Gamification
    participant PushNotif

    Employee->>Mobile: Apply to shift
    Mobile->>API: POST /api/applications<br/>{shift_id, user_id}
    API->>Database: INSERT shift_application
    Database->>Database: Trigger: Calculate response_time
    Database->>Gamification: increment_challenge_progress('accept_3_shifts')
    Gamification->>Database: UPDATE user_weekly_progress
    Database->>Database: Trigger: Award points (+30)
    Database->>Gamification: check_and_unlock_badges()
    Gamification-->>Database: Badges unlocked (if eligible)
    Database-->>API: Application created
    API->>PushNotif: Send notification to manager
    PushNotif-->>API: Notification sent
    API-->>Mobile: {success, application_id, points_gained: 30}
    Mobile-->>Employee: Confirmation + XP toast
```

---

### 3. Weekly Challenge Claim Flow

```mermaid
sequenceDiagram
    participant Employee
    participant Mobile
    participant API
    participant Redis
    participant Database

    Employee->>Mobile: Claim challenge reward
    Mobile->>API: POST /api/gamification/challenges/:id/claim
    API->>Redis: Check rate limit<br/>incr("rate:user:claim")
    Redis-->>API: count = 1 (OK, <20 requests)

    API->>Database: SELECT claim_challenge_reward(user_id, challenge_id, week_start)
    Database->>Database: BEGIN TRANSACTION
    Database->>Database: Check completed = true AND claimed = false
    Database->>Database: Award points (+150)
    Database->>Database: UPDATE user_weekly_progress SET claimed = true
    Database->>Database: COMMIT
    Database-->>API: {success: true, points_awarded: 150}

    API->>Redis: Invalidate cache<br/>del("challenges-overview:user")
    API-->>Mobile: {success, points: 150, new_total: 1350}
    Mobile-->>Employee: Celebration modal<br/>+150 pts!
```

---

## Multi-Tenant Security

### Row Level Security (RLS) Architecture

```mermaid
graph TB
    subgraph "Security Layers"
        JWT[JWT Token<br/>auth.uid]
        RLS[Row Level Security<br/>PostgreSQL]
        Policies[RLS Policies]
    end

    subgraph "Employee Access"
        E1[View own applications]
        E2[View company shifts]
        E3[View own stats]
        E4[Apply to shifts]
    end

    subgraph "Manager Access"
        M1[View establishment shifts]
        M2[Accept/reject applications]
        M3[Create shifts]
        M4[View employees]
    end

    subgraph "Admin Access"
        A1[Full company access]
        A2[Manage establishments]
        A3[View all stats]
        A4[Manage subscription]
    end

    JWT --> RLS
    RLS --> Policies
    Policies --> E1 & E2 & E3 & E4
    Policies --> M1 & M2 & M3 & M4
    Policies --> A1 & A2 & A3 & A4

    style JWT fill:#3ecf8e
    style RLS fill:#ff6b6b
    style E1 fill:#90EE90
    style M1 fill:#FFD700
    style A1 fill:#FF6347
```

---

### Multi-Tenant Data Isolation

```mermaid
graph LR
    subgraph "Company A - Restaurant Chain"
        CA[Company A<br/>Pro Plan]
        EA1[Paris 8th<br/>Champs-Elysees]
        EA2[Lyon Part-Dieu]
        EA3[Marseille Old Port]

        CA --> EA1
        CA --> EA2
        CA --> EA3

        UA1[25 Employees]
        PA1[5 Positions]
        SA1[150 Shifts/month]

        CA --> UA1
        EA1 --> PA1
        EA1 --> SA1
    end

    subgraph "Company B - Hotel Group"
        CB[Company B<br/>Ultra Plan]
        EB1[Nice Hotel]
        EB2[Cannes Hotel]

        CB --> EB1
        CB --> EB2

        UB1[60 Employees]
        PB1[12 Positions]
        SB1[400 Shifts/month]

        CB --> UB1
        EB1 --> PB1
        EB1 --> SB1
    end

    RLS[Row Level Security<br/>Isolation]

    RLS -.company_id Filter.-> CA
    RLS -.company_id Filter.-> CB

    style RLS fill:#FF6347,color:#fff
    style CA fill:#61dafb
    style CB fill:#90EE90
```

---

## Deployment

### Infrastructure Overview

```mermaid
graph TB
    subgraph "Source Code"
        GitHub[GitHub Repository<br/>monorepo]
    end

    subgraph "CI/CD Pipeline"
        Actions[GitHub Actions]
        Tests[Tests + Type Check]
        Migrations[Supabase Migrations]
    end

    subgraph "Production Environment"
        VercelAPI[Vercel<br/>API + Web + CRM]
        SupabaseProd[Supabase Production<br/>PostgreSQL 15]
        RedisProd[Upstash Redis<br/>Serverless]
        EASProd[EAS Build<br/>iOS + Android]
    end

    GitHub --> Actions
    Actions --> Tests
    Tests --> Migrations
    Migrations --> SupabaseProd
    Actions --> VercelAPI
    Actions --> EASProd
    VercelAPI --> SupabaseProd
    VercelAPI --> RedisProd

    style GitHub fill:#24292e,color:#fff
    style Actions fill:#2088FF,color:#fff
    style VercelAPI fill:#000000,color:#fff
    style SupabaseProd fill:#3ecf8e
    style RedisProd fill:#dc382d,color:#fff
```

---

### Deployment Workflow

```mermaid
sequenceDiagram
    participant Dev
    participant GitHub
    participant Actions
    participant Vercel
    participant Supabase
    participant EAS

    Dev->>GitHub: git push origin main
    GitHub->>Actions: Trigger CI/CD workflow

    Actions->>Actions: npm run test (Vitest)
    Actions->>Actions: npm run type-check
    Actions->>Actions: Build API + Web + CRM

    alt Tests Pass
        Actions->>Supabase: npx supabase db push
        Supabase-->>Actions: Migrations applied

        Actions->>Vercel: Deploy API + Web + CRM
        Vercel-->>Actions: Deployed (3 apps)

        par Mobile Build (Optional)
            Actions->>EAS: eas build --platform all
            EAS-->>Actions: APK + IPA ready
        end

        Actions-->>Dev: Deployment success
    else Tests Fail
        Actions-->>Dev: Deployment blocked
    end
```

---

## Scalability

### Horizontal Scaling Strategy

```mermaid
graph TB
    subgraph "Load Distribution"
        LB[Vercel Edge Network<br/>Global CDN]
        API1[API Instance 1<br/>us-east-1]
        API2[API Instance 2<br/>eu-west-1]
        API3[API Instance 3<br/>ap-southeast-1]
    end

    subgraph "Database Layer"
        Primary[Primary DB<br/>Write Operations]
        Replica1[Read Replica 1<br/>Europe]
        Replica2[Read Replica 2<br/>Asia]
    end

    subgraph "Cache Layer"
        Redis[Redis Cluster<br/>Upstash Serverless]
        CDN[Static Assets CDN<br/>Vercel Edge]
    end

    LB --> API1
    LB --> API2
    LB --> API3

    API1 --> Redis
    API2 --> Redis
    API3 --> Redis

    API1 --> Primary
    API2 --> Replica1
    API3 --> Replica2

    Primary -.Replication.-> Replica1
    Primary -.Replication.-> Replica2

    style LB fill:#000000,color:#fff
    style Redis fill:#dc382d,color:#fff
    style Primary fill:#3ecf8e
```

---

### Performance Bottlenecks & Solutions

```mermaid
graph LR
    Request[1000 concurrent users] --> RateLimit{Rate Limiting<br/>Redis}

    RateLimit -->|Pass| Cache{Redis Cache}
    RateLimit -->|Block| Response429[429 Too Many Requests]

    Cache -->|Hit 87%| FastResponse[Fast Response<br/>~15ms]
    Cache -->|Miss 13%| Database{PostgreSQL}

    Database -->|Simple Query| MediumResponse[Medium Response<br/>~50ms]
    Database -->|Complex Query| SlowResponse[Slow Response<br/>~200ms]

    SlowResponse --> Bottleneck[Potential Bottleneck]

    Bottleneck -.Solution 1.-> AddIndex[Add DB Indexes]
    Bottleneck -.Solution 2.-> CacheMore[Increase Cache TTL]
    Bottleneck -.Solution 3.-> Replicas[Use Read Replicas]

    style Bottleneck fill:#ff6b6b,color:#fff
    style FastResponse fill:#51cf66,color:#fff
    style MediumResponse fill:#ffd43b
    style SlowResponse fill:#ffa94d
```

---

### Cache Strategy

```mermaid
graph TB
    Request[API Request] --> CheckCache{Check Redis Cache}

    CheckCache -->|Hit| ReturnCached[Return Cached Data<br/>~15ms]
    CheckCache -->|Miss| QueryDB[Query PostgreSQL]

    QueryDB --> ProcessData[Process Data]
    ProcessData --> StoreCache[Store in Redis<br/>TTL: 1-5min]
    StoreCache --> ReturnFresh[Return Fresh Data<br/>~50-200ms]

    subgraph "Cache Keys"
        K1[leaderboard:weekly<br/>TTL: 5min]
        K2[challenges-overview:user-uuid<br/>TTL: 1min]
        K3[cosmetics:all<br/>TTL: 5min]
        K4[user-stats:user-uuid<br/>TTL: 1min]
    end

    style ReturnCached fill:#51cf66,color:#fff
    style ReturnFresh fill:#ffd43b
```

---

## External Services

### Integrations Architecture

```mermaid
graph TB
    App[Shift Express API]

    App --> Stripe[Stripe<br/>Payments & Subscriptions]
    App --> Expo[Expo Push Notifications<br/>Mobile]
    App --> Resend[Resend<br/>Transactional Emails]
    App --> Sentry[Sentry<br/>Error Tracking]

    Stripe -.Webhooks.-> App
    Supabase -.Webhooks.-> App

    subgraph "Webhook Handlers"
        W1[/api/stripe/webhook<br/>invoice.paid]
        W2[/api/stripe/webhook<br/>subscription.updated]
        W3[/api/supabase/webhook<br/>user.created]
    end

    style Stripe fill:#635bff,color:#fff
    style Expo fill:#000020,color:#fff
    style Resend fill:#000000,color:#fff
    style Sentry fill:#362d59,color:#fff
```

---

## Monitoring & Observability

### Observability Stack

```mermaid
graph TB
    subgraph "Application"
        API[Express API]
        Mobile[Mobile App]
        Web[Web App]
    end

    subgraph "Monitoring Tools"
        Logs[Vercel Logs<br/>Real-time]
        Metrics[Vercel Analytics<br/>Performance]
        Errors[Sentry<br/>Error Tracking]
        DB[Supabase Dashboard<br/>Query Performance]
    end

    subgraph "Alerting"
        Slack[Slack Notifications]
        Email[Email Alerts]
    end

    API --> Logs
    API --> Metrics
    API --> Errors
    API --> DB

    Mobile --> Errors
    Web --> Metrics

    Errors --> Slack
    DB --> Email

    style Errors fill:#362d59,color:#fff
    style Slack fill:#4A154B,color:#fff
```

---

## References

- [ADRs](./adr/README.md) - Architecture Decision Records
- [Database Schema](./DATABASE.md) - Detailed 27 tables schema
- [API Documentation](./API.md) - 20 documented endpoints
- [Gamification System](./GAMIFICATION_SYSTEM.md) - Weekly challenges + Cosmetics
- [Troubleshooting](./TROUBLESHOOTING.md) - Debugging guide

---

**Version**: 2.0.0
**Last updated**: 2025-10-17
**Next review**: Quarterly
