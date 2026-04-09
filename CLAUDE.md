# figma-ds-cli

CLI that controls Figma Desktop directly. No API key needed.

## Quick Reference

| User says | Command |
|-----------|---------|
| "connect to figma" | `node src/index.js connect` |
| "add shadcn colors" | `node src/index.js tokens preset shadcn` |
| "add tailwind colors" | `node src/index.js tokens tailwind` |
| "show colors on canvas" | `node src/index.js var visualize` |
| "create dashboard" | `node src/index.js blocks create dashboard-01` |
| "list blocks" | `node src/index.js blocks list` |
| "create cards/buttons" | `render-batch` + `node to-component` |
| "create a rectangle/frame" | `node src/index.js render '<Frame>...'` |
| "convert to component" | `node src/index.js node to-component "ID"` |
| "list variables" | `node src/index.js var list` |
| "find nodes named X" | `node src/index.js find "X"` |
| "what's on canvas" | `node src/index.js canvas info` |
| "export as PNG/SVG" | `node src/index.js export png` |
| "show all variants" | `node src/index.js combos` |
| "create size variants" | `node src/index.js sizes --base small` |
| "create a slot" | `node src/index.js slot create "Name"` |
| "list slots" | `node src/index.js slot list` |
| "reset slot" | `node src/index.js slot reset` |
| "verify creation" | `node src/index.js verify` |
| "check contrast" | `node src/index.js a11y contrast` |
| "color blindness sim" | `node src/index.js a11y vision` |
| "check touch targets" | `node src/index.js a11y touch` |
| "check text a11y" | `node src/index.js a11y text` |
| "full a11y audit" | `node src/index.js a11y audit` |

**Full command reference:** See REFERENCE.md

---

## Key Rules

1. **Always use `render` for frames** - has smart positioning
2. **Never use `eval` to create** - no positioning, overlaps at (0,0)
3. **Never use `npx figma-use render`** - no smart positioning
4. **For multiple frames:** Use `render-batch`
5. **Convert to components:** `node to-component` after creation
6. **Never show terminal commands to users.** Run silently, give friendly feedback.
7. **NEVER delete existing nodes** - users may have components they want to keep!
8. **Always verify after creating:** `node src/index.js verify "NODE_ID"`

---

## AI Verification

After creating any component, run `verify` to get a small screenshot for validation:

```bash
node src/index.js verify              # Screenshot of selection
node src/index.js verify "123:456"    # Screenshot of specific node
```

Returns JSON with base64 image (max 2000px). This is for internal AI checks, not shown to users.

---

## Blocks (Pre-built UI Layouts)

**ALWAYS use `blocks create` for dashboards and page layouts.** Never build them manually.

```bash
node src/index.js blocks list                    # Show available blocks
node src/index.js blocks create dashboard-01     # Create dashboard in Figma
```

**dashboard-01**: Full analytics dashboard (sidebar, stats cards, area chart, data table). All colors bound to shadcn variables (Light/Dark mode). Block source files: `src/blocks/`

---

## Design Tokens

```bash
node src/index.js tokens preset shadcn   # 244 primitives + 32 semantic (Light/Dark)
node src/index.js tokens tailwind        # 242 primitive colors only
node src/index.js tokens ds              # IDS Base colors
node src/index.js var delete-all         # Delete all variables
node src/index.js var delete-all -c "primitives"  # Only specific collection
```

- `tokens preset shadcn` = Full system (primitives + semantic with Light/Dark mode)
- `tokens tailwind` = Just the Tailwind color palette (primitives only)
- `var list` only SHOWS variables. Use `tokens` commands to CREATE them.

---

## Variable Binding (var: syntax)

Use `var:name` to bind variables at creation time. Works with `render`, `create`, and `set` commands:

```bash
# JSX render
node src/index.js render '<Frame bg="var:card" stroke="var:border" rounded={12} p={24}>
  <Text color="var:foreground" size={18}>Title</Text>
</Frame>'

# Create commands
node src/index.js create rect "Card" --fill "var:card" --stroke "var:border"

# Set commands
node src/index.js set fill "var:primary"
```

**Available shadcn variables:** `background`, `foreground`, `card`, `primary`, `secondary`, `muted`, `accent`, `border`, `input`, `ring`, and their `-foreground` variants.

---

## Connection Modes

**Yolo Mode (Recommended):** `node src/index.js connect` - Patches Figma once, fully automatic.

**Safe Mode:** `node src/index.js connect --safe` - Plugin-based, no Figma modification. Then: Plugins > Development > FigCli.

**Safe Mode caveat:** `render-batch` does NOT render text properly. Use `eval` with direct Figma API for components with text (see REFERENCE.md "Safe Mode Component Creation").

---

## JSX Syntax (render command)

