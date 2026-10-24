<div align="center">

# Resume Matcher

### Find the resumes that align most closely with a job description.

An explainable resume-ranking demo that combines technical-skill overlap with TF-IDF text similarity, then surfaces missing skills and an overall match score.

<br />

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![scikit--learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white)
![Google Gemini](https://img.shields.io/badge/Google%20Gemini-4285F4?style=for-the-badge&logo=googlegemini&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=222222)
![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)

</div>

---

## Overview

Enter a job description, upload resumes, and compare the resulting match scores. The interface sorts candidates from highest to lowest score and displays detected skills and skills missing from each resume.

The project has three main parts:

| Part | What it does |
| --- | --- |
| **Frontend** | Static HTML, CSS, and JavaScript interface for entering a job description, selecting files, and viewing ranked results |
| **Backend API** | FastAPI `/match` endpoint that extracts resume text, runs matching, and returns a JSON result |
| **Matching and extraction** | A frozen scikit-learn TF-IDF vectorizer and predefined skill list produce the match score; Google Gemini extracts candidate skills for display |

## Tech stack

<div align="center">

| Layer | Technologies |
| :--- | :--- |
| **UI** | ![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=222222) ![Tailwind CSS](https://img.shields.io/badge/Tailwind-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white) |
| **API** | ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) ![Uvicorn](https://img.shields.io/badge/Uvicorn-4051B5?style=flat-square&logo=gunicorn&logoColor=white) |
| **Matching** | ![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white) ![TF--IDF](https://img.shields.io/badge/TF--IDF-7C3AED?style=flat-square) ![Gemini](https://img.shields.io/badge/Gemini-4285F4?style=flat-square&logo=googlegemini&logoColor=white) |
| **Resume parsing** | ![pdfplumber](https://img.shields.io/badge/pdfplumber-334155?style=flat-square) |

</div>

## System architecture

```mermaid
flowchart LR
    person["Recruiter"]
    subgraph client["Browser · Frontend"]
        form["Job description<br/>and resume selection"]
        rank["Ranked results<br/>match score · skills"]
    end
    subgraph api["Python · FastAPI"]
        endpoint["POST /match"]
        pdf["Extract resume text<br/>with pdfplumber"]
        response["Build JSON response"]
    end
    subgraph matching["Matching · scikit-learn"]
        tfidf["Frozen TF-IDF vectorizer<br/>cosine similarity"]
        skills["Predefined skill list<br/>skill overlap"]
        score["Weighted match score"]
    end
    gemini["Google Gemini<br/>candidate skill extraction"]

    person --> form
    form -->|"job description + resume"| endpoint
    endpoint --> pdf
    pdf --> tfidf
    pdf --> skills
    tfidf --> score
    skills --> score
    pdf --> gemini
    score --> response
    gemini --> response
    response -->|"score, extracted skills,<br/>missing skills"| rank
    rank --> person

    classDef user fill:#F1F5F9,stroke:#64748B,color:#0F172A
    classDef frontend fill:#CFFAFE,stroke:#0891B2,color:#164E63
    classDef apiNode fill:#DCFCE7,stroke:#16A34A,color:#14532D
    classDef ml fill:#EDE9FE,stroke:#7C3AED,color:#4C1D95
    classDef ai fill:#FEF3C7,stroke:#D97706,color:#78350F
    class person user
    class form,rank frontend
    class endpoint,pdf,response apiNode
    class tfidf,skills,score ml
    class gemini ai
    style client fill:#ECFEFF,stroke:#06B6D4
    style api fill:#F0FDF4,stroke:#22C55E
    style matching fill:#F5F3FF,stroke:#8B5CF6
```

### Match score

The backend combines two signals:

```text
final_score = 0.4 × cosine_similarity + 0.6 × skill_overlap
match_percentage = final_score × 100
```

- **Cosine similarity** compares normalized job-description and resume text transformed by the saved TF-IDF vectorizer.
- **Skill overlap** compares skills found in the job description with skills found in the resume. Its overlap score is the fraction of job-description skills found in the resume.
- **Missing skills** are the job-description skills that were not found in the resume.

The score is a relative matching signal, **not** a probability of success or a hiring recommendation.

## Repository layout

```text
.
├── backend/
│   ├── main.py                  # FastAPI app and /match endpoint
│   ├── requirements.txt         # API dependencies
│   └── ML/
│       ├── ml_logic.py          # TF-IDF, skill extraction, and scoring
│       ├── tfidf_vectorizer.pkl # Pre-trained vectorizer used for matching
│       ├── test_matcher.py      # Sample matcher run
│       └── data/                # Example job descriptions and resumes
└── frontend/
    ├── index.html               # App page
    ├── app.js                   # Upload flow and ranked results
    └── style.css                # UI styles
```

## Getting started

### 1. Start the backend

From the repository root, create and activate a virtual environment, then install the backend dependencies:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Create `backend/.env` and add a Google Gemini API key:

```env
GEMINI_API_KEY=your_api_key
```

Start the API from the `backend` directory:

```powershell
uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

The API root is `http://127.0.0.1:8000/`; interactive API documentation is at `http://127.0.0.1:8000/docs`.

### 2. Open the frontend

In a second terminal, serve the frontend directory:

```powershell
cd frontend
python -m http.server 5500
```

Open `http://127.0.0.1:5500` in a browser. The frontend currently sends requests to the deployed API URL `https://resume-jd-matching-ml.onrender.com/match`. To use the local backend instead, change the URL in [`frontend/app.js`](frontend/app.js) to `http://127.0.0.1:8000/match`.

### 3. Run the matcher example

The sample matcher script uses the bundled vectorizer and example texts:

```powershell
cd backend\ML
python test_matcher.py
```

## API

`POST /match` accepts multipart form data:

| Field | Type | Description |
| --- | --- | --- |
| `job_description` | string | Job description to compare against |
| `file` | file | Resume PDF |

On success, the response contains `filename` and `data` fields with `match_percentage`, `extracted_skills`, and `missing_skills`. The frontend sends each selected resume in a separate request, then sorts successful results by score.

## Current limitations

- **File-type mismatch:** the frontend file picker currently accepts `.txt` files, but the backend extracts text from PDFs. Use a PDF when calling the API directly; align the frontend file picker with the backend before using the complete browser flow.
- **Local API URL:** the frontend defaults to the deployed Render API. Change the URL in `frontend/app.js` to use a locally running backend.
- **Gemini configuration:** skill extraction requires `GEMINI_API_KEY`. If Gemini returns no skills or errors, the backend falls back to the skills found by the matcher.
- **Text-based PDFs:** image-only or scanned PDFs may not yield extractable text.
- **Matching limitations:** TF-IDF is lexical rather than deeply semantic, and skill detection relies on a predefined vocabulary.
- **Privacy:** resume text is sent to the configured backend, and the backend sends resume text to Gemini for skill extraction. Do not upload sensitive documents unless this processing is appropriate for your use case.
- **Hiring use:** scores can reflect wording and data limitations. Use the results only as decision-support and review resumes directly; do not use the score as an automated hiring decision.

## Further reading

- [ML approach and limitations](backend/ML/README.md)
- [Backend API](backend/main.py)
- [Matching logic](backend/ML/ml_logic.py)
