# AI Resume Analyzer & Job Matcher

Upload a resume (PDF or text), paste a job description, and get:

- a **match score** with matched and missing skills
- **AI feedback** from Google Gemini: an honest fit summary, rewritten resume bullets and likely interview questions

Built with **Python, FastAPI, Pydantic, pypdf and the Gemini API**.

## Why it is built this way

- **The score is deterministic.** Skills are extracted with a dictionary of canonical names and aliases (word-boundary regex, so "Java" does not match "JavaScript" and "C" does not match "cloud"). The same input always gives the same score, and you can see exactly why.
- **The LLM only advises.** Gemini is asked for JSON, which is validated with Pydantic (`LLMReview`). If the model fails or returns bad JSON, the score and skill lists still work and the API reports `review_error`.
- **The prompt forbids invention.** The model is told not to make up experience, employers or numbers.
- **Privacy:** resumes are processed in memory and never stored.

```
Resume (PDF/txt) --> pypdf text --> extract_skills() \
                                                      >--> match score + gaps --> Gemini (validated JSON) --> response
Job description ---------------> extract_skills()   /
```

## Quick start

```bash
git clone https://github.com/<your-username>/resume-analyzer.git
cd resume-analyzer
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env        # add GEMINI_API_KEY, or set LLM_PROVIDER=mock
uvicorn app.main:app --reload
```

Open http://localhost:8000 (UI) or http://localhost:8000/docs (Swagger). Docker: `docker build -t resume-analyzer . && docker run -p 8000:8000 --env-file .env resume-analyzer`.

## API

`POST /analyze` (multipart form): `resume` (file) and `job_description` (text). Returns:

```json
{
  "match_score": 62,
  "matched_skills": ["FastAPI", "Python"],
  "missing_skills": ["AWS", "Docker"],
  "extra_skills": ["RAG"],
  "review": { "summary": "...", "strengths": [], "improved_bullets": [], "interview_questions": [] },
  "review_error": null
}
```

## Tests

```bash
pytest -q
```

Covers skill extraction edge cases (`C` vs `C++` vs `cloud`, `Java` vs `JavaScript`), scoring, and parsing of LLM output.

## Limitations and roadmap

- The skill dictionary (`app/skills.py`) is finite; unknown skills are not scored
- Scanned (image-only) PDFs are not supported; add OCR
- Add embedding-based semantic matching (for example "built REST services" vs "API development")
- Export the tailored resume as a DOCX/PDF
- Next.js frontend and a deployed demo

## License

MIT
