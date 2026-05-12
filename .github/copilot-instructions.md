# Copilot Coding Agent Instructions for twentyone Repository

## Repository Overview

This is a full-stack employee directory application with a React frontend and Express backend. The repository is a **monorepo** with two independent projects:
- **Client**: React 18 + TypeScript + Vite (SPA with React Router)
- **Server**: Express + TypeScript (REST API)

**Size**: ~713 lines of code
**Languages**: TypeScript, JavaScript
**Target Runtime**: Node.js (18.x, 20.x, 22.x supported)

## Project Architecture

### Directory Structure
```
/
├── .github/
│   ├── workflows/
│   │   └── node.js.yml          # CI/CD pipeline
│   └── dependabot.yml
├── client/                       # Frontend React application
│   ├── src/
│   │   ├── pages/               # React pages/components
│   │   ├── services/            # API service layer
│   │   ├── App.tsx              # Main app component with routing
│   │   ├── main.tsx             # Application entry point
│   │   └── setupTests.ts        # Test setup
│   ├── package.json
│   ├── tsconfig.json            # TypeScript config (noEmit: true)
│   ├── vite.config.ts           # Vite config with vitest setup
│   ├── eslint.config.js         # ESLint flat config
│   └── index.html
└── server/                       # Backend Express application
    ├── src/
    │   ├── server/
    │   │   ├── index.ts         # Server entry point
    │   │   └── app.ts           # Express app setup
    │   ├── controllers/         # Request handlers
    │   ├── routes/              # Route definitions
    │   └── middlewares/         # Express middlewares
    ├── package.json
    ├── tsconfig.json            # TypeScript config (builds to dist/)
    ├── eslint.config.js         # ESLint flat config
    ├── jest.config.js           # Jest test config
    ├── babel.config.cjs         # Babel config for Jest
    └── .prettierrc              # Prettier config
```

**Important**: There is NO root package.json. Client and server are completely independent.

### Key Application Details
- Client runs on port 5173 (dev) and proxies `/api/v1` to `http://localhost:3000`
- Server runs on port 3000 (configurable via PORT env var)
- Server fetches employee data from JSONPlaceholder API
- Client uses Vite for dev server and building
- Server uses tsx for development and tsc for building

## Build & Test Instructions

### CRITICAL: Always follow these sequences exactly as documented

### Client (React + Vite)

#### Install Dependencies
**IMPORTANT**: Client uses `npm install` (NOT `npm ci`) in CI
```bash
cd client
npm install
```
**Time**: ~5-10 seconds

#### Lint
```bash
cd client
npm run lint
```
Uses ESLint with TypeScript, React hooks, and React refresh plugins.
**Time**: ~1-2 seconds
**Config**: `eslint.config.js` (flat config format)

#### Test
```bash
cd client
npm run test
```
Uses Vitest with jsdom environment. Tests run with globals enabled.
**Time**: ~1 second
**Config**: `vite.config.ts` (contains test config)
**Test Files**: `*.test.tsx` in src/

#### Build
```bash
cd client
npm run build
```
Runs `tsc -b && vite build`. TypeScript compilation checks types (noEmit: true), then Vite builds the bundle.
**Time**: ~1-2 seconds
**Output**: `dist/` directory
**Artifacts to ignore**: `*.tsbuildinfo`, `dist/`

#### Dev Server
```bash
cd client
npm run dev
```
Starts Vite dev server on `http://localhost:5173`

### Server (Express)

#### Install Dependencies
**IMPORTANT**: Server uses `npm ci` in CI (different from client!)
```bash
cd server
npm ci
```
**Time**: ~3-10 seconds
**Warnings Expected**: Several deprecation warnings for @types packages (these are normal)

#### Lint
```bash
cd server
npm run lint
```
Uses ESLint with TypeScript. Custom rules enforce spacing, quotes (single), indentation (2 spaces).
**Time**: ~1-2 seconds
**Config**: `eslint.config.js` (flat config format)

#### Test
```bash
cd server
npm run test
```
Uses Jest with ts-jest and babel-jest transformers.
**Time**: ~1 second
**Config**: `jest.config.js` and `babel.config.cjs`
**Test Files**: `*.test.ts` in src/

#### Build
```bash
cd server
npm run build
```
Runs `npm run clean && tsc`. Cleans dist/ first, then compiles TypeScript.
**Time**: ~1-2 seconds
**Output**: `dist/` directory with compiled JavaScript and source maps
**Artifacts to ignore**: `dist/`

#### Start (Production)
```bash
cd server
npm start
```
Runs the TypeScript source directly using tsx (no build needed for dev)

#### Dev Server
```bash
cd server
npm run dev
```
Uses nodemon with tsx for hot reloading

