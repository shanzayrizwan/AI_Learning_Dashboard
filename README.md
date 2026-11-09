# AI Student Learning Dashboard

An AI-powered study assistant for students. It explains topics, helps with assignments (text or image), generates quizzes, builds course roadmaps, turns raw lecture notes into revision notes, and helps with exam preparation.

The project has two parts:

- **Backend:** FastAPI (Python) server that calls the Google Gemini API.
- **Frontend:** React + Vite single-page app with a white, purple, and black theme.

---

## Features

| Tool | What it does |
|------|--------------|
| **Dashboard** | Home screen with quick links to every tool |
| **Topic Explainer** | Clear, example-based explanation of any concept |
| **Assignment Helper** | Step-by-step help for a typed question or an uploaded question image |
| **Quiz Generator** | Multiple-choice quiz (3, 5 or 10 questions) with instant scoring and explanations |
| **Course Roadmap** | Module-by-module learning path for a subject, for a chosen target level |
| **Smart Notes** | Converts raw lecture text into notes (bullet points, summary, flashcards, etc.) |
| **Exam Prep** | Finds high-frequency topics from past papers, or solves a question image |

Accounts: sign up and sign in with name, email, and password. Emails with an `.edu` domain are saved as student accounts.

---

## Project Structure

```
studyAI/
├── backend/
│   ├── main.py            # FastAPI app: all API routes and Gemini calls
│   ├── requirements.txt   # Python dependencies (pinned)
│   ├── .env               # Your secrets (NOT committed, see setup)
│   └── .gitignore
├── frontend/
│   ├── index.html
│   ├── package.json
│   ├── public/            # favicon and static assets
│   └── src/
│       ├── main.jsx       # React entry point
│       ├── App.jsx        # Main layout, login state, tab routing, AI handlers
│       ├── App.css        # Component styles (theme)
│       ├── index.css      # Global theme variables and base styles
│       ├── services/
│       │   └── api.js     # All fetch calls to the backend
│       └── components/
│           ├── Auth.jsx             # Sign in / sign up
│           ├── Header.jsx           # Top bar with user info and logout
│           ├── TabNavigation.jsx    # Tool tabs
│           ├── Dashboard.jsx        # Home screen
│           ├── TopicExplainer.jsx
│           ├── AssignmentHelper.jsx
│           ├── QuizGenerator.jsx
│           ├── CourseRoadmap.jsx
│           ├── SmartNotes.jsx
│           ├── ExamPrep.jsx
│           ├── MarkdownView.jsx     # Renders AI output as formatted Markdown
│           └── Login.jsx            # Not used (Auth.jsx is used instead)
└── CHANGES.md             # List of fixes and changes
```

---

## Requirements

