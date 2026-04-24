---
name: "figma-to-component"
displayName: "Figma to Component"
description: "Converts Figma designs into production-ready React components following the project's architecture, steering rules, and reusable component library. Fetches Figma data via Framelink MCP, then refactors the output into clean, responsive, standards-compliant code."
keywords: ["figma", "component", "design", "ui", "react", "convert", "mcp", "framelink"]
author: "www.linkedin.com/in/nathan-lunelli-23bb30290"
---

# Figma to Component Power

## Overview

This power bridges the gap between Figma designs and production-ready React components. It leverages the **Framelink Figma MCP** server (`mcp_Framelink_Figma_MCP`) to fetch design data, then applies the project's steering rules, coding standards, and existing component library to produce clean, responsive, well-structured code.

Key capabilities:
- Fetch Figma node data and images via Framelink MCP
- Convert raw Figma output into idiomatic React + Tailwind CSS components
- Enforce all project steering rules automatically
- Reuse existing base components instead of recreating them
- Automatically split complex screens into multiple focused components
- Generate mobile-first responsive layouts (no fixed sizes)
- Produce mock data when no API is available yet

## Onboarding

### Prerequisites
- Figma file URL or file key + node ID for the screen/component to convert
- Framelink Figma MCP server configured and running (see MCP Config below)

### MCP Config

This power includes a ready-to-use `mcp.json` file inside the power folder. The only setup required is replacing the placeholder API key with your Figma personal access token.

1. Open the `mcp.json` file included in this power's folder
2. Replace `YOUR_FIGMA_API_KEY_HERE` with your Figma personal access token (Settings → Personal Access Tokens in Figma)
3. The Kiro IDE will automatically detect and load the MCP server configuration from the power

---

## Core Principles

### 1. Reuse Existing Components — Never Recreate

Before writing any new component, **always scan the project's component directory** (`src/components/` or equivalent). Identify every reusable primitive the project already has: buttons, modals/dialogs, dropdowns, comboboxes, search inputs, tabs, checkboxes, date pickers, pagination, toasts, summary cards, page headers, filter bars, icon components, etc.

**Rules:**
- If the Figma design contains an element that matches an existing component — use it.
- If an existing component needs a new variant (e.g., a new button color), add the variant to the existing component — do NOT create a new component file.
- Only create new components for genuinely new UI patterns that have no match in the codebase.

### 2. Smart Component Splitting

Never generate a single monolithic component for an entire screen. Follow these splitting rules:

- **Page component** (`src/pages/`): Orchestrates layout, data fetching (via hooks), and composes child components. Thin logic only.
- **Feature components** (`src/components/`): Self-contained UI blocks (cards, lists, modals, forms). Each gets its own file.
- **Shared/reusable components** (`src/components/`): Generic UI elements used across multiple features.

**When to split:**
- A section of the screen has its own distinct responsibility (e.g., a filter bar, a data table, a summary card row, a modal)
- A section could be reused elsewhere
- A section has its own state management
- The component exceeds ~80-100 lines of JSX — consider splitting

**Hierarchy pattern:**
```
PageName.tsx (page)
├── PageNameHeader.tsx (page-specific header if custom)
├── PageNameFilters.tsx (filter section)
├── PageNameList.tsx (list/grid of items)
│   └── PageNameCard.tsx (individual card)
├── PageNameModal.tsx (modal if present)
└── PageNameEmptyState.tsx (empty state)
```

### 3. Mobile-First Responsive Design

- **NEVER use fixed pixel widths** for containers or layouts (e.g., `w-[450px]`)
- Use responsive Tailwind classes: `w-full`, `max-w-lg`, `md:grid-cols-2`, `lg:grid-cols-3`
- Use `flex` and `grid` layouts with responsive breakpoints
- Test mental model: "Does this work on a 320px screen?"
- Images and icons should use relative sizing or constrained max dimensions
- Allowed fixed sizes: icon dimensions (`w-5 h-5`), `min-w` on buttons, `max-w` on modals/content containers

