# Business Flows - Shift Express

## Shift Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Pending: Manager creates shift

    Pending --> Published: Manager publishes
    Published --> Filled: Enough applications accepted
    Published --> Expired: Start date passed

    Filled --> Completed: Shift finished successfully
    Filled --> Cancelled: Manager cancels

    Expired --> [*]
    Completed --> [*]
    Cancelled --> [*]

    note right of Published
        Visible to employees
        Can receive applications
    end note

    note right of Filled
        required_employees = filled_employees
        No more applications accepted
    end note

    note right of Completed
        Gamification trigger:
        - Points awarded
        - Challenges verified
        - Stats updated
    end note
```

## Shift Application Flow

```mermaid
sequenceDiagram
    actor E as Employee
    participant M as Mobile App
    participant S as Supabase
    participant N as Notifications
    participant G as Gamification
    actor R as Manager

    E->>M: View available shifts
    M->>S: SELECT shifts WHERE status='published'
    S-->>M: Shifts list

    E->>M: Apply to a shift
    M->>S: INSERT shift_application
    Note over S: Calculate response_time_seconds
    S-->>M: Application created

    S->>N: Notify manager
    N->>R: "New application"

    R->>M: Review application
    R->>M: Accept
    M->>S: UPDATE application status='accepted'

    Note over S: Check if shift filled
    alt Shift filled
        S->>S: UPDATE shift status='filled'
    end

    S->>N: Notify employee
    N->>E: "Application accepted"

    Note over E,G: After the shift
    S->>G: Shift completed
    G->>G: Award points
    G->>G: Verify challenges
    G->>G: Unlock badges
    G->>E: "Badge unlocked!"
```

## Gamification System

```mermaid
flowchart TD
    START[Employee completes shift] --> UPDATE_STATS[Update user_stats]

    UPDATE_STATS --> POINTS{Calculate points}
    POINTS --> ADD_BASE[+100 base points]
    POINTS --> ADD_BONUS{Bonus?}

    ADD_BONUS -->|Response < 2min| FAST[+50 points]
    ADD_BONUS -->|Weekend| WEEKEND[+30 points]
    ADD_BONUS -->|Night| NIGHT[+20 points]

    FAST & WEEKEND & NIGHT --> TOTAL[Total points]
    ADD_BASE --> TOTAL

    TOTAL --> CHECK_CHALLENGES[Check weekly challenges]
    CHECK_CHALLENGES --> UPDATE_PROGRESS[Increment progress]

    UPDATE_PROGRESS --> CHALLENGE_COMPLETE{Challenge completed?}
    CHALLENGE_COMPLETE -->|Yes| REWARD_POINTS[+Reward points]
    CHALLENGE_COMPLETE -->|No| NEXT

    REWARD_POINTS --> CHECK_BADGES[Check badges]
    CHECK_BADGES --> UNLOCK_BADGE{Criteria met?}

    UNLOCK_BADGE -->|Yes| ADD_COSMETIC[Add to user_cosmetics]
    UNLOCK_BADGE -->|No| NEXT

    ADD_COSMETIC --> NOTIFY_UNLOCK[Badge unlock notification]
    NOTIFY_UNLOCK --> NEXT[Check level]

    NEXT --> LEVEL_UP{Enough points?}
    LEVEL_UP -->|Yes| INCREMENT_LEVEL[Level +1]
    LEVEL_UP -->|No| END

    INCREMENT_LEVEL --> END[Gamification complete]

    style START fill:#90EE90
    style END fill:#FFD700
    style ADD_COSMETIC fill:#FF69B4
    style NOTIFY_UNLOCK fill:#FF69B4
```

## Employee Invitation Flow

```mermaid
sequenceDiagram
    actor M as Manager
    participant W as Web App
    participant S as Supabase
    participant E as Email Service
    actor N as New Employee

    M->>W: Invite employee
    W->>S: INSERT team_invitation
    Note over S: Generate unique token
    S-->>W: Invitation created

    S->>E: Trigger email
    E->>N: "You're invited!"
    Note over E,N: Link: /accept-invite?token=xxx

    N->>W: Click link
    W->>S: SELECT invitation WHERE token=xxx
    S-->>W: Valid invitation

    N->>W: Create account
    W->>S: auth.signUp()
    S-->>W: User created

    W->>S: INSERT INTO users
    W->>S: INSERT INTO employees
    W->>S: UPDATE invitation accepted_at=NOW()

    S->>S: Trigger gamification initialization
    Note over S: Create user_stats<br/>Unlock starter cosmetics

    S-->>N: Account created and ready
