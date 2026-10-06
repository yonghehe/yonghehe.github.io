# Publishing the OAuth app: the app-domain requirement

You are here because Google requires **App domain** details (home page, privacy policy, verified authorized
domain) before an app that declares sensitive or restricted scopes can be moved from **Testing** to
**In production**. Gmail scopes are restricted, so the fields are mandatory.

Why it matters: a project whose consent screen is in Testing is issued refresh tokens that expire after
7 days. Moving to In production lifts that, which is the difference between a working daily mail digest and
re-authorizing every week.

Two facts that make this cheap for you:

- **No verification review is needed.** Google documents a personal-use exception: when you are the only
  user, or the only users are people you know personally, you can publish unverified and click through the
  "unverified app" screen. You do not need the Trust and Safety review, a demo video, or a CASA security
  assessment.
- **What you do need is a public page on a domain you can prove you own.** Any static host with a
  verifiable domain works. This guide uses GitHub Pages, which is free and gives you
  `<your-handle>.github.io`, a registrable domain (github.io is on the Public Suffix List, so
  `your-handle.github.io` is treated as its own site).

Budget: about 15 minutes.

---

## Step 0: files

This folder holds:

- `index.html` - the app home page (what the app does, who uses it, link to the policy)
- `privacy.html` - the privacy policy, including the Limited Use disclosure Google looks for

Both must end up served from the same domain. Edit the app name in `index.html` if the **App name** in your
Cloud Console branding page is something other than "Hermes Mail Digest"; they must match.

## Step 1: put them online (GitHub Pages)

*(Your GitHub handle appears to be `yonghehe` from your notification mail. Substitute yours if different.)*

1. Create a **public** repository named exactly `yonghehe.github.io`.
2. Commit `index.html` and `privacy.html` to the root of the default branch.
3. Repository Settings, then Pages. Under Build and deployment set Source to **Deploy from a branch**,
   branch `main`, folder `/ (root)`. Save.
4. Wait a minute or two, then confirm both pages load:
   - `https://yonghehe.github.io/`
   - `https://yonghehe.github.io/privacy.html`

## Step 2: prove ownership in Search Console

Google will not accept an authorized domain that you have not verified as yours.

1. Go to `https://search.google.com/search-console` while signed in as
   `chanyonghweehi@gmail.com` (this account must be an Owner or Editor on the Cloud project).
2. **Add property**, choose the **URL prefix** type, and enter `https://yonghehe.github.io/`.
3. Verify using either method:
   - **HTML tag**: copy the `<meta name="google-site-verification" ...>` line into the `<head>` of
     `index.html`, commit, wait for Pages to rebuild, then click Verify.
   - **HTML file**: download the verification file, commit it to the repo root, then click Verify.
4. Confirm the property shows as verified.

## Step 3: fill the Branding page

Google Cloud Console, **Google Auth Platform**, **Branding**:

| Field | Value |
|---|---|
| App name | must match the name shown on your home page |
| User support email | `chanyonghweehi@gmail.com` |
| Developer contact email | `chanyonghweehi@gmail.com` |
| Application home page | `https://yonghehe.github.io/` |
| Application privacy policy link | `https://yonghehe.github.io/privacy.html` |
| Terms of service link | optional, leave blank |
| Authorized domains | `yonghehe.github.io` |

Save. The status should go to **Ready to publish** after the automated branding check (usually a few
minutes).

## Step 4: publish

On the same page click **Publish app**, then confirm the **Audience** page reads **In production**.

## Step 5: re-authorize (do not skip this)

The refresh token you hold today was issued while the app was in **Testing**, so it is still on the 7-day
clock and dies around 13 October. Publishing does not retroactively extend it. Once the Audience page says
In production, run the authorization flow again.

In this workflow that means: generate a fresh URL with

```
/home/hermes/.hermes/hermes-agent/venv/bin/python \
  /home/hermes/.hermes/skills/productivity/google-workspace/scripts/setup.py --auth-url
```

approve it in the browser, hand back the redirect URL, and the exchange is done as before. Expect the
"Google hasn't verified this app" warning; click Advanced, then Go to <app> (unsafe). That screen is
documented behaviour for an unverified app using restricted scopes, and it does not reappear once consent
has been granted.

To confirm the fix held, check the token a week later with:

```
/home/hermes/.hermes/hermes-agent/venv/bin/python \
  /home/hermes/.hermes/skills/productivity/google-workspace/scripts/setup.py --check
```

`AUTHENTICATED` on or after 13 October means the 7-day clock is gone.

---

## If GitHub Pages does not work out

- **Search Console refuses the github.io property**: some accounts have trouble with hosted subdomains. A
  real domain (about SGD 15 per year, Cloudflare or Namecheap) pointed at GitHub Pages, Cloudflare Pages,
  or Netlify works the same way and is the more conventional answer.
- **The Console rejects the authorized domain**: same fallback, a domain you registered.
- **You would rather not publish at all**: the app can stay in Testing. That costs a re-authorization every
  7 days forever. Workable, but it needs a watchdog, because the failure is quiet: the job keeps posting a
  brief that says it could not read any mail. A daily `setup.py --check` monitor that pings you when auth
  breaks removes the ten-day blind spot either way.

## What publishing does not change

- Consent on the owner's account still shows the unverified-app screen once.
- Google's Security Checkup may list the app as unverified or risky. Expected, and harmless for a
  single-user personal tool.
- Unverified apps are capped at 100 new users, which is irrelevant here.
- A future Google password change on an account with Gmail scopes can still invalidate the refresh token.
  Only the 7-day expiry goes away.
