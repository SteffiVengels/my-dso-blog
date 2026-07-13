# Docusaurus Blog

This page documents how I set up and personalized this blog based on the developer-akademie-starter template.

## Overview

The project is a learning journal / portfolio website built with [Docusaurus](https://docusaurus.io/). It started from a template repository, which I cloned, configured, and personalized. All changes were made on a feature branch called `setup-blog` and merged into `main` via a pull request.

## Setup Steps

### 1. Local setup

- Cloned the repository to my local machine and opened it in VS Code
- Installed all dependencies with `npm install`
- Started the local development server with `npm start`
- Created my local environment configuration by copying the example file: `cp example.env .env`

### 2. Environment variables

- Adjusted the values in the environment configuration to match my GitHub account and repository (`DEPLOYMENT_URL`, `BASE_URL`, `GITHUB_ORG`, `GITHUB_PROJECT`)
- Added a new variable `GIT_REPOSITORY_URL` containing the URL of this repository
- Created a TypeScript variable in `docusaurus.config.ts` that reads `GIT_REPOSITORY_URL` from the environment and falls back to a default value, following the pattern of the existing `blogEnabled` variable

### 3. docusaurus.config.ts

- Changed `title` and `tagline` to reflect that this is my personal learning journal
- Updated the `url` to match my GitHub username
- Changed the `editUrl` in both the docs and the blog configuration to use the new `GIT_REPOSITORY_URL` variable
- Personalized the navbar: new title and corrected GitHub repository link
- Adjusted the footer:
  - Added a link to the projects page (`/docs/projects`) in the Docs column
  - Removed the Community column
  - Replaced the GitHub link in the More column with a link to my repository and added a link to the template repository labeled "Template"
  - Personalized the copyright message and extended it with the required snippet

### 4. README

- Replaced the manual deployment documentation with a short description of the automatic deployment via GitHub Actions
- Removed the Contributing section and all references to the removed NGINX/Docker deployment
- Switched the documented commands from pnpm to npm to match the actual workflow
- Described all remaining files and folders in the Repository Structure section

### 5. Cleanup

- Removed the Docker-related files (`Dockerfile`, `.dockerignore`, the NGINX deployment guide) since deployment is handled exclusively by GitHub Actions

### 6. Deployment

- Enabled GitHub Pages in the repository settings with **GitHub Actions** as the source
- The site is now built and deployed automatically by the preconfigured workflow whenever a commit is pushed to the main branch

## Challenges

- A misconfigured `url` value (it contained a sub-path) prevented the development server from starting. The fix was to split the value correctly: the plain domain goes into `url`, the repository sub-path into `baseUrl`.
- The variable for `GIT_REPOSITORY_URL` initially converted the value into a boolean, which caused a TypeScript error at `editUrl`. Reading the value as a string with a default fallback solved it.