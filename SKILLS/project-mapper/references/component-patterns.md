# Component Patterns — Vite + React + TypeScript + shadcn/ui

## Table of Contents
1. [Stack & architecture](#1-stack--architecture)
2. [Project scaffold](#2-project-scaffold) — full file tree, package.json, configs
3. [globals.css & theming](#3-globalscss--theming) — CSS variables, shadcn tokens
4. [Data types](#4-data-types) — shared TypeScript interfaces
5. [Data embedding](#5-data-embedding) — baking FILE_MAP / HIERARCHY into the app
6. [App entry & routing](#6-app-entry--routing) — App.tsx, sidebar, header
7. [RoleGate](#7-rolegate) — permission gating component
8. [Dashboard page](#8-dashboard-page) — overview cards
9. [Feature Map page](#9-feature-map-page) — dependency graph
10. [State Flow page](#10-state-flow-page) — state machine visualizer
11. [Module page](#11-module-page) — States / Branches / Flow / Features tabs
12. [Governance Config page](#12-governance-config-page) — interactive authority/delegation form
13. [Hierarchy page](#13-hierarchy-page) — permission registry table
14. [User UI conventions](#14-user-ui-conventions) — client-facing portal rules
15. [Custom-only rule](#15-custom-only-rule) — when to go off-shadcn

---

## 1. Stack & architecture

| Layer | Choice |
|-------|--------|
| Bundler | Vite 5 |
| Language | TypeScript (strict) |
| UI components | shadcn/ui (Radix primitives + Tailwind) |
| Styling | Tailwind CSS v3 |
| Icons | lucide-react (shadcn default) |
| Fonts | Geist Sans + Geist Mono (shadcn default) |
| Animations | `tailwindcss-animate` (bundled with shadcn) |

**Each UI (admin-ui, user-ui) is a separate Vite project.** The user runs:
```bash
cd admin-ui && npm install && npm run dev
```

**Do not** produce single-file HTML or CDN-based builds. shadcn requires a proper build pipeline.

---

## 2. Project scaffold

### File tree (admin-ui baseline)
```
admin-ui/
├── package.json
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── vite.config.ts
├── tailwind.config.ts
├── postcss.config.js
├── components.json
├── index.html
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── globals.css
    ├── data.ts              ← all PROJECT_DATA (from FILE_MAP + HIERARCHY)
    ├── types.ts             ← shared TypeScript interfaces
    ├── lib/
    │   └── utils.ts         ← cn() helper
    ├── components/
    │   ├── ui/              ← shadcn generated components (copy verbatim from shadcn docs)
    │   │   ├── button.tsx
    │   │   ├── card.tsx
    │   │   ├── badge.tsx
    │   │   ├── table.tsx
    │   │   ├── tabs.tsx
    │   │   ├── select.tsx
    │   │   ├── switch.tsx
    │   │   ├── dialog.tsx
    │   │   ├── sheet.tsx
    │   │   ├── sidebar.tsx
    │   │   ├── separator.tsx
    │   │   ├── tooltip.tsx
    │   │   ├── scroll-area.tsx
    │   │   └── ...          ← add as needed
    │   ├── role-gate.tsx
    │   ├── app-sidebar.tsx
    │   ├── dashboard-page.tsx
    │   ├── feature-map-page.tsx
    │   ├── state-flow-page.tsx
    │   ├── module-page.tsx
    │   ├── governance-config-page.tsx
    │   └── hierarchy-page.tsx
    └── pages/               ← one file per module page if needed
```

For `user-ui/`, keep the same base toolchain files (`package.json`, TypeScript configs, Vite config, Tailwind config, `index.html`, `src/main.tsx`, `src/App.tsx`, `src/globals.css`, `src/data.ts`, `src/types.ts`, `src/lib/utils.ts`, and the required shadcn `ui/` primitives), but do **not** include admin-only pages such as `governance-config-page.tsx`, `hierarchy-page.tsx`, `feature-map-page.tsx`, or `state-flow-page.tsx`.

### package.json
```json
{
  "name": "<project-name>-<admin-ui|user-ui>",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "@radix-ui/react-dialog": "^1.1.2",
    "@radix-ui/react-dropdown-menu": "^2.1.2",
    "@radix-ui/react-scroll-area": "^1.2.1",
    "@radix-ui/react-select": "^2.1.2",
    "@radix-ui/react-separator": "^1.1.0",
    "@radix-ui/react-slot": "^1.1.0",
    "@radix-ui/react-switch": "^1.1.1",
    "@radix-ui/react-tabs": "^1.1.1",
    "@radix-ui/react-tooltip": "^1.1.3",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "lucide-react": "^0.462.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "tailwind-merge": "^2.5.4"
  },
  "devDependencies": {
    "@types/react": "^18.3.12",
    "@types/react-dom": "^18.3.1",
    "@vitejs/plugin-react": "^4.3.3",
    "autoprefixer": "^10.4.20",
    "postcss": "^8.4.47",
    "tailwindcss": "^3.4.14",
    "tailwindcss-animate": "^1.0.7",
    "typescript": "^5.6.3",
    "vite": "^5.4.10"
  }
}
```

### vite.config.ts
```ts
import path from "path"
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: { "@": path.resolve(__dirname, "./src") },
  },
})
```

### tsconfig.json
```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```

### tsconfig.app.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["src"]
}
```

### tsconfig.node.json
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2023"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["vite.config.ts"]
}
```

### tailwind.config.ts
```ts
import type { Config } from "tailwindcss"
import animate from "tailwindcss-animate"

const config: Config = {
  darkMode: ["class"],
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: { DEFAULT: "hsl(var(--primary))", foreground: "hsl(var(--primary-foreground))" },
        secondary: { DEFAULT: "hsl(var(--secondary))", foreground: "hsl(var(--secondary-foreground))" },
        destructive: { DEFAULT: "hsl(var(--destructive))", foreground: "hsl(var(--destructive-foreground))" },
        muted: { DEFAULT: "hsl(var(--muted))", foreground: "hsl(var(--muted-foreground))" },
        accent: { DEFAULT: "hsl(var(--accent))", foreground: "hsl(var(--accent-foreground))" },
        popover: { DEFAULT: "hsl(var(--popover))", foreground: "hsl(var(--popover-foreground))" },
        card: { DEFAULT: "hsl(var(--card))", foreground: "hsl(var(--card-foreground))" },
        sidebar: {
          DEFAULT: "hsl(var(--sidebar-background))",
          foreground: "hsl(var(--sidebar-foreground))",
          primary: "hsl(var(--sidebar-primary))",
          "primary-foreground": "hsl(var(--sidebar-primary-foreground))",
          accent: "hsl(var(--sidebar-accent))",
          "accent-foreground": "hsl(var(--sidebar-accent-foreground))",
          border: "hsl(var(--sidebar-border))",
          ring: "hsl(var(--sidebar-ring))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      fontFamily: {
        sans: ["Geist", "sans-serif"],
        mono: ["Geist Mono", "monospace"],
      },
    },
  },
  plugins: [animate],
}

export default config
```

### postcss.config.js
```js
export default {
  plugins: { tailwindcss: {}, autoprefixer: {} },
}
```

### components.json (shadcn config)
```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "src/globals.css",
    "baseColor": "zinc",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
```

### index.html
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><Project Name> <Admin UI | User UI></title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### src/lib/utils.ts
```ts
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

---

## 3. globals.css & theming

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 240 10% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 240 10% 3.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 240 10% 3.9%;
    --primary: 240 5.9% 10%;
    --primary-foreground: 0 0% 98%;
    --secondary: 240 4.8% 95.9%;
    --secondary-foreground: 240 5.9% 10%;
    --muted: 240 4.8% 95.9%;
    --muted-foreground: 240 3.8% 46.1%;
    --accent: 240 4.8% 95.9%;
    --accent-foreground: 240 5.9% 10%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 5.9% 90%;
    --input: 240 5.9% 90%;
    --ring: 240 5.9% 10%;
    --radius: 0.5rem;
    --sidebar-background: 240 5.9% 10%;
    --sidebar-foreground: 240 4.8% 95.9%;
    --sidebar-primary: 0 0% 98%;
    --sidebar-primary-foreground: 240 5.9% 10%;
    --sidebar-accent: 240 3.7% 15.9%;
    --sidebar-accent-foreground: 240 4.8% 95.9%;
    --sidebar-border: 240 3.7% 15.9%;
    --sidebar-ring: 217.2 91.2% 59.8%;
  }
  .dark {
    --background: 240 10% 3.9%;
    --foreground: 0 0% 98%;
    --card: 240 10% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 240 10% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 0% 98%;
    --primary-foreground: 240 5.9% 10%;
    --secondary: 240 3.7% 15.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 240 3.7% 15.9%;
    --muted-foreground: 240 5% 64.9%;
    --accent: 240 3.7% 15.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 240 4.9% 83.9%;
    --sidebar-background: 240 5.9% 10%;
    --sidebar-foreground: 240 4.8% 95.9%;
    --sidebar-primary: 224.3 76.3% 48%;
    --sidebar-primary-foreground: 0 0% 100%;
    --sidebar-accent: 240 3.7% 15.9%;
    --sidebar-accent-foreground: 240 4.8% 95.9%;
    --sidebar-border: 240 3.7% 15.9%;
    --sidebar-ring: 217.2 91.2% 59.8%;
  }
}

@layer base {
  * { @apply border-border; }
  body { @apply bg-background text-foreground font-sans; }
}
```

---

## 4. Data types

Role names, hierarchy depth, permission assignments, and governance controls in
the sample code below are placeholders only. Replace them with the exact
source-defined roles and authority surfaces from FILE_MAP.md and HIERARCHY.md.
Do not keep `owner`, `super_admin`, `admin`, or `user` unless the source files
actually use those roles.
Do not keep placeholder sample IDs such as `role_primary`, `role_secondary`, or
`role_viewer` unless the source files actually define those names.

```ts
// src/types.ts

export type RoleLevel = string

export interface StateNode {
  id: string
  label: string
  type: "initial" | "active" | "terminal" | "error" | "warning"
  description: string
  entry: string
  exit: string
}

export interface Transition {
  from: string
  to: string
  label: string
  trigger: string
  sideEffects?: string
}

export interface Branch {
  id: string
  label: string
  condition: string
  pathA: string
  pathB: string
}

export interface FlowStep {
  step: number
  label: string
  detail: string
}

export interface ModuleAction {
  label: string
  minRole: RoleLevel
  variant?: "default" | "destructive" | "outline" | "secondary" | "ghost"
}

export interface Feature {
  id: string
  name: string
  minRole: RoleLevel
  status: "complete" | "in_progress" | "pending" | "blocked"
  acceptance: string
}

export interface Module {
  id: string
  name: string
  description: string
  minRole: RoleLevel
  dependsOn: string[]
  dependedOnBy: string[]
  acceptanceCondition: string
  skills: string[]
  features: Feature[]
  states: StateNode[]
  transitions: Transition[]
  branches: Branch[]
  flow: FlowStep[]
  actions: ModuleAction[]
}

export interface Permission {
  string: string
  assignedRoles: RoleLevel[]
  source: string
  notes?: string
}

export interface Dependency {
  from: string
  to: string
}

export interface GovernanceConfigKey {
  key: string
  label: string
  description?: string
  default: boolean
}

export interface RoleDefinition {
  id: RoleLevel
  label: string
}

export interface ProjectData {
  name: string
  roles: RoleDefinition[]
  roleAccess: Record<RoleLevel, RoleLevel[]>
  // Optional demo-only default for the role switcher. This is not a hierarchy rule.
  defaultDemoRoleId?: RoleLevel
  // Optional explicit role allowed to access Governance Config when such a page is documented.
  governanceRoleId?: RoleLevel
  modules: Module[]
  permissions: Permission[]
  dependencies: Dependency[]
  governanceConfigKeys: GovernanceConfigKey[]
}
```

---

## 5. Data embedding

```ts
// src/data.ts
// ALL content derived from FILE_MAP.md and HIERARCHY.md — nothing invented.
// The role IDs below are illustrative placeholders only.

import type { ProjectData } from "@/types"

export const PROJECT_DATA: ProjectData = {
  name: "Your Project Name",
  roles: [
    { id: "role_primary", label: "Primary Role" },
    { id: "role_secondary", label: "Secondary Role" },
    { id: "role_viewer", label: "Viewer Role" },
  ],
  roleAccess: {
    role_primary: ["role_primary", "role_secondary", "role_viewer"],
    role_secondary: ["role_secondary", "role_viewer"],
    role_viewer: ["role_viewer"],
  },
  defaultDemoRoleId: "role_secondary",
  governanceRoleId: "role_primary",
  modules: [
    {
      id: "form-builder",
      name: "Form Builder",
      description: "Allows the documented operator role to create and manage submission forms.",
      minRole: "role_secondary",
      dependsOn: ["session-management", "file-storage"],
      dependedOnBy: ["client-portal"],
      acceptanceCondition: "The documented operator role can create, edit, publish, and archive forms.",
      skills: ["secure-coding-practices", "design-auditor"],
      features: [
        { id: "f1", name: "Create Form", minRole: "role_secondary", status: "complete", acceptance: "Form saved to DB" },
      ],
      states: [
        { id: "draft", label: "Draft", type: "initial", description: "Form created, not published", entry: "Documented operator role creates form", exit: "Documented operator role publishes" },
        { id: "active", label: "Active", type: "active", description: "Live and accepting submissions", entry: "Published", exit: "Archived" },
        { id: "archived", label: "Archived", type: "terminal", description: "No longer accepting submissions", entry: "Documented operator role archives", exit: "" },
      ],
      transitions: [
        { from: "draft", to: "active", label: "Publish", trigger: "Documented operator role clicks Publish", sideEffects: "Notification sent to the documented next authority when required" },
        { from: "active", to: "archived", label: "Archive", trigger: "Documented operator role clicks Archive" },
      ],
      branches: [
        { id: "auth-check", label: "Authenticated?", condition: "Session token valid", pathA: "dashboard", pathB: "login-redirect" },
      ],
      flow: [
        { step: 1, label: "Trigger", detail: "Documented operator role navigates to Form Builder" },
        { step: 2, label: "Input", detail: "Template or blank form selected" },
        { step: 3, label: "Processing", detail: "Fields validated, form saved to DB" },
        { step: 4, label: "Output", detail: "Form record created, ID returned" },
        { step: 5, label: "Side effects", detail: "Audit log entry written" },
        { step: 6, label: "Error paths", detail: "Validation failure returns inline errors" },
      ],
      actions: [
        { label: "Archive Form", minRole: "role_secondary", variant: "outline" },
        { label: "Delete Form", minRole: "role_primary", variant: "destructive" },
      ],
    },
    // ← one object per module in FILE_MAP.md
  ],
  permissions: [
    { string: "form.create", assignedRoles: ["role_primary", "role_secondary"], source: "§1.2" },
    // ← one object per row in HIERARCHY.md permission registry
  ],
  dependencies: [
    { from: "form-builder", to: "file-storage" },
    // ← one object per edge in FILE_MAP.md dependency matrix
  ],
  governanceConfigKeys: [
    { key: "delegatedExportAccess", label: "Delegated role can export submissions", description: "Example governance control derived from HIERARCHY.md", default: false },
    // ← one key per documented governance or delegation control from HIERARCHY.md
  ],
}
```

---

## 6. App entry & routing

### src/main.tsx
```tsx
import { StrictMode } from "react"
import { createRoot } from "react-dom/client"
import "@/globals.css"
import App from "@/App"

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

### src/App.tsx
```tsx
import { useState } from "react"
import { SidebarProvider, SidebarInset, SidebarTrigger } from "@/components/ui/sidebar"
import { AppSidebar } from "@/components/app-sidebar"
import { Separator } from "@/components/ui/separator"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"
import { Badge } from "@/components/ui/badge"
import { PROJECT_DATA } from "@/data"
import type { RoleLevel } from "@/types"
import { DashboardPage } from "@/components/dashboard-page"
import { FeatureMapPage } from "@/components/feature-map-page"
import { StateFlowPage } from "@/components/state-flow-page"
import { ModulePage } from "@/components/module-page"
import { GovernanceConfigPage } from "@/components/governance-config-page"
import { HierarchyPage } from "@/components/hierarchy-page"

export default function App() {
  const [page, setPage] = useState("dashboard")
  const [role, setRole] = useState<RoleLevel>(
    // Fallback to the first listed role only for demo bootstrapping when no explicit default is supplied.
    PROJECT_DATA.defaultDemoRoleId ?? PROJECT_DATA.roles[0]?.id ?? "viewer",
  )
  const roleLabels = Object.fromEntries(PROJECT_DATA.roles.map((r) => [r.id, r.label])) as Record<RoleLevel, string>
  const hasGovernanceConfig = Boolean(PROJECT_DATA.governanceRoleId && PROJECT_DATA.governanceConfigKeys.length > 0)

  const renderPage = () => {
    if (page === "dashboard")   return <DashboardPage data={PROJECT_DATA} role={role} />
    if (page === "feature-map") return <FeatureMapPage modules={PROJECT_DATA.modules} dependencies={PROJECT_DATA.dependencies} role={role} />
    if (page === "state-flow")  return <StateFlowPage modules={PROJECT_DATA.modules} />
    if (page === "governance-config" && hasGovernanceConfig) {
      return <GovernanceConfigPage configKeys={PROJECT_DATA.governanceConfigKeys} role={role} />
    }
    if (page === "hierarchy")   return <HierarchyPage permissions={PROJECT_DATA.permissions} />
    const mod = PROJECT_DATA.modules.find(m => m.id === page)
    if (mod) return <ModulePage module={mod} role={role} onNavigate={setPage} />
    return <DashboardPage data={PROJECT_DATA} role={role} />
  }

  return (
    <SidebarProvider>
      <AppSidebar modules={PROJECT_DATA.modules} currentPage={page} onNavigate={setPage} />
      <SidebarInset>
        <header className="flex h-16 shrink-0 items-center gap-2 border-b px-4">
          <SidebarTrigger className="-ml-1" />
          <Separator orientation="vertical" className="mr-2 h-4" />
          <span className="font-semibold text-sm">{PROJECT_DATA.name} — Admin</span>
          <div className="ml-auto flex items-center gap-3">
            <span className="text-xs text-muted-foreground">Viewing as</span>
            <Select value={role} onValueChange={(v) => setRole(v as RoleLevel)}>
              <SelectTrigger className="w-36 h-8 text-xs">
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                {PROJECT_DATA.roles.map((r) => (
                  <SelectItem key={r.id} value={r.id}>{r.label}</SelectItem>
                ))}
              </SelectContent>
            </Select>
            <Badge variant="outline" className="text-xs">{roleLabels[role] ?? role}</Badge>
          </div>
        </header>
        <main className="flex flex-1 flex-col gap-4 p-6 animate-in fade-in duration-200">
          {renderPage()}
        </main>
      </SidebarInset>
    </SidebarProvider>
  )
}
```

### src/components/app-sidebar.tsx
```tsx
import { Home, Map, GitBranch, Settings, Shield, ChevronRight } from "lucide-react"
import {
  Sidebar, SidebarContent, SidebarGroup, SidebarGroupLabel, SidebarGroupContent,
  SidebarMenu, SidebarMenuItem, SidebarMenuButton, SidebarFooter,
} from "@/components/ui/sidebar"
import { PROJECT_DATA } from "@/data"
import type { Module } from "@/types"

interface AppSidebarProps {
  modules: Module[]
  currentPage: string
  onNavigate: (page: string) => void
}

export function AppSidebar({ modules, currentPage, onNavigate }: AppSidebarProps) {
  const fixedNav = [
    { id: "dashboard",   label: "Dashboard",   icon: Home },
    { id: "feature-map", label: "Feature Map", icon: Map },
    { id: "state-flow",  label: "State Flow",  icon: GitBranch },
    { id: "hierarchy",   label: "Permissions", icon: Shield },
    ...(PROJECT_DATA.governanceRoleId && PROJECT_DATA.governanceConfigKeys.length > 0
      ? [{ id: "governance-config", label: "Governance Config", icon: Settings }]
      : []),
  ]

  return (
    <Sidebar>
      <SidebarContent>
        <SidebarGroup>
          <SidebarGroupLabel>Navigation</SidebarGroupLabel>
          <SidebarGroupContent>
            <SidebarMenu>
              {fixedNav.map(({ id, label, icon: Icon }) => (
                <SidebarMenuItem key={id}>
                  <SidebarMenuButton isActive={currentPage === id} onClick={() => onNavigate(id)}>
                    <Icon className="size-4" />
                    <span>{label}</span>
                  </SidebarMenuButton>
                </SidebarMenuItem>
              ))}
            </SidebarMenu>
          </SidebarGroupContent>
        </SidebarGroup>
        <SidebarGroup>
          <SidebarGroupLabel>Modules</SidebarGroupLabel>
          <SidebarGroupContent>
            <SidebarMenu>
              {modules.map(mod => (
                <SidebarMenuItem key={mod.id}>
                  <SidebarMenuButton isActive={currentPage === mod.id} onClick={() => onNavigate(mod.id)}>
                    <ChevronRight className="size-4" />
                    <span>{mod.name}</span>
                  </SidebarMenuButton>
                </SidebarMenuItem>
              ))}
            </SidebarMenu>
          </SidebarGroupContent>
        </SidebarGroup>
      </SidebarContent>
      <SidebarFooter className="p-4 text-xs text-sidebar-foreground/50">
        Project Mapper — Admin UI
      </SidebarFooter>
    </Sidebar>
  )
}
```

---

## 7. RoleGate

```tsx
// src/components/role-gate.tsx
import type { RoleLevel } from "@/types"
import { PROJECT_DATA } from "@/data"

interface RoleGateProps {
  minRole: RoleLevel
  currentRole: RoleLevel
  children: React.ReactNode
  hideCompletely?: boolean
}

export function RoleGate({ minRole, currentRole, children, hideCompletely = false }: RoleGateProps) {
  const allowedRoles = PROJECT_DATA.roleAccess[minRole] ?? [minRole]
  const allowed = allowedRoles.includes(currentRole)
  if (allowed) return <>{children}</>
  if (hideCompletely) return null
  return (
    <div className="flex items-center justify-center rounded-lg border border-dashed border-muted-foreground/25 bg-muted/30 p-8 text-center">
      <div>
        <div className="text-2xl mb-2">🔒</div>
        <p className="text-sm text-muted-foreground">
          Requires <span className="font-semibold text-foreground">{minRole.replace("_", " ")}</span> access
        </p>
      </div>
    </div>
  )
}
```

---

## 8. Dashboard page

```tsx
// src/components/dashboard-page.tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { Layers, Shield, AlertTriangle, CheckCircle } from "lucide-react"
import type { ProjectData, RoleLevel } from "@/types"

interface DashboardPageProps { data: ProjectData; role: RoleLevel }

export function DashboardPage({ data }: DashboardPageProps) {
  const tbdCount = data.permissions.filter(p => p.notes?.includes("TBD")).length

  const stats = [
    { title: "Modules Mapped",     value: data.modules.length,      icon: Layers,        description: "From FILE_MAP.md" },
    { title: "Permissions",        value: data.permissions.length,  icon: Shield,        description: `${tbdCount} flagged TBD` },
    { title: "Governance Config Keys",  value: data.governanceConfigKeys.length, icon: AlertTriangle, description: "Require authority decision" },
    { title: "Dependencies",       value: data.dependencies.length, icon: CheckCircle,   description: "Cross-module edges" },
  ]

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold tracking-tight">{data.name}</h1>
        <p className="text-muted-foreground text-sm mt-1">Admin dashboard — project structure overview</p>
      </div>
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        {stats.map(({ title, value, icon: Icon, description }) => (
          <Card key={title}>
            <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
              <CardTitle className="text-sm font-medium">{title}</CardTitle>
              <Icon className="size-4 text-muted-foreground" />
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{value}</div>
              <p className="text-xs text-muted-foreground mt-1">{description}</p>
            </CardContent>
          </Card>
        ))}
      </div>
      {tbdCount > 0 && (
        <Card className="border-amber-200 bg-amber-50 dark:border-amber-800 dark:bg-amber-950">
          <CardHeader>
            <CardTitle className="text-sm text-amber-800 dark:text-amber-200 flex items-center gap-2">
              <AlertTriangle className="size-4" /> {tbdCount} items require authority decision
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-xs text-amber-700 dark:text-amber-300">
              Review HIERARCHY.md gaps and set defaults in Governance Config before building.
            </p>
          </CardContent>
        </Card>
      )}
    </div>
  )
}
```

---

## 9. Feature Map page

```tsx
// src/components/feature-map-page.tsx
// Note: SVG dependency arrows are custom (shadcn has no graph primitive).
// All surrounding chrome uses shadcn Card, Badge, Tooltip.
import { useState } from "react"
import { Card, CardContent } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from "@/components/ui/tooltip"
import { cn } from "@/lib/utils"
import type { Module, Dependency, RoleLevel } from "@/types"
import { PROJECT_DATA } from "@/data"

const ROLE_CARD_STYLES = [
  "border-red-300 bg-red-50 text-red-900 dark:border-red-700 dark:bg-red-900/20 dark:text-red-300",
  "border-green-300 bg-green-50 text-green-900 dark:border-green-700 dark:bg-green-900/20 dark:text-green-300",
  "border-blue-300 bg-blue-50 text-blue-900 dark:border-blue-700 dark:bg-blue-900/20 dark:text-blue-300",
  "border-slate-300 bg-slate-50 text-slate-900 dark:border-slate-700 dark:bg-slate-900/20 dark:text-slate-300",
]

function getRoleCardClass(role: RoleLevel) {
  const idx = PROJECT_DATA.roles.findIndex((entry) => entry.id === role)
  return ROLE_CARD_STYLES[idx >= 0 ? idx % ROLE_CARD_STYLES.length : ROLE_CARD_STYLES.length - 1]
}

interface FeatureMapPageProps {
  modules: Module[]
  dependencies: Dependency[]
  role: RoleLevel
}

export function FeatureMapPage({ modules, dependencies }: FeatureMapPageProps) {
  const [active, setActive] = useState<string | null>(null)

  const isConnected = (id: string) =>
    active !== null && dependencies.some(d =>
      (d.from === active && d.to === id) || (d.to === active && d.from === id)
    )

  return (
    <TooltipProvider>
      <div className="space-y-4">
        <div>
          <h2 className="text-xl font-bold">Feature Interconnection Map</h2>
          <p className="text-sm text-muted-foreground mt-1">Click any module to highlight its dependencies.</p>
          <div className="flex gap-2 mt-2 flex-wrap">
            {PROJECT_DATA.roles.map((r) => (
              <Badge key={r.id} variant="outline" className={cn("text-xs", getRoleCardClass(r.id))}>
                {r.label}
              </Badge>
            ))}
          </div>
        </div>
        <Card>
          <CardContent className="p-6">
            <div className="grid grid-cols-3 gap-3">
              {modules.map(mod => (
                <Tooltip key={mod.id}>
                  <TooltipTrigger asChild>
                    <button
                      onClick={() => setActive(active === mod.id ? null : mod.id)}
                      className={cn(
                        "rounded-lg border-2 p-3 text-left text-sm font-medium transition-all duration-150",
                        getRoleCardClass(mod.minRole),
                        active === mod.id && "ring-2 ring-ring ring-offset-2 scale-105 shadow-md",
                        active && active !== mod.id && !isConnected(mod.id) && "opacity-25",
                        "hover:shadow-sm"
                      )}
                    >
                      <div className="font-semibold text-sm">{mod.name}</div>
                      <div className="text-xs opacity-70 mt-0.5 line-clamp-2">{mod.description}</div>
                    </button>
                  </TooltipTrigger>
                  <TooltipContent>
                    <p className="text-xs">Min role: {mod.minRole.replace("_"," ")}</p>
                    {mod.dependsOn.length > 0 && <p className="text-xs">Depends on: {mod.dependsOn.join(", ")}</p>}
                  </TooltipContent>
                </Tooltip>
              ))}
            </div>
            {dependencies.length > 0 && (
              <div className="mt-6 border-t pt-4">
                <p className="text-xs font-medium text-muted-foreground mb-2">Dependency edges</p>
                <div className="flex flex-wrap gap-2">
                  {dependencies.map((d, i) => (
                    <Badge key={i} variant="outline" className="font-mono text-xs">
                      {d.from} → {d.to}
                    </Badge>
                  ))}
                </div>
              </div>
            )}
          </CardContent>
        </Card>
      </div>
    </TooltipProvider>
  )
}
```

---

## 10. State Flow page

```tsx
// src/components/state-flow-page.tsx
// State nodes and transitions are custom SVG/JSX (no shadcn graph primitive).
// Module selector, detail panel, cards all use shadcn.
import { useState } from "react"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"
import { Badge } from "@/components/ui/badge"
import { Separator } from "@/components/ui/separator"
import { cn } from "@/lib/utils"
import type { Module, StateNode } from "@/types"

const STATE_STYLE: Record<StateNode["type"], string> = {
  initial:  "bg-blue-100 border-blue-400 text-blue-900 dark:bg-blue-900/30 dark:border-blue-500 dark:text-blue-200",
  active:   "bg-green-100 border-green-400 text-green-900 dark:bg-green-900/30 dark:border-green-500 dark:text-green-200",
  terminal: "bg-muted border-border text-muted-foreground",
  error:    "bg-red-100 border-red-400 text-red-900 dark:bg-red-900/30 dark:border-red-500 dark:text-red-200",
  warning:  "bg-amber-100 border-amber-400 text-amber-900 dark:bg-amber-900/30 dark:border-amber-500 dark:text-amber-200",
}

const STATE_PREFIX: Partial<Record<StateNode["type"], string>> = {
  initial: "▶ ", terminal: "■ ",
}

interface StateFlowPageProps { modules: Module[] }

export function StateFlowPage({ modules }: StateFlowPageProps) {
  const [selectedId, setSelectedId] = useState(modules[0]?.id ?? "")
  const [activeState, setActiveState] = useState<StateNode | null>(null)
  const mod = modules.find(m => m.id === selectedId)

  return (
    <div className="space-y-4">
      <div className="flex items-center gap-4">
        <h2 className="text-xl font-bold">State Flow</h2>
        <Select value={selectedId} onValueChange={v => { setSelectedId(v); setActiveState(null) }}>
          <SelectTrigger className="w-56">
            <SelectValue placeholder="Select module" />
          </SelectTrigger>
          <SelectContent>
            {modules.map(m => (
              <SelectItem key={m.id} value={m.id}>{m.name}</SelectItem>
            ))}
          </SelectContent>
        </Select>
      </div>

      {mod && (
        <div className="grid gap-4 lg:grid-cols-3">
          <div className="lg:col-span-2 space-y-4">
            <Card>
              <CardHeader className="pb-3"><CardTitle className="text-sm">States</CardTitle></CardHeader>
              <CardContent>
                <div className="flex flex-wrap gap-2">
                  {mod.states.map(state => (
                    <button
                      key={state.id}
                      onClick={() => setActiveState(activeState?.id === state.id ? null : state)}
                      className={cn(
                        "px-3 py-1.5 rounded-lg border-2 text-sm font-medium transition-all duration-150",
                        STATE_STYLE[state.type],
                        activeState?.id === state.id && "ring-2 ring-ring ring-offset-1"
                      )}
                    >
                      {STATE_PREFIX[state.type]}{state.label}
                    </button>
                  ))}
                </div>
              </CardContent>
            </Card>

            <Card>
              <CardHeader className="pb-3"><CardTitle className="text-sm">Transitions</CardTitle></CardHeader>
              <CardContent className="space-y-2">
                {mod.transitions.length > 0 ? mod.transitions.map((t, i) => (
                  <div key={i} className="flex items-center gap-2 rounded-md border bg-muted/30 px-3 py-2 text-sm">
                    <Badge variant="outline" className="font-mono text-xs">{t.from}</Badge>
                    <span className="text-muted-foreground text-xs">──{t.label}──▶</span>
                    <Badge variant="outline" className="font-mono text-xs">{t.to}</Badge>
                    {t.sideEffects && <span className="ml-auto text-xs text-muted-foreground italic">{t.sideEffects}</span>}
                  </div>
                )) : <p className="text-sm text-muted-foreground italic">No transitions defined.</p>}
              </CardContent>
            </Card>

            {mod.branches.length > 0 && (
              <Card>
                <CardHeader className="pb-3"><CardTitle className="text-sm">Branch Points</CardTitle></CardHeader>
                <CardContent className="space-y-3">
                  {mod.branches.map(b => (
                    <div key={b.id} className="rounded-lg border border-amber-200 bg-amber-50 px-4 py-3 dark:border-amber-800 dark:bg-amber-950">
                      <p className="font-medium text-amber-800 dark:text-amber-200 text-sm mb-1">◆ {b.label}</p>
                      <p className="text-xs text-amber-700 dark:text-amber-300 mb-2">{b.condition}</p>
                      <div className="flex gap-4 text-xs">
                        <span className="text-green-700 dark:text-green-400">✓ True → <strong>{b.pathA}</strong></span>
                        <span className="text-red-700 dark:text-red-400">✗ False → <strong>{b.pathB}</strong></span>
                      </div>
                    </div>
                  ))}
                </CardContent>
              </Card>
            )}
          </div>

          <div>
            {activeState ? (
              <Card className="animate-in fade-in slide-in-from-right-2 duration-150">
                <CardHeader className="pb-3"><CardTitle className="text-sm">State Detail</CardTitle></CardHeader>
                <CardContent className="space-y-3 text-sm">
                  <span className={cn("inline-flex px-2 py-0.5 rounded border text-xs font-medium", STATE_STYLE[activeState.type])}>
                    {STATE_PREFIX[activeState.type]}{activeState.label}
                  </span>
                  <Separator />
                  <div><p className="text-xs font-medium text-muted-foreground mb-1">Description</p><p>{activeState.description}</p></div>
                  <div><p className="text-xs font-medium text-muted-foreground mb-1">Entry condition</p><p>{activeState.entry || "—"}</p></div>
                  <div><p className="text-xs font-medium text-muted-foreground mb-1">Exit condition</p><p>{activeState.exit || "—"}</p></div>
                </CardContent>
              </Card>
            ) : (
              <Card className="border-dashed">
                <CardContent className="flex items-center justify-center p-8 text-center">
                  <p className="text-sm text-muted-foreground">Click a state node to see its details.</p>
                </CardContent>
              </Card>
            )}
          </div>
        </div>
      )}
    </div>
  )
}
```

---

## 11. Module page

```tsx
// src/components/module-page.tsx
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs"
import { Card, CardContent } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { Button } from "@/components/ui/button"
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
import { RoleGate } from "@/components/role-gate"
import { cn } from "@/lib/utils"
import type { Module, RoleLevel } from "@/types"
import { PROJECT_DATA } from "@/data"

const SKILL_COLOR: Record<string, string> = {
  "design-auditor":           "bg-pink-100 text-pink-700 border-pink-200",
  "email-html-mjml":          "bg-blue-100 text-blue-700 border-blue-200",
  "secure-coding-practices":  "bg-red-100 text-red-700 border-red-200",
  "security-best-practices":  "bg-rose-100 text-rose-700 border-rose-200",
  "playwright-skill":         "bg-green-100 text-green-700 border-green-200",
  "playwright":               "bg-teal-100 text-teal-700 border-teal-200",
  "doc":                      "bg-sky-100 text-sky-700 border-sky-200",
}

const ROLE_BADGE_STYLES = [
  "bg-red-100 text-red-700 border-red-200",
  "bg-green-100 text-green-700 border-green-200",
  "bg-blue-100 text-blue-700 border-blue-200",
  "bg-slate-100 text-slate-700 border-slate-200",
]

function getRoleBadgeClass(role: RoleLevel) {
  const idx = PROJECT_DATA.roles.findIndex((entry) => entry.id === role)
  return ROLE_BADGE_STYLES[idx >= 0 ? idx % ROLE_BADGE_STYLES.length : ROLE_BADGE_STYLES.length - 1]
}

const STATUS_STYLE: Record<string, string> = {
  complete:    "bg-green-100 text-green-700",
  in_progress: "bg-yellow-100 text-yellow-700",
  pending:     "bg-muted text-muted-foreground",
  blocked:     "bg-red-100 text-red-700",
}

interface ModulePageProps {
  module: Module
  role: RoleLevel
  onNavigate: (id: string) => void
}

export function ModulePage({ module: mod, role, onNavigate }: ModulePageProps) {
  return (
    <div className="max-w-4xl space-y-5 animate-in fade-in duration-150">
      <div>
        <div className="flex items-center gap-3 mb-1">
          <h2 className="text-2xl font-bold">{mod.name}</h2>
          <Badge className={cn("text-xs", getRoleBadgeClass(mod.minRole))}>
            {mod.minRole.replace("_"," ")}
          </Badge>
        </div>
        <p className="text-muted-foreground text-sm">{mod.description}</p>
      </div>

      {mod.skills.length > 0 && (
        <div className="flex flex-wrap gap-1.5">
          {mod.skills.map(s => (
            <span key={s} className={cn(
              "inline-flex items-center px-2 py-0.5 rounded border text-xs font-medium",
              SKILL_COLOR[s] ?? "bg-muted text-muted-foreground border-border"
            )}>
              🔧 {s}
            </span>
          ))}
        </div>
      )}

      <div className="flex flex-wrap gap-2">
        {mod.dependsOn.map(dep => (
          <Button key={dep} variant="outline" size="sm" className="h-7 text-xs" onClick={() => onNavigate(dep)}>
            ↳ Depends on: {dep}
          </Button>
        ))}
        {mod.dependedOnBy.map(dep => (
          <Button key={dep} variant="outline" size="sm" className="h-7 text-xs" onClick={() => onNavigate(dep)}>
            ← Used by: {dep}
          </Button>
        ))}
      </div>

      <div className="rounded-lg border border-amber-200 bg-amber-50 p-4 text-sm dark:border-amber-800 dark:bg-amber-950">
        <span className="font-semibold text-amber-800 dark:text-amber-200">Acceptance: </span>
        <span className="text-amber-700 dark:text-amber-300">{mod.acceptanceCondition}</span>
      </div>

      <Tabs defaultValue="states">
        <TabsList>
          <TabsTrigger value="states">States</TabsTrigger>
          <TabsTrigger value="branches">Branches</TabsTrigger>
          <TabsTrigger value="flow">Flow</TabsTrigger>
          <TabsTrigger value="features">Features</TabsTrigger>
        </TabsList>

        <TabsContent value="states" className="mt-4">
          <Card>
            <CardContent className="p-0">
              <Table>
                <TableHeader>
                  <TableRow>
                    <TableHead>State</TableHead>
                    <TableHead>Type</TableHead>
                    <TableHead>Entry condition</TableHead>
                    <TableHead>Exit condition</TableHead>
                  </TableRow>
                </TableHeader>
                <TableBody>
                  {mod.states.map(s => (
                    <TableRow key={s.id}>
                      <TableCell className="font-medium">{s.label}</TableCell>
                      <TableCell><Badge variant="outline" className="text-xs">{s.type}</Badge></TableCell>
                      <TableCell className="text-xs text-muted-foreground">{s.entry}</TableCell>
                      <TableCell className="text-xs text-muted-foreground">{s.exit || "—"}</TableCell>
                    </TableRow>
                  ))}
                </TableBody>
              </Table>
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="branches" className="mt-4 space-y-3">
          {mod.branches.length > 0 ? mod.branches.map(b => (
            <div key={b.id} className="rounded-lg border border-amber-200 bg-amber-50 px-4 py-3 dark:border-amber-800 dark:bg-amber-950">
              <p className="font-medium text-amber-800 dark:text-amber-200 text-sm mb-1">◆ {b.label}</p>
              <p className="text-xs text-amber-700 dark:text-amber-300 mb-2">{b.condition}</p>
              <div className="flex gap-4 text-xs">
                <span className="text-green-700 dark:text-green-400">✓ True → <strong>{b.pathA}</strong></span>
                <span className="text-red-700 dark:text-red-400">✗ False → <strong>{b.pathB}</strong></span>
              </div>
            </div>
          )) : <p className="text-sm text-muted-foreground italic">No branch points defined.</p>}
        </TabsContent>

        <TabsContent value="flow" className="mt-4 space-y-2">
          {mod.flow.map(f => (
            <div key={f.step} className="flex gap-4 rounded-md border bg-muted/30 px-4 py-3 text-sm">
              <span className="font-bold text-primary w-5 shrink-0">{f.step}.</span>
              <div>
                <p className="font-medium">{f.label}</p>
                <p className="text-xs text-muted-foreground mt-0.5">{f.detail}</p>
              </div>
            </div>
          ))}
        </TabsContent>

        <TabsContent value="features" className="mt-4">
          <RoleGate minRole={mod.minRole} currentRole={role}>
            <Card>
              <CardContent className="p-0">
                <Table>
                  <TableHeader>
                    <TableRow>
                      <TableHead>Feature</TableHead>
                      <TableHead>Min Role</TableHead>
                      <TableHead>Status</TableHead>
                      <TableHead>Acceptance</TableHead>
                    </TableRow>
                  </TableHeader>
                  <TableBody>
                    {mod.features.map(f => (
                      <TableRow key={f.id}>
                        <TableCell className="font-medium">{f.name}</TableCell>
                        <TableCell>
                          <Badge variant="outline" className={cn("text-xs", getRoleBadgeClass(f.minRole))}>
                            {f.minRole.replace("_"," ")}
                          </Badge>
                        </TableCell>
                        <TableCell>
                          <span className={cn("inline-flex px-2 py-0.5 rounded-full text-xs font-medium", STATUS_STYLE[f.status] ?? STATUS_STYLE.pending)}>
                            {f.status.replace("_"," ")}
                          </span>
                        </TableCell>
                        <TableCell className="text-xs text-muted-foreground max-w-xs">{f.acceptance}</TableCell>
                      </TableRow>
                    ))}
                  </TableBody>
                </Table>
              </CardContent>
            </Card>
          </RoleGate>
        </TabsContent>
      </Tabs>

      {mod.actions.length > 0 && (
        <div className="flex flex-wrap gap-2 pt-2">
          {mod.actions.map((action, i) => (
            <RoleGate key={i} minRole={action.minRole} currentRole={role} hideCompletely>
              <Button variant={action.variant ?? "default"} size="sm">{action.label}</Button>
            </RoleGate>
          ))}
        </div>
      )}
    </div>
  )
}
```

---

## 12. Governance Config page

```tsx
// src/components/governance-config-page.tsx
import { useState } from "react"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Switch } from "@/components/ui/switch"
import { Button } from "@/components/ui/button"
import { Separator } from "@/components/ui/separator"
import { RoleGate } from "@/components/role-gate"
import { CheckCircle, Download } from "lucide-react"
import type { GovernanceConfigKey, RoleLevel } from "@/types"
import { PROJECT_DATA } from "@/data"

const STORAGE_KEY = "governance_config"

interface GovernanceConfigPageProps { configKeys: GovernanceConfigKey[]; role: RoleLevel }

export function GovernanceConfigPage({ configKeys, role }: GovernanceConfigPageProps) {
  const governanceRole = PROJECT_DATA.governanceRoleId
  const [config, setConfig] = useState<Record<string, boolean>>(() => {
    try {
      // localStorage is intentional here: this runs in the user's browser, not in a Claude artifact sandbox.
      const saved = localStorage.getItem(STORAGE_KEY)
      if (saved) return JSON.parse(saved)
    } catch { /* ignore */ }
    return Object.fromEntries(configKeys.map(k => [k.key, k.default]))
  })
  const [saved, setSaved] = useState(false)

  const handleToggle = (key: string) => {
    setConfig(prev => ({ ...prev, [key]: !prev[key] }))
    setSaved(false)
  }

  const handleSave = () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(config))
    setSaved(true)
    setTimeout(() => setSaved(false), 2500)
  }

  const handleExport = () => {
    const blob = new Blob([JSON.stringify(config, null, 2)], { type: "application/json" })
    const a = document.createElement("a")
    a.href = URL.createObjectURL(blob)
    a.download = "governance-config.json"
    a.click()
  }

  if (!governanceRole) {
    return null
  }

  return (
    <RoleGate minRole={governanceRole} currentRole={role}>
      <div className="max-w-2xl space-y-6">
        <div>
          <h2 className="text-xl font-bold">Governance Configuration</h2>
          <p className="text-sm text-muted-foreground mt-1">
            Configure documented authority and delegation controls. Changes only take effect after saving.
          </p>
        </div>
        <Card>
          <CardHeader>
            <CardTitle className="text-sm">Governance Controls</CardTitle>
            <CardDescription className="text-xs">Derived from HIERARCHY.md documented delegation or authority controls.</CardDescription>
          </CardHeader>
          <CardContent className="divide-y">
            {configKeys.length > 0 ? configKeys.map(({ key, label, description }) => (
              <div key={key} className="flex items-center justify-between py-4 first:pt-0 last:pb-0">
                <div>
                  <p className="text-sm font-medium">{label}</p>
                  {description && <p className="text-xs text-muted-foreground mt-0.5">{description}</p>}
                </div>
                <Switch checked={config[key] ?? false} onCheckedChange={() => handleToggle(key)} />
              </div>
            )) : (
              <p className="text-sm text-muted-foreground italic py-4">No governance controls found in HIERARCHY.md.</p>
            )}
          </CardContent>
        </Card>
        <Separator />
        <div className="flex gap-3">
          <Button onClick={handleSave} className="flex-1">
            {saved ? <><CheckCircle className="size-4 mr-2" />Saved</> : "Save Configuration"}
          </Button>
          <Button variant="outline" onClick={handleExport}>
            <Download className="size-4 mr-2" />Export JSON
          </Button>
        </div>
      </div>
    </RoleGate>
  )
}
```

---

## 13. Hierarchy page

```tsx
// src/components/hierarchy-page.tsx
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table"
import { Badge } from "@/components/ui/badge"
import { CheckCircle, XCircle, AlertTriangle } from "lucide-react"
import { cn } from "@/lib/utils"
import type { Permission } from "@/types"

const Check = () => <CheckCircle className="size-4 text-green-500 mx-auto" />
const Cross = () => <XCircle className="size-4 text-muted-foreground/25 mx-auto" />

interface HierarchyPageProps { permissions: Permission[] }

export function HierarchyPage({ permissions }: HierarchyPageProps) {
  const gaps = permissions.filter(p => p.notes?.includes("TBD"))

  return (
    <div className="space-y-6">
      <div>
        <h2 className="text-xl font-bold">Permission Registry</h2>
        <p className="text-sm text-muted-foreground mt-1">Full RBAC registry from HIERARCHY.md</p>
      </div>
      <Card>
        <CardContent className="p-0">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Permission</TableHead>
                <TableHead>Assigned Roles</TableHead>
                <TableHead>Source</TableHead>
                <TableHead>Notes</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {permissions.map(p => (
                <TableRow key={p.string} className={cn(p.notes?.includes("TBD") && "bg-amber-50/50 dark:bg-amber-950/20")}>
                  <TableCell className="font-mono text-xs font-medium">{p.string}</TableCell>
                  <TableCell className="text-xs text-muted-foreground">{p.assignedRoles.join(", ") || "TBD"}</TableCell>
                  <TableCell className="text-xs text-muted-foreground">{p.source}</TableCell>
                  <TableCell>
                    {p.notes?.includes("TBD") ? (
                      <Badge variant="outline" className="text-xs border-amber-400 text-amber-700 bg-amber-50">
                        <AlertTriangle className="size-3 mr-1" />TBD
                      </Badge>
                    ) : p.notes ? (
                      <span className="text-xs text-muted-foreground">{p.notes}</span>
                    ) : null}
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
      {gaps.length > 0 && (
        <Card className="border-amber-200 bg-amber-50 dark:border-amber-800 dark:bg-amber-950">
          <CardHeader>
            <CardTitle className="text-sm text-amber-800 dark:text-amber-200">
              {gaps.length} permission(s) require authority decision
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="flex flex-wrap gap-2">
              {gaps.map(p => (
                <Badge key={p.string} variant="outline" className="font-mono text-xs border-amber-400 text-amber-800">
                  {p.string}
                </Badge>
              ))}
            </div>
          </CardContent>
        </Card>
      )}
    </div>
  )
}
```

---

## 14. User UI conventions

The user-ui project uses the **same stack and scaffold** as admin-ui with these differences:

- No sidebar sections for admin-only modules
- No `RoleGate` wrapping (all shown content is already user-accessible)
- No Governance Config, Hierarchy, Feature Map, or State Flow pages
- No skill tag badges, permission strings, or dependency info visible to users
- Module state shown only where user-visible (e.g. order status, request status) — still uses shadcn `Badge` for state display
- All components still use shadcn/ui — `Card`, `Badge`, `Table`, `Tabs`, `Button`, etc.
- Same `globals.css`, same Tailwind config, same TypeScript types from `@/types`

---

## 15. Custom-only rule

**Default to shadcn for everything.** Use custom JSX/CSS only when shadcn provably cannot cover it:

| Need | Use |
|------|-----|
| Layout, spacing, color | Tailwind utility classes |
| Buttons, inputs, selects, switches | `@/components/ui/*` |
| Tables | shadcn `Table` + `TableHeader` / `TableBody` etc. |
| Dialogs, sheets, drawers | shadcn `Dialog` / `Sheet` |
| Tooltips | shadcn `Tooltip` + `TooltipProvider` |
| Tabs | shadcn `Tabs` + `TabsList` / `TabsTrigger` / `TabsContent` |
| Badges, labels | shadcn `Badge` with `variant` or custom `className` |
| Sidebar | shadcn `Sidebar` + `SidebarProvider` — never build custom |
| Animations | `tailwindcss-animate` classes: `animate-in`, `fade-in`, `slide-in-from-left`, `zoom-in`, etc. |
| Icons | `lucide-react` only — never use other icon libraries |
| Fonts | Geist via `globals.css` — never swap to system fonts |
| Class merging | `cn()` from `@/lib/utils` — never raw string concat |
| **Dependency graph edges** | Custom SVG — shadcn has no graph primitive |
| **State machine node canvas** | Custom inline SVG if needed — shadcn has no canvas primitive |

For the two custom cases, use inline SVG or div-based node rendering with Tailwind for node styling. All surrounding chrome (cards, selects, badges, panels) still uses shadcn.
