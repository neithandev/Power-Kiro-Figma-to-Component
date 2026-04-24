---
inclusion: manual
---

# Figma-to-Component Conversion Guide

This steering file provides detailed instructions for converting Figma MCP output into production-ready React components. Activate this guide when using the Figma to Component power.

## Pre-Conversion Checklist

Before writing any code from Figma data, complete this checklist:

1. **Inventory existing components** — Read the project's component directory (`src/components/` or equivalent) and identify which base components already exist that match elements in the design.
2. **Read steering files** — Load all `.md` files from `.kiro/steering/` to understand the project's specific rules for styling, pages, components, hooks, forms, DTOs, utils, and general architecture.
3. **Plan the component tree** — Decide how to split the screen before writing a single line of code.

## Component Mapping Rules

Before generating any code, **scan the project's component directory** and read the source of every reusable component found. Build a mental inventory of what already exists.

### Discovery Process

1. List all files in `src/components/` (and subdirectories).
2. For each file, read its exported interface/props to understand what it accepts and what variants it supports.
3. While analyzing the Figma design, for **every UI element** you encounter, ask: "Does something in the project already do this?"

### Common Frontend Elements to Check For

These are UI elements that commonly exist in frontend projects. For each one, search the component directory before creating anything new:

- **Buttons** — Look for a button component with variant support. If a new visual style is needed, add a variant to the existing button — never create a separate button component.
- **Modals / Dialogs** — Look for a dialog or modal wrapper component. Customize via its props instead of building a new overlay.
- **Dropdowns / Selects** — Look for simple dropdown and searchable/combobox select components.
- **Inputs** — Look for text inputs, search inputs, textarea, number inputs, password inputs.
- **Checkboxes / Toggles / Switches** — Look for existing form control components.
- **Radio buttons** — Look for radio or radio group components.
- **Date / Time pickers** — Look for date picker or calendar input components.
- **Tabs / Tab bars** — Look for tab navigation components (may have mobile/desktop variants).
- **Pagination** — Look for page navigation components.
- **Toasts / Notifications** — Look for toast or snackbar notification systems.
- **Cards** — Look for generic card containers or domain-specific card components.
- **Headers / Page headers** — Look for page title or header bar components.
- **Footers** — Look for footer or bottom bar components.
- **Sidebars / Navigation menus** — Look for sidebar, drawer, or nav menu components.
- **Breadcrumbs** — Look for breadcrumb navigation components.
- **Badges / Tags / Chips** — Look for label or status badge components.
- **Avatars** — Look for user avatar or image circle components.
- **Tooltips / Popovers** — Look for tooltip or popover components.
- **Accordions / Collapsibles** — Look for expandable section components.
- **Tables** — Look for data table or list components.
- **Skeletons / Loading states** — Look for skeleton loader components.
- **Empty states** — Look for empty state or placeholder components.
- **Filter bars** — Look for filter control bar components.
- **Progress bars / Indicators** — Look for progress or loading bar components.
- **Icons** — Look for an icons folder or icon components. Check existing icons before creating new SVGs.
- **Layouts / Wrappers** — Look for layout containers, grid wrappers, or page shells.

### Rules

- If a matching component exists → **use it**.
- If a matching component exists but needs a small extension (new variant, new prop) → **extend the existing component**.
- If no match exists → **create a new component** in the appropriate directory.
- **The goal is zero unnecessary component duplication.**

## Splitting Strategy

### When to Create a New Component File

- The UI block has a clear, single responsibility (e.g., a card, a form section, a chart wrapper)
- The UI block manages its own local state
- The UI block could potentially be reused in another screen
- The JSX for the block exceeds ~40-50 lines

### Component Hierarchy Pattern

```
PageName.tsx (page)
├── PageNameHeader.tsx (page-specific header if custom)
├── PageNameFilters.tsx (filter section)
├── PageNameList.tsx (list/grid of items)
│   └── PageNameCard.tsx (individual card)
├── PageNameModal.tsx (modal if present)
└── PageNameEmptyState.tsx (empty state)
```