## Continuous Integration

### GitHub Actions Workflow
**File**: `.github/workflows/node.js.yml`

The CI pipeline runs TWO separate jobs (server and client) in parallel:

#### Server Job
Node versions: 18.x, 20.x, 22.x
```bash
cd server
npm ci              # Install dependencies
npm run lint        # Lint code
npm run test        # Run tests
npm run build       # Build TypeScript
```

#### Client Job
Node versions: 18.x, 20.x, 22.x
```bash
cd client
npm install         # Install dependencies (NOT npm ci!)
npm run lint        # Lint code
npm run test        # Run tests
npm run build       # Build with Vite
```

### CRITICAL DIFFERENCE
- **Server**: Uses `npm ci` in CI
- **Client**: Uses `npm install` in CI
Always match this in your local testing to replicate CI behavior.

## Common Pitfalls & Solutions

### Build Failures

**Problem**: TypeScript compilation fails
- **Solution**: Run `npm run lint` first to catch TypeScript errors early
- **Note**: Both projects use strict TypeScript settings

**Problem**: Client build creates `*.tsbuildinfo` files
- **Solution**: Already in `.gitignore` - these are TypeScript incremental build artifacts

**Problem**: Tests fail after dependency changes
- **Solution**: Delete `node_modules` and reinstall:
  - Server: `rm -rf node_modules && npm ci`
  - Client: `rm -rf node_modules && npm install`

### Linting Issues

**Server Specific**:
- Single quotes required (not double)
- 2-space indentation (enforced)
- Semicolons required
- Specific spacing rules around operators, functions, objects

**Client Specific**:
- React component export checks
- React hooks rules enforced
- JSX-specific linting

### Testing Issues

**Client Tests**:
- Vitest runs in jsdom environment
- Setup file: `src/setupTests.ts`
- Console output during tests is normal (not an error)

**Server Tests**:
- Jest uses babel-jest and ts-jest
- Tests run in Node environment
- Mock data is expected

## Code Change Workflow

### Making Changes

1. **Identify the project**: Determine if you're working in client/ or server/
2. **Install dependencies** (if fresh clone):
   - Server: `cd server && npm ci`
   - Client: `cd client && npm install`
3. **Make your changes**
4. **Lint immediately**: `npm run lint` in the appropriate directory
5. **Run tests**: `npm run test` in the appropriate directory
6. **Build**: `npm run build` in the appropriate directory
7. **Verify**: Check that no unintended files were created in dist/ or node_modules/

### Testing Your Changes Locally

Always replicate the CI workflow exactly:

**For Server Changes**:
```bash
cd server
npm ci
npm run lint
npm run test
npm run build
```

**For Client Changes**:
```bash
cd client
npm install
npm run lint
npm run test
npm run build
```

### Environment Variables

Both projects use `.env` files (already in `.gitignore`):
- Server: PORT, VITE_APP_CLIENT_PORT
- Client: VITE_APP_API_URL

Default values are set in code, so `.env` files are optional.

## Code Style Guidelines

### TypeScript
- Strict mode enabled in both projects
- No implicit any
- No unused locals/parameters (client only)

### Server Code Style
- Single quotes for strings
- 2-space indentation
- Semicolons required
- Spaces around operators
- Space in object curly braces: `{ key: value }`
- No space in array brackets: `[1, 2, 3]`

### Client Code Style
- Follows React best practices
- Functional components with hooks
- React Router v7 for routing

## Additional Notes

- **No Docker**: This project doesn't use containerization
- **No Monorepo Tools**: No Lerna, Nx, or Turborepo - projects are managed independently
- **Dependabot**: Configured for npm, Docker (none present), and GitHub Actions
- **Node Version**: CI tests on 18.x, 20.x, 22.x; local development uses 20.x
- **API Endpoints**: Server proxies to JSONPlaceholder API for demo data
- **CORS**: Configured in server to allow client origin

## Quick Command Reference

| Action | Client | Server |
|--------|--------|--------|
| Install | `npm install` | `npm ci` |
| Lint | `npm run lint` | `npm run lint` |
| Test | `npm run test` | `npm run test` |
| Build | `npm run build` | `npm run build` |
| Dev | `npm run dev` | `npm run dev` |
| Start | `npm run preview` | `npm start` |

## Final Reminders

1. **Always work in the correct directory** (client/ or server/)
2. **Match CI behavior**: npm ci for server, npm install for client
3. **Lint before you commit**: Catches most issues early
4. **Check .gitignore**: Don't commit node_modules/, dist/, .env, or *.tsbuildinfo
5. **Run the full CI sequence locally** before assuming your changes will pass

**Trust these instructions**. Only search or explore further if you find these instructions incomplete or incorrect for your specific task.
