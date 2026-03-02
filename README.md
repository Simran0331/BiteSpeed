Bitespeed Identity Reconciliation Service

A backend API that detects and unifies customer identities across multiple checkouts—even when shoppers use different emails and/or phone numbers over time.

Hosted Endpoint

Base URL: https://bitespeed-tngy.onrender.com

POST URL: https://bitespeed-tngy.onrender.com/identify

GET URL: https://bitespeed-tngy.onrender.com/contacts

Tech Stack

Runtime: Node.js + TypeScript

Framework: Express.js

ORM: Prisma (v7)

Database: PostgreSQL (Supabase)

Getting Started
Prerequisites

Node.js >= 20.19.0

npm

Installation
# Clone the repository
git clone <repo-url>
cd projectpp

# Install dependencies
npm install

# Create a .env file and set your database URL
# DATABASE_URL="postgresql://user:password@host:5432/dbname"
# If Supabase on 5432 fails, try the pooler port 6543:
# DATABASE_URL="postgresql://postgres.<project-ref>:[password]@aws-1-ap-south-1.pooler.supabase.com:6543/postgres"

# Generate Prisma client and sync schema to the DB
npx prisma generate
npx prisma db push
Running the Server
# Start (production)
npm start

# Start (development)
npm run dev

By default the server runs on http://localhost:5000. You can override this using the PORT environment variable.

API Endpoints
POST /identify

Accepts contact details and returns the merged identity for that customer.

Request Body (JSON):

{
  "email": "alex.chen@acme.io",
  "phoneNumber": "9988776655"
}

Provide at least one of email or phoneNumber.

You may send both together.

Response (200 OK):

{
  "contact": {
    "primaryContactId": 101,
    "emails": ["alex.chen@acme.io", "a.chen@workmail.com"],
    "phoneNumbers": ["9988776655", "8877665544"],
    "secondaryContactIds": [202, 203]
  }
}

primaryContactId: the chosen primary contact ID

emails: all associated emails (primary email first)

phoneNumbers: all associated phone numbers (primary phone first)

secondaryContactIds: all linked secondary contact IDs

GET /contacts

Fetches every contact row currently stored.

Response (200 OK):

{
  "count": 2,
  "contacts": [
    {
      "id": 101,
      "phoneNumber": "9988776655",
      "email": "alex.chen@acme.io",
      "linkedId": null,
      "linkPrecedence": "primary",
      "createdAt": "2026-03-02T08:20:00.267Z",
      "updatedAt": "2026-03-02T08:20:00.267Z",
      "deletedAt": null
    },
    {
      "id": 202,
      "phoneNumber": "8877665544",
      "email": "a.chen@workmail.com",
      "linkedId": 101,
      "linkPrecedence": "secondary",
      "createdAt": "2026-03-02T09:10:12.120Z",
      "updatedAt": "2026-03-02T09:10:12.120Z",
      "deletedAt": null
    }
  ]
}
How Identity Linking Works

Brand-new customer
If no record matches the provided email/phone, the service creates a new primary contact.

Match found + new detail arrives
If an existing contact matches by email or phone, but the request introduces a new email/phone, the service adds a secondary contact linked to the primary.

Two identity groups become connected
If a single request ties together two previously separate primaries (e.g., email matches one group and phone matches another), the newer primary is converted into a secondary and linked under the older primary.

Database Schema
Contact {
  id              Int         Primary key, auto-increment
  phoneNumber     String?     Optional phone number
  email           String?     Optional email
  linkedId        Int?        ID of the linked primary contact
  linkPrecedence  String      "primary" or "secondary"
  createdAt       DateTime    Auto-set on creation
  updatedAt       DateTime    Auto-updated on modification
  deletedAt       DateTime?   Soft delete timestamp
}
Project Structure
├── prisma/
│   └── schema.prisma          # PostgreSQL schema for Prisma
├── prisma.config.ts           # Prisma CLI configuration
├── src/
│   ├── index.ts               # Express app bootstrap
│   ├── lib/
│   │   └── prisma.ts          # Prisma client setup (PrismaPg adapter)
│   ├── routes/
│   │   └── identify.ts        # POST /identify route
│   └── services/
│       └── contact.service.ts # Identity reconciliation logic
├── package.json
├── tsconfig.json
└── .env                       # DATABASE_URL (not committed)
Environment Variables
Variable	Description	Default
PORT	Port the server listens on	5000
DATABASE_URL	PostgreSQL connection string (Supabase)	--
GIVE ME README.MD CODE
# Bitespeed Identity Reconciliation Service

