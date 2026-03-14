# Tracklet --- Public Technical Overview

**Tracklet** (https://tracklet.me) is a minimal, table-first assignment
tracker designed for students who prefer spreadsheet-style organization
over kanban boards or productivity dashboards.

Think:

> **Google Sheets --- purpose-built for assignments.**

This repository provides a **technical overview of the system
architecture** behind Tracklet.

The application source code remains **private**, but this documentation
outlines:

-   System architecture
-   Database schema
-   API design
-   Infrastructure
-   Security architecture
-   Engineering decisions

The goal is **technical transparency without exposing proprietary
business logic**.

------------------------------------------------------------------------

# System Overview

Tracklet is a SaaS web application built for students to manage
assignments, deadlines, and coursework.

Key design principles:

-   Minimal friction
-   Spreadsheet-style workflows
-   Fast inline editing
-   Reliable deadline tracking
-   Subscription-based SaaS model

Unlike typical productivity apps, Tracklet prioritizes:

-   speed
-   clarity
-   data density

------------------------------------------------------------------------

# Tech Stack

  Layer             Technology
  ----------------- ---------------------------------
  Framework         Next.js 16 (App Router)
  Language          TypeScript (strict mode)
  Styling           Tailwind CSS v4
  UI Components     shadcn/ui
  Table Engine      TanStack Table v8
  Database ORM      Prisma v7
  Database          PostgreSQL 17 (Neon serverless)
  Authentication    NextAuth v5
  Payments          Stripe
  Email             Resend
  Storage           Vercel Blob
  Hosting           Vercel
  CI                GitHub Actions
  Package Manager   pnpm

------------------------------------------------------------------------

# High Level Architecture

```
                ┌─────────────────────┐
                │       Browser       │
                │    Next.js Client   │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │     Next.js App     │
                │ Server Actions + API│
                └──────────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
 ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
 │ PostgreSQL  │   │   Stripe    │   │   Resend    │
 │    (Neon)   │   │ Billing API │   │ Email API   │
 └─────────────┘   └─────────────┘   └─────────────┘
```

**External Integrations**

- Google OAuth  
- Vercel Blob Storage  
- Vercel Cron Jobs

------------------------------------------------------------------------

# Core Product Architecture

Tracklet is built around a **table-first interaction model**.

The assignment table is the primary interface where users:

-   create assignments
-   edit fields inline
-   filter and sort
-   track deadlines
-   manage course workloads

Core architectural patterns:

-   Optimistic UI updates
-   Minimal page revalidation
-   Server-side subscription enforcement
-   Relational database modeling

------------------------------------------------------------------------

# Database Schema

Tracklet uses **8 core database models**.

## Enums

PlanStatus: FREE, PRO

AssignmentStatus: SUBMITTED, IN_PROGRESS, NOT_STARTED

AssignmentType: ASSIGNMENT, LAB, QUIZ, TEST, EXAM, PROJECT, OTHER

------------------------------------------------------------------------

## User

Stores account information and subscription state.

Fields:

id\
name\
email\
password\
image\
planStatus\
stripeCustomerId\
stripeSubscriptionId\
timezone\
reminderEnabled\
reminderDays\
weeklySummaryEnabled\
calendarToken\
clearStreak\
onTimeStreak\
lastStreakCheckDate

------------------------------------------------------------------------

## Tracker

Represents a collection of assignments.

Fields:

id\
name\
order\
isArchived\
shareId\
isPublic\
userId

------------------------------------------------------------------------

## Course

Courses group assignments within a tracker.

Constraint:

UNIQUE(trackerId, name)

------------------------------------------------------------------------

## Assignment

Primary record representing academic work.

Fields:

id\
title\
status\
type\
dueDate\
dueTime\
todo\
checklist\
pointsEarned\
pointsPossible\
weight\
courseId\
trackerId\
isOnTimeStreakCounted

The checklist field stores JSON data.

------------------------------------------------------------------------

# Authentication

Tracklet supports two authentication strategies.

## OAuth

Google OAuth handled through NextAuth v5.

## Email + Password

Credential authentication includes:

-   bcrypt password hashing
-   OTP verification
-   password reset flow

Security protections:

-   login rate limiting
-   brute force protection
-   email enumeration prevention

------------------------------------------------------------------------

# API Design

## Authentication

/api/auth/\[...nextauth\]

Handles OAuth and credential sessions.

------------------------------------------------------------------------

## Assignment Updates

PATCH /api/assignments/\[id\]

Used for inline editing inside the assignment table.

Goal: avoid expensive page revalidation.

------------------------------------------------------------------------

## Stripe Webhooks

POST /api/webhooks/stripe

Handles:

-   checkout completion
-   subscription updates
-   payment failures

Webhook signatures are verified.

------------------------------------------------------------------------

## Calendar Export

GET /api/calendar/\[token\]

Returns an iCalendar (.ics) feed compliant with RFC 5545.

Features:

-   VALARM reminders
-   token-based authentication
-   regeneration capability

------------------------------------------------------------------------

# Background Jobs

## Daily Reminder Job

/api/cron/reminders

Runs daily at 08:00 UTC.

Responsibilities:

-   detect upcoming assignments
-   send reminder emails

------------------------------------------------------------------------

## Weekly Summary Job

/api/cron/weekly-summary

Runs every Monday.

Includes:

-   upcoming assignments
-   overdue work

Emails are timezone-aware.

------------------------------------------------------------------------

# Subscription System

Plans:

FREE --- max 2 trackers\
PRO --- \$3.99/month unlimited trackers, reminders, weekly summaries,
calendar sync

Enforcement occurs at two layers:

1.  UI layer (disabled controls)
2.  Server layer (hard enforcement)

On subscription downgrade:

-   existing trackers preserved
-   new tracker creation blocked

------------------------------------------------------------------------

# Security Architecture

HTTP Security Headers:

-   HSTS
-   X-Frame-Options: DENY
-   X-Content-Type-Options: nosniff

Password hashing:

bcrypt (cost factor 12)

Rate limiting:

10 attempts per 15 minutes

OTP protection:

max 5 attempts per token

Stripe webhook signatures verified.

------------------------------------------------------------------------

# Performance Design

Optimistic UI updates allow inline table edits without full page reload.

Hydration-safe localStorage logic uses:

-   useEffect synchronization
-   suppressHydrationWarning

Timezone handling:

Dates stored as UTC midnight.\
Times stored as strings to preserve timezone accuracy.

------------------------------------------------------------------------

# Infrastructure

Hosting: Vercel\
Database: Neon (PostgreSQL)\
Payments: Stripe\
Email: Resend\
Storage: Vercel Blob

------------------------------------------------------------------------

# CI Pipeline

GitHub Actions pipeline:

pnpm typecheck\
pnpm lint\
pnpm build

Build fails on any error.

------------------------------------------------------------------------

# Environment Variables

DATABASE_URL\
AUTH_SECRET\
AUTH_GOOGLE_ID\
AUTH_GOOGLE_SECRET\
STRIPE_SECRET_KEY\
STRIPE_PRICE_ID_MONTHLY\
STRIPE_PRICE_ID_ANNUAL\
STRIPE_WEBHOOK_SECRET\
NEXT_PUBLIC_APP_URL\
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY\
RESEND_API_KEY\
RESEND_FROM_EMAIL\
CRON_SECRET\
BLOB_READ_WRITE_TOKEN

------------------------------------------------------------------------

# Status

Tracklet is currently:

-   production deployed
-   feature complete
-   CI verified
-   actively maintained

------------------------------------------------------------------------

# Why the Source Code is Private

Tracklet is a commercial SaaS product.

This repository documents system architecture while keeping proprietary
implementation details private.

The goal is to balance:

-   technical transparency
-   engineering credibility
-   sustainable business development