### 4. Mock Data Strategy

Since APIs may not exist yet:
- Create mock data constants at the top of the component file or in a dedicated section
- Use realistic content that matches the project's domain and language
- Structure mock data to match the expected DTO shape so swapping to real data later is trivial
- Comment mock data clearly: `// TODO: Replace with API data`
- Use 3-5 items for lists (not just 1-2) and include edge cases (long names, zero values, empty arrays)

### 5. Follow Project Steering Rules

Before generating any code, **read all steering files** present in the project's `.kiro/steering/` directory. These contain the project's specific rules for:
- Styling (Tailwind usage, forbidden patterns, allowed exceptions)
- Pages (structure, naming, patterns)
- Components (isolation, naming, placement)
- Hooks (naming, return patterns, data fetching)
- Forms (edit mode, loading states, validation)
- DTOs (optional properties, naming conventions)
- Utils (static classes, no dependencies)
- General architecture (file naming, imports, TypeScript rules)

**Every generated component must comply with all active steering rules.**

---

## Workflow

### Step 1: Fetch Figma Data

Use the Framelink MCP to get the design data:

```
Tool: mcp_Framelink_Figma_MCP_get_figma_data
Parameters:
  fileKey: "<figma-file-key>"
  nodeId: "<node-id>"   // optional, use if targeting a specific frame
```

If the design contains images or icons that need to be downloaded:

```
Tool: mcp_Framelink_Figma_MCP_download_figma_images
Parameters:
  fileKey: "<figma-file-key>"
  nodes: [{ nodeId: "...", fileName: "icon_name.svg" }]
  localPath: "public/images/icons"   // or "src/assets" depending on project convention
```

### Step 2: Analyze the Design

Before writing any code:

1. **Scan existing components** — List the project's `src/components/` directory and identify which base components already exist that match elements in the design.
2. **Read all steering files** — Load every `.md` file from `.kiro/steering/` to understand the project's rules.
3. **Identify reusable components** — Which parts of the design map to existing components?
4. **Plan the component tree** — How should this screen be split? What is the page, what are the feature components, what are shared components?
5. **Identify responsive breakpoints** — Where does the layout need to change for mobile vs desktop?
6. **Identify state** — What interactive states exist (loading, empty, error, hover, active)?
7. **Identify data shape** — What mock data structure is needed?

### Step 3: Generate Components

For each component identified in Step 2:

1. **Create the component** following all steering rules:
   - Component interface at the top of the file
   - Proper TypeScript typing (no `any`, no type casts)
   - Tailwind-only styling (no inline styles except project-allowed exceptions)
   - Responsive classes for mobile support
   - Reuse existing base components

2. **Place the file** in the correct location per the project's conventions:
   - Reusable component → `src/components/`
   - Page-specific component → page subfolder or `src/components/` with scoped naming
   - Page → `src/pages/`

### Step 4: Review and Refine

After generating all components:

1. Verify no existing component was recreated
2. Verify no inline styles were used (except allowed exceptions per steering)
3. Verify responsive design (no fixed widths on containers)
4. Verify component interfaces are properly typed
5. Verify mock data matches expected DTO shapes
6. Verify each component file has a single responsibility
7. Verify all steering rules are respected

---

## Common Patterns

### Modal from Figma

When the Figma design is a modal/dialog, use the project's existing Dialog/Modal component:

```tsx
import { Dialog, DialogHeader } from '../components/Dialog';
import { Button } from '../components/Button';

interface MyModalProps {
  isOpen: boolean;
  onClose: () => void;
}

export const MyModal: React.FC<MyModalProps> = ({ isOpen, onClose }) => {
  return (
    <Dialog isOpen={isOpen} onClose={onClose}>
      <DialogHeader title="Modal Title" onClose={onClose} />
      <div className="flex flex-col gap-4">
        {/* Modal content */}
      </div>
      <div className="flex justify-end gap-3 pt-6">
        <Button variant="outline" onClick={onClose}>Cancel</Button>
        <Button variant="primary" onClick={handleSave}>Save</Button>
      </div>
    </Dialog>
  );
};
```

