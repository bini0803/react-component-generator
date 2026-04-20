# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**React 컴포넌트 생성기** - An AI-powered React component generator that creates components from prompts with real-time preview. Users can generate, customize, and interact with components instantly using either Anthropic Claude or Google Gemini APIs.

## Architecture

### Frontend (React + Vite)
- **Main App**: `src/App.tsx` - Manages UI state, provider selection, API key input, and component list
- **Components**:
  - `PromptInput`: Form for entering component descriptions with example prompts
  - `ComponentCard`: Displays generated component with live preview, code view, and actions
  - `LivePreview`: Renders component code using react-live in an isolated scope
  - `CodeView`: Syntax-highlighted code display with copy button
- **Hook**: `useComponentGenerator` - Handles /api/generate calls, manages component history, error handling
- **Type**: `Provider = 'anthropic' | 'google'` defines AI provider selection

### Backend (Bun Server)
- **Single file**: `server/index.ts` running on port 3002
- **Key endpoints**:
  - `GET /api/config` - Returns which API keys are configured in .env
  - `POST /api/generate` - Accepts `{prompt, apiKey?, provider}`, returns generated component code
- **Providers**:
  - Anthropic: Uses `claude-haiku-4-5-20251001` with 4096 max_tokens
  - Google: Uses `gemini-2.5-flash` with 8192 max_tokens
- **API Key Resolution**: Prefers client-provided key over .env (via `resolveApiKey`)
- **Code Processing**:
  - `stripCodeFences`: Removes markdown code block syntax
  - `ensureRenderCall`: Adds `render(<ComponentName />)` if missing

### System Prompt Constraints
Located at top of `server/index.ts` - critical for component generation:
- **Output Format**: Plain JavaScript only (no TypeScript syntax, no type annotations)
- **Styling**: Inline styles only, no CSS imports or CSS modules
- **Scope**: React global + react-live render function available
- **Pattern**: Define component function, then call `render(<ComponentName />)` at end
- **Dependencies**: No import statements - React is pre-scoped by react-live
- **Code Quality**: Responsive design, interactive states, modern color palettes, descriptive names

## Common Development Commands

```bash
# Install dependencies
bun install

# Run API server + Vite dev server concurrently (default port 5173)
bun run dev

# Watch and rebuild Bun server only
bun run server

# Type check + build for production
bun run build

# Lint with ESLint
npm run lint
```

## Key Files & Responsibilities

| File | Purpose |
|------|---------|
| `server/index.ts` | Bun server, API routing, provider proxying, code cleanup |
| `src/App.tsx` | Main UI container, provider/API key state, component orchestration |
| `src/hooks/useComponentGenerator.ts` | Fetch wrapper, component storage, error handling |
| `src/components/LivePreview.tsx` | Executes untrusted code safely via react-live Scope |
| `src/components/ComponentCard.tsx` | Component display card with preview, code, and actions |

## Important Implementation Details

### Code Generation & Execution
- **Generated code is untrusted** - executed at runtime inside `LivePreview` using `react-live`
- `react-live` Scope includes: React namespace, render function, common utilities
- No ability to modify `window` or access unsafe APIs from generated code
- All examples in system prompt must be valid within this scope

### Multi-Provider Logic
- Frontend: Stores selected provider in state, sends to backend with each request
- Backend: Routes to `callAnthropic` or `callGoogle` based on request
- API Key sources (in order of precedence):
  1. Client-provided key (UI input)
  2. Server-side .env variable
  3. Error if neither exists

### Error Handling
- Network errors from AI APIs show user-friendly Korean messages
- 503 (overload) and 429 (rate limit) get specific messages
- MAX_TOKENS from Gemini suggests simpler component
- Parse errors from code cleanup logged to console but don't block

### Styling & Appearance
- Latest commit: `style(ui): 레트로 8-bit 디자인 전체 리스타일` - 8-bit retro aesthetic
- Uses CSS custom properties for consistency
- See `src/App.css` for design token definitions

## Development Workflow

1. **Feature changes in components**: Test via `bun run dev` at http://localhost:5173
2. **Server logic changes**: Changes auto-reload due to `--watch` flag
3. **System prompt changes**: Modify string at top of `server/index.ts`, test via UI
4. **New providers**: Add function like `callAnthropic`, provider option to enum, endpoint routing

## Testing the Application

- Manual testing via browser at http://localhost:5173 is primary validation
- No automated test suite present - verify component renders, code execution, API calls
- Test both providers (Anthropic/Google) with various prompts
- Verify error states: missing API key, malformed code, API failures
- Test provider switching behavior (clears UI input correctly)

## Relevant External Documentation

- **react-live**: Uses `<LiveProvider>`, `<LiveEditor>`, `<LivePreview>`, `<Scope>` for code execution
- **Bun**: Server runtime with native fetch and Node compatibility; see `server/index.ts` for Bun.serve API
- **Vite**: Frontend build tool configured via `vite.config.ts` (not present, using defaults)
