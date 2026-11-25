# Hosting neelimasharma.com on GitHub Pages

_As of 25 November 2026_

## Overview

GitHub Pages builds and serves the site from the `master` branch of `NeelimaSharma/neelimasharma.github.io`; Squarespace (the domain registrar) holds the DNS that points neelimasharma.com at GitHub. Netlify plays no part. Recreating the setup takes four steps: configure GitHub, set four A records and one CNAME at Squarespace, clear Netlify, then turn on HTTPS.

```
Visitor ──looks up the domain──▶ Squarespace DNS ──answers with GitHub's IPs──▶ GitHub Pages
types neelimasharma.com          4 A records + www CNAME                         serves the site over HTTPS
                                                                                        ▲
                                                                                        │ each git push rebuilds the site
                                                                                        │
                                                                               Repo, master branch
                                                                               neelimasharma.github.io
```

Every value you need, in one place:

| Where | Setting | Value |
| --- | --- | --- |
| GitHub repo | Name | `neelimasharma.github.io` |
| GitHub Pages | Source | Deploy from a branch: `master`, folder `/ (root)` |
| GitHub Pages | Custom domain | `neelimasharma.com` |
| Repo file | `CNAME` | `neelimasharma.com` (one line) |
| Repo file | `_config.yml` | `url: https://neelimasharma.com` |
| Squarespace DNS | A record, host `@` | `185.199.108.153` |
| Squarespace DNS | A record, host `@` | `185.199.109.153` |
| Squarespace DNS | A record, host `@` | `185.199.110.153` |
| Squarespace DNS | A record, host `@` | `185.199.111.153` |
| Squarespace DNS | CNAME record, host `www` | `neelimasharma.github.io` |
| GitHub Pages | Enforce HTTPS | On, once the certificate is issued |

## Step 1: GitHub

GitHub builds the Jekyll site on every push and serves it; these settings tell it which branch to build and which domain to answer for.

1. Create a public repository named exactly `<username>.github.io` (here `NeelimaSharma/neelimasharma.github.io`) and push the site to the `master` branch.
2. In the repo, open **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to *Deploy from a branch*, branch `master`, folder `/ (root)`, and click **Save**. The site is now live at `https://neelimasharma.github.io`.
4. Under **Custom domain**, type `neelimasharma.com` and click **Save**. GitHub commits a file named `CNAME` containing `neelimasharma.com` to the repo; pull it locally (`git pull`) so your next push doesn't conflict.
5. In `_config.yml`, set `url: https://neelimasharma.com` (no `www`). The menu builds its links from this value, so a `www` here sends every click through an extra redirect.
6. Commit and push. Each push triggers a rebuild that takes about a minute.

If the `CNAME` file is ever deleted from the repo, GitHub forgets the custom domain; re-enter it in step 4.

## Step 2: Squarespace DNS

The domain is registered with Squarespace Domains (which took over Google Domains), and its live DNS is served there. This is where the domain gets pointed at GitHub.

1. Sign in at [account.squarespace.com/domains](https://account.squarespace.com/domains), click **neelimasharma.com**, then **DNS** in the left menu. Squarespace may ask you to re-verify with Google before it lets you edit.
2. Under **Custom records**, click **Add record** once for each A record below, then **Save**:

   | Type | Host | Data |
   | --- | --- | --- |
   | A | `@` | `185.199.108.153` |
   | A | `@` | `185.199.109.153` |
   | A | `@` | `185.199.110.153` |
   | A | `@` | `185.199.111.153` |
   | CNAME | `www` | `neelimasharma.github.io` |

3. For `www`, edit the existing CNAME record (pencil icon) rather than adding a second one; a host can hold only one CNAME.
4. Delete any other A records on `@`, such as Netlify's `75.2.60.5` and `99.83.190.102`. A leftover record sends some visitors to the old host.
5. Leave these alone: the **Google Workspace** MX records (your email), the `_domainconnect` CNAME, and the `zn2qocrunoos` CNAME (a Google site-verification record).

Add the new records before deleting the old ones; that keeps the gap where the site is unreachable to a few minutes.

## Step 3: Netlify

Nothing in Netlify is needed for this setup; only tidy it so it can't claim the domain.

1. For any Netlify project that lists `neelimasharma.com` under **Domain management → Production domains**, remove the domain (and `www.neelimasharma.com`). Do this after Step 2 has taken effect, or the old site goes offline before the new one is reachable.
2. Netlify's own **DNS zone** for neelimasharma.com (team **Domains** page) is not in use, because the domain's name servers point to Squarespace, not Netlify. Editing records there changes nothing. It is empty and can be deleted or left.
3. Projects you no longer need can be deleted under **Project configuration → Configuration → Danger zone**. Today one project remains: `neelimasharma-html` (at neelimasharma-html.netlify.app), with no custom domain.

To check which DNS host is in charge, run `dig +short NS neelimasharma.com`: `googledomains.com` name servers mean Squarespace; `nsone.net` ones would mean Netlify DNS.

## Step 4: HTTPS and checks

GitHub issues a free certificate once it sees the DNS pointing at it, usually within an hour of Step 2.

1. In **Settings → Pages**, click **Check again** next to the custom domain until it shows *DNS check successful*.
2. When the **Enforce HTTPS** box becomes clickable, tick it. `http://` visits then redirect to `https://`.
3. Confirm from a terminal:

   ```
   dig +short A neelimasharma.com @8.8.8.8
   dig +short CNAME www.neelimasharma.com @8.8.8.8
   curl -sI http://neelimasharma.com | grep -i location
   ```

   Expected: the four `185.199.x.153` addresses, then `neelimasharma.github.io.`, then `Location: https://neelimasharma.com/`.

4. Open the site in a private window to avoid a cached copy of the old one.

## Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| GitHub says the domain is *improperly configured* | Its last DNS check ran before the records changed | Click **Check again**; if it persists, remove the custom domain, save, re-add it, save |
| **Enforce HTTPS** stays greyed out | Certificate not issued yet | Wait up to an hour, then **Check again**; the remove-and-re-add trick restarts the request |
| `https://www.` shows a certificate warning | The `www` certificate lags the main one | Usually clears within an hour; check the `www` CNAME is `neelimasharma.github.io` |
| Menu links go to `www.neelimasharma.com` | `_config.yml` `url` includes `www` | Set `url: https://neelimasharma.com` and push |
| Site shows the old Netlify version | A leftover Netlify A record, or a cached page | Delete other A records on `@`; hard refresh (Cmd+Shift+R) or use a private window |
| New fonts or styles don't appear | Browser cached the old stylesheet | Hard refresh (Cmd+Shift+R) |
| Email stops working | MX records were deleted in Step 2 | Restore the Google Workspace preset under **DNS Presets → Add preset** |
