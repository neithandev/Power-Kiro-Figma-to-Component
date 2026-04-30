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

### 5. Design Token Mapping (Colors, Fonts, Spacing)

The Figma MCP returns raw values (hex colors, px font sizes, px spacing). These must be **mapped to the project's design system** before generating any code.

**Process:**
1. Read the project's `tailwind.config` (or equivalent) to understand the existing color palette, font scale, and spacing scale.
2. For every color extracted from Figma, find the closest match in the project's Tailwind palette.
3. For every font size, map to the nearest Tailwind text class (`text-xs`, `text-sm`, `text-base`, etc.).
4. For every spacing value, map to the nearest Tailwind spacing class (`gap-2`, `p-4`, `mt-6`, etc.).

**Color mapping rules:**
- If the Figma color matches an existing Tailwind color → use it (e.g., `#EF4444` → `red-500`).
- If the Figma color is close to an existing color (within ~5% hue/saturation) → use the existing color.
- If the Figma color has **no match** in the project palette → **register it as a new custom color** in `tailwind.config` using the Tailwind shade convention, then use the new token. Example: Figma uses `#1A1A2E` which has no match → add `'midnight-900': '#1A1A2E'` to the Tailwind config `colors` section, then use `bg-midnight-900` in the component.
- **Never use arbitrary hex values** like `text-[#1A1A2E]` in components. Always create a named token first.
- When creating a new color, pick a descriptive name and assign the appropriate shade number (50-950) based on lightness.

**Font mapping rules:**
- Map Figma font sizes to the closest Tailwind text utility.
- If the project uses a custom font scale in Tailwind config, use that scale.
- Never use arbitrary font sizes like `text-[17px]` — round to the nearest Tailwind class.

### 6. Icon and Asset Strategy

Icons are the most common asset extracted from Figma. Handle them systematically:

**Discovery first:**
1. Check if the project uses an icon library (Lucide, Heroicons, Phosphor, React Icons, etc.) — look in `package.json` and existing imports.
2. Check if the project has a custom icons folder (`src/assets/icons/`, `src/components/icons/`, `public/icons/`).
3. Check if the project has an Icon wrapper component that standardizes size/color props.

**Rules:**
- If the Figma icon has an equivalent in the project's icon library → **use the library icon**. Do not download the SVG.
- If the Figma icon has no equivalent in the library → **download the SVG** via `mcp_Framelink_Figma_MCP_download_figma_images` and create a component following the project's icon pattern.
- Never paste raw SVG markup inline in a component's JSX. Always create a dedicated icon component or use the project's icon system.
- Icon components should accept `size` (or `className`) and `color` props to remain flexible.
- Place downloaded icons in the project's existing icons directory.
- Name icon files consistently with the project's convention (e.g., `IconArrowLeft.tsx`, `arrow-left.svg`).

**Icon component pattern (when no library match exists):**
```tsx
interface IconCustomProps {
  size?: number;
  className?: string;
}

export const IconCustom: React.FC<IconCustomProps> = ({ size = 24, className }) => {
  return (
    <svg
      width={size}
      height={size}
      viewBox="0 0 24 24"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      className={className}
    >
      {/* SVG paths from Figma */}
    </svg>
  );
};
```

### 7. Follow Project Steering Rules

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

### Step 2: Map Design Tokens

Before analyzing the design structure, map all raw Figma values to the project's design system:

1. **Read `tailwind.config`** (or equivalent) to understand the existing color palette, font scale, and spacing scale.
2. **Extract all unique colors** from the Figma data and map each to the closest Tailwind token. If no match exists, register a new custom color in `tailwind.config` with a descriptive name and shade number.
3. **Extract all font sizes** and map to Tailwind text utilities (`text-xs`, `text-sm`, `text-base`, etc.).
4. **Extract all spacing values** and map to Tailwind spacing utilities (`gap-2`, `p-4`, `m-6`, etc.).
5. **Document the mapping** — before generating code, list the token mapping so the user can review it (e.g., "Figma `#1A1A2E` → new token `midnight-900`").

### Step 3: Analyze the Design

Before writing any code:

