# Deployment & DevOps Guide: SafeOrbit

## 1. Deployment

### 1.1. Deploying Frontend to Vercel

1.  Connect the GitHub repository to a new Vercel project.
2.  Configure the build settings:
    *   **Framework Preset:** `Vite`
    *   **Build Command:** `pnpm build`
    *   **Output Directory:** `dist`
3.  Add the necessary environment variables for the Supabase URL and anon key.
4.  Vercel will automatically deploy the frontend on every push to the `main` branch.

### 1.2. Provisioning Supabase Project

1.  Create a new project in the Supabase dashboard.
2.  Save the project's URL, anon key, and service role key.
3.  Use the SQL editor in the Supabase dashboard to run the database schema migrations.

### 1.3. Configuring Environment Variables

The following environment variables need to be configured in both the Vercel and Supabase projects:

*   `SUPABASE_URL`
*   `SUPABASE_ANON_KEY`
*   `SUPABASE_SERVICE_ROLE_KEY`
*   `CLOUDFLARE_API_TOKEN`
*   `BUNNY_API_KEY`

## 2. CI/CD Strategy

The CI/CD pipeline will be managed through GitHub Actions and will be designed to work seamlessly with AI-assisted development tools.

*   **On Pull Request:**
    *   Run linting and unit tests.
    *   A preview deployment will be created by Vercel.
*   **On Merge to `main`:**
    *   The same linting and testing jobs will run.
    *   If the jobs pass, the production frontend will be deployed to Vercel.
    *   Database migrations will be run against the production Supabase project.
*   **AI-Assisted Development:**
    *   AI tools will be used to generate code and propose changes via pull requests.
    *   All pull requests, whether from a human or an AI, will require a human review and approval before being merged.
