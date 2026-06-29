---
inclusion: fileMatch
fileMatchPattern: "**/*.{vue,js}"
---

# Frontend Standards

## Component Architecture

### Composition API Preferred
New components use `<script setup>`. Existing Options API components can be refactored incrementally.

```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

// 1. Props & emits (top)
// 2. Composable usage
// 3. Reactive state
// 4. Computed properties
// 5. Methods
// 6. Lifecycle hooks (bottom)
</script>

<template>
  <!-- Template -->
</template>
```

### Component Types

**Views** (`views/tasks/`) — Route-level pages, handle data fetching:
```
TaskIndex.vue     — List all tasks
TaskCreate.vue    — Create form
TaskEdit.vue      — Edit form
```

**Components** (`components/`) — Reusable UI elements:
```
DataTable.vue                          — Table with actions
FormFields/InputGroupText.vue          — Text input group
FormFields/InputGroupCheckbox.vue      — Checkbox input group
FormFields/InputFields/InputFieldText.vue
FormFields/InputFields/InputFieldCheckbox.vue
```

### Component Rules
- Views fetch data (via composables), components receive via props
- Props down, events up — no direct parent mutation
- Keep templates readable — extract complex logic to computed properties
- One component per file

## Composables (Shared Logic)

All API interaction lives in composables, not in components directly.

```javascript
// composables/tasks.js
export function useTasks() {
    const tasks = ref([])
    const task = ref({})
    const errors = ref({})

    async function listTask() { /* axios call */ }
    async function createTask(data) { /* axios call */ }
    async function updateTask(id, data) { /* axios call */ }
    async function deleteTask(id) { /* axios call */ }

    return { tasks, task, errors, listTask, createTask, updateTask, deleteTask }
}
```

### Composable Rules
- Prefix with `use` (`useTasks`, `useAuth`)
- Return reactive state and methods
- Handle errors inside the composable, expose error state
- Don't call composable methods from within other composables (keep them independent)

## API Interaction

- Use axios (configured in `bootstrap.js` with base URL and CSRF)
- All API calls go through composables
- Handle loading and error states consistently
- Base URL: `/api`

### Error Handling Pattern
```javascript
async function createTask(data) {
    errors.value = {}
    try {
        const response = await axios.post('/api/tasks', data)
        return response.data
    } catch (error) {
        if (error.response?.status === 422) {
            errors.value = error.response.data.errors
        }
        throw error
    }
}
```

## Forms

- Form state is local (ref in the component)
- Submit calls a composable method
- Display server-side validation errors per field
- Disable submit button during submission
- Redirect on success

```vue
<script setup>
const { createTask, errors } = useTasks()
const form = ref({ title: '' })
const isSubmitting = ref(false)

async function handleSubmit() {
    isSubmitting.value = true
    try {
        await createTask(form.value)
        router.push({ name: 'tasks.index' })
    } finally {
        isSubmitting.value = false
    }
}
</script>
```

## Routing (Vue Router)

- Named routes for all navigation
- Auth guard on task routes (handled server-side via Blade `auth` middleware)
- Route naming: dot notation (`tasks.index`, `tasks.create`, `tasks.edit`)

```javascript
const routes = [
    { path: '/dashboard', name: 'dashboard', component: Dashboard },
    { path: '/tasks', name: 'tasks.index', component: TaskIndex },
    { path: '/tasks/create', name: 'tasks.create', component: TaskCreate },
    { path: '/tasks/:id/edit', name: 'tasks.edit', component: TaskEdit },
]
```

## Styling (Tailwind CSS)

- Utility-first — compose styles directly in templates
- No custom CSS unless unavoidable
- Consistent spacing using Tailwind's default scale
- Responsive: mobile-first approach
- Use Tailwind classes for colors, spacing, typography — don't write custom CSS for these

## Data Flow

```
View (fetch via composable)
  └── passes data as props to child components
        └── child emits events back to parent
              └── parent calls composable to mutate data
```

- Views own the data lifecycle
- Components are pure UI (props in, events out)
- Composables own API communication

## Accessibility

- All form inputs have associated labels
- Interactive elements are keyboard-navigable
- Use semantic HTML (`<button>`, `<nav>`, `<main>`, `<table>`)
- Color is not the only indicator of state (use icons + text)
- Focus management on route changes

## File Organization

```
resources/js/
├── app.js                  # Vue 3 entry, mounts on #app
├── bootstrap.js            # Axios config
├── router/
│   └── index.js            # Route definitions
├── composables/
│   └── tasks.js            # useTasks()
├── views/
│   └── tasks/
│       ├── TaskIndex.vue
│       ├── TaskCreate.vue
│       └── TaskEdit.vue
└── components/
    ├── DataTable.vue
    └── FormFields/
        ├── InputGroupText.vue
        ├── InputGroupCheckbox.vue
        └── InputFields/
            ├── InputFieldText.vue
            └── InputFieldCheckbox.vue
```
