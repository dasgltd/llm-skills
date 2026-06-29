---
name: firebase-spark-deploy-workaround
description: Use this skill whenever the user wants to deploy a web app (Next.js, Vite, React, Angular) to Firebase but is stuck on the free "Spark" plan, or gets an error saying "Blaze plan is required". Also trigger this whenever the user wants to set up CI/CD or GitHub Actions for Firebase Hosting without using App Hosting or Cloud Functions. This provides a workaround to avoid paying for Firebase.
---

# Firebase Spark Plan Deploy Workaround

## Why this skill exists
Firebase heavily promotes **Firebase App Hosting** and **Firebase Web Frameworks** (which power the default `firebase deploy` for Next.js/Angular/React). However, both of these modern solutions automatically provision **Cloud Functions** underneath to handle SSR and image optimization. 

Because Cloud Functions are exclusively available on the **Blaze (pay-as-you-go) plan**, users on the free **Spark plan** will encounter an error when trying to deploy, blocking their progress.

When the user wants continuous deployment (CI/CD) via GitHub but refuses to upgrade to Blaze, we must circumvent App Hosting entirely.

## The Workaround Strategy
Instead of dynamic App Hosting, we force a static build and deploy it to **Firebase Hosting (Classic)** using **GitHub Actions**. This bypasses all Cloud Function requirements while still giving the user automated "git push" deployments.

### Execution Steps:

**1. Force a Static Build**
We must ensure the framework outputs plain HTML/CSS/JS.
- **For Vite/React:** Usually ready by default. The output directory is `dist`.
- **For Next.js:** We MUST instruct the user/modify the code to edit `next.config.js` and set `output: 'export'`. The output directory will be `out`.

**2. Initialize Classic Hosting**
Run the following in the terminal:
```bash
npx -y firebase-tools@latest init hosting
```
- When asked for the public directory, use the correct static folder (`dist` for Vite, `out` for Next.js).
- Configure as a single-page app (rewrites to `index.html`) if it's a typical React/Vite app.

**3. Setup GitHub Actions (The "Indirect GitHub Deploy")**
To provide the CI/CD experience without App Hosting, run:
```bash
npx -y firebase-tools@latest init hosting:github
```
- Authenticate and select the GitHub repository.
- Ensure the generated YAML (`.github/workflows/firebase-hosting-merge.yml`) uses the exact build script (e.g., `npm ci && npm run build`).

### What this accomplishes
By completing these steps, the user gets automated deployments on every `git push` directly to Firebase Hosting. Because we generate static files and use GitHub Actions for the build step, no Cloud Functions are provisioned, and **the deployment succeeds on the free Spark plan**.