- Python 3.10 or newer
- Node.js 18 or newer, with npm
- A Google Gemini API key from [Google AI Studio](https://aistudio.google.com/)

---

## Setup

### 1. Backend

```bash
cd backend

# Create and activate a virtual environment
python -m venv venv
# Windows (PowerShell):
.\venv\Scripts\activate
# macOS / Linux:
source venv/bin/activate

pip install -r requirements.txt
```

Create a file named `.env` inside `backend/`:

```env
GEMINI_API_KEY=your_api_key_here
# Optional: change the model if your key does not have access to the default
GEMINI_MODEL=gemini-3.6-flash
```

Start the server:

```bash
uvicorn main:app --reload
```

The API runs at `http://127.0.0.1:8000`.
- Health check: `http://127.0.0.1:8000/api/test`
- Interactive API docs: `http://127.0.0.1:8000/docs`

### 2. Frontend

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Open the address shown in the terminal, usually `http://localhost:5173`.

To point the frontend at a different backend, create `frontend/.env.local`:

```env
VITE_API_URL=http://127.0.0.1:8000
```

---

## API Reference

All routes are under `/api`. Request and response bodies are JSON unless noted.

### Accounts

| Method | Route | Body | Returns |
|--------|-------|------|---------|
| POST | `/api/signup` | `name`, `email`, `password`, `account_type` | `user` (`name`, `email`, `account_type`) |
| POST | `/api/login` | `email`, `password` | `user` (`name`, `email`, `account_type`) |

Emails with an `.edu` domain are always saved as `student`, whatever `account_type` is sent.

### Tools used by the UI

| Method | Route | Body | Returns |
|--------|-------|------|---------|
| POST | `/api/explain-topic` | `topic` | `explanation` |
| POST | `/api/assignment-help` | `assignment_detail` (text only) | `solution` |
| POST | `/api/analyze-image` | `image_base64`, `mime_type`, `prompt_text` | `result` |
| POST | `/api/generate-quiz` | `topic`, `grade`, `num_questions` | `quiz` (questions with `answer` and `explanation`) |
| POST | `/api/course-roadmap` | `course_name`, `target_level` | `roadmap` (Markdown text) |
| POST | `/api/voice-notes` | `voice_transcript`, `format` | `formatted_notes` |
| POST | `/api/analyze-past-paper` | `subject_name`, `past_paper_text` | `analysis` |

The Assignment Helper sends text to `/api/assignment-help`. When the user uploads an image, it goes to `/api/analyze-image` instead. Exam Prep uses the same image route for question images.

### Backend only (not yet in the UI)

| Method | Route | Body | Returns |
|--------|-------|------|---------|
| GET | `/api/tasks/{email}` | none | User's tasks |
| POST | `/api/tasks` | `title`, `subject`, `user_email`, `completed` | Created task |
| PATCH | `/api/tasks/{task_id}/toggle` | none | `completed` |
| DELETE | `/api/tasks/{task_id}` | none | Confirmation |
| POST | `/api/generate-roadmap/{task_id}` | `title`, `subject` | `roadmap` (saved on the task) |
| POST | `/api/youtube-notes` | `video_url_or_transcript` | `notes` |
| POST | `/api/pdf-summary` | `document_text` | `summary` |
| POST | `/api/generate-mock-exam` | `subject_name`, `duration_minutes` | `mock_exam` |
| GET | `/api/test-ai` | none | Gemini connection check |
| GET | `/api/test` | none | Backend running check |

---

## Configuration

| Variable | Where | Required | Default |
|----------|-------|----------|---------|
| `GEMINI_API_KEY` | `backend/.env` | Yes, for AI features | none |
| `GEMINI_MODEL` | `backend/.env` | No | `gemini-3.6-flash` |
| `VITE_API_URL` | `frontend/.env.local` | No | `http://127.0.0.1:8000` |

CORS is allowed only for `http://localhost:5173` and `http://127.0.0.1:5173`. If Vite runs on another port, add that origin in `backend/main.py`.

---

## Troubleshooting

- **`{"detail":"Not Found"}` at the root URL:** This is expected. The backend has no route at `/`. Use `/api/test` or `/docs`.
- **AI features fail with "model not found":** Set `GEMINI_MODEL` in `backend/.env` to a model your API key can access.
- **AI features fail with an API key error:** Check `GEMINI_API_KEY` in `backend/.env`, and restart the server after editing it.
- **Frontend cannot reach the backend:** Make sure the backend is running on port 8000, and that the frontend is opened on `localhost:5173`.
- **`npm audit` warnings:** Most are in development tools (Vite, ESLint). They do not affect the production build. `npm audit fix` (without `--force`) is safe.

---

## Known Limitations

- Users and tasks are stored in memory. They are lost when the server restarts. A database is needed for persistence.
- Login returns the user object but no session token. Pages are not protected on the server side.
- Passwords are hashed (PBKDF2), but this is a student project and has not been security-audited.
- The Gemini model name in the code is a default. Check it against what your API key supports.

---

## Security Notes

- Never commit `backend/.env`. The provided `.gitignore` already excludes it.
- If an API key was ever shared (for example in a zip file), create a new key and delete the old one.

---

## License

Add your license here. For example: MIT.
