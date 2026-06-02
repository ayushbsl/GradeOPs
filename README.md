<div align="center">


# 🎓 GradeOps

**AI-Powered Exam Grading & Results Management Platform for Instructors**

![Python](https://img.shields.io/badge/Python-3.12-blue?style=flat&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-Latest-009688?style=flat&logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-19.2-61DAFB?style=flat&logo=react&logoColor=black)
![Qwen2-VL](https://img.shields.io/badge/Qwen2--VL-OCR-orange?style=flat)
![Vite](https://img.shields.io/badge/Vite-8.0-646CFF?style=flat&logo=vite&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat)

</div>
---


## Overview

**GradeOps** is a full-stack, AI-assisted exam grading platform designed to eliminate the bottleneck of manual answer sheet evaluation in academic environments. It addresses the inefficiency and inconsistency of human grading by combining Vision-Language Model OCR with structured marking scheme evaluation to produce objective, reasoned scores in real-time.

Instructors can create exams, register students, upload marking schemes (JSON or PDF), and submit scanned answer sheets (image or PDF) — the system extracts handwritten or typed text via the **Qwen2-VL-2B-OCR** model and evaluates each response against predefined expected points and keywords, returning per-question scores, matched/missing criteria, and an overall reasoning summary.

Built by **Ayush Bansal**, **Lalit Deshmane**, **Krish Patel** as a self-contained grading backend with a single-page instructor dashboard.

---

## Key Features

- **Vision-Language OCR Grading:** Qwen2-VL-2B-OCR model (via HuggingFace Transformers) extracts text from handwritten or printed answer sheets supplied as images (JPEG, PNG) or multi-page PDFs.
- **Structured Marking Scheme Engine:** Upload marking schemes as JSON files or text-based PDFs. Each question defines expected points, keywords, max marks, and a strictness level (`low` / `medium` / `high`) that controls scoring leniency.
- **Per-Question Reasoning:** Every graded submission returns matched criteria, missing criteria, awarded marks, and a natural-language justification for each question alongside an overall reasoning summary.
- **Unified Single-Page Dashboard:** React 19 frontend with a persistent sticky sidebar and collapsible accordion sections — Exams, Students, and Results are all visible on one page without navigation.
- **JWT-Protected Instructor API:** All endpoints require a Bearer token issued on login. Passwords are hashed with bcrypt via Passlib.
- **Zero-Dependency Storage:** All data (users, exams, students, schemes, submissions, evaluations) is persisted as flat JSON files — no database setup required.
- **PDF Support:** Both marking schemes and answer sheets accept PDFs. Multi-page answer PDFs are stitched into a single image before OCR; text-based scheme PDFs are parsed directly without re-rendering.

---

## System Architecture

GradeOps operates as a decoupled two-tier system to separate AI inference from the instructor-facing API:

1. **Instructor Portal (React Frontend):** Single-page dashboard served by Vite. Communicates with the backend exclusively via Axios REST calls using a stored JWT token.

2. **API Orchestrator (FastAPI Backend):** Manages authentication, data persistence, file uploads, and coordination between the OCR and evaluation services. Six route modules handle distinct domains.

3. **AI Grading Engine:**
   - **Layer 1 — File Normalisation:** PyMuPDF renders PDF pages to PIL images at 200 DPI; JPEG/PNG files are opened directly with Pillow.
   - **Layer 2 — Vision OCR:** Qwen2-VL-2B-OCR tokenises the image and generates a plain-text transcript of the answer sheet (up to 2048 tokens, configurable).
   - **Layer 3 — Scheme Evaluation:** An LLM-based evaluator compares the extracted transcript against each question's `expected_points` and `keywords`, applying the configured strictness to award partial or full marks.
   - **Layer 4 — Score Aggregation:** Per-question results are aggregated into a total score, percentage, and overall reasoning string, then persisted as an evaluation record.

4. **JSON File Store:** All records are written to structured directories under `data/` — no external database or migration tooling required.

---

## Project Structure

```
GradeOPs/
├── backend/
│   ├── app/
│   │   ├── main.py                      # FastAPI app, CORS middleware, router registration
│   │   ├── config.py                    # Env config — model name, JWT settings, data paths
│   │   ├── routes/
│   │   │   ├── auth.py                  # POST /auth/register, /auth/login
│   │   │   ├── exams.py                 # POST /exams/create, GET /exams/
│   │   │   ├── students.py              # POST /students/create, GET /students/
│   │   │   ├── evaluate.py              # POST /evaluate/upload-scheme, /evaluate/grade
│   │   │   ├── ocr.py                   # POST /ocr/ — raw OCR endpoint
│   │   │   └── submissions.py           # GET /submissions/exam/{id}, /submissions/student/{id}
│   │   ├── services/
│   │   │   ├── ocr_service.py           # Qwen2-VL model loading & inference
│   │   │   ├── evaluation_service.py    # Answer text scoring against marking scheme
│   │   │   ├── marking_scheme_service.py # Scheme parsing — JSON & text-based PDF
│   │   │   └── submission_service.py    # Submission & evaluation record creation
│   │   ├── schemas/
│   │   │   └── evaluation_schema.py     # Pydantic models — MarkingScheme, EvaluationResponse
│   │   ├── utils/
│   │   │   ├── image_loader.py          # PIL + PyMuPDF PDF-to-image pipeline
│   │   │   └── auth_guard.py            # JWT Bearer token dependency
│   │   └── storage/                     # JSON file-based CRUD for each domain
│   │       ├── exam_store.py
│   │       ├── student_store.py
│   │       ├── scheme_store.py
│   │       └── submission_store.py
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── App.jsx                      # Route definitions & legacy redirects
│   │   ├── main.jsx                     # React root — BrowserRouter + AuthProvider
│   │   ├── pages/
│   │   │   ├── Dashboard.jsx            # Unified dashboard — Exams, Students, Results
│   │   │   ├── LandingPage.jsx          # Public landing page
│   │   │   └── LoginPage.jsx            # Instructor login / register
│   │   ├── components/
│   │   │   ├── Sidebar.jsx              # Sticky sidebar with IntersectionObserver scroll-spy
│   │   │   ├── ExamCard.jsx             # Exam summary card
│   │   │   ├── AuthCard.jsx             # Login / register form component
│   │   │   └── ProtectedRoute.jsx       # JWT auth guard wrapper
│   │   ├── services/
│   │   │   ├── authService.js           # Register & login API calls
│   │   │   ├── examService.js           # Exam & student API calls
│   │   │   └── evaluationService.js     # Scheme upload, grading, results API calls
│   │   ├── context/
│   │   │   └── AuthContext.jsx          # Global JWT auth state (token, user, login, logout)
│   │   └── styles/                      # Per-component CSS — dark theme (#0d1117, purple accent)
│   ├── vite.config.js
│   └── package.json
└── generate_demo_pdfs.py                # Generates demo_scheme.pdf & demo_answer_sheet.pdf
```

---

## Getting Started

### Prerequisites

- Python 3.12+
- Node.js 18+
- A CUDA-capable GPU is recommended for Qwen2-VL inference (CPU fallback works but is slow)

### Backend Setup

```bash
cd backend
python -m venv venv
venv\Scripts\activate          # Windows
pip install -r requirements.txt
uvicorn app.main:app --reload
```

The API will be available at `http://127.0.0.1:8000`. Interactive docs at `/docs`.

### Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

The dashboard will be available at `http://localhost:5173`.

---

## Environment Variables

Create a `.env` file inside `backend/` to override defaults:

| Variable | Default | Description |
|---|---|---|
| `MODEL_NAME` | `JackChew/Qwen2-VL-2B-OCR` | HuggingFace model ID for OCR |
| `MAX_NEW_TOKENS` | `2048` | Max tokens generated per OCR pass |
| `UPLOAD_DIR` | `uploads` | Directory for saved answer sheet files |
| `DATA_DIR` | `data` | Root directory for all JSON stores |
| `JWT_SECRET_KEY` | *(insecure default)* | **Must be changed in production** |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `60` | JWT token lifetime |

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/auth/register` | Register a new instructor |
| `POST` | `/auth/login` | Login and receive a JWT token |
| `POST` | `/exams/create` | Create a new exam |
| `GET` | `/exams/` | List all exams |
| `POST` | `/students/create` | Register a new student |
| `GET` | `/students/` | List all students |
| `POST` | `/evaluate/upload-scheme` | Upload a marking scheme (JSON or PDF) |
| `POST` | `/evaluate/grade` | Grade an answer sheet (image or PDF) |
| `GET` | `/submissions/exam/{exam_id}` | Fetch all results for an exam |
| `GET` | `/submissions/student/{student_id}` | Fetch all results for a student |

All endpoints except `/auth/*` require an `Authorization: Bearer <token>` header.

---

## Demo Files

Run the included generator to create sample PDFs for testing:

```bash
python generate_demo_pdfs.py
```

This produces on your Desktop:
- **`demo_scheme.pdf`** — 3-question marking scheme (OS, RAM, CPU) — 30 marks total
- **`demo_answer_sheet.pdf`** — Partial student answers for `STU001` (Q1 full, Q2 partial, Q3 blank)
updated finally
