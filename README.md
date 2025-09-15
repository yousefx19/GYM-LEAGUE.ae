#!/usr/bin/env bash
set -e

# === EDIT THESE BEFORE RUNNING ===
GITHUB_OWNER="your-github-username-or-org"   # e.g. "yasser" or "my-org"
REPO_NAME="gym-league.ae"                   # GitHub repo name
REPO_VISIBILITY="public"                    # public or private
EMAIL="you@example.com"
# ================================

echo "Creating local repo structure for ${REPO_NAME}..."

# create folders
mkdir -p "${REPO_NAME}"
cd "${REPO_NAME}"

mkdir -p server client .github/workflows

# --------------------------
# Root files: README, .gitignore, LICENSE, CONTRIBUTING
# --------------------------
cat > README.md <<'README'
# GYM LEAGUE.ae

GYM LEAGUE.ae — fitness progress tracking web app.

## Overview
- Users can track workouts, progress, body measurements.
- Coaches can form teams, challenge other gyms.
- Minimal viable product (MVP) contains:
  - Authentication (email/password)
  - User profile & measurements
  - Workout logging & progress charts
  - Leaderboards / Teams

## Tech stack (suggested)
- Frontend: React + Vite (or Create React App)
- Backend: Node.js + Express
- Database: PostgreSQL (or MongoDB)
- CI/CD: GitHub Actions

## Quick start (local)
1. `cd server && npm install && npm run dev`
2. `cd ../client && npm install && npm run dev`

## Contributing
See CONTRIBUTING.md
README

cat > .gitignore <<'.GITIGNORE'
# Node
node_modules/
npm-debug.log
.env
.DS_Store

# Logs
logs
*.log

# Build
dist/
build/

# Editor
.vscode/
.idea/
.GITIGNORE

cat > LICENSE <<'LICENSE'
MIT License

Copyright (c) $(date +%Y) ${GITHUB_OWNER}

Permission is hereby granted, free of charge, to any person obtaining a copy
... (standard MIT license text continues) ...
LICENSE

cat > CONTRIBUTING.md <<'CONTRIB'
# Contributing to GYM LEAGUE.ae

Thanks for your interest! Please:
- Open issues for bugs and feature requests.
- Follow the branching model: `main` for stable, feature branches `feat/<name>`.
- Run tests before PRs.
CONTRIB

# --------------------------
# Server: basic Express API with a sample endpoint
# --------------------------
cat > server/package.json <<'PKGSRV'
{
  "name": "gym-league-server",
  "version": "0.1.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"No tests yet\""
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}
PKGSRV

cat > server/index.js <<'INDEXJS'
/**
 * Basic Express server scaffold
 * Extend with routes, auth, DB connection
 */
const express = require('express');
const app = express();
const port = process.env.PORT || 4000;

app.use(express.json());

// sample health-check
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', app: 'GYM LEAGUE.ae', env: process.env.NODE_ENV || 'dev' });
});

// sample user route (placeholder)
app.get('/api/users/sample', (req, res) => {
  res.json({ id: 1, name: 'Sample User', role: 'athlete' });
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
INDEXJS

# --------------------------
# Client: minimal React app scaffold (using Vite-like structure)
# --------------------------
cat > client/package.json <<'PKGCLIENT'
{
  "name": "gym-league-client",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "echo \"No tests yet\""
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.4.0"
  },
  "devDependencies": {
    "vite": "^5.0.0"
  }
}
PKGCLIENT

mkdir -p client/src
cat > client/index.html <<'HTML'
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>GYM LEAGUE.ae</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
HTML

cat > client/src/main.jsx <<'MAIN'
import React from "react";
import { createRoot } from "react-dom/client";

function App() {
  return (
    <div style={{fontFamily: "system-ui, sans-serif", padding: 20}}>
      <h1>GYM LEAGUE.ae</h1>
      <p>Welcome — scaffold ready. Connect this client to the server API at <code>/api</code>.</p>
    </div>
  );
}

createRoot(document.getElementById("root")).render(<App />);
MAIN

# --------------------------
# GitHub Actions workflow for Node.js CI
# --------------------------
cat > .github/workflows/nodejs.yml <<'YML'
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x]

    steps:
      - uses: actions/checkout@v4
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - name: Install server dependencies
        working-directory: ./server
        run: npm ci
      - name: Run server tests
        working-directory: ./server
        run: npm test
      - name: Install client dependencies
        working-directory: ./client
        run: npm ci
      - name: Build client
        working-directory: ./client
        run: npm run build
      - name: Run client tests
        working-directory: ./client
        run: npm test
YML

# --------------------------
# Optional: sample ENV template
# --------------------------
cat > server/.env.example <<'ENV'
# Example environment variables
PORT=4000
DATABASE_URL=postgres://user:pass@localhost:5432/gym_league
JWT_SECRET=replace-with-secure-secret
ENV

# --------------------------
# Initialize git and commit
# --------------------------
git init
git config user.email "${EMAIL}"
git config user.name "${GITHUB_OWNER}"

git add .
git commit -m "chore: init gym-league.ae scaffold (server + client + CI)"

# --------------------------
# Optionally create GitHub repo and push
# --------------------------
if command -v gh >/dev/null 2>&1; then
  echo "Found gh (GitHub CLI). Creating remote repo ${GITHUB_OWNER}/${REPO_NAME} (${REPO_VISIBILITY})..."
  gh repo create "${GITHUB_OWNER}/${REPO_NAME}" --${REPO_VISIBILITY} --source=. --remote=origin --push || {
    echo "gh repo create failed - maybe repo exists or auth issue"
  }
else
  echo "gh CLI not found. Skipping automatic remote creation. You can create a repo manually and run:"
  echo "  git remote add origin git@github.com:${GITHUB_OWNER}/${REPO_NAME}.git"
  echo "  git branch -M main"
  echo "  git push -u origin main"
fi

echo "Done. Next steps:"
echo "  cd ${REPO_NAME}/server && npm install && npm run dev"
echo "  cd ../client && npm install && npm run dev"
echo ""
echo "Edit README.md, set up DB, and add authentication & tests."
 # GYM-LEAGUE.ae
