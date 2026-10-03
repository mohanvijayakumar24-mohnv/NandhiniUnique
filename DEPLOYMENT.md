# Nandini Unique Deployment

## Production URLs

- Storefront: https://nandhiniunique.com
- Canonical `www` URL: https://www.nandhiniunique.com (redirects to the apex domain)
- API: https://nandini-unique-api.onrender.com
- API health check: https://nandini-unique-api.onrender.com/api/health

The production site does not depend on VS Code or the local computer. Netlify hosts the frontend and Render hosts the API.

## Hosting

### Frontend: Netlify

- Source: GitHub repository `NandhiniUnique`
- Publish directory: `frontend`
- Configuration: `netlify.toml`
- Custom domains: `nandhiniunique.com` and `www.nandhiniunique.com`
- `frontend/js/config.js` uses the local API only on `localhost` and uses the public Render API everywhere else.
- The API configuration is served with `Cache-Control: no-store` so an old local or tunnel URL is not retained by the CDN.

### Backend: Render

The Render Blueprint is in `render.yaml`.

- Service name: `nandini-unique-api`
- Root directory: `backend`
- Build command: `npm install`
- Start command: `npm start`
- Health check: `/api/health`

The Blueprint also describes the static frontend service `nandini-unique-web`. The currently live frontend is served by Netlify; use one frontend host at a time to avoid confusion.

## Required Render environment variables

Set these in the Render service dashboard. Never commit their values.

- `MONGO_URI` - MongoDB Atlas connection string
- `JWT_SECRET` - Render can generate this value
- `FRONTEND_URL` - `https://nandhiniunique.com`
- `STORE_WHATSAPP_NUMBER` - business number with country code, without `+` or spaces
- `STORE_EMAIL` - store contact email
- `SMTP_HOST` - SMTP provider hostname
- `SMTP_PORT` - usually `587`
- `SMTP_USER` - SMTP account
- `SMTP_PASS` - SMTP password or app password

After changing environment variables, redeploy the Render API service.

## DNS

The domain must point to the hosting provider configured for the frontend. DNS is currently active for the Netlify deployment. If reconnecting the domain, use the DNS records shown in the Netlify domain settings rather than mixing them with Render records.

For the Render frontend service instead, add the records Render displays in its custom-domain panel. Do not configure both Netlify and Render as the active frontend for the same domain.

## Deploying changes

1. Make and test changes locally.
2. Check the deployment diff:

   ```bash
   git status
   git diff --check
   ```

3. Commit only the intended files:

   ```bash
   git add <files>
   git commit -m "Describe the change"
   ```

4. Push the branch connected to Netlify and Render:

   ```bash
   git push origin main
   ```

5. Wait for both providers to finish their deploys.
6. Verify the public site and API:

   ```bash
   curl -I https://nandhiniunique.com/
   curl https://nandini-unique-api.onrender.com/api/health
   ```

GitHub authentication requires a personal access token with write access to the repository. Enter tokens directly in the terminal; do not put them in this document, source files, chat, or commit messages.

## Local development

Run the API and static frontend locally when needed:

```bash
cd backend
npm install
npm start
```

In a second terminal:

```bash
python3 -m http.server 5500 --directory frontend
```

Open http://localhost:5500. Local pages use `http://localhost:5000`; deployed pages use the public Render API automatically.

## Troubleshooting

- `401` or `403` from the API: check the Render environment variables and API logs.
- Frontend shows a local or tunnel API URL: confirm the deployed `frontend/js/config.js` has only the localhost branch and the public Render URL, then redeploy Netlify.
- Domain shows an old page: trigger a new Netlify deploy and check DNS plus CDN cache headers.
- Render health check fails: open `/api/health`, then inspect the Render logs and verify `MONGO_URI`.
