
# Bitespeed Identity Reconciliation Service

A lightweight backend API that **recognizes, links, and consolidates customer identities** across multiple checkouts—useful when the same shopper shows up over time with different **emails** and/or **phone numbers**.

---

## Live Deployment

**Base URL:** `https://bitespeed-tngy.onrender.com`

- **POST** `/identify` → `https://bitespeed-tngy.onrender.com/identify`
- **GET** `/contacts` → `https://bitespeed-tngy.onrender.com/contacts`

> If your deployment URL is different, replace the Base URL accordingly.

---

## Built With

- **Node.js** + **TypeScript**
- **Express.js**
- **Prisma ORM (v7)**
- **PostgreSQL** (Supabase)

---

## Quick Start

### Requirements

- Node.js **>= 20.19.0**
- npm

### Install & Configure

```bash
# Clone the repository
git clone <repo-url>
cd projectpp

# Install packages
npm install

# Create your environment file
# .env
# DATABASE_URL="postgresql://user:password@host:5432/dbname"
#
# If Supabase port 5432 is blocked, try the pooler port 6543:
# DATABASE_URL="postgresql://postgres.<project-ref>:<password>@aws-1-<region>.pooler.supabase.com:6543/postgres"

# Generate Prisma client
npx prisma generate

# Push schema to database
npx prisma db push
````

### Run the Server

```bash
# Production
npm start

# Development (watch mode)
npm run dev
```

Default server address: `http://localhost:5000`
To change the port, set `PORT` in your environment.

---

## Environment Variables

| Variable       | Purpose                                 | Default |
| -------------- | --------------------------------------- | ------- |
| `PORT`         | Port where the API listens              | `5000`  |
| `DATABASE_URL` | PostgreSQL connection string (Supabase) | —       |

---

## API Reference

### POST `/identify`

Sends contact info and returns the **merged identity view** for that shopper.

#### Request body

Provide **at least one** of `email` or `phoneNumber` (you can send both):

```json
{
  "email": "alex.chen@acme.io",
  "phoneNumber": "9988776655"
}
```

#### Response (200 OK)

```json
{
  "contact": {
    "primaryContactId": 101,
    "emails": ["alex.chen@acme.io", "a.chen@workmail.com"],
    "phoneNumbers": ["9988776655", "8877665544"],
    "secondaryContactIds": [202, 203]
  }
}
```

**Field meaning**

* `primaryContactId`: ID of the chosen primary identity
* `emails`: all known emails for the identity (primary email first)
* `phoneNumbers`: all known phone numbers (primary phone first)
* `secondaryContactIds`: IDs of linked secondary rows

---

### GET `/contacts`

Returns **all contact rows** currently stored in the database.

#### Response (200 OK)

```json
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
```

---

## How Identity Resolution Works

The service follows a simple, deterministic linking strategy:

1. **Completely new shopper**

   * If neither email nor phone matches anything in the DB, a **new primary** contact is created.

2. **Match exists, but request includes new information**

   * If a record matches by email or phone, and the incoming request introduces a **new email/phone**, a **secondary** contact is created and linked under the primary identity.

3. **A request connects two previously separate identities**

   * If one request matches two different identity clusters (e.g., email ties to one primary, phone ties to another), the service merges them:

     * the **older primary** remains primary
     * the **newer primary** is downgraded to **secondary** and linked to the older primary

---

## Data Model

### `Contact` table

```txt
Contact
- id             Int       (PK, auto-increment)
- phoneNumber    String?   (nullable)
- email          String?   (nullable)
- linkedId       Int?      (nullable, points to primary contact)
- linkPrecedence String    ("primary" | "secondary")
- createdAt      DateTime  (auto-set)
- updatedAt      DateTime  (auto-updated)
- deletedAt      DateTime? (soft delete timestamp)
```

---

## Project Layout

```txt
.
├── prisma/
│   └── schema.prisma           # Prisma schema
├── prisma.config.ts            # Prisma CLI config
├── src/
│   ├── index.ts                # Express bootstrap
│   ├── lib/
│   │   └── prisma.ts           # Prisma client setup (PrismaPg adapter)
│   ├── routes/
│   │   └── identify.ts         # POST /identify route
│   └── services/
│       └── contact.service.ts  # Identity reconciliation logic
├── package.json
├── tsconfig.json
└── .env                        # DATABASE_URL (not committed)
```

---

## Notes

* `deletedAt` enables soft deletes (rows remain in DB but can be treated as inactive).
* Prisma schema is pushed via `npx prisma db push` (no migrations required for quick setup).
