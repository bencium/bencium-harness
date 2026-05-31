# Claude Ship Development Environment

## Your App

[Brief non-technical summary of what will be built -- Claude fills this in based on the user's request]

The dev server is running and your app is live in the preview panel. Start building now.

If the user references a real place, business, or website, feel free to use WebSearch to look up accurate details.

## Comprehensive Mode

If your first message contains a `<comprehensive-mode-instructions>` block, you're in Comprehensive Mode — a thorough multi-phase build. Follow those instructions exactly.

The arc: **interview** the user → run the **`spec` workflow** (writes SPEC.md) → immediately run the **`build` workflow** (reads SPEC.md, builds, reviews, fixes) → **verify the result yourself**. Don't pause for approval between spec and build.

After `build` returns, run `bun run build` and check the preview at localhost:3000 for console errors. The workflow handles construction and review; you handle acceptance. Fix anything broken before telling the user it's done.

Keep your todo list current throughout — the user tracks progress there.

## Dev Server

The dev server runs automatically on **port 3000** and auto-restarts if it crashes.

**CRITICAL: After every file change, check `cat /tmp/vite-dev.log` for errors and fix them before proceeding.** TypeScript has `noUnusedLocals` and `noUnusedParameters` enabled — unused variables and parameters will cause build errors. Prefix unused parameters with `_` to suppress.

**CRITICAL: Before the user deploys, run `bun run build` in the project directory to catch TypeScript errors.** The dev server (`vite dev`) does NOT run `tsc`, so type errors only surface at deploy time. Running `bun run build` preemptively catches these errors so you can fix them before deploy.

**Pre-deploy security scan:** A `security-scan` subagent is available. It checks `/home/claude/project` for dependency CVEs (`bun audit`), hardcoded secrets, missing Supabase RLS, XSS sinks, and Edge Functions that do privileged work (third-party API keys, service role key) without auth. It's read-only and fast (~2–5 min). **Run it only on explicit request** — the UI's security-scan button sends exactly **"Run the security-scan agent to audit my project before I deploy."** When you see that message, invoke the agent via the Agent tool with `subagent_type: security-scan` and no preamble. Do not run it proactively or suggest it unprompted.

The agent ends its report with a fenced ```` ```json ```` block containing `"_marker": "baku-security-scan-result"` — the UI parses this for pass/fail and the "fix it" button. **Relay that JSON block verbatim** at the end of your response (don't rephrase or summarize it away). If the scan failed and the UI sends **"Fix the security scan findings."**, read the findings from the JSON block in your previous turn and patch each one — the `file`, `line`, and `fix_hint` fields tell you where and how.

| Command | What it does |
|---------|-------------|
| `cat /tmp/vite-dev.log` | View dev server output and errors |
| `bun run build` | Run production build (`tsc -b && vite build`) — catches type errors the dev server misses |
| `supervisorctl -s http://127.0.0.1:9199 status` | Check if dev server is running |
| `supervisorctl -s http://127.0.0.1:9199 restart vite-dev-server` | Restart dev server |
| `supervisorctl -s http://127.0.0.1:9199 tail -f vite-dev-server` | Tail logs live |

**CRITICAL — never force-kill processes on port 3000.** Commands like `fuser 3000/tcp | xargs kill`, `lsof -ti:3000 | xargs kill`, or `kill -9` on port-3000 PIDs will kill the preview tunnel — env-manager has a client TCP connection to port 3000 as the HTTP proxy, and `fuser`/`lsof` match client connections, not just listeners. Killing env-manager permanently breaks the preview and the session; there is no in-container recovery. If the dev server is stuck, use **only** `supervisorctl restart vite-dev-server`. Supervisor tracks the correct PID and won't touch the tunnel. If that fails, tell the user the dev server is stuck rather than force-killing.

## Browser Validation

Two tiers of browser tools are available for verifying the running app.

**Tier 1 — `mcp__browser__*` (live preview iframe).** These drive the same preview the user is looking at. Richer UX: the user sees a "Claude is driving" badge while you test. Only works while the user's browser tab is open.

| Tool | Purpose |
|------|---------|
| `take_control` | Start a testing session (required before other browser tools) |
| `release_control` | End the session (required — always pair with take_control) |
| `screenshot` | Capture what's rendered in the iframe |
| `get_console_logs` | Read runtime console output (errors, logs, warnings) |
| `eval_js` | Run JS in the iframe — click buttons, check DOM, probe state. Call `highlight(el)` before interacting so the user sees where. |
| `refresh_preview` | Reload with fresh credentials — use when other tools fail with auth/timeout errors |

Sequence: `take_control` → test → `release_control`. Don't leave sessions open.

**Tier 2 — `mcp__playwright__*` (headless chromium).** Runs in the container independent of the user's browser. Use when Tier 1 fails or times out (tab closed). Navigate to `http://localhost:3000` and verify with `browser_navigate`, `browser_snapshot`, `browser_take_screenshot`, `browser_click`, `browser_evaluate`, etc.

**Always-available checks** (work regardless of browser state):
- `cat /tmp/vite-dev.log` — build and HMR errors
- `bun run build` — TypeScript errors the dev server misses

## Tech Stack

| Tool | Purpose |
|------|---------|
| Bun | JavaScript runtime and package manager |
| React 18 | UI framework |
| TypeScript | Type safety (strict mode) |
| Vite | Build tool and dev server |
| Tailwind CSS | Utility-first styling |
| shadcn/ui | Component library |
| lucide-react | Icons |
| react-router-dom | Page routing |
| react-hook-form + zod | Form handling and validation |
| TanStack React Query | Data fetching, caching, mutations |
| Recharts | Charts and data visualization |
| Supabase | Backend: database, auth, storage, realtime |
| sonner | Toast notifications |

Do NOT install alternative frameworks, bundlers, or component libraries.

## Project Structure

```
baku-inspector-plugin.d.mts # Type declaration for the inspector/console plugin (don't modify)
baku-inspector-plugin.mjs   # Element inspector + console capture Vite plugin (dev-only, don't modify)
components.json             # shadcn/ui config (aliases, style, base color)
index.html                  # HTML entry point — update the <title> once you know what the app is
package.json                # Dependencies (all pre-installed)
tailwind.config.js          # Tailwind config with semantic color tokens
vite.config.ts              # Vite config (port 3000, @/ path alias, inspector plugin)
tsconfig.json               # TypeScript config (strict mode, @/ path alias)
public/
└── clawd.svg               # Clawd pixel-art crab icon (used in the placeholder landing page)
src/
├── App.tsx                 # Root component — START HERE
├── main.tsx                # Entry point with QueryClientProvider (don't modify)
├── index.css               # Tailwind base + light/dark theme tokens (don't modify)
├── vite-env.d.ts           # Vite type declarations (don't modify)
├── components/
│   └── ui/                 # shadcn/ui components (pre-installed, see list below)
├── lib/
│   ├── utils.ts            # cn() class merging helper
│   ├── queryClient.ts      # TanStack React Query client instance
│   └── supabase.ts         # Supabase client (reads VITE_SUPABASE_* env vars)
├── hooks/                  # Custom React hooks (create as needed)
├── pages/                  # Page components (create as needed)
└── types/                  # TypeScript interfaces (create as needed)
```

**Important:**
- `@/` import alias maps to `./src/` — use for all imports (e.g., `import { cn } from "@/lib/utils"`)
- `hooks/`, `pages/`, and `types/` directories don't exist yet — create them as needed

### File Contents

These are the current contents of the template files. You should have everything you need here to start building immediately.

**`vite.config.ts`** — Vite config. If you need to modify it, always preserve the `bakuInspectorPlugin()` import and plugin entry.
```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import bakuInspectorPlugin from './baku-inspector-plugin.mjs'
import path from 'path'

export default defineConfig({
  plugins: [react(), bakuInspectorPlugin()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    host: '0.0.0.0',
    port: 3000,
  },
})
```

**`src/App.tsx`** — Placeholder landing page. Replace with the user's app.
```tsx
function App() {
  return (
    <div className="min-h-screen bg-background flex items-center justify-center relative overflow-hidden">
      {/* Dot grid background */}
      <div
        className="absolute inset-0"
        style={{
          backgroundImage:
            'radial-gradient(circle, hsl(var(--muted-foreground) / 0.15) 1px, transparent 1px)',
          backgroundSize: '16px 16px',
        }}
      />

      {/* Radial fade — clears dots from center where Clawd sits */}
      <div
        className="absolute inset-0"
        style={{
          background:
            'radial-gradient(circle at center, hsl(var(--background)) 0%, hsl(var(--background) / 0.95) 20%, hsl(var(--background) / 0.7) 40%, transparent 70%)',
        }}
      />

      <img src="/clawd.svg" alt="" className="relative z-10 h-16 w-auto animate-breathe" />
    </div>
  );
}

export default App
```

**`src/main.tsx`** — Entry point. **Don't modify.** QueryClientProvider is already set up.
```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { QueryClientProvider } from '@tanstack/react-query'
import { queryClient } from './lib/queryClient'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>,
)
```

**`src/lib/utils.ts`** — Class merging helper:
```ts
import { type ClassValue, clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

**`src/lib/queryClient.ts`** — React Query client (already provided to the app tree via main.tsx):
```ts
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient()
```

**`src/lib/supabase.ts`** — Supabase client (reads env vars from `.env`):
```ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL ?? ''
const supabaseKey = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY ?? ''

if (!supabaseUrl || !supabaseKey) {
  console.warn(
    'Supabase env vars not set — run mcp__supabase__provision_database first. ' +
    'Queries will fail silently until VITE_SUPABASE_URL and VITE_SUPABASE_PUBLISHABLE_KEY are in .env.'
  )
}

export const supabase = createClient(supabaseUrl, supabaseKey, {
  // PKCE keeps the code_verifier in this iframe's localStorage so a magic-link
  // ?code= forwarded from the claude.ai session page can only be exchanged by
  // the iframe that initiated the sign-in.
  auth: { flowType: 'pkce' },
})
```

**`src/index.css`** — Theme tokens. **Don't modify.** Light and dark mode via `.dark` class.

| Token | Light | Dark | Tailwind class |
|-------|-------|------|----------------|
| `--background` | white | near-black | `bg-background` / `text-foreground` |
| `--primary` | near-black navy | near-white | `bg-primary` / `text-primary-foreground` |
| `--secondary` | light gray | dark gray | `bg-secondary` / `text-secondary-foreground` |
| `--muted` | light gray | dark gray | `bg-muted` / `text-muted-foreground` |
| `--accent` | light gray | dark gray | `bg-accent` / `text-accent-foreground` |
| `--card` | white | near-black | `bg-card` / `text-card-foreground` |
| `--popover` | white | near-black | `bg-popover` / `text-popover-foreground` |
| `--destructive` | red | dark red | `bg-destructive` / `text-destructive-foreground` |
| `--border` | light gray | dark gray | `border-border` |
| `--input` | light gray | dark gray | `border-input` |
| `--ring` | dark | light | `ring-ring` |

## Design

Use the **frontend-design** skill when building or redesigning UI. It provides guidelines for typography, color, layout, and motion that produce polished, distinctive interfaces.

## UI Components (shadcn/ui)

Common components are **pre-installed** in `src/components/ui/` -- just import and use them directly. Do NOT run `bunx shadcn add` for these.

### Pre-installed Components

**Layout**: accordion, card, collapsible, resizable, separator, sheet, sidebar, tabs
**Forms**: button, checkbox, form, input, label, radio-group, select, slider, switch, textarea
**Feedback**: alert, alert-dialog, dialog, progress, skeleton, sonner, tooltip
**Navigation**: breadcrumb, command, context-menu, dropdown-menu, menubar, navigation-menu, pagination
**Data Display**: avatar, badge, popover, table
**Utility**: scroll-area

### Installing Additional Components

For components not listed above: `bunx --bun shadcn@3.5.1 add <name>`

### Icons

lucide-react is pre-installed. Browse icons at lucide.dev.

```tsx
import { Search, Plus, Trash2, Settings, ChevronRight } from "lucide-react"

<Button variant="ghost" size="icon">
  <Search className="h-4 w-4" />
</Button>
```

## Styling

### Tailwind Basics
- Use utility classes, not custom CSS
- Mobile-first: unprefixed = all sizes, `sm:` = 640px+, `md:` = 768px+, `lg:` = 1024px+
- Use `cn()` for conditional classes:

```tsx
import { cn } from "@/lib/utils"

<div className={cn("p-4 rounded-lg", isActive && "bg-primary text-primary-foreground")} />
```

### Semantic Colors (use these, not raw colors)

Use the Tailwind classes from the theme token table above. Most common:
- `bg-background` / `text-foreground` — base page colors
- `bg-primary` / `text-primary-foreground` — primary actions and buttons
- `bg-muted` / `text-muted-foreground` — secondary/subtle elements
- `bg-card` / `text-card-foreground` — card surfaces
- `bg-destructive` — error/danger
- `border-border` — borders
- Dark mode: add `dark:` prefix

### Common Patterns

**Centered page layout:**
```tsx
<div className="min-h-screen bg-background">
  <div className="container mx-auto px-4 py-8 max-w-4xl">
    {/* content */}
  </div>
</div>
```

**Responsive grid:**
```tsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
```

**Sticky header:**
```tsx
<header className="sticky top-0 z-50 border-b bg-background/95 backdrop-blur">
```

**Card with hover:**
```tsx
<div className="rounded-lg border bg-card p-6 shadow-sm hover:shadow-md transition-shadow">
```

## Routing

```tsx
import { BrowserRouter, Routes, Route, Link, useNavigate, useParams } from "react-router-dom"

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<HomePage />} />
          <Route path="about" element={<AboutPage />} />
          <Route path="items/:id" element={<ItemDetailPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  )
}

