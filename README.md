##Architecture diagram
<img width="3208" height="3221" alt="Talentflow-sequence-diagram-dark" src="https://github.com/user-attachments/assets/0340ec26-eea8-4d24-a4a4-d400ede892a2" />
# TalentFlow Application: Technical Documentation

## Overview

TalentFlow is a comprehensive HR management platform built using Next.js, React, and various other libraries.  The application features a protected area for managing jobs, candidates, and assessments, including an AI-powered assessment generation tool.  This document details the architecture and key workflows of the system, focusing on assessment saving and retrieval, candidate management via a Kanban board, and job reordering.

## Modules/Components

The application consists of several key modules:

* **`.gitignore`:**  Configures Git to ignore specific files and directories, maintaining a clean repository.
* **`README.md`:**  Provides project information and instructions for developers.
* **`components.json`:** Configures the `shadcn/ui` component library, defining styling and aliases.
* **`bun.lock`:** Records the exact versions of project dependencies, ensuring consistent builds.
* **`eslint.config.mjs`:** Defines ESLint rules and settings for code quality.
* **`package.json`:** Specifies project metadata, dependencies, and scripts.
* **`tsconfig.json`:** Configures the TypeScript compiler.
* **`public/file.svg`, `public/globe.svg`, `public/logo.svg`, `public/next.svg`, `public/vercel.svg`, `public/window.svg`:** SVG files containing various icons and logos.
* **`public/mockServiceWorker.js`:** Implements Mock Service Worker (MSW) for API mocking.
* **`next.config.ts`:** Configures Next.js build settings.
* **`postcss.config.mjs`:** Configures PostCSS for Tailwind CSS integration.
* **`lib/utils.ts`:** Provides utility functions, including CSS class name merging.
* **`src/app/(protected)/assessments/[id]/FormPreview.tsx`:** Renders and handles user input for dynamic forms.
* **`src/app/globals.css`:** Defines global CSS styles and theming.
* **`src/app/page.tsx`:** Implements the application's homepage.
* **`src/app/providers.tsx`:** Sets up application providers, including database seeding and MSW.
* **`src/app/(protected)/layout.tsx`:** Defines the layout for protected routes.
* **`src/app/(protected)/assessments/page.tsx`:** Displays and manages job listings.
* **`src/app/(protected)/assessments/[id]/page.tsx`:** Implements the form builder for assessments.
* **`src/app/(protected)/assessments/[id]/savedAssessments.tsx`:** Component for selecting saved assessments.
* **`src/app/(protected)/assessments/[id]/kiko.tsx` & `src/app/(protected)/assessments/[id]/kikoPromptModal.tsx`:** Components for AI-powered assessment generation.
* **`src/app/(protected)/assessments/[id]/publisher.tsx`:** Component for publishing and sharing assessments.
* **`src/app/(protected)/candidates/page.tsx`:** Manages a table of candidates.
* **`src/app/(protected)/candidates/addCandidateModal.tsx`:**  Modal for adding new candidates.
* **`src/app/(protected)/create/page.tsx`:** Form for creating new job postings.
* **`src/app/(protected)/jobs/[jobId]/page.tsx`:** Kanban board for managing candidates within a job.
* **`src/app/(protected)/jobs/[jobId]/modal.tsx`:** Help modal for the candidate pipeline.
* **`src/app/published/[id]/layout.tsx`:** Layout for published assessments.
* **`src/app/published/[id]/page.tsx`:** Fillable assessment form for published assessments.
* **`src/app/(protected)/global.css`:** Defines global CSS styles for the protected routes.
* **`src/app/(protected)/sidebar.tsx`:**  Defines the sidebar navigation component.
* **`src/app/(protected)/assessments/[id]/HelpModal.tsx`:** Help modal for assessment building.
* **`src/app/(protected)/dashboard/page.tsx`:** Recruitment dashboard visualizing candidate flow.
* **`src/app/(protected)/jobs/page.tsx`:**  Component for managing and filtering jobs.
* **`src/app/api/ai/route.ts`:** API endpoint for AI assessment generation.
* **`src/components/ui/*`:** Contains reusable UI components (Badge, Card, Button, CodeBlock, Input, Loader, Sidebar, Textarea, DropdownMenu, Label, Main-Spotlight, Stateful-Button, Tooltip).
* **`src/db/schema.ts`:** Defines the database schema.
* **`src/db/seed.ts`:** Seeds the database with mock data.
* **`src/mockApis/browser.ts`:** Sets up MSW for browser API mocking.
* **`src/mockApis/handle.ts`:** Defines MSW handlers for mock API routes.
* **`src/lib/gemini.ts`:**  Provides functions for interacting with the Google Gemini API.
* **`src/lib/utils.ts`:** Contains utility functions.


