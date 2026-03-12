# Axium

Temporary file transfer that deletes itself. Upload a file, get a shareable link, set an expiry — the file is gone when the timer runs out.

![file transfer auto delete](https://ibb.co/nMyvTQ6Y)

## Features

- Drag and drop file upload
- Files upload directly to S3 — they never pass through the app server
- Expiry options: 10 minutes, 1 hour, 2 hours
- Password protection
- One-time download mode
- Download count limits
- Automatic cleanup via cron endpoint

## Stack

- **Framework:** Next.js 15, React 19, TypeScript
- **Storage:** Amazon S3 (presigned uploads)
- **Database:** Vercel Postgres (file metadata)
- **Styling:** Tailwind CSS

## Getting Started

### Prerequisites

- Node.js 20+
- AWS account with an S3 bucket

### Setup

```bash
git clone https://github.com/4shil/Axium-TempFiles.git
cd Axium-TempFiles
npm install
cp .env.example .env.local
```

Edit `.env.local`:

```
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_S3_BUCKET=
POSTGRES_URL=
CRON_SECRET=
```

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### S3 Setup

1. Create a private S3 bucket.
2. Create an IAM user with `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject` on your bucket.
3. Generate access keys for the IAM user.
4. Set a CORS policy on the bucket to allow uploads from your frontend origin.

### File Cleanup

Set up a cron job (or Vercel Cron) to call `/api/cleanup` every 15 minutes:

```
Authorization: Bearer <CRON_SECRET>
```

## Project Structure

```
Axium-TempFiles/
├── app/
│   ├── api/
│   │   ├── upload/     # Presigned URL generation
│   │   ├── files/      # File metadata CRUD
│   │   └── cleanup/    # Expired file deletion
│   └── page.tsx        # Upload UI
├── components/         # UI components
└── lib/                # S3 client, DB helpers
```

## Architecture

Files upload directly from the browser to S3 using presigned URLs. The Next.js backend only handles authorization, metadata storage, and cleanup — it never touches the file bytes.

```
Browser  -->  S3 (file upload, presigned URL)
Browser  -->  Next.js API (metadata, expiry, links)
Cron     -->  Next.js API /cleanup  -->  S3 delete + DB purge
```

## Environment Variables

| Variable               | Description                          |
|------------------------|--------------------------------------|
| `AWS_REGION`           | S3 bucket region                     |
| `AWS_ACCESS_KEY_ID`    | IAM access key                       |
| `AWS_SECRET_ACCESS_KEY`| IAM secret key                       |
| `AWS_S3_BUCKET`        | Bucket name                          |
| `POSTGRES_URL`         | Vercel Postgres connection string    |
| `CRON_SECRET`          | Bearer token for cleanup endpoint    |
| `MAX_FILE_SIZE`        | Max upload in bytes (default 500MB)  |

## License

MIT