A backend API that detects and unifies customer identities across multiple checkouts—even when shoppers use different email addresses and/or phone numbers over time.

## Hosted Endpoint

- **Base URL:** `https://bitespeed-identity-wqp5.onrender.com`
- **POST** `https://bitespeed-identity-wqp5.onrender.com/identify`

## Tech Stack

- **Runtime:** Node.js + TypeScript
- **Framework:** Express.js
- **ORM:** Prisma (v7)
- **Database:** PostgreSQL (Supabase)

## Getting Started

### Prerequisites

- Node.js **>= 20.19.0**
- npm

### Installation

```bash
# Clone the repository
git clone <repo-url>
cd projectpp

# Install dependencies
npm install

# Create a .env file and set your database URL
# Example:
# DATABASE_URL="postgresql://user:password@host:5432/dbname"
#
# If Supabase on 5432 fails, try the pooler port 6543:
# DATABASE_URL="postgresql://postgres.<project-ref>:[password]@aws-1-ap-south-1.pooler.supabase.com:6543/postgres"

# Generate Prisma client and sync schema to the DB
npx prisma generate
npx prisma db push
Running the Server
# Start (production)
npm start

# Start (development)
npm run dev

The server runs on http://localhost:5000 by default. Set the PORT environment variable to change this.

API Endpoints
POST /identify

Accepts contact details and returns the merged identity for that customer.

Request Body (JSON):

{
  "email": "alex.chen@acme.io",
  "phoneNumber": "9988776655"
}

Provide at least one of email or phoneNumber.

You may send both together.

Response (200 OK):

{
  "contact": {
    "primaryContactId": 101,
    "emails": ["alex.chen@acme.io", "a.chen@workmail.com"],
    "phoneNumbers": ["9988776655", "8877665544"],
    "secondaryContactIds": [202, 203]
  }
}

primaryContactId: the chosen primary contact ID

emails: all associated emails (primary email first)

phoneNumbers: all associated phone numbers (primary phone first)

secondaryContactIds: all linked secondary contact IDs

GET /contacts

Fetches every contact row currently stored.

Response (200 OK):

{
  "count": 2,
  "contacts": [
    {
      "id": 101,
      "phoneNumber": "9988776655",
      "email": "alex.chen@acme.io",
      "linkedId": null,
      "linkPrecedence": "primary",
      "createdAt": "2026-03-02T08:20:00.267Z",
      "updatedAt": "2026-03-02T08:20:00.267Z",
      "deletedAt": null
    }
  ]
}
How Identity Linking Works

Brand-new customer

If no record matches the provided email/phone, the service creates a new primary contact.

Match found + new detail arrives

If an existing contact matches by email or phone, but the request introduces a new email/phone, the service adds a secondary contact linked to the primary.

Two identity groups become connected

If a single request ties together two previously separate primaries (e.g., email matches one group and phone matches another), the newer primary is converted into a secondary and linked under the older primary.

Database Schema
Contact {
  id              Int         Primary key, auto-increment
  phoneNumber     String?     Optional phone number
  email           String?     Optional email
  linkedId        Int?        ID of the linked primary contact
  linkPrecedence  String      "primary" or "secondary"
  createdAt       DateTime    Auto-set on creation
  updatedAt       DateTime    Auto-updated on modification
  deletedAt       DateTime?   Soft delete timestamp
}
Project Structure
├── prisma/
│   └── schema.prisma          # PostgreSQL schema for Prisma
├── prisma.config.ts           # Prisma CLI configuration
├── src/
│   ├── index.ts               # Express app bootstrap
│   ├── lib/
│   │   └── prisma.ts          # Prisma client setup (PrismaPg adapter)
│   ├── routes/
│   │   └── identify.ts        # POST /identify route
│   └── services/
│       └── contact.service.ts # Identity reconciliation logic
├── package.json
├── tsconfig.json
└── .env                       # DATABASE_URL (not committed)
Environment Variables
Variable	Description	Default
PORT	Port the server listens on	5000
DATABASE_URL	PostgreSQL connection string (Supabase)	--
