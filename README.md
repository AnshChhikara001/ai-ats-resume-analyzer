# ATS Resume Scorer

A web app that scores how well a resume matches a job description and returns actionable feedback. Built with FastAPI + Streamlit. Llama 3.3 70B (via Groq) parses the resume and job description into structured fields; spaCy and Sentence Transformers do the scoring.

## What it does

1. Upload a resume (PDF / DOC / DOCX) and paste a job description.
2. The backend parses the resume, extracts skills and experience, and compares them to the JD using semantic similarity.
3. You get an ATS score, a breakdown by category (formatting, keywords, content, skill validation, ATS compatibility), and a list of specific issues with suggestions for what to improve.
4. Past analyses are saved to your account so you can revisit them.

## How the score works

- **Parsing.** The LLM returns the resume and the job description as JSON (skills, projects, experience, keywords, contact details). Invalid JSON gets one stricter retry before the request fails.
- **ATS score.** Five components, each scored separately: formatting, keywords, content, skill validation and ATS compatibility.
- **Skill validation.** Every skill the resume lists is checked against the project and experience text: an exact match first, then embedding similarity of at least 0.6. Skills with no supporting evidence are flagged.
- **Job description match.** 60% keyword overlap plus 40% semantic similarity between `all-MiniLM-L6-v2` embeddings of the resume and the job description.

## Tech stack

- **Frontend:** Streamlit
- **Backend:** FastAPI (Python)
- **NLP:** spaCy (`en_core_web_md`), Sentence Transformers (`all-MiniLM-L6-v2`)
- **LLM:** Groq API (Llama 3.3 70B) for structured resume and job description parsing
- **Auth + Database:** Supabase (email/password and Google OAuth)
- **PDF report export:** WeasyPrint + Jinja2

## Project structure

```
ai-ats-resume-analyzer/
├── backend/              FastAPI app, NLP services, API routes
├── frontend/             Streamlit app, views, components
├── notebooks/            Research and dataset prep (not used at runtime)
├── requirements.txt      Combined backend + frontend dependencies
└── .env.example          Template for environment variables
```

## Setup

### 1. Clone and create a virtual environment

```bash
git clone https://github.com/AnshChhikara001/ai-ats-resume-analyzer.git
cd ai-ats-resume-analyzer
python -m venv venv
source venv/bin/activate         # Windows: venv\Scripts\activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
python -m spacy download en_core_web_md
```

WeasyPrint needs system libraries on Linux:

```bash
# Fedora
sudo dnf install -y cairo pango gdk-pixbuf2 libffi

# Debian / Ubuntu
sudo apt install -y libcairo2 libpango-1.0-0 libpangoft2-1.0-0 libffi-dev
```

### 3. Configure environment variables

Copy the template and fill in your keys:

```bash
cp .env.example .env
```

You need:

- A **Supabase** project — grab `SUPABASE_URL`, `SUPABASE_KEY` (service role), and `SUPABASE_ANON_KEY` from Project Settings → API.
- A **Groq** API key from [console.groq.com](https://console.groq.com).
- (Optional) Google OAuth set up in the Supabase dashboard if you want Google sign-in.

The Streamlit frontend also reads Supabase config from `frontend/.streamlit/secrets.toml`. Copy `secrets.toml.example` to `secrets.toml` and fill it in.

### 4. Run the backend

From the project root:

```bash
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

The API is now at `http://localhost:8000`.

### 5. Run the frontend

In a new terminal (with the venv activated):

```bash
streamlit run frontend/streamlit_app.py
```

The app opens at `http://localhost:8501`.

## Notes

- **Never commit `.env` or `secrets.toml`** — they hold API keys. Both are in `.gitignore`; check before you push.
- The first run downloads the Sentence Transformer model (~80 MB). It's cached afterwards.
- A Groq API key is required: the resume and job description are parsed by the LLM before anything is scored.
- `notebooks/` is for experimentation and isn't required to run the app.