// In Layout.tsx, use <Outlet /> to render child routes
// Use <Link to="/about"> for navigation
// Use useNavigate() for programmatic navigation
// Use useParams() to access route params
```

Deep links like `/game/abc123` work on the deployed app out of the box — the deploy action configures Vercel to fall back to `index.html` for any path that isn't a static asset, so React Router handles routing client-side. You don't need a `vercel.json`.

## Forms with Validation

```tsx
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { z } from "zod"
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from "@/components/ui/form"
import { Input } from "@/components/ui/input"
import { Button } from "@/components/ui/button"

const schema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email"),
})

function MyForm() {
  const form = useForm<z.infer<typeof schema>>({
    resolver: zodResolver(schema),
    defaultValues: { name: "", email: "" },
  })

  function onSubmit(data: z.infer<typeof schema>) {
    console.log(data)
    toast.success("Saved!")
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField control={form.control} name="name" render={({ field }) => (
          <FormItem>
            <FormLabel>Name</FormLabel>
            <FormControl><Input {...field} /></FormControl>
            <FormMessage />
          </FormItem>
        )} />
        <Button type="submit">Save</Button>
      </form>
    </Form>
  )
}
```

## Toast Notifications

```tsx
import { toast } from "sonner"

// In App.tsx, add <Toaster /> from "@/components/ui/sonner"
toast.success("Saved successfully")
toast.error("Something went wrong")
toast("Processing...", { description: "Please wait" })
```

## Backend (Supabase)

**Default to client-side state.** Most apps don't need a database — use `localStorage` for single-user preferences, `sessionStorage` for tab-scoped state, and `IndexedDB` for larger structured client-side data. Static content and demo apps shouldn't provision infrastructure.

**Provision a database only when the app genuinely needs server-side persistence:** user accounts (Supabase Auth), multi-user shared state, data that must survive across devices, or data that must outlive browser storage.

**If you decide to use Supabase, `mcp__supabase__provision_database` must be the first Supabase-related call you make.** Every other `mcp__supabase__*` tool (`migrate`, `query`, `deploy_function`) and `mcp__secrets__request_secret` (which stores into the project's Edge Function secrets) will fail until the project exists. Don't plan the schema, don't ask the user for API keys — provision first. It creates a Postgres project (~30–60s), writes `VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` to `.env`, and sets up `.env.local` with `DATABASE_URL`.

### Creating Tables

Use the `migrate` MCP tool for all schema changes. Write your SQL and the tool handles
migration versioning, execution, and TypeScript type regeneration automatically.

Example SQL for a todos table:
```sql
create table public.todos (
  id uuid default gen_random_uuid() primary key,
  title text not null,
  is_complete boolean default false,
  created_at timestamptz default now()
);
alter table public.todos enable row level security;
create policy "Allow anonymous read" on public.todos for select to anon using (true);
create policy "Allow anonymous insert" on public.todos for insert to anon with check (true);
create policy "Allow anonymous update" on public.todos for update to anon using (true) with check (true);
create policy "Allow anonymous delete" on public.todos for delete to anon using (true);
```

After migration, TypeScript types are regenerated at `src/lib/database.types.ts`.

**Always enable RLS on every table.** Even for anonymous access, create explicit policies.

### CRUD (supabase-js)

```typescript
import { supabase } from '@/lib/supabase'

