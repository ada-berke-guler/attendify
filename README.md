# Attendify

Smart attendance management system using QR codes, facial recognition, and liveness detection. Built with FastAPI, PostgreSQL (pgvector), React, and a Flask ML service.

## Structure

```
attendify/
├── backend/               FastAPI backend (auth, courses, sessions, attendance)
├── ml-service/             Flask microservice (face embedding, comparison, liveness)
├── frontend-admin/         Admin dashboard (React)
├── frontend-instructor/    Instructor dashboard (React)
└── frontend-student/       Student web app (React, deployed on Vercel)
```

## Tech Stack

- **Backend:** FastAPI, SQLAlchemy, Alembic, PostgreSQL + pgvector, JWT auth
- **ML Service:** Flask, face_recognition (dlib), Python 3.11
- **Frontend:** React, Vite, Tailwind CSS

## Setup

See setup instructions in each subfolder's README, or refer to `SETUP.md` for a full local environment guide.

### Quick start

```bash
# Backend
cd backend
python -m venv venv && venv\Scripts\activate
pip install -r requirements.txt
python seed.py
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# ML Service (requires Python 3.11)
cd ml-service
py -3.11 -m venv venv311 && venv311\Scripts\activate
pip install -r requirements.txt
py -3.11 face_app.py

# Frontends
cd frontend-instructor && npm install && npm run dev
cd frontend-admin && npm install && npm run dev
cd frontend-student && npm install && npm run dev
```

## Test Credentials

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@test.com | admin123 |
| Instructor | hoca@test.com | hoca123 |

## License

MIT
