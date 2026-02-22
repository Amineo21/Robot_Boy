# AGENTS.md - Health Robot Project

This document provides guidance for AI coding agents working in this repository.

## Project Overview

Health Robot is an intelligent robot assistant system for nursing homes (EHPAD). The project is a monorepo containing:
- **frontend/health-robot-front**: TanStack Start React application
- **backend**: Python FastAPI server
- **infra**: Docker, Mosquitto MQTT broker, Nginx configuration

## Tech Stack

| Layer    | Technology                                           |
|----------|------------------------------------------------------|
| Frontend | TanStack Start, TanStack Router, React 19, TypeScript |
| Styling  | Tailwind CSS v4.1                                    |
| Build    | Vite 7, Nitro                                        |
| Backend  | FastAPI, Python 3.11, Uvicorn                        |
| Infra    | Docker, Docker Compose, Mosquitto (MQTT)             |

---

## Build/Lint/Test Commands

### Frontend (from `frontend/health-robot-front/`)

```bash
# Development
npm run dev           # Start dev server on port 3000

# Build
npm run build         # Production build
npm run preview       # Preview production build

# Testing (Vitest)
npm run test          # Run all tests once
npx vitest            # Run tests in watch mode
npx vitest run        # Run all tests once
npx vitest <file>     # Run single test file
npx vitest -t "test name"  # Run test by name pattern

# Examples for single test
npx vitest src/components/Header.test.tsx
npx vitest --run --reporter=verbose src/components/Header.test.tsx
```

### Backend (from `backend/`)

```bash
# Install dependencies
pip install -r requirements.txt

# Run server
uvicorn app.main:app --reload --port 4000

# Run with Docker
docker compose -f infra/docker-compose.yml up --build
```

### Infrastructure (from `infra/`)

```bash
docker compose up --build        # Start all services
docker compose down              # Stop all services
docker compose logs -f frontend  # Follow frontend logs
```

---

## Code Style Guidelines

### TypeScript/React (Frontend)

#### Import Order
```typescript
// 1. External packages
import { Link, useRouter } from '@tanstack/react-router'
// 2. React
import { useState, useEffect } from 'react'
// 3. Icons and UI libraries
import { Home, Menu, X } from 'lucide-react'
// 4. Local components
import Header from '../components/Header'
// 5. Styles
import appCss from '../styles.css?url'
```

#### Naming Conventions
| Type       | Convention   | Example                          |
|------------|--------------|----------------------------------|
| Components | PascalCase   | `Header.tsx`, `RobotStatus.tsx`  |
| Functions  | camelCase    | `getRouter`, `handleClick`       |
| Routes     | lowercase    | `index.tsx`, `__root.tsx`        |
| Constants  | UPPER_SNAKE  | `API_URL`, `MAX_RETRIES`         |
| Types      | PascalCase   | `RobotState`, `UserProps`        |

#### Component Structure
```typescript
// Route definition first (TanStack Router pattern)
export const Route = createFileRoute('/path')({
  component: ComponentName,
})

// Component function
function ComponentName() {
  // 1. Hooks and state
  const [state, setState] = useState(initial)
  
  // 2. Derived data / computations
  const computed = useMemo(() => ..., [deps])
  
  // 3. Event handlers
  const handleClick = () => { ... }
  
  // 4. JSX return
  return (
    <div className="...">
      ...
    </div>
  )
}
```

#### TypeScript Guidelines
- Use strict mode (enabled in tsconfig.json)
- Prefer explicit return types for exported functions
- Use path aliases: `@/*` maps to `./src/*`
- Avoid `any` - use `unknown` and narrow types
- Props typing inline for simple components: `{ children }: { children: React.ReactNode }`

#### React Patterns
- Functional components only (no class components)
- Use TanStack Router's `createFileRoute` for route definitions
- Default exports for components
- Named exports for route configurations

### Python (Backend)

```python
# FastAPI endpoint pattern
@app.get("/endpoint")
async def endpoint_name():
    """Docstring describing the endpoint."""
    return {"key": "value"}
```

- Use snake_case for functions and variables
- Type hints for function parameters and returns
- Async functions for I/O operations

---

## Styling Guidelines (Tailwind CSS)

- Use utility classes directly in JSX
- Responsive prefixes: `sm:`, `md:`, `lg:`, `xl:`
- Dark theme palette: slate-800/900 backgrounds
- Avoid inline styles - use Tailwind utilities
- Group related utilities logically

---

## Error Handling

### Frontend
```typescript
try {
  const result = await apiCall()
} catch (error) {
  if (error instanceof SpecificError) {
    // Handle specific error
  }
  console.error('Context:', error)
  // Show user-friendly message
}
```

### Backend
```python
from fastapi import HTTPException

@app.get("/resource/{id}")
async def get_resource(id: str):
    resource = await fetch_resource(id)
    if not resource:
        raise HTTPException(status_code=404, detail="Resource not found")
    return resource
```

---

## Git Conventions

### Branch Naming
```
feature/<feature-name>    # New features
fix/<issue-description>   # Bug fixes
docs/<doc-topic>          # Documentation
```

### Commit Messages
```
[FEAT]: Add robot navigation component
[FIX]: Resolve MQTT connection timeout
[DOCS]: Update API documentation
[REFACTOR]: Simplify state management
[TEST]: Add unit tests for Header component
```

---

## File Structure

```
frontend/health-robot-front/
├── src/
│   ├── components/    # Reusable React components
│   ├── routes/        # TanStack Router file-based routes
│   ├── router.tsx     # Router configuration
│   └── styles.css     # Global Tailwind styles
├── public/            # Static assets
└── package.json
```

---

## Important Notes

1. **Generated Files**: Do NOT edit `routeTree.gen.ts` - it's auto-generated by TanStack Router
2. **Port Configuration**: Frontend: 3000 (dev), Backend: 4000, MQTT: 1883/9001
3. **Testing**: Uses Vitest with @testing-library/react and jsdom
4. **Path Aliases**: Use `@/` instead of relative paths (e.g., `@/components/Header`)

---

## Environment Variables

Create `.env` files based on these templates:

### Frontend (.env)
```
VITE_API_URL=http://localhost:4000
VITE_MQTT_URL=ws://localhost:9001
```

### Backend (.env)
```
MQTT_BROKER=mqtt://localhost:1883
```

---

## Testing Guidelines

### Test File Naming
- Component tests: `ComponentName.test.tsx`
- Utility tests: `utilName.test.ts`
- Place tests next to source files or in `__tests__/` directory

### Test Structure
```typescript
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import Component from './Component'

describe('Component', () => {
  it('renders correctly', () => {
    render(<Component />)
    expect(screen.getByText('Expected Text')).toBeInTheDocument()
  })
})
```