### Card Component from Figma

```tsx
interface FeatureCardProps {
  title: string;
  description: string;
  icon: React.ReactNode;
  onClick?: () => void;
}

export const FeatureCard: React.FC<FeatureCardProps> = ({ title, description, icon, onClick }) => {
  return (
    <div
      className="flex items-start gap-3 p-4 rounded-xl border bg-white cursor-pointer hover:shadow-md transition-shadow"
      onClick={onClick}
    >
      <div className="flex items-center justify-center w-10 h-10 rounded-full bg-green-50">
        {icon}
      </div>
      <div className="flex flex-col gap-1 flex-1 min-w-0">
        <span className="font-bold text-sm truncate">{title}</span>
        <span className="text-xs text-gray-500">{description}</span>
      </div>
    </div>
  );
};
```

### Page with Tabs from Figma

```tsx
import { useState } from 'react';
import { TabBar } from '../components/TabBar';

type TabKey = 'overview' | 'details';

const TABS: { key: TabKey; label: string }[] = [
  { key: 'overview', label: 'Overview' },
  { key: 'details', label: 'Details' },
];

export default function FeaturePage() {
  const [activeTab, setActiveTab] = useState<TabKey>('overview');

  return (
    <div className="flex flex-col gap-6 p-6">
      <TabBar tabs={TABS} activeTab={activeTab} onTabChange={setActiveTab} />
      {activeTab === 'overview' && <OverviewSection />}
      {activeTab === 'details' && <DetailsSection />}
    </div>
  );
}
```

### List Page with Filters

```tsx
import { useState } from 'react';

// TODO: Replace with API data
const MOCK_ITEMS = [
  { id: 1, name: 'Item One', status: 'active' },
  { id: 2, name: 'Item Two', status: 'inactive' },
  { id: 3, name: 'Item Three', status: 'active' },
];

export default function ListPage() {
  const [search, setSearch] = useState('');

  return (
    <div className="flex flex-col gap-6 p-6">
      <PageHeader title="Items" />
      <FiltersBar search={search} onSearchChange={setSearch} />
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {MOCK_ITEMS.map((item) => (
          <ItemCard key={item.id} item={item} />
        ))}
      </div>
    </div>
  );
}
```

---

## Troubleshooting

### Figma MCP Returns Empty or Minimal Data

- Verify the `fileKey` and `nodeId` are correct
- Check that your Figma API key has read access to the file
- Try fetching without `nodeId` first to confirm file access, then drill into specific nodes

### Design Has Custom Fonts

- Use the project's standard font (check Tailwind config or steering files)
- Map any Figma font to the project font unless explicitly told otherwise

### Design Uses Colors Not in the Project Palette

- Check if the color is close to an existing palette color and use that
- If it's genuinely new, use the exact hex value with Tailwind arbitrary values: `text-[#HEXCODE]`

### Component Seems Too Large

- If a single component file exceeds ~100 lines of JSX, split it
- Extract repeated patterns into sub-components
- Extract complex logic into hooks or utils

---

## Best Practices

- Always scan the project's component directory before creating anything new
- When in doubt about splitting, split — smaller components are easier to maintain
- Keep page components thin — they orchestrate, they don't implement
- Use semantic HTML elements (`section`, `nav`, `header`, `main`, `aside`) where appropriate for accessibility
- Add `aria-label` to interactive elements that lack visible text
- Use `gap` instead of margins for spacing between siblings
- Prefer `grid` for 2D layouts and `flex` for 1D layouts
- Name mock data variables clearly: `MOCK_USERS`, `MOCK_PRODUCTS`
- Always type component props with an interface — never inline object types
- Follow the project's file naming convention (check steering for snake_case, kebab-case, etc.)
- Check the project's existing icon library before creating new SVG icon components

---

**MCP Server:** Framelink_Figma_MCP
**Package:** `figma-developer-mcp`
