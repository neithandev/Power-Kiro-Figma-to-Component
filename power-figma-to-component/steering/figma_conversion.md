---
inclusion: always
---

# Figma-to-Component Conversion Guide

This steering file provides detailed instructions for converting Figma MCP output into production-ready React components. Activate this guide when using the Figma to Component power.

> **⚠️ IMPORTANT: If the project already has an established folder structure, naming convention, or organizational pattern, ALWAYS follow the existing pattern over the defaults in this guide. Scan the project first and adapt accordingly.**

## Pre-Conversion Checklist

Before writing any code from Figma data, complete this checklist:

1. **Inventory existing components** — Read the project's component directory (`src/components/` or equivalent) and identify which base components already exist that match elements in the design.
2. **Read steering files** — Load all `.md` files from `.kiro/steering/` to understand the project's specific rules for styling, pages, components, hooks, forms, DTOs, utils, and general architecture.
3. **Read `tailwind.config`** — Understand the project's color palette, font scale, spacing scale, and any custom tokens.
4. **Scan icon library** — Check `package.json` for icon libraries (Lucide, Heroicons, Phosphor, React Icons, etc.) and the project's icon folder.
5. **Plan the component tree** — Decide how to split the screen before writing a single line of code.

## Design Token Mapping

### Color Mapping

Before generating any component code, map every color from the Figma design to the project's design system:

1. **Read `tailwind.config`** and list all existing color tokens.
2. **Extract all unique colors** from the Figma MCP data.
3. **For each Figma color:**
   - If it matches an existing Tailwind token → use it (e.g., `#EF4444` → `red-500`).
   - If it's close to an existing token (within ~5% hue/saturation/lightness) → use the existing token.
   - If it has **no match** → **register a new custom color** in `tailwind.config`:
     - Pick a descriptive name based on the color's appearance or purpose (e.g., `midnight`, `ocean`, `coral`).
     - Assign the appropriate shade number (50-950) based on lightness: lighter colors get lower numbers (50, 100, 200), darker colors get higher numbers (700, 800, 900).
     - Add it to the `colors` section of `tailwind.config`.
     - Use the new token in components (e.g., `bg-midnight-900`).

```tsx
// ❌ NEVER — arbitrary hex values in components
<div className="bg-[#1A1A2E] text-[#E94560]">

// ✅ CORRECT — named tokens (existing or newly registered)
<div className="bg-midnight-900 text-coral-500">
```

**Example `tailwind.config` addition:**
```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        // New colors from Figma design (registered during conversion)
        'midnight': {
          900: '#1A1A2E',
          800: '#16213E',
        },
        'coral': {
          500: '#E94560',
          400: '#FF6B6B',
        },
      },
    },
  },
};
```

### Font Size Mapping

Map Figma font sizes to the nearest Tailwind text utility:

| Figma px | Tailwind class |
|----------|---------------|
| 10-11px  | `text-xs`     |
| 12-13px  | `text-sm`     |
| 14-15px  | `text-base`   |
| 16-17px  | `text-lg`     |
| 18-19px  | `text-xl`     |
| 20-23px  | `text-2xl`    |
| 24-29px  | `text-3xl`    |
| 30-35px  | `text-4xl`    |
| 36+px    | `text-5xl`+   |

- If the project has a custom font scale in `tailwind.config`, use that scale instead.
- Never use arbitrary font sizes like `text-[17px]` — round to the nearest Tailwind class.

### Spacing Mapping

Map Figma spacing values to Tailwind spacing utilities:

- 4px → `1`, 8px → `2`, 12px → `3`, 16px → `4`, 20px → `5`, 24px → `6`, 32px → `8`, 40px → `10`, 48px → `12`, 64px → `16`
- For values between standard steps, round to the nearest Tailwind spacing class.
- If the project has custom spacing in `tailwind.config`, use that scale.

## Icon and Asset Strategy

### Discovery Process

Before handling any icon from the Figma design:

1. **Check `package.json`** for icon libraries: `lucide-react`, `@heroicons/react`, `phosphor-react`, `react-icons`, etc.
2. **Check the project's icon folder** — look for `src/assets/icons/`, `src/components/icons/`, `public/icons/`, or similar.
3. **Check for an Icon wrapper component** — some projects have a generic `<Icon name="..." />` component.
4. **Read existing icon imports** in a few components to understand the pattern used.
5. **Check if the project groups icons by feature** — some projects place icons alongside feature components.

