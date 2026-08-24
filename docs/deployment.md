# Deploying to GitHub Pages

The frontend is a single self-contained `index.html` — no build step, no bundler, no relative-path API calls. That makes this close to the simplest possible Pages deployment.

## Steps

1. **Point the app at your own n8n instance first.** Update `ANALYZE_URL`, `TAILOR_URL`, and `EMAIL_URL` near the top of the `<script>` block in `index.html` — see [`n8n-setup.md`](./n8n-setup.md) for where these come from.
2. **Push this repo to GitHub** (see the root README / your own `git push` for the exact commands).
3. In the repo on GitHub: **Settings → Pages → Build and deployment → Source: Deploy from a branch**. Pick `main` and `/ (root)`, then Save.
4. Wait a minute or two — GitHub will publish to `https://<your-username>.github.io/<repo-name>/`.

## Things that only show up once you're live (not on `file://`)

- **CORS preflight.** All three n8n webhooks are configured with `allowedOrigins: "*"` and respond with `Access-Control-Allow-Origin: *`, so any origin — including your `github.io` URL — should be allowed. But `file://` doesn't exercise a real cross-origin `OPTIONS` preflight the way an actual HTTPS origin does. Run through analyze → tailor → email once from the live Pages URL before considering this done; this is the first real test of that config.
- **n8n workflow must be Active.** If you get a 404 instead of a CORS error, the workflow is probably still toggled inactive in n8n rather than a frontend problem.
- **Payload size over the real network.** The email step sends a base64-encoded PDF in the request body. This behaves identically to local testing in practice, but it's the step worth re-verifying end-to-end on the deployed version since it's had the most iteration.

## Updating the app later

Since it's a single static file, updates are just: edit `index.html`, commit, push. GitHub Pages redeploys automatically within a minute or so — no separate build/deploy step.
