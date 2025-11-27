# 🔄 Flows Métier - Shift Express

## 📋 Cycle de Vie d'un Shift

```mermaid
stateDiagram-v2
    [*] --> Pending: Manager crée shift

    Pending --> Published: Manager publie
    Published --> Filled: Assez de candidatures acceptées
    Published --> Expired: Date de début passée

    Filled --> Completed: Shift terminé avec succès
    Filled --> Cancelled: Manager annule

    Expired --> [*]
    Completed --> [*]
    Cancelled --> [*]

    note right of Published
        Visible par les employés
        Peut recevoir candidatures
    end note

    note right of Filled
        required_employees = filled_employees
        Plus de candidatures acceptées
    end note

    note right of Completed
        Trigger gamification :
        - Points accordés
        - Défis vérifiés
        - Stats mises à jour
    end note
```

## 🙋 Flow de Candidature à un Shift

```mermaid
sequenceDiagram
    actor E as Employee
    participant M as Mobile App
    participant S as Supabase
    participant N as Notifications
    participant G as Gamification
    actor R as Manager

    E->>M: Voir shifts disponibles
    M->>S: SELECT shifts WHERE status='published'
    S-->>M: Liste shifts

    E->>M: Postuler à un shift
    M->>S: INSERT shift_application
    Note over S: Calcul response_time_seconds
    S-->>M: Application créée

    S->>N: Notifier manager
    N->>R: 📱 "Nouvelle candidature"

    R->>M: Examiner candidature
    R->>M: Accepter
    M->>S: UPDATE application status='accepted'

    Note over S: Vérifier si shift rempli
    alt Shift rempli
        S->>S: UPDATE shift status='filled'
    end

    S->>N: Notifier employee
    N->>E: 📱 "Candidature acceptée"

    Note over E,G: Après le shift
    S->>G: Shift completed
    G->>G: Accorder points
    G->>G: Vérifier défis
    G->>G: Débloquer badges
    G->>E: 🏆 "Badge débloqué!"
```

## 🎮 Système de Gamification

```mermaid
flowchart TD
    START[Employee complète shift] --> UPDATE_STATS[Mettre à jour user_stats]

    UPDATE_STATS --> POINTS{Calculer points}
    POINTS --> ADD_BASE[+100 points base]
    POINTS --> ADD_BONUS{Bonus ?}

    ADD_BONUS -->|Réponse < 2min| FAST[+50 points]
    ADD_BONUS -->|Weekend| WEEKEND[+30 points]
    ADD_BONUS -->|Nuit| NIGHT[+20 points]

    FAST & WEEKEND & NIGHT --> TOTAL[Total points]
    ADD_BASE --> TOTAL

    TOTAL --> CHECK_CHALLENGES[Vérifier défis hebdo]
    CHECK_CHALLENGES --> UPDATE_PROGRESS[Incrémenter progress]

    UPDATE_PROGRESS --> CHALLENGE_COMPLETE{Défi complété ?}
    CHALLENGE_COMPLETE -->|Oui| REWARD_POINTS[+Points récompense]
    CHALLENGE_COMPLETE -->|Non| NEXT

    REWARD_POINTS --> CHECK_BADGES[Vérifier badges]
    CHECK_BADGES --> UNLOCK_BADGE{Critère atteint ?}

    UNLOCK_BADGE -->|Oui| ADD_COSMETIC[Ajouter à user_cosmetics]
    UNLOCK_BADGE -->|Non| NEXT

    ADD_COSMETIC --> NOTIFY_UNLOCK[📱 Notif badge débloqué]
    NOTIFY_UNLOCK --> NEXT[Vérifier niveau]

    NEXT --> LEVEL_UP{Assez points ?}
    LEVEL_UP -->|Oui| INCREMENT_LEVEL[Niveau +1]
    LEVEL_UP -->|Non| END

    INCREMENT_LEVEL --> END[✅ Gamification terminée]

    style START fill:#90EE90
    style END fill:#FFD700
    style ADD_COSMETIC fill:#FF69B4
    style NOTIFY_UNLOCK fill:#FF69B4
```

## 📧 Flow d'Invitation Employé

