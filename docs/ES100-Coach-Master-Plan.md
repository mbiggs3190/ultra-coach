# ES100 Running Coach App — Master Build Plan

## Project Overview

A Progressive Web App (PWA) running coach built specifically to train for and win the Eastern States 100 (August 8, 2026). The app syncs automatically from Coros via Strava, generates an adaptive training plan calibrated to historical winner data, and provides a living predicted finish time that updates weekly based on actual training.

-----

## Athlete Profile

- **Goal**: Win Eastern States 100 — 2026
- **Target finish time**: Sub-20:30 (competitive winning range based on historical data)
- **Race date**: August 8, 2026
- **Location**: Pittsburgh, PA
- **Equipment**: Treadmill (12% max incline), dumbbells, 12lb weighted vest, Coros watch, high school track
- **Terrain**: Pittsburgh hills (punchy, 15-25% grade), riverfront flat trails, public staircases
- **Data pipeline**: Coros → Strava (auto-sync, already configured) → App via Strava API

-----

## Race Intelligence

### Eastern States 100 — Course Facts

- 103 miles, 98% single/double-track
- 20,000+ feet of elevation gain
- Allegheny Plateau, north-central Pennsylvania (Pine Creek watershed)
- Trails: Mid State Trail, Black Forest Trail
- August timing — heat and humidity are major factors
- Average finish rate: 49% (drops to 30% in hot years)
- 2024 was cancelled due to weather

### Historical Men’s Winners

|Year|Winner         |Time         |
|----|---------------|-------------|
|2025|Michael Busada |22:30:27     |
|2023|Reagan McCoy   |22:23:58     |
|2022|Ryan Clifford  |20:13:26     |
|2021|Ben Quatromoni |22:31:28     |
|2019|Wesley Atkinson|18:23:47 (CR)|
|2017|Jayson Kolb    |21:13:16     |
|2016|Devon Olson    |20:30:36     |
|2015|Michael Wardian|21:21:27     |
|2014|Ryan Welts     |21:29:59     |

**Winning target for 2026**: 20:30 anchor. Good-day ceiling needs to be sub-21 hours. No dominant repeat winners — the race is genuinely open to well-prepared regional competitors.

-----

## Training Philosophy

### Core Framework

- **Structure**: Jason Koops (Training Essentials for Ultrarunning) — periodized block training, base/build/peak/taper phases, ruthless protection of easy days
- **Adaptive voice**: David Roche (SWAP Athletics, Some Work All Play podcast) — high easy volume, strides as near-daily staple, downhill running as a specific skill, individualized effort approach
- **No reference to David Roche’s book**

### Key ES100-Specific Demands

1. Eccentric leg strength (downhill volume is non-negotiable)
1. Technical footing at pace and under fatigue
1. Heat and humidity tolerance (late July/early August)
1. Back-half durability — miles 60-80 on descents break most runners
1. Time on feet over distance — terrain matters more than flat mileage

### Effort System (Three-Tier)

- **Easy**: Fully conversational, could go much longer
- **Moderate-Hard**: Comfortably uncomfortable, purposeful
- **Hard**: Race-simulation effort, limited use

### Pittsburgh Terrain Usage

- **Flat riverfront** (North Shore, Montour Trail, Three Rivers Heritage Trail): Easy aerobic days, stay in correct effort zone
- **Neighborhood hills / bluffs** (15-25% grade): Strength work, uphill intervals — steeper than race grade, builds strength efficiently
- **Public staircases**: Uphill intervals with built-in recovery, low-impact on ascent
- **Downhill repeats**: The descent IS the workout, not recovery. Run intentionally at pace
- **High school track**: Strides, controlled tempos, heat exposure sessions
- **Treadmill (12% max)**: Uphill interval simulation, weighted vest walks — no downhill
- **Weighted vest (12lb)**: Incline walks and strength circuits only, not for outdoor runs
- **Dumbbells**: Single-leg RDL, Bulgarian split squats, step-ups, hip/ankle stability

-----

## Tech Stack

|Layer          |Technology                           |
|---------------|-------------------------------------|
|Framework      |Next.js 14 (App Router)              |
|Language       |TypeScript                           |
|Styling        |Tailwind CSS + shadcn/ui             |
|Database ORM   |Prisma                               |
|Database       |PostgreSQL (Supabase or Neon)        |
|Auth           |NextAuth.js (Strava OAuth provider)  |
|Data sync      |Strava API v3 + Webhooks             |
|Deployment     |Vercel                               |
|Version control|GitHub                               |
|PWA            |Next.js PWA manifest + service worker|

### Strava Integration Facts

- API is public, free, no approval required
- Register app at strava.com/settings/api
- OAuth 2.0 — authenticate once, tokens refresh automatically (expire every 6 hours)
- Webhooks fire on activity create/update/delete — automatic sync after every run
- Rate limits: 200 requests/15 min, 2,000/day — generous for personal use
- Dev mode: limited to 1 connected athlete (fine — this is a personal app)
- All data needed (distance, elevation gain/loss, moving time, HR, sport type, grade-adjusted pace) available on free Strava account
- Webhook requires publicly deployed URL — deploy to Vercel before registering

### Coros Notes

- Coros official API is private/gated — not used
- Coros → Strava auto-sync is already configured and working
- FIT file upload available as fallback if needed

-----

## Architecture Principles

### Feature-Based Folder Structure

Organize by feature, not by type. Each feature owns its own components, hooks, types, and utilities.