### Rules

- If the Figma icon has an equivalent in the project's icon library → **use the library icon**. Do NOT download the SVG.
- If the Figma icon has no equivalent → **create an SVG icon component** inside the feature's component folder: `/components/{feature}/IconName.tsx`.
- **Never paste raw SVG markup inline** in a component's JSX. Always create a dedicated icon component.
- Icon components should accept `width`, `height`, `color`, and `className` props.
- Place icon components alongside the feature components that use them (e.g., `/components/pomodoro/PomodoroPlayIcon.tsx`).
- Name icon files with the feature prefix and descriptive name.
- **If the project already has an established icon pattern/folder structure, follow that pattern instead.**

### Icon Component Pattern

When creating a new icon component (no library match):

```tsx
interface IconProps {
  width?: number;
  height?: number;
  color?: string;
  className?: string;
}

export const PomodoroPlayIconLarge: React.FC<IconProps> = ({
  width = 32,
  height = 32,
  color = '#629737',
  className = '',
}) => {
  return (
    <svg
      width={width}
      height={height}
      viewBox="0 0 32 32"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      className={`flex-shrink-0 ${className}`}
    >
      <path d="M11 7L25 16L11 25V7Z" fill={color} />
    </svg>
  );
};
```

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
src/pages/PageName.tsx (page orchestrator)
src/components/{feature}/
├── FeatureHeader.tsx (page-specific header if custom)
├── FeatureFilters.tsx (filter section)
├── FeatureList.tsx (list/grid of items)
│   └── FeatureCard.tsx (individual card)
├── FeatureModal.tsx (modal if present)
├── FeatureEmptyState.tsx (empty state)
└── FeatureIcon.tsx (SVG icons for this feature)
src/consts/feature-config.ts (constants/config)
src/utils/feature.utils.ts (utility functions)
```

### File Placement

- **Reusable across pages** → project's shared components directory (`/components/shared/` or root `/components/`)
- **Used only in one feature** → feature subfolder (`/components/{feature}/`)
- **Multiple components for same feature** → always group in subfolder (`/components/{feature}/`)
- **Page orchestrator** → project's pages directory
- **Constants/config** → `/consts/{name}.ts`
- **Utility functions** → `/utils/{subject}.utils.ts`
- **SVG icons** → inside the feature's component folder
- **If the project already has an established pattern → follow it**

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
- **Never use arbitrary hex values** in components — always use named Tailwind tokens. If a color doesn't exist, register it in `tailwind.config` first.
- **Gradients exception** — complex `linear-gradient` / `radial-gradient` with rgba stops are allowed as inline `style` attributes. Store gradient values in `/consts/` and reference them. Simple gradients should still use Tailwind (`bg-gradient-to-b from-blue-50 to-white`).
- **Check steering for `style` attribute exceptions** — typically only allowed for complex gradients, JS variable interpolation, complex boxShadow, or vendor-specific properties
- **Check steering for margin rules** — many projects forbid margins in favor of gap/padding/space utilities
- **Use conditional classes** for dynamic states: `${isActive ? 'bg-blue-500' : 'bg-gray-200'}`
- **Use the project's font family** (check Tailwind config or steering)
- **Use the project's color palette** (check Tailwind config, steering, or existing components)
- **Order Tailwind classes consistently** — follow the ordering convention defined in the Tailwind Class Ordering section

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

## File Organization (Constants, Utils, Component Grouping)

**IMPORTANT: If the project already has an established folder structure or pattern, always follow the existing pattern. The rules below are defaults when no pattern exists.**

### Constants Extraction

Configuration objects, state mappings, option arrays, and similar static data must be extracted into dedicated files:

- Place in `/consts/` folder (or the project's equivalent)
- Name the file descriptively: `/consts/pomodoro-states.ts`, `/consts/product-categories.ts`
- Never leave large config objects inside component files
- Export as named `const` with descriptive names

```tsx
// /consts/pomodoro-states.ts
export const STATE_CONFIG = {
  initial: {
    gradient: 'linear-gradient(180deg, rgba(239, 248, 255, 1) 0%, rgba(255, 255, 255, 1) 100%)',
    borderColor: '#B2DDFF',
    chipLabel: 'Pomodoro',
  },
  study: {
    gradient: 'linear-gradient(180deg, rgba(255, 192, 188, 1) 0%, rgba(255, 255, 255, 1) 100%)',
    borderColor: '#FFB6B5',
    chipLabel: 'Estudar',
  },
  break: {
    gradient: 'linear-gradient(180deg, rgba(207, 255, 210, 1) 0%, rgba(255, 255, 255, 1) 100%)',
    borderColor: '#BBD6A5',
    chipLabel: 'Descansar',
  },
};
```

### Utility Functions Extraction

Calculation functions, formatters, parsers, and helper logic must be extracted into dedicated util files:

- Place in `/utils/` folder (or the project's equivalent)
- Name with the pattern: `/utils/{subject}.utils.ts`
- Examples: `/utils/time.utils.ts`, `/utils/currency.utils.ts`, `/utils/pomodoro.utils.ts`
- Never leave utility functions inside component files
- Functions should be pure (no side effects) and well-typed

```tsx
// /utils/time.utils.ts
export const formatTime = (seconds: number): string => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
};
```

### Component Grouping by Feature

When creating multiple components for the same feature/domain, **always group them in a subfolder**:

- Structure: `/components/{feature}/Component1.tsx`, `/components/{feature}/Component2.tsx`
- Icon components for that feature go in the same folder
- This keeps related components together and easy to find

```
src/components/
├── pomodoro/
│   ├── PomodoroTimer.tsx
│   ├── PomodoroControls.tsx
│   ├── PomodoroPlayIconLarge.tsx
│   └── PomodoroPauseIcon.tsx
├── product/
│   ├── ProductCard.tsx
│   ├── ProductList.tsx
│   └── ProductCartIcon.tsx
└── shared/
    ├── Button.tsx
    └── Dialog.tsx