```

## Notification System

```mermaid
flowchart LR
    subgraph "Triggers"
        T1[New application]
        T2[Application accepted]
        T3[Shift cancelled]
        T4[Badge unlocked]
        T5[Challenge completed]
        T6[Invitation received]
    end

    subgraph "Notification Engine"
        CHECK{User has device?}
    end

    subgraph "Channels"
        PUSH[Push Notification<br/>Expo]
        EMAIL[Email<br/>Resend]
        IN_APP[In-App<br/>Red badge]
    end

    T1 & T2 & T3 & T4 & T5 & T6 --> CHECK

    CHECK -->|Yes| PUSH
    CHECK --> EMAIL
    CHECK --> IN_APP

    PUSH --> USER[User]
    EMAIL --> USER
    IN_APP --> USER

    style T4 fill:#FFD700
    style T5 fill:#FFD700
    style PUSH fill:#90EE90
```

## Subscription Flow (Stripe)

```mermaid
sequenceDiagram
    actor A as Admin
    participant W as Web App
    participant API as API Server
    participant ST as Stripe
    participant S as Supabase

    A->>W: Choose plan (Pro/Enterprise)
    W->>API: POST /create-subscription

    API->>ST: Create Customer
    ST-->>API: customer_id

    API->>S: UPDATE companies SET stripe_customer_id

    API->>ST: Create Checkout Session
    ST-->>API: session_url
    API-->>W: Redirect URL

    W->>ST: Redirect to Stripe Checkout
    A->>ST: Card payment

    ST->>API: Webhook: checkout.session.completed
    API->>S: UPDATE subscription_status='active'
    API->>S: UPDATE subscription_plan='professional'

    ST->>API: Webhook: invoice.paid (monthly)
    API->>S: Record payment

    alt Payment failed
        ST->>API: Webhook: invoice.payment_failed
        API->>S: UPDATE subscription_status='past_due'
        API->>A: Payment reminder email
    end
```

## Weekly Challenge Reset

```mermaid
flowchart TD
    CRON[Cron Job<br/>Every Monday 00:00] --> GET_WEEK[Calculate week_start_date]

    GET_WEEK --> RESET_STATS[Reset weekly stats]
    RESET_STATS --> RESET_WEEKLY{For each user}

    RESET_WEEKLY --> R1[weekly_points = 0]
    RESET_WEEKLY --> R2[responses_under_2min = 0]
    RESET_WEEKLY --> R3[morning_shifts_accepted = 0]
    RESET_WEEKLY --> R4[...]

    R1 & R2 & R3 & R4 --> UPDATE_PROGRESS[Reset user_weekly_progress]

    UPDATE_PROGRESS --> NEW_WEEK{For each active challenge}
    NEW_WEEK --> INSERT[INSERT new progress row]
    INSERT --> DONE[New week ready]

    style CRON fill:#FFD700
    style DONE fill:#90EE90
```

## No-Show Management

```mermaid
sequenceDiagram
    actor M as Manager
    participant W as Web App
    participant S as Supabase
    participant G as Gamification
    actor E as Employee

    Note over M: Shift ended,<br/>Employee absent

    M->>W: Report no-show
    W->>S: UPDATE shift_application<br/>SET no_show=true

    S->>G: Trigger penalty

    G->>S: Remove points (-50)
    G->>S: Increment total_shifts_cancelled
    G->>S: Reset no_cancellation_streak = 0
    G->>S: Lower reliability_score

    Note over G: If too many no-shows:<br/>Suspend account

    G->>E: "No-show warning"

    alt 3 no-shows in 1 month
        S->>S: UPDATE employees<br/>SET active=false
        G->>E: "Account suspended"
    end
```

---

**Note:** These diagrams are in Mermaid format and render automatically on GitHub!
