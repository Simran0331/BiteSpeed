# Bitespeed Identity Reconciliation Service

A backend web service that identifies and consolidates customer contact information across multiple purchases, even when different email addresses and phone numbers are used.

## Hosted Endpoint

> **Base URL:** `<YOUR_HOSTED_URL>`
>
> **POST** `/identify`

## Tech Stack

- **Runtime:** Node.js with TypeScript
- **Framework:** Express.js
- **ORM:** Prisma (v7)
- **Database:** PostgreSQL (hosted on Supabase)

## Getting Started

### Prerequisites

- Node.js >= 20.19.0
- npm

### Installation

```bash
# Clone the repository
git clone <repo-url>
cd projectpp

# Install dependencies
npm install

# Create a .env file with your database URL
# DATABASE_URL="postgresql://user:password@host:5432/dbname"

# Generate Prisma client and push schema to database
npx prisma generate
npx prisma db push
```

### Running the Server

```bash
# Start the server
npm start

# Or use development mode
npm run dev
```

The server starts on `http://localhost:3000` by default. Set the `PORT` environment variable to change this.

## API Endpoints

### `POST /identify`

Receives contact information and returns a consolidated identity.

**Request Body (JSON):**

```json
{
  "email": "example@domain.com",
  "phoneNumber": "123456"
}
```

At least one of `email` or `phoneNumber` must be provided. Both can be provided together.

**Response (200 OK):**

```json
{
  "contact": {
    "primaryContatctId": 1,
    "emails": ["lorraine@hillvalley.edu", "mcfly@hillvalley.edu"],
    "phoneNumbers": ["123456"],
    "secondaryContactIds": [23]
  }
}
```

- `primaryContatctId`: ID of the primary contact
- `emails`: All emails linked to this identity (primary contact's email first)
- `phoneNumbers`: All phone numbers linked to this identity (primary contact's phone first)
- `secondaryContactIds`: IDs of all secondary contacts

### `GET /contacts`

Returns all contacts stored in the database.

**Response (200 OK):**

```json
{
  "count": 2,
  "contacts": [
    {
      "id": 1,
      "phoneNumber": "123456",
      "email": "lorraine@hillvalley.edu",
      "linkedId": null,
      "linkPrecedence": "primary",
      "createdAt": "2026-03-02T08:20:00.267Z",
      "updatedAt": "2026-03-02T08:20:00.267Z",
      "deletedAt": null
    }
  ]
}
```

## How Identity Linking Works

1. **New customer:** If no existing contacts match the email or phone, a new primary contact is created.
2. **Existing customer, new info:** If a match is found by email or phone but the request contains new information (a new email or phone), a secondary contact is created and linked to the primary.
3. **Merging primaries:** If the request links two previously separate primary contacts (e.g., email matches one group and phone matches another), the newer primary is demoted to secondary and linked to the older primary.

## Database Schema

```
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
```

## Project Structure

```
├── prisma/
│   └── schema.prisma          # Database schema (PostgreSQL)
├── prisma.config.ts           # Prisma CLI configuration
├── src/
│   ├── index.ts               # Express app entry point
│   ├── lib/
│   │   └── prisma.ts          # Prisma client instance (PrismaPg adapter)
│   ├── routes/
│   │   └── identify.ts        # POST /identify route
│   └── services/
│       └── contact.service.ts # Core reconciliation logic
├── package.json
├── tsconfig.json
└── .env                       # DATABASE_URL (not committed)
```

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `PORT` | Server port | `3000` |
| `DATABASE_URL` | PostgreSQL connection string (Supabase) | -- |