```jsx
// Layout
flex="row"              // or "col"
gap={16}                // spacing
p={24}                  // padding all sides
px={16} py={8}          // padding x/y
pt={8} pr={16} pb={8} pl={16}

// Alignment
justify="center"        // main axis: start, center, end, between
items="center"          // cross axis: start, center, end

// Size
w={320} h={200}         // fixed
w="fill" h="fill"       // fill parent
minW={100} maxW={500} minH={50} maxH={300}

// Appearance
bg="#fff"               // fill color
bg="var:card"           // bind to variable
stroke="#000"           // stroke
strokeWidth={2}         strokeAlign="inside"
opacity={0.8}           blendMode="multiply"

// Corners & Effects
rounded={16}            // all corners
roundedTL={8} roundedTR={8} roundedBL={0} roundedBR={0}
cornerSmoothing={0.6}   // iOS squircle
shadow="4px 4px 12px rgba(0,0,0,0.25)"
blur={8}                overflow="hidden"       rotate={45}

// Auto-Layout
wrap={true}             // flow to next row (HORIZONTAL only)
rowGap={12}             // gap between rows
grow={1}                // fill remaining space
stretch={true}          // fill cross-axis
position="absolute" x={12} y={12}  // must have name for x/y

// Text
<Text size={18} weight="bold" color="#000" font="Inter">Hello</Text>

// Icons (real SVG via Iconify API)
<Icon name="lucide:home" size={20} color="#fff" />
<Icon name="lucide:check" size={14} color="var:primary-foreground" />

// Slots (inside components)
<Slot name="Content" flex="col" gap={8} w="fill" />
```

**Common mistakes (silently ignored, no error!):**
```
WRONG                    RIGHT
layout="horizontal"   →  flex="row"
padding={24}          →  p={24}
fill="#fff"           →  bg="#fff"
cornerRadius={12}     →  rounded={12}
fontSize={18}         →  size={18}
fontWeight="bold"     →  weight="bold"
```

---

## Critical Pitfalls

### 1. Text gets cut off (MOST COMMON BUG)

**Rule:** For text to wrap, BOTH parent AND every Text element need `w="fill"`:

```jsx
// BAD: Text clips to single line
<Frame flex="col" gap={8}>
  <Text size={16} weight="semibold" color="#fff">Long title gets cut off</Text>
  <Text size={14} color="#a1a1aa">Description also cut off</Text>
</Frame>

// GOOD: w="fill" on parent AND ALL text elements
<Frame flex="col" gap={8} w="fill">
  <Text size={16} weight="semibold" color="#fff" w="fill">Long title wraps correctly</Text>
  <Text size={14} color="#a1a1aa" w="fill">Description wraps correctly</Text>
</Frame>
```

This applies to ALL text: titles, descriptions, labels, any multi-word text.

### 2. Toggle switches: use flex, not absolute positioning

```jsx
// ON (knob right)
<Frame w={52} h={28} bg="#3b82f6" rounded={14} flex="row" items="center" p={2} justify="end">
  <Frame w={24} h={24} bg="#fff" rounded={12} />
</Frame>
// OFF (knob left)
<Frame w={52} h={28} bg="#27272a" rounded={14} flex="row" items="center" p={2} justify="start">
  <Frame w={24} h={24} bg="#52525b" rounded={12} />
</Frame>
```

### 3. Buttons need flex for centered text

```jsx
<Frame bg="#3b82f6" px={16} py={10} rounded={10} flex="row" justify="center" items="center">
  <Text color="#fff">Button</Text>
</Frame>
```

### 4. No emojis: use Lucide icons or shapes

```jsx
// BAD: Emojis render inconsistently
<Text>🏠</Text>

// GOOD: Real Lucide icons
<Icon name="lucide:home" size={20} color="#fff" />

// OK: Shape placeholders
<Frame w={20} h={20} rounded={4} stroke="#fff" strokeWidth={2} />
```

### 5. Three-dot menu / Star rating with shapes

```jsx
// Three dots
<Frame flex="row" gap={3} justify="center" items="center">
  <Frame w={4} h={4} bg="#52525b" rounded={2} />
  <Frame w={4} h={4} bg="#52525b" rounded={2} />
  <Frame w={4} h={4} bg="#52525b" rounded={2} />
</Frame>

// Star rating
<Frame flex="row" gap={4}>
  <Frame w={14} h={14} bg="#fbbf24" rounded={2} />
  <Frame w={14} h={14} bg="#fbbf24" rounded={2} />
  <Frame w={14} h={14} bg="#fbbf24" rounded={2} />
</Frame>
```

### 6. Push items to edges (navbar): use grow spacer

```jsx
// justify="between" doesn't work reliably
<Frame flex="row" items="center">
  <Frame>Logo</Frame>
  <Frame grow={1} />
  <Frame>Buttons</Frame>
</Frame>
```

### 7. Slots: isSlot does NOT work in eval

Setting `frame.isSlot = true` in eval will NOT create a slot. Use:
```bash
node src/index.js slot convert "frame-id" --name "SlotName"
```

---

## Slots (Quick Reference)

```bash
node src/index.js slot create "Content" --flex col --gap 8 --padding 16
node src/index.js slot list
node src/index.js slot preferred "Slot#1:2" "comp-id-1" "comp-id-2"
node src/index.js slot reset
node src/index.js slot convert --name "Actions"    # Convert frame to slot
node src/index.js slot add "slot-id" --component "comp-id"
```

JSX: `<Slot name="Content" flex="col" gap={8} w="fill" />` (creates real slot when parent is component)

---

## Onboarding

1. Run `npm install` silently
2. Ask connection mode (Yolo or Safe)
3. Run `node src/index.js connect` (or `--safe`)
4. When connected, say: "Connected! What would you like to create?"

If permission error (macOS): System Settings > Privacy > Full Disk Access > Add Terminal

---

## Other Commands

```bash
node src/index.js var visualize              # Show colors on canvas
node src/index.js var visualize "primitives" # Filter by collection
node src/index.js recreate-url "URL"         # Recreate webpage in Figma
node src/index.js screenshot-url "URL"       # Screenshot webpage
node src/index.js daemon status              # Check daemon
node src/index.js daemon restart             # Restart daemon
```

For eval patterns, layout examples, and Safe Mode templates, see REFERENCE.md.
