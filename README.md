# Interactive Blog Statistics

An interactive educational blog about how messy data becomes interpretable evidence.

Live deployment: [[https://interactive-blog-statistics.onrender.com](https://aaron-zeller.github.io/statistics-blog.io/)]([https://interactive-blog-statistics.onrender.com](https://aaron-zeller.github.io/statistics-blog.io/))

## Overview

This project is a guided, explorable statistics blog built for a course context in educational technology. It combines:

- a narrative, section-by-section learning flow
- interactive visual explanations
- a pre-assessment and post-assessment
- an optional learner feedback questionnaire
- simple backend persistence for collected responses

The goal is not just to present statistical ideas, but to let learners move through them in a structured way: first checking prior understanding, then working through the blog, then revisiting the same ideas in a post-assessment.

The current version is designed to support:

- local preview and iteration during development
- deployment on Render
- storage of assessment and feedback data either locally or in Postgres
- export of collected results for later analysis

## Educational Purpose

The blog is designed as a didactic explorable explanation around common topics in introductory statistics and data interpretation. It focuses on issues such as:

- reading distributional shape from plots
- understanding the relationship between summary statistics and data shape
- interpreting p-values and confidence intervals
- checking when statistical assumptions are plausible
- choosing between common test types
- understanding multiple testing and false positives

The structure is intentionally staged:

1. learners complete a pre-assessment
2. they move through the blog one act at a time
3. they complete a post-assessment using the same ten questions
4. they can optionally leave structured feedback about the learning experience
5. they see a brief wrap-up page with key takeaways

## Main Features

### Guided learning flow

The blog is split into acts and scenes, and only one main section is shown at a time. This reduces cognitive overload and keeps the pacing closer to a guided teaching sequence than to a long scrolling article.

### Pre- and post-assessment

The blog includes:

- one pre-assessment before the content
- one post-assessment after the content

Both assessments use the same ten questions so the results are directly comparable. The backend stores:

- total score
- total number of questions
- a boolean correctness array for each question
- submission timestamps

### Optional learner feedback

After the post-assessment, learners can optionally complete a short five-question feedback questionnaire. This collects high-level impressions of the blog's:

- engagement
- interactivity
- clarity
- pacing
- effect on learner confidence

### Data export

Collected data can be exported as JSON or CSV through a protected export endpoint.

### Render-ready backend

The backend can run:

- locally with file storage
- on Render with Postgres via `DATABASE_URL`

## Tech Stack

- Frontend: plain HTML, CSS, and JavaScript
- Backend: Node.js HTTP server, no framework
- Database: local JSON file for development, Postgres for deployment
- Hosting: Render
- Package manager: Yarn

This project intentionally stays lightweight. There is no frontend build step and no framework-specific runtime.

## Project Structure

```text
interactive-blog-statistics/
|- index.html
|- main.js
|- server.js
|- package.json
|- yarn.lock
|- LICENSE
|- README.md
|- data/
   |- assessment-results.json
```

### Important files

- [`index.html`](/Users/aaronzeller/Documents/FS26/Design%20in%20Educational%20Technology/Interactive%20Blog/interactive-blog-statistics/index.html)
  Main page markup and styling. This contains the blog structure, assessment panels, section layout, and page-level CSS.

- [`main.js`](/Users/aaronzeller/Documents/FS26/Design%20in%20Educational%20Technology/Interactive%20Blog/interactive-blog-statistics/main.js)
  Frontend behavior. This includes:
  - assessment rendering and grading
  - chart drawing and interaction logic
  - guided act navigation
  - optional feedback questionnaire logic
  - dynamic UI updates across the blog

- [`server.js`](/Users/aaronzeller/Documents/FS26/Design%20in%20Educational%20Technology/Interactive%20Blog/interactive-blog-statistics/server.js)
  Simple Node server for:
  - serving the static app
  - saving assessment results
  - saving optional feedback
  - exporting stored data

- `data/assessment-results.json`
  Local development storage when Postgres is not configured.

## Running the Project Locally

### Requirements

- Node.js
- Yarn 1.x

### Install dependencies

```bash
yarn
```

### Start the local server

From the project root:

```bash
yarn start
```

By default, the app runs at:

- [http://localhost:8000](http://localhost:8000)

The server uses:

- `PORT` if provided
- otherwise `8000`

Example:

```bash
PORT=5000 yarn start
```

## Data Storage Behavior

The backend supports two storage modes.

### 1. Local file storage

If `DATABASE_URL` is not set, the app stores results in:

```text
data/assessment-results.json
```

This mode is useful for local testing and development.

### 2. Postgres storage

If `DATABASE_URL` is present, the app automatically uses Postgres.

This is the expected production setup on Render.

The default table name is:

```text
assessment_results
```

This can be overridden with:

```bash
POSTGRES_TABLE=your_table_name
```

## Environment Variables

### Core runtime

- `PORT`
  Port for the local or hosted server.

- `DATABASE_URL`
  Enables Postgres storage when present.

- `DATABASE_SSL`
  Set to `true` if your Postgres connection requires SSL.

- `POSTGRES_TABLE`
  Optional override for the Postgres table name.

- `ASSESSMENT_STORAGE`
  Optional explicit backend selection:
  - `file`
  - `postgres`

If not set, the server auto-detects:

- `postgres` when `DATABASE_URL` exists
- `file` otherwise

### Export

- `ASSESSMENT_EXPORT_TOKEN`
  Required to use the export endpoint.

## API Endpoints

### Save or update assessment state

```http
POST /api/assessment
```

Used by both the pre- and post-assessment.

### Fetch a participant record

```http
GET /api/assessment?userId=...
```

Used by the frontend to restore an existing learner state.

### Save optional blog feedback

```http
POST /api/feedback
```

Stores the optional learner feedback in the same participant record.

### Export all records

```http
GET /api/assessment/export?token=YOUR_TOKEN
GET /api/assessment/export?token=YOUR_TOKEN&format=csv
```

The export endpoint returns the full stored dataset, including:

- pre-assessment score
- post-assessment score
- per-question correctness arrays
- optional feedback responses
- timestamps

## Exporting Results

### JSON export

```bash
curl "https://interactive-blog-statistics.onrender.com/api/assessment/export?token=YOUR_TOKEN"
```

### CSV export

```bash
curl "https://interactive-blog-statistics.onrender.com/api/assessment/export?token=YOUR_TOKEN&format=csv"
```

Replace `YOUR_TOKEN` with the value of `ASSESSMENT_EXPORT_TOKEN`.

## Data Model

Each participant record can contain:

- `userId`
- `createdAt`
- `updatedAt`
- `preScore`
- `preTotal`
- `preCorrect`
- `preSavedAt`
- `postScore`
- `postTotal`
- `postCorrect`
- `postSavedAt`
- `feedbackResponses`
- `feedbackSavedAt`

This allows the project to store the whole learner journey in one record:

- initial assessment
- final assessment
- optional experience feedback

## Deployment on Render

This project is currently deployed on Render.

### Recommended setup

- Web service for the Node app
- Render Postgres for persistent storage

### Typical configuration

Build command:

```bash
yarn
```

Start command:

```bash
yarn start
```

Recommended environment variables:

```bash
DATABASE_URL=...
ASSESSMENT_EXPORT_TOKEN=...
```

Optional:

```bash
DATABASE_SSL=true
POSTGRES_TABLE=assessment_results
```

### Important note

Do not rely on local file storage in production hosting environments. Use Postgres on Render so collected results are persistent.

## Notes for Development

### No build system

This project intentionally does not use a frontend bundler. That keeps the structure simple for course work and makes the deployed app easy to inspect.

### Runtime data

The `data/` folder is used for local runtime storage. It should not be treated as source code.

### Frontend architecture

The frontend is still a single-file JavaScript implementation in `main.js`. That keeps it straightforward to deploy, but it also means most frontend logic currently lives in one place.

## Why this project exists

This project was created as part of a course context in design and educational technology. It is meant to demonstrate:

- interactive teaching design
- explorable explanation techniques
- integration of assessment into a learning experience
- collection of structured learner response data

In other words, it is both:

- a teaching artifact
- and a small educational web application

## License

This repository includes a [`LICENSE`](/Users/aaronzeller/Documents/FS26/Design%20in%20Educational%20Technology/Interactive%20Blog/interactive-blog-statistics/LICENSE) file. See that file for the full license text.
