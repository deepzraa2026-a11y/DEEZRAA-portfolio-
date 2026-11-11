# deezraa — Curated Scene Packs

Premium, image-only scene pack catalog with a secure admin dashboard.

## Stack

- **Frontend:** React 19, React Router 7, Vite 8, Tailwind CSS 4
- **Backend:** Express 4, better-sqlite3, multer (JPG/JPEG/PNG/WEBP uploads)
- **Auth:** bcrypt password hashing + HMAC-signed session tokens stored in SQLite

## Local setup

**Prerequisites:** Node.js 22.x and npm.

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create your environment file and set secure values:

   ```bash
   cp .env.example .env
   ```

   | Variable | Required | Purpose |
   | --- | --- | --- |
   | `ADMIN_USERNAME` | Yes | Admin dashboard username |
   | `ADMIN_PASSWORD` | Yes | Admin dashboard password |
   | `SESSION_SECRET` | Yes | Long random secret used to sign admin sessions |
   | `PORT` | No | Local server port (default `3000`) |
   | `DATA_DIR` | No | SQLite storage directory (default `./data`) |
   | `UPLOADS_DIR` | No | Uploaded image directory (default `./uploads`) |
   | `BLOB_STORE_ID` | Vercel only | Private Vercel Blob store for the durable SQLite snapshot |
   | `DEEZRAA_IMAGES_STORE_ID` | Vercel only | Public Vercel Blob store for uploaded scene images |

   Admin credentials and secrets are never hardcoded in the application source.

3. Start the development server:

   ```bash
   npm run dev
   ```

   Open http://localhost:3000 — the Scene Packs landing page is the default homepage.

## Production build

```bash
npm run build
```

This generates the static client bundle in `dist/`. Run the production server with:

```bash
# Windows PowerShell
$env:NODE_ENV = "production"; npm start

# macOS / Linux
NODE_ENV=production npm start
```

The Express server serves the `dist/` assets, SPA routes, `/api/*`, and `/uploads/*` on port `3000` (or `PORT`).

## Routes

| Route | Description |
| --- | --- |
| `/` | Scene Packs landing page (default homepage) |
| `/scene-packs` | Landing page alias |
| `/scene-packs/all` | Full catalog with search, category and sort filters |
| `/scene-packs/:slug` | Individual scene pack gallery |
| `/admin/login` | Admin login |
| `/admin` | Admin dashboard (protected) |

## API overview

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/packs` | Public packs (`category`, `q`, `sort`, `featured`) |
| `GET` | `/api/packs/:slug` | Single pack |
| `POST` | `/api/packs/:slug/download` | Register a download and return the Google Drive URL |
| `POST` | `/api/admin/login` | Admin login |
| `POST` | `/api/admin/logout` | Admin logout |
| `GET/POST/PUT/DELETE` | `/api/admin/packs` | Pack management |
| `POST` | `/api/upload` | Upload one image (admin only) |
| `POST` | `/api/upload-multiple` | Upload up to 20 images (admin only) |
| `DELETE` | `/api/upload/:filename` | Delete an uploaded image (admin only) |
| `DELETE` | `/api/upload?url=` | Delete an uploaded image by URL/path (admin only) |

## Deploying to Vercel

`vercel.json` configures the Vite build, SPA rewrites, and the serverless API entry point at `api/index.ts`. The current SQLite database and `uploads/` folder are bundled with the function via `includeFiles` and copied into writable `/tmp` storage on the first cold start, so the deployed site shows your real catalog instead of seeded demo content.

1. Import the repository into Vercel.
2. Add the required environment variables (`ADMIN_USERNAME`, `ADMIN_PASSWORD`, `SESSION_SECRET`) in **Project → Settings → Environment Variables**.
3. Deploy. The frontend is served as static assets from `dist/`, while `/api/*` and `/uploads/*` are handled by the serverless function.

**Persistent storage on Vercel:** Two Vercel Blob stores keep admin changes durable across cold starts and redeploys:

- `BLOB_STORE_ID` points to a **private** store. The SQLite database is checkpointed and uploaded there after every pack/category/settings mutation, and restored before the database opens on each cold start.
- `DEEZRAA_IMAGES_STORE_ID` points to a **public** store. New uploads are written there and served from `*.public.blob.vercel-storage.com` URLs.

Both variables are added automatically when the stores are connected in **Project → Storage**. Without them the deployment falls back to the bundled catalog snapshot, and admin changes only last for the life of the function instance. Vercel Functions limit request bodies to roughly 4.5 MB, so the admin UI compresses images and uploads them one file per request.

## Notes

- Public users can download full packs through Google Drive links managed in the admin dashboard.
- All scene packs are image-only; video players and video uploads were removed.
- Uploaded images are stored on disk locally and in a public Vercel Blob store in production.
- The bundled `data/app.db` is a sanitized catalog snapshot that preserves the current packs and uploaded covers, but contains no admin users or session tokens. The admin account is created from `ADMIN_USERNAME` and `ADMIN_PASSWORD` on first startup.
