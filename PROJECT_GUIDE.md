# Nandini Unique Project Guide

This is the day-to-day reference for maintaining the Nandini Unique saree and dress store.

## Current production setup

- Website: https://nandhiniunique.com
- API: https://nandini-unique-api.onrender.com
- API health: https://nandini-unique-api.onrender.com/api/health
- Frontend host: Netlify
- Backend host: Render
- Database: MongoDB, normally MongoDB Atlas in production
- Source repository: GitHub repository connected to Netlify and Render
- Production branch: `main`

The website and API run in the cloud. VS Code and this computer are only needed to edit and push code; the local development servers are not required for the public site.

## Important files

### Frontend

- `frontend/index.html` - storefront home page
- `frontend/shop.html` - product listing, search, and filters
- `frontend/product.html` - product details and purchase options
- `frontend/cart.html` - shopping cart
- `frontend/checkout.html` - checkout and payment reference entry
- `frontend/my-orders.html` - customer order history
- `frontend/track-order.html` - public order tracking
- `frontend/admin/` - owner dashboard pages
- `frontend/css/style.css` - shared styling
- `frontend/js/api.js` - API calls, authentication, cart, and shared frontend logic
- `frontend/js/config.js` - local versus production API URL selection

### Backend

- `backend/server.js` - Express app and route registration
- `backend/config/db.js` - MongoDB connection
- `backend/models/` - User, Product, Order, Offer, and Complaint models
- `backend/routes/` - authentication, products, orders, offers, complaints, and admin routes
- `backend/utils/` - email and WhatsApp helpers
- `backend/seed.js` - optional initial admin and sample-data seed
- `backend/uploads/` - uploaded product images

### Deployment

- `netlify.toml` - Netlify publish directory and response headers
- `render.yaml` - Render Blueprint for the API and optional static frontend service
- `DEPLOYMENT.md` - detailed hosting and DNS reference
- `backend/.env.example` - names and examples of environment variables

## Local development

Install dependencies once:

```bash
npm install
cd backend
npm install
cd ..
```

Start both local services:

```bash
npm start
```

This starts:

- Frontend: http://localhost:5500
- API: http://localhost:5000
- Health check: http://localhost:5000/api/health

Start services separately when needed:

```bash
npm run frontend
npm run backend
```

For backend auto-reload during development:

```bash
cd backend
npm run dev
```

The frontend uses `http://localhost:5000` only when opened on `localhost` or `127.0.0.1`. Any deployed hostname uses the public Render API from `frontend/js/config.js`.

## Environment variables

Create `backend/.env` locally from `backend/.env.example`. Never commit `.env` files, database URLs, passwords, SMTP credentials, JWT secrets, or GitHub tokens.

Important values:

- `MONGO_URI` - local MongoDB or MongoDB Atlas connection string
- `PORT` - local API port, normally `5000`; Render supplies its own port
- `JWT_SECRET` - long random secret for login tokens
- `FRONTEND_URL` - `http://localhost:5500` locally and `https://nandhiniunique.com` in production
- `STORE_WHATSAPP_NUMBER` - store number with country code and no `+` or spaces
- `STORE_EMAIL` - store email address
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` - optional email delivery settings
- `TWILIO_*` - optional automatic WhatsApp messaging settings
- `RAZORPAY_*` - reserved for a future online payment integration

Set production values in the Render service environment dashboard, not in source files.

## Owner/admin workflow

Open the admin panel at:

`https://nandhiniunique.com/admin/login.html`

Routine tasks:

1. Add or edit products in **Products**.
2. Upload product images and set product type, fabric/type, details, price, discount, sizes, and stock.
3. Review new orders in **Orders**.
4. Verify the customer's UTR or transaction ID.
5. Move orders through their status sequence: Placed, Confirmed, Packed, Shipped, Out for Delivery, Delivered.
6. Add offers and coupon codes in **Offers**.
7. Read and respond to customer messages in **Complaints**.

Change the initial seeded admin password immediately after first use. Do not record admin passwords in this document.

## Deploying code changes

Before committing:

```bash
git status
git diff --check
node --check frontend/js/config.js
```

Commit only the intended files:

```bash
git add <files>
git commit -m "Describe the change"
git push origin main
```

Netlify redeploys the frontend and Render redeploys the API from the connected branch. Check both provider dashboards after a push.

Verify production after deployment:

```bash
curl -I https://nandhiniunique.com/
curl https://nandini-unique-api.onrender.com/api/health
```

Never paste a GitHub token into chat, source code, a document, or a commit. Use a personal access token directly at the terminal prompt. Revoke any token that has been exposed.

## Database and data safety

- Use MongoDB Atlas backups for production data.
- Do not run seed commands against production unless the script and target database have been verified.
- Product images are stored under `backend/uploads/`; confirm the production storage strategy before relying on local disk uploads during a redeploy.
- Before schema changes, inspect the existing Mongoose models and routes and test with a non-production database.

## Troubleshooting

### Website is down

1. Check Netlify deploy status and deployment logs.
2. Check DNS for `nandhiniunique.com` and `www.nandhiniunique.com`.
3. Test the homepage with `curl -I`.

### Website loads but products or login fail

1. Open the API health URL.
2. Check Render logs.
3. Check `MONGO_URI`, `JWT_SECRET`, and `FRONTEND_URL` in Render.
4. Confirm `frontend/js/config.js` contains the public Render API URL and no localtunnel URL.

### Changes are not visible

1. Confirm the intended commit reached the connected `main` branch.
2. Check the Netlify or Render deploy log.
3. Reload after deployment; `config.js` is configured with `Cache-Control: no-store`.

### Port already in use locally

Check which process owns the ports, or use the existing running service. Do not start duplicate API/frontend servers on ports `5000` and `5500`.

## Safe change checklist

- Keep public APIs and route names backward compatible where possible.
- Validate user input on the backend, not only in HTML.
- Do not expose secrets in browser JavaScript.
- Test login, product listing, cart, checkout, admin orders, and order tracking after shared API changes.
- Keep deployment configuration in `netlify.toml` and `render.yaml` aligned with the actual hosting providers.
