# Attendify

Smart attendance management system using QR codes, facial recognition, and liveness detection. Built with FastAPI, PostgreSQL (pgvector), React, and a Flask ML service.

## Structure

```
attendify/
├── backend/               FastAPI backend (auth, courses, sessions, attendance)
├── ml-service/             Flask microservice (face embedding, comparison, liveness)
├── frontend-admin/         Admin dashboard (React)
├── frontend-instructor/    Instructor dashboard (React)
└── frontend-student/       Student web app (React)
```

## Tech Stack

- **Backend:** FastAPI, SQLAlchemy, Alembic, PostgreSQL + pgvector, JWT auth
- **ML Service:** Flask, face_recognition (dlib), Python 3.11
- **Frontend:** React, Vite, Tailwind CSS

---

## Requirements

| Tool | Version | Check |
|------|---------|-------|
| Python | 3.11 | `python --version` or `py -3.11 --version` |
| Node.js | 18+ | `node --version` |
| PostgreSQL | 14+ | `psql --version` |
| ngrok (optional, for mobile testing) | any | `ngrok --version` |

---

## 1. Database Setup

Install PostgreSQL if you don't have it: https://www.postgresql.org/download/

Create the database and user:

```sql
psql -U postgres
CREATE USER attendify WITH PASSWORD 'admin123';
CREATE DATABASE attendify OWNER attendify;
GRANT ALL PRIVILEGES ON DATABASE attendify TO attendify;
\q
```

Enable the pgvector extension (used to store face embeddings):

```sql
psql -U attendify -d attendify
CREATE EXTENSION IF NOT EXISTS vector;
\q
```

---

## 2. Backend Setup

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

pip install -r requirements.txt
```

Create a `.env` file in `backend/` (copy from `.env.example` and adjust):

```env
DATABASE_URL=postgresql://attendify:admin123@localhost:5432/attendify
SECRET_KEY=attendify-secret-key-2024-min-32-chars-ok
ML_SERVICE_URL=http://localhost:5001
ML_SERVICE_ENABLED=True
FACE_SIMILARITY_THRESHOLD=0.35
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_FROM=
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_TLS=True
MAIL_SSL=False
```

> Leave `MAIL_*` empty if you don't need the forgot-password email feature.

Create tables and seed test data:

```bash
python -c "from app.db.session import Base, engine; from app.models import models; Base.metadata.create_all(bind=engine)"
python seed.py
```

Run the backend:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

API docs available at `http://localhost:8000/docs`.

---

## 3. ML Service Setup

Requires **Python 3.11** specifically (the `face_recognition` library depends on dlib, which is version-sensitive).

```bash
cd ml-service
py -3.11 -m venv venv311

# Windows
venv311\Scripts\activate
# macOS/Linux
source venv311/bin/activate

pip install -r requirements.txt
py -3.11 face_app.py
```

> If `face_recognition`/`dlib` fails to install, you may need Visual Studio Build Tools (Windows) or `cmake` + `build-essential` (Linux/Mac).

Runs on `http://localhost:5001`.

---

## 4. Frontend Setup (Instructor & Admin)

These run locally and connect directly to `http://localhost:8000`.

```bash
cd frontend-instructor
npm install
npm run dev
# → http://localhost:5173

cd frontend-admin
npm install
npm run dev
# → http://localhost:5175 (or next available port)
```

---

## 5. Student Frontend Setup

The student app is meant to be tested on a real phone (camera access for QR scanning + face verification), so it needs to reach your local backend over the network.

### Option A — Local network testing (same Wi-Fi)

```bash
cd frontend-student
npm install
```

Create a `.env` file in `frontend-student/`:

```env
VITE_API_BASE_URL=http://<YOUR_LOCAL_IP>:8000/api/v1
```

Find your local IP with `ipconfig` (Windows) or `ifconfig` (Mac/Linux). Then run:

```bash
npm run dev -- --host
```

Open the printed network URL on your phone (must be on the same Wi-Fi network).

### Option B — ngrok tunnel (works from anywhere)

```bash
ngrok http 8000
```

Copy the generated `https://xxxx.ngrok-free.app` URL, set it in `frontend-student/.env`:

```env
VITE_API_BASE_URL=https://xxxx.ngrok-free.app/api/v1
```

```bash
npm install
npm run dev
```

> Every time you restart ngrok, the URL changes — update `.env` accordingly.

### Option C — Deploy to Vercel

```bash
npm install -g vercel
cd frontend-student
vercel
```

Set `VITE_API_BASE_URL` as an environment variable in the Vercel dashboard, pointing to your ngrok URL (or a deployed backend URL).

---

## 6. Creating & Using a Student Account

1. Log in to the **admin panel** (`frontend-admin`, default `http://localhost:5175`) with:
   - Email: `admin@test.com`
   - Password: `admin123`
2. Go to **Students → New Student**, fill in name, email, student number, password.
3. Open the **student app** (`frontend-student`) on a phone or browser.
4. Log in with the email/password just created.
5. You'll be prompted to **register your face** (`Register Face`) — this is required before submitting attendance. Allow camera access and follow the on-screen steps.
6. Once enrolled, ask an instructor to start a live session, then scan the QR code shown to submit attendance (includes a liveness/blink check + face match).

---

## 7. Running Everything Together

Open 5 terminals:

```bash
# Terminal 1 — Backend
cd backend && uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 2 — ML Service
cd ml-service && py -3.11 face_app.py

# Terminal 3 — ngrok (only if testing student app on a phone)
ngrok http 8000

# Terminal 4 — Instructor frontend
cd frontend-instructor && npm run dev

# Terminal 5 — Admin frontend
cd frontend-admin && npm run dev
```

Student frontend runs separately (`npm run dev` in `frontend-student/`, or via Vercel).

---

## Test Credentials

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@test.com | admin123 |
| Instructor | hoca@test.com | hoca123 |

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `Could not connect to database` | Make sure PostgreSQL service is running |
| `ML service unavailable` | Confirm `face_app.py` is running on port 5001 |
| `Failed to fetch` on student app | Check `VITE_API_BASE_URL` matches your current ngrok/local IP |
| `face-recognition` won't install | Install Visual Studio Build Tools (Windows) or cmake (Mac/Linux) first |
| QR scan fails repeatedly | QR rotates every 15s — make sure you're scanning the current code |

## License

MIT