```mermaid
sequenceDiagram
    actor M as Manager
    participant W as Web App
    participant S as Supabase
    participant E as Email Service
    actor N as New Employee

    M->>W: Inviter employé
    W->>S: INSERT team_invitation
    Note over S: Générer token unique
    S-->>W: Invitation créée

    S->>E: Trigger email
    E->>N: 📧 "Vous êtes invité!"
    Note over E,N: Lien: /accept-invite?token=xxx

    N->>W: Cliquer lien
    W->>S: SELECT invitation WHERE token=xxx
    S-->>W: Invitation valide

    N->>W: Créer compte
    W->>S: auth.signUp()
    S-->>W: User créé

    W->>S: INSERT INTO users
    W->>S: INSERT INTO employees
    W->>S: UPDATE invitation accepted_at=NOW()

    S->>S: Trigger initialisation gamification
    Note over S: Créer user_stats<br/>Débloquer cosmétiques de départ

    S-->>N: ✅ Compte créé et prêt
```

## 🔔 Système de Notifications

```mermaid
flowchart LR
    subgraph "Triggers"
        T1[Nouvelle candidature]
        T2[Candidature acceptée]
        T3[Shift annulé]
        T4[Badge débloqué]
        T5[Défi complété]
        T6[Invitation reçue]
    end

    subgraph "Notification Engine"
        CHECK{User a device?}
    end

    subgraph "Channels"
        PUSH[Push Notification<br/>Expo]
        EMAIL[Email<br/>Resend]
        IN_APP[In-App<br/>Badge rouge]
    end

    T1 & T2 & T3 & T4 & T5 & T6 --> CHECK

    CHECK -->|Oui| PUSH
    CHECK --> EMAIL
    CHECK --> IN_APP

    PUSH --> USER[📱 User]
    EMAIL --> USER
    IN_APP --> USER

    style T4 fill:#FFD700
    style T5 fill:#FFD700
    style PUSH fill:#90EE90
```

## 💳 Flow de Souscription (Stripe)

```mermaid
sequenceDiagram
    actor A as Admin
    participant W as Web App
    participant API as API Server
    participant ST as Stripe
    participant S as Supabase

    A->>W: Choisir plan (Pro/Enterprise)
    W->>API: POST /create-subscription

    API->>ST: Créer Customer
    ST-->>API: customer_id

    API->>S: UPDATE companies SET stripe_customer_id

    API->>ST: Créer Checkout Session
    ST-->>API: session_url
    API-->>W: Redirect URL

    W->>ST: Redirection Stripe Checkout
    A->>ST: Paiement carte

    ST->>API: Webhook: checkout.session.completed
    API->>S: UPDATE subscription_status='active'
    API->>S: UPDATE subscription_plan='professional'

    ST->>API: Webhook: invoice.paid (mensuel)
    API->>S: Enregistrer paiement

    alt Paiement échoué
        ST->>API: Webhook: invoice.payment_failed
        API->>S: UPDATE subscription_status='past_due'
        API->>A: 📧 Email relance paiement
    end
```

## 🔄 Reset Hebdomadaire des Défis

```mermaid
flowchart TD
    CRON[Cron Job<br/>Chaque lundi 00:00] --> GET_WEEK[Calculer week_start_date]

    GET_WEEK --> RESET_STATS[Reset stats hebdo]
    RESET_STATS --> RESET_WEEKLY{Pour chaque user}

    RESET_WEEKLY --> R1[weekly_points = 0]
    RESET_WEEKLY --> R2[responses_under_2min = 0]
    RESET_WEEKLY --> R3[morning_shifts_accepted = 0]
    RESET_WEEKLY --> R4[...]

    R1 & R2 & R3 & R4 --> UPDATE_PROGRESS[Reset user_weekly_progress]

    UPDATE_PROGRESS --> NEW_WEEK{Pour chaque challenge actif}
    NEW_WEEK --> INSERT[INSERT new progress row]
    INSERT --> DONE[✅ Nouvelle semaine prête]

    style CRON fill:#FFD700
    style DONE fill:#90EE90
```

## 🚨 Gestion No-Show

```mermaid
sequenceDiagram
    actor M as Manager
    participant W as Web App
    participant S as Supabase
    participant G as Gamification
    actor E as Employee

    Note over M: Shift terminé,<br/>Employee absent

    M->>W: Signaler no-show
    W->>S: UPDATE shift_application<br/>SET no_show=true

    S->>G: Trigger pénalité

    G->>S: Retirer points (-50)
    G->>S: Incrémenter total_shifts_cancelled
    G->>S: Reset no_cancellation_streak = 0
    G->>S: Baisser reliability_score

    Note over G: Si trop de no-shows:<br/>Suspendre compte

    G->>E: 📧 "Attention no-show"

    alt 3 no-shows en 1 mois
        S->>S: UPDATE employees<br/>SET active=false
        G->>E: 📧 "Compte suspendu"
    end
```

---

**Note :** Ces diagrammes sont en Mermaid et s'affichent automatiquement sur GitHub !
