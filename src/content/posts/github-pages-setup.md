---
title: "GitHub Pages Setup"
published: 2025-09-13
description: "Step-by-step guide to set up and deploy a Fuwari blog with GitHub Pages."
image: "/assets/github-pages-setup/github-pages-setup.png"
tags: ["fuwari", "github-pages", "deployment"]
category: "Projects"
draft: false
lang: "en"
---
# 📘 ShadowNotes – Deploy Fuwari Blog to GitHub Pages

A quick, step-by-step guide to set up and deploy your **Fuwari Astro blog** on **GitHub Pages**.  

---

## ✅ Requirements

Before starting, ensure the following are installed on your system:

```bash
node 
npm 
pnpm 
```

---

## 🔧 Installation

Before starting, make sure **Node.js** and **NPM** are installed on your system.

### 1️⃣  Install Node.js and NPM

Go to the official Node.js [website](https://nodejs.org/en/download).

- Download the LTS version (recommended for most users)
- Run the installer and follow the prompts.

This automatically installs Node.js and NPM.

### Verify installation

```bash
node -v   # Should show version >= 20
npm -v    # Should show version >= 9
```

### 2️⃣ Install PNPM Globally

PNPM is a fast and efficient Node.js package manager used by Fuwari:

```bash
npm install -g pnpm
pnpm -v   # Verify PNPM installation

```

✅ Now your system is ready to install dependencies and run Fuwari.

---

## 🌍 Deploy on GitHub Pages  

### 1️⃣ Create Repo  

```bash
# On GitHub → New Repository → "blogs" (Public)
```

---

### 2️⃣ Clone Repo  

```bash
git clone https://github.com/username/blogs.git
cd blogs
```

---

### 3️⃣ Install Fuwari  

```bash
pnpm create fuwari@latest .
cd <created dir>
pnpm install
```

```bash
pnpm dev
```

👉 Open in browser:  [http://localhost:4321](http://localhost:4321)  

---

## 📝 Customize Your Blog  

- `src/config.ts` → Site config (site name, links, theme, etc.)
- `src/content/spec/about.md` → About page (edit your profile details)
- `astro.config.mjs` → Astro settings  (update site URL for GitHub Pages)
- `public/favicon/*` → Favicons (replace with your own logo)

---

## 🛠️ Useful pnpm Commands  

```bash
pnpm new-post my-first-post   # create new post
pnpm build && pnpm preview    # build for production
pnpm dev                      # run dev server
```

---

### 4️⃣ Update Astro Config  

`astro.config.mjs`  

```js
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  site: 'https://username.github.io/blogs', // GitHub Pages URL
  base: '/blogs/',                             // repo name
  integrations: [tailwind()],
});
```

---

### 5️⃣ GitHub Actions Workflow  

Create `.github/workflows/deploy.yml`  

```yaml
name: Deploy Astro Blog to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - uses: pnpm/action-setup@v2
        with:
          version: 9
      - run: pnpm install
      - run: pnpm build
      - uses: actions/upload-artifact@v4
        with:
          name: github-pages
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deploy.outputs.page_url }}
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: github-pages
          path: ./dist
      - id: deploy
        uses: actions/deploy-pages@v4
        with:
          artifact_name: github-pages
```

---

### 6️⃣ Build and Test Locally  

```bash
pnpm build && pnpm preview
```

---

### 7️⃣ Push to GitHub  

```bash
git init
git add .
git commit -m "Setup Fuwari Blog with GitHub Pages"
git checkout -b main
git remote add origin https://github.com/username/notes.git
git push origin main
```

---

### 8️⃣ Enable GitHub Pages

1. Go to your repository → **Settings** → **Pages**.
2. Under **Build and Deployment**, select:
   - **Source** → **GitHub Actions**
3. Click **Save** ✅

---

## 🎉 Result  

Your blog is live at:  

👉 <https://username.github.io/blogs>