```

**Rules:**
- If creating 2+ components for the same feature → create a subfolder
- If creating 1 component that is truly shared/generic → place in `/components/shared/` or root `/components/`
- Icon SVG components go inside the feature folder, not in a separate icons folder

## Inline Style Exceptions (Gradients)

Complex CSS gradients **are allowed as inline `style` attributes**. These cannot be cleanly expressed as Tailwind classes.

### Allowed inline styles

- Complex `linear-gradient` / `radial-gradient` with multiple stops or rgba values
- `background` with gradient values from constants/config
- Dynamic values interpolated from JavaScript variables
- `borderColor` when the value comes from a config/constant

### Pattern

```tsx
// ✅ CORRECT — gradient from constants, applied via style
<div
  style={{ background: STATE_CONFIG[currentState].gradient, borderColor: STATE_CONFIG[currentState].borderColor }}
  className="flex flex-col items-center gap-4 rounded-2xl border p-6"
>
```

```tsx
// ✅ CORRECT — simple gradient that fits Tailwind
<div className="bg-gradient-to-b from-blue-50 to-white">
```

```tsx
// ❌ WRONG — trying to force complex gradients into Tailwind arbitrary values
<div className="bg-[linear-gradient(180deg,rgba(239,248,255,1)_0%,rgba(255,255,255,1)_100%)]">
```

### Rule

If a gradient is simple enough for Tailwind (e.g., `bg-gradient-to-b from-blue-50 to-white`), use Tailwind. If it has specific rgba stops, multiple color stops, or complex angles, use inline `style` with the value coming from a constant in `/consts/`.

## Tailwind Class Ordering

All Tailwind classes in `className` attributes must follow a consistent order. This improves readability and makes diffs cleaner.

**Standard order:**

1. **Layout** — `flex`, `grid`, `block`, `inline`, `hidden`, `relative`, `absolute`, `fixed`, `sticky`, `z-*`
2. **Flex/Grid modifiers** — `flex-col`, `flex-row`, `items-*`, `justify-*`, `grid-cols-*`, `col-span-*`, `gap-*`
3. **Sizing** — `w-*`, `h-*`, `min-w-*`, `min-h-*`, `max-w-*`, `max-h-*`
4. **Spacing** — `p-*`, `px-*`, `py-*`, `pt-*`, `m-*`, `mx-*`, `my-*`, `mt-*`
5. **Typography** — `text-*` (size), `font-*`, `leading-*`, `tracking-*`, `truncate`, `line-clamp-*`
6. **Colors** — `text-*` (color), `bg-*`, `placeholder-*`
7. **Borders** — `border`, `border-*`, `rounded-*`, `divide-*`
8. **Effects** — `shadow-*`, `opacity-*`, `ring-*`
9. **Transitions** — `transition-*`, `duration-*`, `ease-*`
10. **Interactive states** — `hover:*`, `focus:*`, `active:*`, `disabled:*`
11. **Responsive** — `sm:*`, `md:*`, `lg:*`, `xl:*`

```tsx
// ✅ CORRECT — ordered: layout → flex → sizing → spacing → typography → colors → borders → effects → transitions → hover → responsive
<div className="flex items-center gap-3 w-full p-4 text-sm text-gray-700 bg-white border rounded-xl shadow-sm transition-shadow hover:shadow-md md:p-6">