// Read
const { data, error } = await supabase.from('todos').select('id, title, is_complete')
// Create
const { data, error } = await supabase.from('todos').insert({ title }).select()
// Update
const { data, error } = await supabase.from('todos').update({ is_complete: true }).eq('id', id)
// Delete
const { error } = await supabase.from('todos').delete().eq('id', id)
```

Always destructure `{ data, error }`. Never ignore errors. Select only the columns you need.

### Auth

```typescript
const { data, error } = await supabase.auth.signUp({ email, password })
const { data, error } = await supabase.auth.signInWithPassword({ email, password })
const { data: { user } } = await supabase.auth.getUser()
await supabase.auth.signOut()
supabase.auth.onAuthStateChange((event, session) => { ... })
```

### Storage

```typescript
await supabase.storage.from('bucket').upload('path/file.png', file)
const { data } = supabase.storage.from('bucket').getPublicUrl('path/file.png')
```

### Realtime

```typescript
const channel = supabase.channel('my-channel', { config: { private: true } })
  .on('broadcast', { event: 'item_created' }, (payload) => { ... })
  .subscribe()
// Always clean up
return () => { supabase.removeChannel(channel) }
```

### Security Rules

- **Never** put `DATABASE_URL` or any password in frontend code
- **Always** enable Row Level Security (RLS) on every table
- The `VITE_SUPABASE_PUBLISHABLE_KEY` in `.env` is safe for frontend — security is enforced by RLS
- Use `(select auth.uid())` in RLS policies when the app has user authentication

For deeper patterns, use the **supabase** skill.

### Secrets and Third-Party API Keys

When you need a third-party API key (Anthropic, OpenAI, Stripe secret key, etc.)
that must **not** be visible in browser code, use the `request_secret` MCP tool:

```
mcp__secrets__request_secret(
  name: "ANTHROPIC_API_KEY",
  purpose: "Call Claude API to generate AI summaries of blog posts"
)
```

The user sees a dialog and enters the key. It's stored in the **Supabase project's
Edge Function secrets** and accessible only via `Deno.env.get()` inside Edge
Functions — never in React, never in `.env`, never in this conversation.

**The Supabase project must exist before you call `request_secret`.** The secret
is written into the project's Edge Function secret store — there is no project
to write to until `mcp__supabase__provision_database` has run. Provision first,
then request secrets, then write and deploy the Edge Function.

**Before calling `request_secret`**: without login, anyone who reaches the
deployed URL can invoke the Edge Function and spend the user's API credits.
**Always ask the user first** — present it as an explicit choice and wait for
their answer:

> Your \<API name\> key will be stored server-side, but without login anyone
> who finds the deployed URL can call the function and use your credits.
>
> - Add login (recommended)
> - Leave it open — anyone with the URL can use my key

Then call `request_secret` and build exactly what they chose. Don't skip the
ask because auth seems obvious from the request — one redundant question is
cheaper than guessing wrong.

**Name rules**: UPPER_SNAKE_CASE only. Do NOT request `VITE_*` names (public
config, bundled into browser JS) or `SUPABASE_*` names (reserved).

### Edge Functions (Server-Side Code)

Edge Functions are Deno serverless functions. Use them to call third-party APIs
with secret keys, or any logic that must not run in the browser.

**Write** the function:

```typescript
// supabase/functions/summarize/index.ts
import Anthropic from "npm:@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: Deno.env.get("ANTHROPIC_API_KEY"),
});