### File Placement

- **Reusable across pages** → project's shared components directory
- **Used only in one page** → page subfolder or shared components with scoped naming
- **Page orchestrator** → project's pages directory

## Responsive Design Rules

### Layout Patterns

```tsx
// Grid that adapts: 1 col mobile → 2 cols tablet → 3 cols desktop
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">

// Flex that wraps on small screens
<div className="flex flex-col md:flex-row gap-4">

// Full width on mobile, constrained on desktop
<div className="w-full max-w-2xl">
```

### Forbidden Patterns

```tsx
// ❌ NEVER — fixed width containers
<div className="w-[450px]">
<div className="w-[800px]">

// ❌ NEVER — inline styles for layout (check steering for allowed exceptions)
<div style={{ width: '450px', marginTop: '16px' }}>

// ✅ CORRECT — responsive + gap/padding
<div className="w-full max-w-lg">
<div className="flex flex-col gap-4 p-6">
```

### Allowed Fixed Sizes

- Icon dimensions: `w-5 h-5`, `w-10 h-10` (small, bounded elements)
- Border radius values in arbitrary notation when Tailwind classes don't match
- `min-w` on specific interactive elements like buttons
- `max-w` on modals and content containers

## Styling Rules

Always check the project's steering files for specific styling rules. Common patterns to follow:

- **Prefer Tailwind CSS classes** over inline styles
- **Check steering for `style` attribute exceptions** — typically only allowed for complex gradients, JS variable interpolation, complex boxShadow, or vendor-specific properties
- **Check steering for margin rules** — many projects forbid margins in favor of gap/padding/space utilities
- **Use conditional classes** for dynamic states: `${isActive ? 'bg-blue-500' : 'bg-gray-200'}`
- **Use the project's font family** (check Tailwind config or steering)
- **Use the project's color palette** (check Tailwind config, steering, or existing components)

## Mock Data Guidelines

### Structure

```tsx
// At the top of the component or in a constants section
// TODO: Replace with API data
const MOCK_ITEMS = [
  { id: 1, name: 'First Item', status: 'active', progress: 75 },
  { id: 2, name: 'Second Item', status: 'inactive', progress: 45 },
  { id: 3, name: 'Third Item', status: 'active', progress: 90 },
];
```

### Rules

- Use content that matches the project's domain and language
- Structure data to match the expected DTO shape from the project's DTOs directory
- Mark all mock data with `// TODO: Replace with API data`
- Use realistic quantities (not just 1-2 items — use 3-5 for lists)
- Include edge cases in mock data (long names, zero values, empty arrays)

## Code Quality Checklist

After generating each component, verify:

- [ ] No existing component was recreated
- [ ] No `any` type used
- [ ] No type casts (`as Type`) unless explicitly allowed by steering
- [ ] No inline `style` attribute (except project-allowed exceptions)
- [ ] Spacing follows project conventions (check steering for margin rules)
- [ ] No fixed widths on containers (responsive classes used)
- [ ] Component interface defined at top of file
- [ ] Props are properly typed with an interface
- [ ] Project's standard font used for text
- [ ] Interactive elements have proper cursor and hover states
- [ ] Loading/disabled states handled where applicable
- [ ] File placed in correct directory per project conventions
- [ ] Component has single responsibility
- [ ] Mock data clearly marked with TODO comments
- [ ] File naming follows project convention (check steering for snake_case, kebab-case, etc.)

## Conversion Process Summary

```
1. Fetch Figma data (mcp_Framelink_Figma_MCP_get_figma_data)
2. Download images/icons if needed (mcp_Framelink_Figma_MCP_download_figma_images)
3. Scan src/components/ for existing reusable components
4. Read all .kiro/steering/*.md files
5. Plan component tree (page → sections → cards/modals)
6. Generate each component following steering rules
7. Run quality checklist on every generated file
```