```
/app
  /api
    /strava
      /webhook        ← public endpoint, Strava fires here
      /auth           ← NextAuth Strava provider
  /(protected)        ← all authenticated routes
    /dashboard        ← Today screen
    /week             ← Week screen
    /plan             ← Full calendar
    /settings         ← Goal time, profile

/features
  /strava-sync        ← OAuth, webhook handling, historical import
  /training-plan      ← Plan generation, adaptation logic, phase management
  /predicted-finish   ← Prediction engine, confidence scoring, trend history
  /competitive-intel  ← Historical winner data, course profile, benchmarks
  /strength           ← Strength session library and prescriptions
  /heat-protocol      ← Heat acclimatization module

/lib
  /prisma             ← Database client
  /strava             ← Strava API client and rate limit handling
  /types              ← Shared TypeScript types
  /utils              ← Shared utilities

/components
  /ui                 ← Shared UI primitives (shadcn)
  /layout             ← Mobile shell, nav, headers

/prisma
  schema.prisma
  /migrations
  /seed
```

### Key Architecture Decisions

- **Server Actions over API routes** where possible — cleaner, less boilerplate
- **API routes only** for public endpoints (Strava webhook)
- **Types first** — define TypeScript types before writing components or queries
- **Feature branches** — one branch per phase, never build two phases on the same branch

### GitHub / Vercel Workflow

- `main` branch → auto-deploys to production on Vercel
- Feature branches: `feature/phase-0-foundation`, `feature/phase-1-auth`, etc.
- Each phase gets its own PR and merge to main when complete
- Vercel preview URLs auto-generated for every open PR — test on phone before merging

-----

## Mobile UI (Three Screens)

### Today Screen

- Prescribed workout for today
- Yesterday’s sync summary
- Coaching flags (easy day too hard, HRV trending down, weekly vert behind target)

### Week Screen

- Plan vs actual
- Vert tracker
- Effort distribution
- Workout swap interface (tap to swap sessions across days)

### Plan Screen

- Full calendar from today to race day (August 8, 2026)
- Phase indicators
- Predicted finish time widget

### Swap Logic

- Tap and hold → drag to new day, or simple swap button between two sessions
- Manual placements are **sticky** — algorithm never moves them silently
- Scheduled layer (what you’ve committed to) sits on top of prescribed layer (what algorithm suggests)
- `userModified` boolean flag + `swappedWith` reference in database

-----

## Key App Modules

### Training Plan (Living Plan)

- Two layers: prescribed (algorithm) + scheduled (your commitments)
- Full calendar view from today to race day
- Weekly targets: mileage, vert, downhill volume, effort distribution
- Adaptive logic recalculates forward weeks based on actual training data
- Rules: never auto-move sticky sessions, missed weeks redistribute forward (don’t try to make them up)

### Historical Import (Runs Once at Onboarding)

- Pulls 12 months of Strava activities in background after first auth
- Calculates baseline: avg weekly mileage/vert, downhill volume, long run history, effort distribution, consistency score
- Shows progress indicator during import
- Triggers plan generation when complete

### Predicted Finish Time Module

- Living number, recalculates every Sunday
- Three scenarios: good day / normal day / hard day
- Confidence score (low early, high by week 16)
- Goal gap indicator (current trajectory vs 20:30 target, closing or widening)
- Trend history — see the number moving over time
- Input signals: recent long run performance vs effort, weekly vert vs target, downhill volume trend, consistency score, easy day HR trend, weeks remaining

### Competitive Intelligence

- Historical winner data seeded at build time
- 2024 cancellation noted as weather-risk data point
- Winning time range by conditions (good/normal/hard day benchmarks)
- Default goal: 20:30. Athlete can adjust
- Required moving pace: ~12-12:30 min/mile over 103 miles at 20,000 feet gain
- Course profile: climb/descent character, technical sections, back-half difficulty

### Heat Protocol Module

- Auto-activates at 6 weeks before race
- Sauna protocol, hot weather run timing, hydration guidance
- Flags heat-relevant weeks on calendar

-----

## Phase Summary

|Phase|Name             |Description                                       |
|-----|-----------------|--------------------------------------------------|
|0    |Foundation       |Project scaffold, PWA setup, Vercel deployment    |
|1    |Auth             |Strava OAuth, token management, route protection  |
|2    |Schema           |All database models defined and migrated          |
|3    |Historical Import|12-month Strava backfill, baseline calculation    |
|4    |Webhook          |Live activity sync pipeline                       |
|5    |Training Plan    |Plan generation, adaptive logic, phase management |
|6    |Competitive Intel|Winner data, course profile, goal time benchmarks |
|7    |Predicted Finish |Prediction engine, scenarios, confidence, goal gap|
|8    |Mobile UI        |Three screens, swap interface, coaching flags     |
|9    |Strength         |Strength session library, vest/track prescriptions|
|10   |Heat Protocol    |6-week pre-race heat acclimatization module       |

-----

## Training Phase Structure (Race Date: August 8, 2026)

|Phase|Approximate Timing|Focus                                                                   |
|-----|------------------|------------------------------------------------------------------------|
|Base |Weeks 1-8         |Aerobic engine, easy volume, strides, strength 2x/week                  |
|Build|Weeks 9-16        |Uphill intervals, downhill-specific sessions, back-to-back long runs    |
|Peak |Weeks 17-20       |Race simulation, tune-up effort (50K or similar), conservative intensity|
|Taper|Weeks 21-24       |Volume drops, psychological scaffolding, heat protocol active           |

-----

*Last updated: February 2026*
*Race: Eastern States 100 — August 8, 2026*
*Goal: Win outright. Target sub-20:30.*