## Interactions

* **Database Interactions:**  `src/db/seed.ts` populates the database (`src/db/schema.ts`) with mock data.  The application components interact with the database directly through the `db` object.  `src/mockApis/handle.ts` mocks these interactions for development.
* **API Interactions:**  Frontend components make API calls (mocked in development via `src/mockApis/browser.ts` and `src/mockApis/handle.ts`). The `/api/ai` route (`src/app/api/ai/route.ts`) uses the Gemini API (`src/lib/gemini.ts`).
* **UI Component Interactions:**  Many UI components (`src/components/ui/*`) interact with each other and with application logic components. For example, the `FormPreview` component displays data structured by the `FormBuilder` component.
* **Protected Route Layout:** The `src/app/(protected)/layout.tsx` file provides a consistent layout for protected pages, including the sidebar (`src/app/(protected)/sidebar.tsx`).
* **Assessment Workflow:** The assessment creation and management flow involves `src/app/(protected)/assessments/[id]/page.tsx` (form builder), `src/app/(protected)/assessments/[id]/kiko.tsx` and `src/app/(protected)/assessments/[id]/kikoPromptModal.tsx` (AI generation),  `src/app/(protected)/assessments/[id]/publisher.tsx` (publishing), and `src/app/published/[id]/page.tsx` (published assessment rendering).

## Usage/Flow

### Assessment Save and Retrieval Flow

1. **Creation:** Assessments are created using the `FormBuilder` component (`src/app/(protected)/assessments/[id]/page.tsx`).  This component manages form data and allows adding/removing sections and questions.  The AI copilot functionality (`src/app/(protected)/assessments/[id]/kiko.tsx`, `src/app/(protected)/assessments/[id]/kikoPromptModal.tsx`) utilizes the `/api/ai` endpoint (`src/app/api/ai/route.ts`) which leverages the Google Gemini API (`src/lib/gemini.ts`) to generate assessment questions.
2. **Saving:**  The `handleSave` function within `FormBuilder` sends the form data to the `/mock/assessments` endpoint (mocked by `src/mockApis/handle.ts`).
3. **Retrieval:**  The `AssessmentPicker` component (`src/app/(protected)/assessments/[id]/savedAssessments.tsx`) fetches saved assessments from `/mock/assessments?jobId=${jobId}` (mocked by `src/mockApis/handle.ts`) and displays them in a dropdown.  Published assessments are retrieved and rendered by `src/app/published/[id]/page.tsx` from `/mock/assessments/${assessmentId}`.

### Kanban Board and Job Reordering Flow

1. **Kanban Board:** The `KanbanBoard` component (`src/app/(protected)/jobs/[jobId]/page.tsx`) displays a Kanban board for managing candidates.  It fetches candidate data based on the `jobId` from the route parameters (mocked via `/mock/candidates`).
2. **Drag and Drop:** React DnD (`@hello-pangea/dnd`) enables drag-and-drop reordering of candidates within stages and across stages on the Kanban board.
3. **Persistence:**  The `moveCardToColumn` and `removeCardFromBoard` functions update the local state, and the `updateCandidateStage` and `deleteCandidateAPI` functions persist changes by making API calls (mocked by `src/mockApis/handle.ts`).
4. **Job Reordering:** The `src/app/(protected)/jobs/page.tsx` component manages a list of jobs and utilizes `@hello-pangea/dnd` for drag-and-drop reordering.  The persistence mechanism for this reordering is not explicitly defined in the provided summaries.


## Deployment

The application is deployable to Vercel (as mentioned in `README.md`).


## External Dependencies

The application relies on several external libraries, including:

* Next.js
* React
* Radix UI
* Tailwind CSS
* TypeScript
* Dexie.js
* MSW
* `@hello-pangea/dnd`
* `class-variance-authority`
* `react-syntax-highlighter`
* `@tabler/icons-react`
* `motion`
* `@faker-js/faker`
* `clsx`
* `tailwind-merge`
* `react-hook-form`
* `lucide-react`
* `@google/generative-ai` (for Gemini API integration)


This documentation is based solely on the provided file summaries.  Further details may be available within the codebase itself.