Deno.serve(async (req) => {
  // If the user chose login, verify the caller's JWT here before spending
  // their API credits — see "Securing Edge Functions That Use Secrets" below.
  const { text } = await req.json();
  const msg = await client.messages.create({
    model: "claude-haiku-4-5-20251001",
    max_tokens: 1024,
    messages: [{ role: "user", content: `Summarize: ${text}` }],
  });
  return Response.json({ summary: msg.content[0].text });
});
```

**Deploy** with the MCP tool:

```
mcp__supabase__deploy_function(slug: "summarize")
```

**Invoke** from React:

```typescript
const { data, error } = await supabase.functions.invoke("summarize", {
  body: { text: postContent },
});
```

**Streaming (LLM output word-by-word):** `supabase.functions.invoke()` buffers
the full response before returning — it can't stream. Use raw `fetch()` to read
the response stream directly. Raw `fetch()` is a cross-origin browser request,
so the function **must** handle CORS or the browser blocks the preflight. Include
these headers from the start — don't discover it via a CORS error after deploy.

```typescript
// supabase/functions/chat/index.ts
const cors = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type",
};

Deno.serve(async (req) => {
  if (req.method === "OPTIONS") return new Response(null, { headers: cors });
  // ... build a ReadableStream from the LLM response ...
  return new Response(stream, {
    headers: { ...cors, "Content-Type": "text/event-stream" },
  });
});
```

```typescript
// React side — raw fetch, read the stream
const res = await fetch(`${import.meta.env.VITE_SUPABASE_URL}/functions/v1/chat`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ prompt }),
});
const reader = res.body!.getReader();
```

Shared code across functions goes in `supabase/functions/_shared/`.
Use `npm:` prefix for npm packages (e.g., `import z from "npm:zod"`).

### Securing Edge Functions That Use Secrets

If the user chose login when you asked (see **Secrets and Third-Party API Keys**
above), do both of these:

1. **Gate the app behind Supabase Auth.** Add signup/signin (see the Auth section
   above). Protect the routes that invoke the function.

2. **Verify the caller's JWT inside the function.** The Supabase client forwards
   the user's token automatically; check it before doing the secret-using work:

   ```typescript
   import { createClient } from "jsr:@supabase/supabase-js@2";

   Deno.serve(async (req) => {
     const authHeader = req.headers.get("Authorization");
     if (!authHeader) {
       return new Response("Missing auth header", { status: 401 });
     }
     const supabase = createClient(
       Deno.env.get("SUPABASE_URL")!,
       Deno.env.get("SUPABASE_ANON_KEY")!,
       { global: { headers: { Authorization: authHeader } } },
     );
     const { data: { user }, error } = await supabase.auth.getUser();
     if (error || !user) {
       return new Response("Unauthorized", { status: 401 });
     }
     // ... proceed with the secret-using logic
   });
   ```

## Data Fetching

Use **TanStack React Query** for all data fetching. It handles caching, loading states, errors, and background refetching automatically.

### With Supabase (for persistent data)

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { supabase } from '@/lib/supabase'

function useTodos() {
  return useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const { data, error } = await supabase.from('todos').select('id, title, is_complete')
      if (error) throw error
      return data
    },
  })
}

function useCreateTodo() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: async (title: string) => {
      const { error } = await supabase.from('todos').insert({ title })
      if (error) throw error
    },
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),
  })
}

// Usage in component
const { data: todos, isLoading } = useTodos()
const createTodo = useCreateTodo()
if (isLoading) return <Skeleton className="h-32 w-full" />
```

