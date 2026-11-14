# Libsa

**AI-powered daily outfit planning, built for Tunisia and the Maghreb.**

Libsa (Tunisian Arabic for "outfit") turns the clothes you already own into a ready-to-wear look for your day. Tell the app what is on your calendar and it returns one outfit in about 30 seconds, adapted to the occasion, the weather and the image you want to project. Available in Derja, French and English.

> Project code name: **Yalla**

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Data Model](#data-model)
- [Security and Privacy](#security-and-privacy)
- [Getting Started](#getting-started)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Most wardrobe apps start from the closet and lose users during setup. Libsa starts from **the day**: a presentation, an interview, a wedding, a henna night. The app combines the user's closet, the event, the dress code and the forecast to propose a single, explained outfit.

Libsa is designed around local needs often overlooked by global apps: Derja and French support, Tunisian events and seasons (weddings, henna, Ramadan, Aïd), and modest-friendly styling built into every flow.

## Key Features

| Feature | Description |
|---|---|
| **Today's Look** | Select an event and receive one outfit with a short "why it works" explanation. Shuffle for an alternative. |
| **Snap & Go** | Add many items at once through a pile scan, a closet video sweep or gallery import. AI performs cutout, cleanup and tagging. |
| **Big Days Calendar** | Save important events, plan the whole week and receive four reminders per event (7 days, 3 days, the evening before, the morning of). |
| **Stylist Chat** | Ask styling questions by text or voice and get answers in Derja, French or English. |
| **Smart Closet** | Wear tracking, cost per wear and "forgotten treasures" (items not worn for 90 days). |
| **Modest-Friendly Styling** | Preference-aware outfit generation across all features. |

## Architecture

Libsa uses a client-server architecture with a **modular monolith backend** (Supabase) and a **dedicated AI service**. Slow tasks such as scanning and reminders run as background jobs.

```
┌──────────────┐   HTTPS   ┌────────────────────────────────────┐
│ Flutter app  │ ────────► │ Supabase (EU)                      │
│ Android/iOS  │           │ Auth · PostgreSQL + pgvector ·     │
└──────────────┘           │ Storage · Edge Functions           │
                           └──────────────┬─────────────────────┘
                                          │ internal calls only
                                          ▼
                           ┌────────────────────────────────────┐
                           │ AI service (FastAPI, Cloud Run)    │
                           │ Scan · Tagging · Outfits · Chat    │
                           └────────────────────────────────────┘
```

**Logical layers:** Presentation (Flutter) → Business (backend modules) → AI service → Data and file storage.

**Design principles**

- One feature, one folder.
- Screens never access the database directly.
- Only the backend calls the AI service.
- Database changes are made exclusively through numbered migration files.

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile app | Dart, Flutter |
| AI service | Python, FastAPI, Google Cloud Run |
| Database | PostgreSQL with pgvector (Supabase, EU region) |
| Backend functions | TypeScript, Supabase Edge Functions |
| Authentication | Supabase Auth (JWT) |
| Notifications | Firebase Cloud Messaging |
| External services | Weather API, calendar integrations, LLM and vision APIs |

## Repository Structure

```
libsa/
├── app/             Flutter mobile application
├── backend/         Supabase migrations, security rules, edge functions
├── ai-service/      Python FastAPI service (scan, tagging, outfits, chat)
└── docs/            Architecture, API contract, design decisions
```

Flutter feature layout:

```
app/lib/
├── core/            Theme, shared widgets, router, services
└── features/
    ├── auth/
    ├── closet/
    ├── snap/
    ├── today/
    ├── big_days/
    ├── stylist_chat/
    └── profile/
        ├── presentation/   Screens, widgets, controllers
        ├── domain/         Models, use cases
        └── data/           Repositories
```

## Data Model

The data model is built around the `User` and comprises 14 entities:

`User` · `Profile` · `Item` · `ItemPhoto` · `ScanJob` · `Outfit` · `OutfitItem` · `BigDay` · `DayPlan` · `Reminder` · `WearLog` · `OutfitFeedback` · `ChatMessage` · `Subscription`

Core business rules:

- An outfit must contain a top and a bottom, or a dress, with optional shoes, bag and extras.
- A user has at most one `DayPlan` per date.
- Creating a `BigDay` automatically creates its four `Reminder` entries.
- `OutfitFeedback` (wear it, shuffle, like, dislike) feeds the outfit ranking model.

Each item stores a vector embedding (pgvector) used for compatibility search.

## Security and Privacy

- HTTPS for all communication
- JWT-based authentication
- Row-level security: users can only read and modify their own data
- Private photo storage with expiring signed links
- Minimal data collection, with explicit user consent
- Designed to align with Tunisian data-protection requirements (INPDP)

## Getting Started

### Prerequisites

- Flutter SDK
- Python 3.11+
- Supabase CLI
- Node.js (for edge functions)

### Installation

```bash
git clone https://github.com/<your-org>/libsa.git
cd libsa
```

**Mobile app**

```bash
cd app
flutter pub get
flutter run
```

**AI service**

```bash
cd ai-service
pip install -r requirements.txt
uvicorn app.main:app --reload
```

**Backend**

```bash
cd backend
supabase start
supabase db reset
```

Environment variables are documented in `docs/`. Copy `.env.example` to `.env` and fill in your own keys.

> Commands above are provisional and will be updated as the project scaffolding is finalised.

## Roadmap

- [ ] **MVP:** Today's Look, Snap & Go onboarding, Big Days calendar, text stylist
- [ ] **V2:** Voice stylist (Derja, French, English), event and culture packs, Signature Uniform mode
- [ ] **V3:** Cost per wear, resale integration, Libsa Plus subscription
- [ ] **Later:** Local brand suggestions, expansion across the Maghreb

## Contributing

Contributions are welcome. Please open an issue to discuss significant changes before submitting a pull request, and follow the folder and migration conventions described above.

## License

To be defined.

---

Built by **Jihene** and team.
