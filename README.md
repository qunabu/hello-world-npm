# hello-world-npm

Hello World. I'm curious how does npm publishing with CI/CD works

## GitHub Actions Workflows

This repository uses GitHub Actions for automated testing and npm publishing with GitHub OpenID Connect (OIDC) authentication.

### 1. Tests Workflow (`.github/workflows/tests.yml`)

**Triggers:**
- Push to `main` branch
- Push to `release/**` branches
- Pull requests to any branch

**Purpose:**
- Runs automated tests to ensure code quality before publishing
- Triggers the pre-release publishing workflow when tests complete successfully

**Jobs:**
- `test`: Runs basic validation tests

### 2. Publish Workflow (`.github/workflows/publish.yml`)

This workflow handles npm publishing with two distinct jobs based on the trigger event.

#### Job: `pre-release`

**Triggers:**
- Manual workflow dispatch (`workflow_dispatch`)
- After `Tests` workflow completes successfully on:
  - `main` branch
  - `release/**` branches (excluding `release/**-**` patterns)

**Purpose:**
- Publishes development/preview versions to npm
- Uses commit SHA for versioning

**Process:**
1. Checks out the code
2. Sets up Node.js 20
3. Generates a version using the format: `0.0.0-{short-sha}` (e.g., `0.0.0-a1b2c3d`)
4. Updates `package.json` version
5. Publishes to npm with the `develop` tag

**npm Tag:** `develop`

#### Job: `publish`

**Triggers:**
- When a GitHub Release is created (`release` event with type `created`)

**Purpose:**
- Publishes official releases to npm
- Automatically updates `package.json` version to match the release tag
- Publishes with appropriate npm tags based on release settings

**Process:**
1. Checks out the code
2. Sets up Node.js 20
3. Extracts version from the release tag (removes `v` prefix if present)
4. Updates `package.json` and `package-lock.json` with the extracted version
5. Commits and pushes the version update back to the release target branch
6. Determines if this is the latest release by checking GitHub API
7. Publishes to npm with appropriate tag:
   - **Pre-release**: If release is marked as pre-release → `prerelease` tag
   - **Latest release**: If this is the latest release → `latest` tag
   - **Previous release**: If not the latest → `previous` tag

**npm Tags:**
- `latest` - For the most recent stable release
- `previous` - For older releases
- `prerelease` - For pre-release versions

## Authentication

This project uses **GitHub OpenID Connect (OIDC)** for npm authentication instead of traditional tokens. This provides:
- Enhanced security (no long-lived tokens to manage)
- Automatic token rotation
- Provenance information for published packages

**Setup Required:**
1. Configure OIDC trust relationship in npm (see [npm trusted publishers documentation](https://docs.npmjs.com/trusted-publishers))

## Important Notes

- **Single Trusted Publisher**: Due to [npm's restriction](https://docs.npmjs.com/trusted-publishers#supported-cicd-providers) that each package can only have one trusted publisher configured at a time, all publishing must be set up in the same workflow file (`.github/workflows/publish.yml`)

- **Automatic Version Updates**: Package.json version is updated automatically on the selected target branch during release publishing. **Do not update it manually** for releases.

- **Pre-release Deprecation**: When a pre-release is published, it should ideally be deprecated automatically, but this is currently **not possible with OIDC** on npm. According to [npm's documentation](https://docs.npmjs.com/trusted-publishers#limitations-and-future-improvements), OIDC authentication is currently limited to the `publish` operation only. Other npm commands like `deprecate` still require traditional authentication methods.

- **Release Tagging**: 
  - When "Set as the latest release" is selected in GitHub → published with `latest` tag
  - When "Set as a pre-release" is selected → published with `prerelease` tag
  - Older releases → published with `previous` tag 
  - Branch `develop`  → published with `develop` tag 