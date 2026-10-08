# 📄 ATS Resume Checker

Upload a resume (PDF, DOCX or TXT) and get an **estimated ATS score at the top**, followed by a score
breakdown, strengths, prioritised improvements, missing keywords and formatting issues.
Optionally paste a job description for a role-specific keyword match.

**Stack:** Streamlit (UI) · Google Gemini Flash (AI) · pypdf / python-docx (file parsing)

## How the score works

Gemini scores five sections (0-100 each). The app then computes the overall ATS score as a weighted
average, so the number is explainable and consistent:

| Section                  | Weight |
|--------------------------|-------:|
| Keywords & Skills        | 25%    |
| Experience & Impact      | 30%    |
| Formatting & Readability | 20%    |
| Structure & Completeness | 15%    |
| Language & Grammar       | 10%    |

> The score is an AI-based estimate, not the output of a real ATS (Workday, Greenhouse, Lever...). Use it as guidance.

## Project structure

```
ats-checker/
├── app.py
├── requirements.txt
└── README.md
```

## Run locally

1. **Get a free Gemini API key:** https://aistudio.google.com/apikey
2. **Create a virtual environment and install dependencies** (Python 3.10+):
   ```bash
   python -m venv venv
   source venv/bin/activate        # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. **Add your key** (choose one):
   - Paste it into the sidebar when the app runs, **or**
   - Create `.streamlit/secrets.toml`:
     ```toml
     GEMINI_API_KEY = "your-key-here"
     # GEMINI_MODEL = "gemini-3.8-flash"   # optional
     ```
   - Or set an environment variable: `export GEMINI_API_KEY="your-key-here"`
4. **Start the app:**
   ```bash
   streamlit run app.py
   ```

### Choosing the Gemini model

The default is `gemini-3.8-flash`. To use another model, set `GEMINI_MODEL` (secret or env var) or type it into the sidebar.
Google retires models regularly (for example `gemini-2.5-flash` is no longer available to new users), so the app is built to cope:
if the chosen model is retired it skips to a backup (`gemini-3.7-flash`, `gemini-3.5-flash`, `gemini-3.1-flash-lite`), and if all of
those are gone it asks the API which Flash models your key can use. Current names: https://ai.google.dev/gemini-api/docs/models

## Push to GitHub

> **Never commit your API key.** The `.gitignore` below keeps `secrets.toml` out of the repo.

1. Create an **empty** repository on GitHub (https://github.com/new), e.g. `ats-resume-checker`. Do not add a README there.
2. In your project folder run:
   ```bash
   # keep secrets and clutter out of git
   printf ".streamlit/secrets.toml\nvenv/\n__pycache__/\n.env\n" > .gitignore

   git init
   git add .
   git commit -m "Initial commit: ATS resume checker"
   git branch -M main
   git remote add origin https://github.com/<your-username>/ats-resume-checker.git
   git push -u origin main
   ```
   (GitHub no longer accepts account passwords for `git push`. When prompted, use a
   [Personal Access Token](https://github.com/settings/tokens), or sign in via GitHub CLI: `gh auth login`.)
3. Check on GitHub that `secrets.toml` is **not** listed.

## Deploy on Streamlit Community Cloud (free)

1. Go to https://share.streamlit.io and sign in with GitHub (authorise access to your repo if asked).
2. Click **Create app** → **Deploy a public app from GitHub**.
3. Choose your **repository**, **branch** `main`, and **main file path** `app.py`.
4. Open **Advanced settings**:
   - Pick Python **3.11** or **3.12**.
   - In **Secrets**, paste:
     ```toml
     GEMINI_API_KEY = "your-key-here"
     ```
5. Click **Deploy**. The first build takes a few minutes; you get a public `*.streamlit.app` URL.

To update the app later, just `git push` to `main`, and Streamlit redeploys automatically.
You can edit secrets any time under the app's **Settings → Secrets**.

**Public-app warning:** anyone with the link can use your API key's quota. Either restrict viewers in the
app's sharing settings, leave the secret out so each user pastes their own key in the sidebar, or set a quota on the key in Google AI Studio.

## Troubleshooting

| Problem | Fix |
|---|---|
| "Very little text could be extracted" | The resume is probably a scanned image. Export a text-based PDF/DOCX. |
| "Gemini rejected the API key" | Re-copy the key; make sure it's from Google AI Studio. |
| "None of the Gemini models are available" (404) | Google retired the model. Set the sidebar model to the current one (e.g. `gemini-3.8-flash`) and check your `GEMINI_MODEL` secret isn't set to an old name. |
| "Gemini is overloaded" (503) | Temporary on Google's side. The app retries 3x per model and falls back to the backup models in `FALLBACK_MODELS`. If all fail, wait a minute and retry. Edit `FALLBACK_MODELS` in `app.py` to change the backups. |
| Rate limit / quota message | Wait a minute, or enable billing on your Google AI project. |
| `ModuleNotFoundError` on deploy | Make sure `requirements.txt` is in the repo root. |

## Privacy

Resume text is sent to Google's Gemini API for analysis. The app itself does not store uploaded files or results
(they live only in your browser session). Review Google's Gemini API data-use terms before processing other people's resumes.