// ❌ WRONG — random order
<div className="hover:shadow-md border text-sm p-4 flex bg-white shadow-sm rounded-xl gap-3 items-center w-full text-gray-700 transition-shadow md:p-6">
```

If the project uses `prettier-plugin-tailwindcss`, defer to its automatic ordering. Otherwise, follow the order above.

## Props and Handler Naming Convention

### Props Naming

All component props must follow consistent naming patterns:

**Boolean props** — use `is` or `has` prefix:
```tsx
interface ModalProps {
  isOpen: boolean;        // ✅ state boolean
  isLoading: boolean;     // ✅ state boolean
  isDisabled: boolean;    // ✅ state boolean
  hasError: boolean;      // ✅ existence boolean
  hasHeader: boolean;     // ✅ existence boolean
}
```

```tsx
// ❌ WRONG — ambiguous boolean names
interface ModalProps {
  open: boolean;
  loading: boolean;
  disabled: boolean;
  error: boolean;
}
```

**Event handler props** — use `on` prefix:
```tsx
interface FormProps {
  onSubmit: (data: FormData) => void;   // ✅
  onChange: (value: string) => void;     // ✅
  onClose: () => void;                  // ✅
  onDelete: (id: string) => void;       // ✅
}
```

**Internal handler functions** — use `handle` prefix:
```tsx
export const MyForm: React.FC<FormProps> = ({ onSubmit, onClose }) => {
  const handleSubmit = () => {    // ✅ internal handler
    // validation logic
    onSubmit(data);               // calls the prop
  };

  const handleClose = () => {     // ✅ internal handler
    // cleanup logic
    onClose();                    // calls the prop
  };
};
```

**Data props** — use descriptive nouns:
```tsx
interface UserCardProps {
  user: UserDTO;              // ✅ domain object
  title: string;              // ✅ simple value
  items: ProductDTO[];        // ✅ collection
  selectedId: string | null;  // ✅ selected state
}
```

### Naming Summary Table

| Type | Prefix | Examples |
|------|--------|---------|
| Boolean state | `is` | `isOpen`, `isLoading`, `isDisabled`, `isActive`, `isSelected` |
| Boolean existence | `has` | `hasError`, `hasHeader`, `hasFooter`, `hasData` |
| Event handler (prop) | `on` | `onClick`, `onClose`, `onSubmit`, `onChange`, `onDelete` |
| Event handler (internal) | `handle` | `handleClick`, `handleClose`, `handleSubmit`, `handleChange` |
| Render prop / slot | `render` | `renderHeader`, `renderFooter`, `renderItem` |

## Visual Fidelity Review

After generating all components, perform a visual fidelity review comparing the output against the original Figma design:

### Fidelity Checklist

1. **Layout** — Does the component structure match the Figma layout? Are elements in the correct order and hierarchy?
2. **Spacing** — Do gaps, paddings, and margins visually match the design? (using mapped Tailwind tokens)
3. **Colors** — Are all colors correctly mapped to project tokens? No raw hex values?
4. **Typography** — Are font sizes, weights, and line heights consistent with the design?
5. **Icons** — Are all icons present, correctly sized, and using the project's icon system?
6. **States** — Are hover, active, focus, and disabled states implemented for all interactive elements?
7. **Border radius** — Do rounded corners match the design?
8. **Shadows** — Are box shadows consistent with the design (mapped to project tokens)?

### Documenting Divergences

When the generated code intentionally deviates from the Figma design (e.g., using a project standard instead of the exact Figma value), **always add a comment** explaining the reason:

```tsx
// NOTE: Using project standard shadow-md instead of Figma shadow-xl per design system
<div className="shadow-md">

