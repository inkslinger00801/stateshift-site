# Deploying state-shift.com with GitHub Pages

A complete walkthrough from "site files on disk" to "state-shift.com live over HTTPS." No prior GitHub Pages experience needed. Expect ~15 minutes of hands-on work plus ~1 hour of waiting for DNS and SSL.

## What you need

- A GitHub account (free at https://github.com/signup)
- The `website/` folder from this project
- Access to the DNS settings where you registered state-shift.com (registrar login)
- Terminal access

## 1. Create a GitHub repo

1. Go to https://github.com/new
2. Repo name: `stateshift-site` (or anything you like)
3. Visibility: **Public** — GitHub Pages requires public repos on the free tier
4. Do **not** initialize with a README, .gitignore, or license — we're pushing an existing folder
5. Click "Create repository"

GitHub will show you a page titled "Quick setup". Keep it open.

## 2. Push the site

Open a terminal and run:

```bash
cd "~/Library/Mobile Documents/com~apple~CloudDocs/Claude/State Shift App/State Shift App/website"

# Initialize a fresh repo in the website folder
git init
git add .
git commit -m "Initial site"

# Connect to GitHub (replace YOUR-USERNAME)
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/stateshift-site.git
git push -u origin main
```

If git asks for credentials and rejects your password, that's expected — GitHub requires a personal access token instead of your password. Create one at https://github.com/settings/tokens (classic, repo scope), then paste it when prompted.

Refresh the GitHub repo page. You should see `index.html`, `styles.css`, `privacy.html`, `support.html`, `terms.html`, `CNAME`, and `DEPLOY.md`.

## 3. Enable GitHub Pages

1. In the repo, click **Settings** (top nav)
2. In the left sidebar, click **Pages**
3. Under "Build and deployment":
   - Source: **Deploy from a branch**
   - Branch: **main** / root (`/`)
4. Click **Save**

GitHub will start building. In about a minute you'll see a banner:
> Your site is live at https://YOUR-USERNAME.github.io/stateshift-site/

Click that link and make sure the page loads. This is your site running on GitHub's default subdomain. Next step: point state-shift.com at it.

## 4. Configure DNS at your registrar

Log in wherever you registered state-shift.com (Namecheap, Google Domains, Squarespace, Porkbun, whoever). Find the DNS settings — usually called "DNS," "DNS Records," or "Name Servers → Advanced DNS."

**Add four A records for the apex domain (state-shift.com):**

| Type | Host / Name | Value            | TTL  |
|------|-------------|------------------|------|
| A    | @           | 185.199.108.153  | Auto |
| A    | @           | 185.199.109.153  | Auto |
| A    | @           | 185.199.110.153  | Auto |
| A    | @           | 185.199.111.153  | Auto |

(`@` means the root domain. Some registrars use a blank field or the domain itself.)

**Add one CNAME for www:**

| Type  | Host / Name | Value                           | TTL  |
|-------|-------------|---------------------------------|------|
| CNAME | www         | YOUR-USERNAME.github.io         | Auto |

(Replace `YOUR-USERNAME` with your actual GitHub username. Note: the value is **your username**, not the repo name — GitHub figures out the rest from the CNAME file we already added to the repo.)

If the registrar has existing records for `@` or `www` pointing somewhere else, delete them. Save changes.

## 5. Tell GitHub about the custom domain

1. Back in your repo: **Settings → Pages**
2. Under "Custom domain," enter: `state-shift.com`
3. Click **Save**

GitHub will verify the DNS. This takes anywhere from a few minutes to a few hours depending on how long DNS propagation takes at your registrar. You'll see a green check when it's verified.

## 6. Turn on HTTPS

Once the custom domain is verified, the **Enforce HTTPS** checkbox on the same Pages settings page becomes available. Check it.

GitHub will provision a free Let's Encrypt certificate for state-shift.com. This can take another 15 minutes to a couple of hours. You'll know it's done when https://state-shift.com loads without a certificate warning.

## 7. Verify

Check all four routes:

- https://state-shift.com
- https://state-shift.com/privacy.html
- https://state-shift.com/support.html
- https://state-shift.com/terms.html

And that `www.state-shift.com` redirects to the apex domain (it will, automatically, via the www CNAME).

## Updating the site later

To push changes:

```bash
cd "~/Library/Mobile Documents/com~apple~CloudDocs/Claude/State Shift App/State Shift App/website"
git add .
git commit -m "Update copy"
git push
```

GitHub Pages rebuilds in about a minute.

## Email forwarding

The site references `support@state-shift.com` and `privacy@state-shift.com`. You'll need email forwarding set up at your registrar so messages to those addresses reach an inbox you actually check. Most registrars offer free email forwarding in their DNS settings — look for "Email Forwarding" or "Mail Settings." Point both addresses to your personal email for now.

## Troubleshooting

**"Domain does not resolve to the GitHub Pages server" error in Pages settings**
DNS hasn't propagated yet. Wait 30 minutes and click the refresh button on the Pages settings page. If it's still failing after a few hours, double-check the A records are exactly the four addresses above.

**Site loads but over HTTP only, not HTTPS**
The SSL cert is still being provisioned. Wait another hour. If it's been more than 24 hours, try un-checking and re-checking the custom domain to force a reprovision.

**Site shows 404 after pushing a change**
GitHub Pages takes about a minute to rebuild. Refresh the Actions tab in your repo to watch the build status.

**CSS or fonts not loading**
Most common cause is a wrong path. All asset links in this site are relative (`styles.css`, not `/styles.css`) so they'll work both on github.io and on the custom domain.