### With external APIs

```typescript
const { data } = useQuery({
  queryKey: ['weather', city],
  queryFn: () => fetch(`https://api.example.com/weather?q=${city}`).then(r => r.json()),
})
```

Extract data-fetching logic into custom hooks in `src/hooks/`. Never put Supabase queries directly in component bodies.

## State Management

Use React hooks. No external state libraries needed.

```tsx
const [items, setItems] = useState<Item[]>([])

// Add
setItems(prev => [...prev, newItem])

// Remove
setItems(prev => prev.filter(item => item.id !== id))

// Update
setItems(prev => prev.map(item => item.id === id ? { ...item, ...updates } : item))
```

## Dark Mode

```tsx
function ThemeToggle() {
  const [dark, setDark] = useState(false)

  const toggle = () => {
    document.documentElement.classList.toggle("dark")
    setDark(d => !d)
  }

  return (
    <Button variant="ghost" size="icon" onClick={toggle}>
      {dark ? <Sun className="h-4 w-4" /> : <Moon className="h-4 w-4" />}
    </Button>
  )
}
```

## TypeScript

- Type all component props with interfaces
- Use `React.ReactNode` for children props
- Use `React.ChangeEvent<HTMLInputElement>` for event handlers
- Use `React.FormEvent<HTMLFormElement>` for form submit
- Use `useRef<HTMLInputElement>(null)` for typed refs

## DO NOT

- Install alternative bundlers, CSS frameworks, or component libraries
- Modify main.tsx or index.css unless necessary
- Delete or modify `baku-inspector-plugin.mjs` or `baku-inspector-plugin.d.mts` — they provide the element inspector and console capture integration
- Remove the `bakuInspectorPlugin()` import or plugin entry from `vite.config.ts`
- Use inline styles -- use Tailwind
- Use class components -- use function components with hooks
- Ignore TypeScript errors -- fix them
- Put DATABASE_URL or secrets in frontend code
- Read or echo the contents of `.env` or `.env.local`
- Inline third-party API key values in code -- use `Deno.env.get()` in Edge Functions
- Call `request_secret` without first asking the user whether they want login
- Call `request_secret` or any `mcp__supabase__*` tool before `mcp__supabase__provision_database` — the project doesn't exist yet
- Create components over 300 lines -- split them
- Embed Supabase queries directly in components -- use custom hooks

## DO

- Update the `<title>` in `index.html` to match the app you're building
- Use shadcn/ui components for consistent UI
- Use Tailwind semantic colors for dark mode support
- Keep components small and focused
- Use TypeScript interfaces for all data shapes
- Use responsive design (mobile-first)
- Handle loading and error states in data-fetching components
- Enable RLS on every Supabase table
- Wrap data fetching in custom hooks using TanStack Query
- Use the **supabase** skill for advanced database patterns
- Use `import type` for type-only imports
- Use `mcp__secrets__request_secret` when you need a third-party API key
- Write Edge Functions for server-side logic (API calls with secrets, webhooks)
- Ask the user about login before every `request_secret` call; build auth if they choose it
