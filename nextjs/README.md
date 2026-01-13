## Docker File Overview

This project uses a **multi-stage Docker build** optimized for **Next.js standalone output**, **pnpm**, and **production security**.

### Key goals
- Deterministic dependency installs
- Fast rebuilds with Docker layer caching
- Minimal production image size
- Non-root runtime for security
- Consistent behavior in local development and CI

---

## Dockerfile Structure

The Dockerfile is split into **four stages**, each with a single responsibility.

### 1. Base stage (`base`)
- Uses `node:20-alpine`
- Shared base image for all stages to ensure consistency

---

### 2. Dependencies stage (`deps`)
- Installs required system dependencies (`libc6-compat`)
- Enables Corepack and uses **pnpm** as the package manager
- Installs Node.js dependencies using a frozen lockfile for reproducibility
- Copies only `package.json` and `pnpm-lock.yaml` to maximize Docker cache reuse

---

### 3. Build stage (`builder`)
- Reuses `node_modules` from the `deps` stage
- Copies the full application source code
- Builds the Next.js application using `pnpm run build`
- Requires `output: "standalone"` in `next.config.js`

---

### 4. Production stage (`runner`)
- Contains only the minimal runtime output required to run the app
- Copies:
  - `public/`
  - `.next/standalone`
  - `.next/static`
- Runs the application as a non-root user
- Exposes port `3000`
- Starts the server with `node server.js`

---

## Package Manager

- Uses **pnpm** via **Corepack**
- pnpm version is pinned in `package.json` using:
  ```json
  "packageManager": "pnpm@<version>"