// NOTE: Using text-gray-700 (project palette) instead of Figma #4A4A4A (close match)
<span className="text-gray-700">

// NOTE: Rounded to text-sm (14px) from Figma 13px — nearest Tailwind class
<p className="text-sm">
```

This ensures the team can review divergences and decide if they need adjustment.

## Component Export Rule

**Always use `export const` with arrow function syntax for all components.** Never use `export default function` or `export function` declarations.

```tsx
// ✅ CORRECT
export const Login: React.FC<LoginProps> = ({ onSubmit }) => {
  return <div>...</div>;
};

// ❌ WRONG — do NOT use function declarations
export default function Login() { ... }
export function Login() { ... }
```

This applies to all components: pages, feature components, and shared/reusable components.

## Code Quality Checklist

After generating each component, verify:

**Export & Structure:**
- [ ] Component uses `export const` arrow function syntax (no `export default function` or `export function`)
- [ ] Component interface defined at top of file
- [ ] Props are properly typed with an interface
- [ ] Component has single responsibility
- [ ] File placed in correct directory per project conventions
- [ ] File naming follows project convention (check steering for snake_case, kebab-case, etc.)

**Reuse & Duplication:**
- [ ] No existing component was recreated
- [ ] Icons use the project's icon library or a dedicated icon component (no inline SVG)

**File Organization:**
- [ ] Constants/config objects extracted to `/consts/{name}.ts` (not inside components)
- [ ] Utility functions extracted to `/utils/{subject}.utils.ts` (not inside components)
- [ ] Multiple components for same feature grouped in `/components/{feature}/` subfolder
- [ ] SVG icon components placed inside the feature folder (not inline in JSX)
- [ ] Existing project folder patterns respected

**TypeScript:**
- [ ] No `any` type used
- [ ] No type casts (`as Type`) unless explicitly allowed by steering

**Design Tokens:**
- [ ] All colors use named Tailwind tokens (no arbitrary hex values like `text-[#HEXCODE]`)
- [ ] New colors registered in `tailwind.config` with descriptive names and shade numbers
- [ ] Font sizes mapped to Tailwind text utilities (no arbitrary `text-[17px]`)
- [ ] Spacing mapped to Tailwind spacing utilities

**Styling:**
- [ ] No inline `style` attribute (except gradients with rgba stops and dynamic values from constants)
- [ ] Complex gradients use inline `style` with values from `/consts/` (not Tailwind arbitrary values)
- [ ] Spacing follows project conventions (check steering for margin rules)
- [ ] No fixed widths on containers (responsive classes used)
- [ ] Tailwind classes ordered consistently (layout → sizing → spacing → typography → colors → borders → effects → transitions → states → responsive)
- [ ] Project's standard font used for text

**Interactivity & States:**
- [ ] Interactive elements have proper cursor and hover states
- [ ] Focus states implemented for keyboard navigation
- [ ] Loading/disabled states handled where applicable
- [ ] States not in Figma documented with `// TODO: Design pending for [state] state`

**Naming:**
- [ ] Boolean props use `is`/`has` prefix (`isOpen`, `hasError`)
- [ ] Event handler props use `on` prefix (`onClick`, `onClose`)
- [ ] Internal handlers use `handle` prefix (`handleClick`, `handleClose`)

**Data:**
- [ ] Mock data clearly marked with `// TODO: Replace with API data`
- [ ] Mock data uses realistic quantities (3-5 items for lists)

**Visual Fidelity:**
- [ ] Layout matches Figma structure
- [ ] Intentional divergences from Figma documented with `// NOTE:` comments

## Conversion Process Summary

```
1. Fetch Figma data (mcp_Framelink_Figma_MCP_get_figma_data)
2. Download images/icons if needed (mcp_Framelink_Figma_MCP_download_figma_images)
3. Read tailwind.config and map all Figma colors/fonts/spacing to project tokens
4. Register new colors in tailwind.config if no match exists (never use arbitrary hex)
5. Scan src/components/ for existing reusable components
6. Scan icon library (package.json + icon folder) — map Figma icons to library icons
7. Read all .kiro/steering/*.md files
8. Plan component tree (page → sections → cards/modals)
9. Generate each component following all steering rules, using mapped tokens
10. Run full quality checklist on every generated file
11. Perform visual fidelity review — document any divergences with // NOTE: comments
```
