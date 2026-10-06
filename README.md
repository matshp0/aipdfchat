# aipdfchat

Chat with your PDFs. Upload a PDF, it gets parsed and indexed, then ask questions about it.

## How it works

1. Client requests a presigned URL from the backend and uploads the PDF straight to S3.
2. Lambdas process the file: `ParsePdfToText` extracts text, `IndexDocument` chunks it and stores embeddings in Pinecone, `UpdateStatusSuccess` / `UpdateStatusError` set the status in DynamoDB.
3. Client polls the document status, then sends questions. Backend retrieves relevant chunks from Pinecone and asks OpenAI.

## Stack

- **client** – React, Vite, Tailwind, TanStack Query
- **backend** – NestJS
- **lambda** – AWS Lambda (TypeScript, esbuild)
- **infra** – S3, DynamoDB, Pinecone, OpenAI

## API

| Method | Path | Description |
| --- | --- | --- |
| POST | `/documents/upload-url` | Get presigned S3 upload URL |
| GET | `/documents/:id` | Get document status |
| POST | `/ai/documents/:id` | Ask a question about a document |

## Setup

Backend `.env`:

```
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
AWS_DOCUMENT_BUCKET=
AWS_DYNAMODB_DOCUMENT_TABLE=
PINECONE_API_KEY=
PINECONE_INDEX_NAME=
OPENAI_API_KEY=
OPENAI_MODEL=
PORT=3000
```

Run backend:

```
cd backend && npm install && npm run start:dev
# or: docker compose up --build
```

Run client:

```
cd client && npm install && npm run dev
```

Lambdas: `cd lambda/<name> && npm install && node build.mjs`, then deploy the bundle. They need the same AWS / Pinecone env vars (`AWS_DOCUMENT_TABLE` for the table name).
