# AGENTS.md

This document provides instructions for LLM agents working with the David R. Comba Lareu DevLog site.

## Repository Structure

```
davidc.github.io/
├── src/                    # Source code (Vite + Vue + Vuetify)
│   ├── public/
│   │   └── devlog.json    # Dev log entries data
│   └── src/
│       ├── App.vue        # Main application component
│       ├── main.js       # Vue app entry point
│       └── style.css     # Global styles
└── .git/
```

## Tech Stack

- **Vue 3** with Composition API (`<script setup>`)
- **Vuetify 4** with auto-import via `vite-plugin-vuetify`
- **Vite 8** as build tool
- **@mdi/font** for Material Design icons

## Build Commands

```bash
cd src

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production (outputs to src/dist/)
npm run build

# Preview production build
npm run preview
```

**Note:** This project does not have a test framework configured. Do not add tests.

## Code Style Guidelines

### Vue 3 Composition API

- Use `<script setup>` syntax for all Vue components
- Prefer composition functions (`composables`) over mixins
- Use `ref` for primitives, `reactive` for objects
- Destructure props with `defineProps` using reactive defaults

```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  item: { type: Object, required: true }
})

const searchQuery = ref('')
const isLoading = ref(false)

const filteredItems = computed(() => {
  return props.items.filter(item => 
    item.name.includes(searchQuery.value)
  )
})

onMounted(() => {
  console.log('Component mounted')
})
</script>
```

### Template Conventions

- Use kebab-case for HTML attributes and directives
- Prefer `v-for` with `:key` on list items
- Use `v-slot` for named slots
- Keep templates readable; extract complex logic to computed properties

```vue
<!-- Good -->
<v-btn icon @click="handleClick">
  <v-icon>mdi-plus</v-icon>
</v-btn>

<!-- Bad -->
<v-btn icon @click="() => { count++ }"></v-btn>
```

### Vuetify Component Usage

- Use Vuetify 4 component variants (`variant="tonal"`, `variant="outlined"`)
- Follow Vuetify spacing conventions (`pa-4`, `ma-2`, `py-8`)
- Use built-in colors (`primary`, `secondary`, `error`, etc.)
- Leverage Vuetify's density system (`density="compact"`)

```vue
<v-card variant="elevated" class="pa-4">
  <v-btn color="primary" variant="tonal">Action</v-btn>
</v-card>
```

### TypeScript

This project uses JavaScript (not TypeScript). However:
- Use JSDoc comments for complex functions
- Document prop types in comments when non-obvious
- Keep type-like clarity in variable naming

### Naming Conventions

- **Components**: PascalCase (`AppHeader.vue`, `UserCard.vue`)
- **Variables/functions**: camelCase (`isLoading`, `handleSubmit`)
- **Constants**: SCREAMING_SNAKE_CASE for config objects
- **CSS classes**: kebab-case with BEM-like prefixes (`.activity-block`, `.month-column`)
- **Vue refs**: prefix with meaningful context (`logs`, `selectedTags`)

### Import Order

```javascript
// 1. Vue core
import { ref, computed, onMounted } from 'vue'

// 2. External libraries
import { createVuetify } from 'vuetify'
import '@mdi/font/css/materialdesignicons.css'

// 3. Internal modules
import App from './App.vue'
import './style.css'
```

### Error Handling

- Use try/catch for async operations
- Log errors with descriptive messages
- Display user-friendly error states via Vuetify alerts

```javascript
onMounted(async () => {
  try {
    const response = await fetch('/data.json')
    data.value = await response.json()
  } catch (error) {
    console.error('Failed to load data:', error)
    errorMessage.value = 'Failed to load data'
  }
})
```

### CSS/Style Guidelines

- Use scoped styles in Vue components
- Prefer Vuetify utility classes over custom CSS when possible
- Use CSS custom properties for theme colors
- Follow existing patterns in `style.css` and component `<style>` blocks

## Dev Log JSON Structure

The dev log entries are stored in `src/public/devlog.json`. Each entry:

```json
{
  "id": 1,
  "date": "2026-03-14",
  "title": "Entry Title",
  "content": "Description of the update",
  "tags": ["update", "launch"],
  "link": "https://example.com",
  "images": ["https://example.com/image1.jpg"],
  "commits": 15
}
```

### Available Tags

| Tag         | Color   | Description                          |
|-------------|---------|--------------------------------------|
| `apps`      | Blue    | Application releases or updates      |
| `launch`    | Green   | New launches (counts as "ships")     |
| `code`      | Amber   | Code-related updates                 |
| `update`    | Orange  | General updates                      |
| `content`   | Purple  | Content creation                     |
| `3d-print`  | Cyan    | 3D printing projects                 |
| `design`    | Pink    | Design work                          |

## Deploying

```bash
cd src
npm run build

# Copy to public branch
git checkout public
cp -r src/dist/* .
git add -A
git commit -m "Update site: $(date +%Y-%m-%d)"
git checkout main
```

## GitHub API (gh CLI)

```bash
# Count commits since date
gh api "repos/{owner}/{repo}/commits?since=2026-03-01" --jq 'length'

# Get latest release
gh api repos/{owner}/{repo}/releases/latest --jq '.tag_name'
```

## Activity Grid Behavior

- Colored blocks represent days with updates
- Block color matches primary tag of that day's entry
- Clicking filters to show that day's entry, then others from same month
- Shows most recent year with data
