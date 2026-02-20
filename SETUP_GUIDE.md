# 🍜 Schiegaarden Kinarestaurant — Hosting Setup Guide

This guide sets up your restaurant website using:
- **Docker + Caddy** → serves your website
- **Cloudflare Tunnel** → exposes it to the internet for free, securely

---

## 📁 Project Structure

```
schiegaarden/
├── docker-compose.yml   ← orchestrates everything
├── Caddyfile            ← web server config
├── .env                 ← your secret tunnel token (never share this)
├── site/
│   └── index.html       ← the restaurant website
└── SETUP_GUIDE.md       ← you are here
```

---

## ✅ Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- A free Cloudflare account (instructions below)

---

## STEP 1 — Create a Free Cloudflare Account

1. Go to → https://dash.cloudflare.com/sign-up
2. Sign up with your email (it's free, no credit card needed)
3. Verify your email address

---

## STEP 2 — Create a Cloudflare Tunnel

1. Log in to https://dash.cloudflare.com
2. In the left sidebar click **"Zero Trust"**
3. If prompted, create a free Zero Trust organization (pick any team name, choose the **Free plan**)
4. Go to **Networks → Tunnels**
5. Click **"Create a tunnel"**
6. Choose **"Cloudflared"** → click Next
7. Give your tunnel a name, e.g. `schiegaarden` → click Save
8. On the next screen, select **Docker** as your environment
9. You'll see a command like:
   ```
   docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token eyJhGci...
   ```
10. **Copy the long token** (the part after `--token`) — it starts with `eyJ...`
11. Paste it into your `.env` file:
    ```
    CLOUDFLARE_TUNNEL_TOKEN=eyJhGci...your_full_token_here
    ```

---

## STEP 3 — Configure the Tunnel Public Hostname

Still in the Cloudflare tunnel setup page (after saving the connector):

1. Click the **"Public Hostname"** tab
2. Click **"Add a public hostname"**
3. Fill in:
   - **Subdomain**: leave blank or type something like `restaurant`
   - **Domain**: choose a free `*.trycloudflare.com` OR add your own domain later
   - **Service Type**: `HTTP`
   - **URL**: `caddy:80`  ← this is the name of the Caddy container inside Docker
4. Click **Save**

> 💡 If you don't have a domain yet, Cloudflare gives you a free URL like:
> `https://restaurant.yourdomain.workers.dev` or you can use the quick tunnel method below.

---

## STEP 4 — Start Everything

Open a terminal, navigate to this folder, and run:

```bash
docker compose up -d
```

That's it! Docker will:
1. Pull Caddy and Cloudflared images automatically
2. Start your website on port 8080 locally
3. Connect to Cloudflare's network via the tunnel

---

## STEP 5 — Visit Your Live Website

Go back to **Cloudflare → Zero Trust → Networks → Tunnels**
Click your tunnel → **Public Hostnames** tab
You'll see your public URL there — share it with anyone! 🎉

---

## 🚀 Quick Alternative (No Account Needed — For Testing Only)

If you just want a quick public URL RIGHT NOW without setting up an account:

1. Comment out the `cloudflared` service in `docker-compose.yml`
2. Start only Caddy: `docker compose up caddy -d`
3. Run this separately:
   ```bash
   docker run --rm -it --network schiegaarden_web cloudflare/cloudflared:latest tunnel --no-autoupdate --url http://caddy:80
   ```
4. Cloudflare will print a temporary `https://random-name.trycloudflare.com` URL — no login needed!
   (URL changes every restart — good for demos, not permanent)

---

## 🔧 Useful Commands

| Command | What it does |
|---|---|
| `docker compose up -d` | Start everything in background |
| `docker compose down` | Stop everything |
| `docker compose logs -f` | See live logs |
| `docker compose restart` | Restart all containers |
| `docker compose ps` | Check container status |

---

## 🌐 Adding Your Own Domain Later

When you buy a domain (e.g. `schiegaarden.no`):

1. Add it to Cloudflare (free DNS management)
2. Update the Public Hostname in your tunnel to use that domain
3. Your site will be live at `https://schiegaarden.no` with automatic HTTPS ✅

---

## 📝 Update Website Content

To update the website, just edit `site/index.html` — changes are reflected **instantly**
without restarting Docker (Caddy serves files live from the folder).

---

## 💬 Need Help?

If anything goes wrong, run:
```bash
docker compose logs cloudflared
docker compose logs caddy
```
And check the error messages there first.