1. **Scan existing components** — List the project's `src/components/` directory and identify which base components already exist that match elements in the design.
2. **Read all steering files** — Load every `.md` file from `.kiro/steering/` to understand the project's rules.
3. **Scan icon library** — Check `package.json` for icon libraries and the project's icon folder. Map Figma icons to existing library icons where possible.
4. **Identify reusable components** — Which parts of the design map to existing components?
5. **Plan the component tree** — How should this screen be split? What is the page, what are the feature components, what are shared components?
6. **Identify responsive breakpoints** — Where does the layout need to change for mobile vs desktop?
7. **Identify all UI states** — Map every interactive state visible in the design: default, hover, active, focus, disabled, loading, empty, error. For states not shown in the Figma, add `// TODO: Design pending for [state] state`.
8. **Identify data shape** — What mock data structure is needed?

### Step 4: Generate Components

For each component identified in Step 3:

1. **Create the component** following all steering rules:
   - Component interface at the top of the file
   - Proper TypeScript typing (no `any`, no type casts)
   - Tailwind-only styling (no inline styles except project-allowed exceptions)
   - Responsive classes for mobile support
   - Reuse existing base components
   - Use mapped design tokens (never raw hex values)
   - Use icon library components (never inline SVG)

2. **Place the file** in the correct location per the project's conventions:
   - Reusable component → `src/components/`
   - Page-specific component → page subfolder or `src/components/` with scoped naming
   - Page → `src/pages/`

### Step 5: Visual Fidelity Review

After generating all components, compare the output against the original Figma design (screenshot or MCP data):

1. **Layout fidelity** — Does the component structure match the Figma layout? Are elements in the correct order and hierarchy?
2. **Spacing fidelity** — Do gaps, paddings, and margins visually match the design? (using the mapped Tailwind tokens)
3. **Color fidelity** — Are all colors correctly mapped to project tokens? No raw hex values in components?
4. **Typography fidelity** — Are font sizes, weights, and line heights consistent with the design?
5. **Icon fidelity** — Are all icons present and correctly sized?
6. **State fidelity** — Are hover, active, focus, and disabled states implemented?
7. **Document divergences** — If any aspect intentionally deviates from the Figma (e.g., using project's `shadow-md` instead of Figma's `shadow-xl`), add a comment explaining why: `// NOTE: Using project standard shadow-md instead of Figma shadow-xl per design system`
8. Verify no existing component was recreated
9. Verify responsive design (no fixed widths on containers)
10. Verify component interfaces are properly typed
11. Verify mock data matches expected DTO shapes
12. Verify each component file has a single responsibility
13. Verify all steering rules are respected

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

export const FeaturePage = () => {
  const [activeTab, setActiveTab] = useState<TabKey>('overview');

  return (
    <div className="flex flex-col gap-6 p-6">
      <TabBar tabs={TABS} activeTab={activeTab} onTabChange={setActiveTab} />
      {activeTab === 'overview' && <OverviewSection />}
      {activeTab === 'details' && <DetailsSection />}
    </div>
  );
};
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

export const ListPage = () => {
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
};
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

- Read the project's `tailwind.config` to understand the existing palette.
- If the Figma color is close to an existing palette color (within ~5% hue/saturation) → use the existing token.
- If the Figma color has no match → **register it as a new custom color** in `tailwind.config` with a descriptive name and shade number (e.g., `'midnight-900': '#1A1A2E'`), then use the new token (`bg-midnight-900`).
- **Never use arbitrary hex values** like `text-[#1A1A2E]` in components.

### Component Seems Too Large

- If a single component file exceeds ~100 lines of JSX, split it
- Extract repeated patterns into sub-components
- Extract complex logic into hooks or utils

---

## Best Practices

- **Always use `export const` arrow function syntax** for all components — never use `export default function` or `export function`. Example: `export const MyComponent: React.FC<Props> = (props) => { ... };`
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
- **Never use raw hex values** in components — always map to a named Tailwind token (existing or newly registered)
- **Never paste raw SVG inline** in component JSX — always use the project's icon system or create a dedicated icon component
- **Order Tailwind classes consistently**: layout → sizing → spacing → typography → colors → borders → effects → transitions → responsive modifiers. If the project uses `prettier-plugin-tailwindcss`, follow its ordering.
- **Props naming convention**: boolean props use `is`/`has` prefix (`isOpen`, `isLoading`, `hasError`); event handler props use `on` prefix (`onClick`, `onClose`, `onSubmit`); internal handler functions use `handle` prefix (`handleClick`, `handleClose`, `handleSubmit`)
- **Document visual divergences** — when the generated code intentionally differs from the Figma design (e.g., using project standard instead of Figma value), add a comment explaining the reason

---

**MCP Server:** Framelink_Figma_MCP
**Package:** `figma-developer-mcp`
