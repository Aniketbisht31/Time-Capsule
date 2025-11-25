

---

# Time Capsule

A secure, encrypted, time-locked digital vault for memories, files, and messages.

Time Capsule allows users to lock files, images, videos, and written messages for a chosen period. When the unlock time arrives, the system automatically decrypts the contents and delivers them to all selected recipients via email.

This project is built entirely from scratch, with a focus on real-world backend engineering concepts including encryption, scheduling, workers, storage, and email automation.

---

## Features

### End-to-End Encryption

Content is encrypted on the client using AES-GCM before upload. The server never sees plaintext until the scheduled unlock event.

### Secure Storage (S3-Compatible)

Encrypted files are uploaded using presigned URLs. Supports large files, multipart uploads, and strict access permissions.

### Time-Locked Capsules

Each capsule contains:

* Uploads (files, images, videos, documents)
* A message
* Unlock timestamp
* A list of recipients

### Distributed Scheduler

A background worker system periodically checks for capsules whose unlock time has arrived. It performs:

* Key unwrapping
* File decryption
* Email generation and sending
* Audit logging

### Automated Email Delivery

When a capsule unlocks:

* Recipients receive an email containing attachments or secure download links
* Sender receives a status confirmation
* Delivery status is stored and tracked

### User Dashboard

Users can:

* Create and manage capsules
* Upload encrypted memories
* Add or remove recipients
* View locked, pending, or opened capsules
* Track email delivery history

---

## System Architecture

```
Frontend (React/Next.js)
  - Client-side AES encryption
  - Presigned upload requests
  - Capsule creation interface

Backend (Node.js)
  - REST or RPC API
  - Key wrapping and metadata storage
  - Authentication layer

PostgreSQL
  - Capsules metadata
  - Recipients
  - Encryption keys
  - File metadata

S3 Storage
  - Encrypted binary files
  - Metadata objects

Redis + BullMQ Worker
  - Scheduled unlock jobs
  - Decrypt and send workflow
  - Retry and backoff logic

Email Provider (SMTP / SES / SendGrid)
  - Sends final capsule emails
  - Attachments or signed links
```

---

## Database Schema (Core Tables)

### capsules

| Column      | Type      | Description                        |
| ----------- | --------- | ---------------------------------- |
| id          | UUID      | Primary key                        |
| user_id     | UUID      | Owner ID                           |
| unlock_at   | timestamp | Scheduled unlock time              |
| wrapped_key | bytea     | AES key encrypted using server key |
| status      | text      | locked, opening, opened, failed    |
| created_at  | timestamp | Creation time                      |

### capsule_contents

| Column   | Type   | Description            |
| -------- | ------ | ---------------------- |
| s3_key   | text   | Path to encrypted file |
| iv       | bytea  | AES IV for the file    |
| filename | text   | Original file name     |
| size     | bigint | File size              |

### recipients

| Column  | Type      | Description           |
| ------- | --------- | --------------------- |
| email   | text      | Recipient address     |
| status  | text      | pending, sent, failed |
| sent_at | timestamp | Delivery timestamp    |

---

## Encryption Workflow

1. The browser generates an AES-GCM key using WebCrypto.
2. Each file is encrypted with a unique initialization vector.
3. Encrypted blobs are uploaded to S3 via presigned URLs.
4. The AES key is sent to the backend and wrapped using a server-side key (or KMS).
5. A worker unwraps the key at unlock time.
6. Files are downloaded, decrypted, and included in the outgoing email.

---

## S3 Storage Structure

```
/capsules/{capsule_id}/
    encrypted/
        file1.enc
        file2.enc
    metadata.json
```

Optional decrypted versions exist only temporarily for email processing, then removed.

---

## Scheduler Workflow

The worker performs:

1. Select all capsules where `unlock_at <= now AND status = "locked"`.
2. Mark capsule as `opening`.
3. Decrypt the wrapped AES key.
4. Fetch encrypted files from S3.
5. Decrypt files and prepare email content.
6. Send email to all recipients.
7. Mark capsule as `opened`.
8. Log delivery status for each recipient.

If any step fails, the job is retried with exponential backoff.

---

## Email Delivery

Emails include:

* Capsule title and message
* File attachments (if small enough)
* Signed download links for large files
* Delivery logs visible in the dashboard

Supports:

* SMTP
* AWS SES
* SendGrid
* Resend

---

## Technology Stack

Frontend:

* React or Next.js
* WebCrypto API
* TailwindCSS

Backend:

* Node.js (Express or Fastify)
* PostgreSQL
* Prisma or Drizzle ORM

Storage:

* AWS S3 or MinIO

Workers:

* Redis
* BullMQ
* Optional AWS KMS for key management

---

## Running Locally

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/time-capsule
cd time-capsule
```

### 2. Install dependencies

```bash
npm install
```

### 3. Start PostgreSQL, Redis, and MinIO

```bash
docker-compose up -d
```

### 4. Run local dev server

```bash
npm run dev
```

### 5. Start the worker

```bash
npm run worker
```

---

## Testing

Includes tests for:

* Encryption correctness
* S3 upload and retrieval
* Worker process
* Email sending
* Database interactions

---

## Contributing

Contributions are welcome.
Please open an issue to discuss significant changes before submitting a pull request.

---

## License

MIT License

---

