# Resume Tailor

A free, self-hosted web app that scores your resume against a job description, AI-tailors it to match, and exports a polished PDF or Word doc — all without a paid backend.

**Live app:** _add your GitHub Pages URL here after enabling Pages_

## How it works

1. Paste a job description and upload your resume (PDF).
2. Get an instant AI match score with matched/missing keywords.
3. Optionally have the resume AI-tailored to the job description.
4. Edit the result, pick an accent color and font, and download as PDF or Word.
5. Optionally email yourself a copy (with the job description) to track the application.

There's also a **Consultancy mode** that adds a custom letterhead banner designer above your own resume header — drag/resize images and text, upload custom fonts, and flatten the design into a single reusable banner image.

## Stack

- **Frontend:** a single self-contained `index.html` (no build step, no framework) — see [`index.html`](./index.html). Uses [jsPDF](https://github.com/parallax/jsPDF) for PDF export and [docx.js](https://github.com/dolanmiu/docx) for Word export, both loaded from cdnjs.
- **Backend:** three [n8n](https://n8n.io) webhook workflows (JSON exports in [`/n8n`](./n8n)) using [Google Gemini](https://aistudio.google.com/) (free tier via Google AI Studio) as the LLM.
- **Hosting:** GitHub Pages (frontend) + n8n Cloud or self-hosted n8n (backend).

## Repo structure

```
index.html              the entire frontend app
n8n/
  analyze-match.json     webhook: JD + resume -> match score, matched/missing keywords
  tailor-resume.json     webhook: JD + resume -> fully AI-tailored, structured resume
  email-resume.json      webhook: emails a PDF + the JD to the user via Gmail API
docs/
  n8n-setup.md            how to import and configure the three workflows
  deployment.md           how to deploy the frontend to GitHub Pages
```

## Quick start

1. Import the three JSON files in [`/n8n`](./n8n) into your n8n instance — see [`docs/n8n-setup.md`](./docs/n8n-setup.md).
2. Update the three webhook URL constants near the top of `index.html`'s `<script>` block (`ANALYZE_URL`, `TAILOR_URL`, `EMAIL_URL`) to point at your own n8n instance.
3. Deploy `index.html` to GitHub Pages — see [`docs/deployment.md`](./docs/deployment.md).

## Security notes

- No API keys or credentials are stored in this repo. The n8n workflow JSONs use placeholder credential references (`REPLACE_WITH_YOUR_...`) that you connect to your own credentials after import.
- The Gmail integration uses OAuth2 with the minimal `gmail.compose` scope only — not broad read/delete access.
- All three n8n webhooks respond with CORS headers scoped to `Access-Control-Allow-Origin: *`, since this is a public, credential-free-on-the-client app. If you fork this for private/internal use, consider restricting that to your own frontend's origin.

## License

MIT — see [LICENSE](./LICENSE).
