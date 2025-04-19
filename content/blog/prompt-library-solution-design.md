+++
 title = "Prompt Library Solution Design"
 description = "Solution design for the Cozy Coder prompt library."
 date = 2025-04-19T18:14:00+00:00
 updated = 2025-04-19T18:14:00+00:00
 draft = false
 template = "blog/page.html"

 [taxonomies]
 authors = ["jehrhardt"]

 [extra]
 lead = "This is the solution design for the Cozy Coder prompt library. It provides a detailed overview of the architecture, goals, and non-goals of the application."
 +++

## Context
Cozy Coder is a desktop application built with Tauri that allows users to create, organize, and use prompts for AI systems. The application persists data using Automerge CRDT (Conflict-free Replicated Data Type) documents, enabling consistent data management with potential for future collaborative features.

The application uses a two-layer architecture:
- **Front-end**: TypeScript and Solid.js running in a webview
- **Back-end**: Rust providing file system access and data management

## Goals / Non-goals

### Goals
- Create a desktop application for managing AI prompts
- Store prompts in an Automerge document for conflict-free data management
- Provide intuitive navigation and organization of prompts
- Enable basic CRUD operations for prompts and prompt categories
- Support rich text editing for prompts

### Non-goals
- Cloud synchronization (added in future versions)
- Multi-user real-time collaboration (though Automerge supports this for future expansion)
- Integration with specific AI platforms

## Solution Design

### Prompt Library Structure in Automerge Document

The Automerge document will use the following structure:

```typescript
interface PromptLibrary {
  version: string;
  lastModified: number; // Unix timestamp
  categories: Map<string, Category>; // Map of category IDs to categories
  prompts: Map<string, Prompt>; // Map of prompt IDs to prompts
}

interface Category {
  id: string;
  name: string;
  description: string;
  color: string; // Hex color code for UI display
  parentId: string | null; // For nested categories
  createdAt: number;
  updatedAt: number;
}

interface Prompt {
  id: string;
  title: string;
  content: string; // The actual prompt text
  description: string;
  categoryId: string;
  favorite: boolean;
  createdAt: number;
  updatedAt: number;
}
```

Automerge's built-in data types will be used to handle concurrent modifications:
- `Map` for categories and prompts to enable efficient lookups by ID (UUID v4)
- `Set` for tags to prevent duplicates
- Arrays for variable lists

#### Handling Text Content

The `content` field of prompts will store rich text as a string, using Markdown format. For more complex text editing needs, we could use Automerge's text type, but a simple string should be sufficient for most prompt use cases.

### Document Storage on File System

The Automerge document will be stored on the file system as follows:

1. **Location**: The application will store data in the standard application data directory for each platform by using Tauri's [`appDataDir`](https://tauri.app/reference/javascript/api/namespacepath/#appdatadir) method.:

2. **File Format**: The main document will be stored as `{document_id}.cozycoder` in binary format, where `document_id` is a UUID v4 used by Automerge.

3. **Saving Strategy**:
   - Manual save option available in the UI

### Backend API (Rust)

The Rust backend will expose the following commands via Tauri's command system:

#### Prompt Library Management

```rust
// Create a new empty prompt library
fn init_prompt_library() -> Result<PromptLibrary, Error>;

// Load p filesystem
fn load_prompt_library(path: Option<String>) -> Result<PromptLibrary, Error>;
```

#### Prompt Management

```rust
// Get all prompts
fn get_prompts() -> Result<Vec<PromptSummary>, Error>;

// Get prompt by ID
fn get_prompt(id: String) -> Result<Prompt, Error>;

// Create new prompt
fn create_prompt(prompt: NewPrompt) -> Result<Prompt, Error>;

// Update prompt
fn update_prompt(id: String, changes: PromptChanges) -> Result<Prompt, Error>;

// Delete prompt
fn delete_prompt(id: String) -> Result<(), Error>;

// Search prompts
fn search_prompts(query: SearchQuery) -> Result<Vec<PromptSummary>, Error>;
```

#### Category Management

```rust
// Get all categories
fn get_categories() -> Result<Vec<Category>, Error>;

// Create category
fn create_category(category: NewCategory) -> Result<Category, Error>;

// Update category
fn update_category(id: String, changes: CategoryChanges) -> Result<Category, Error>;

// Delete category
fn delete_category(id: String, reassign_to: Option<String>) -> Result<(), Error>;
```

#### Error Handling

The backend will use a structured error handling approach:

```rust
enum ErrorKind {
    LibraryNotFound,
    InvalidLibrary,
    PromptNotFound,
    CategoryNotFound,
    ValidationError,
    IOError,
    SerializationError,
}
```

### Frontend Implementation (Solid.js)

#### State Management

The frontend will use Solid.js stores to manage application state:

```typescript
// Global store
const createLibraryStore = () => {
  const [state, setState] = createStore({
    prompts: {} as Record<string, PromptSummary>,
    categories: {} as Record<string, Category>,
    isLoading: true,
    selectedPromptId: null as string | null,
    selectedCategoryId: null as string | null,
  });

  return [state, setState] as const;
};
```

#### Routing

Routing will be handled using Solid Router with the following structure:

```typescript
const routes = [
  {
    path: "/",
    component: Dashboard,
    children: [
      { path: "/", component: PromptList },
      { path: "/prompts/:id", component: PromptDetail },
      { path: "/categories", component: CategoryManagement },
    ],
  },
];
```

URL patterns will be designed to support direct navigation to specific prompts:
- `/prompts/:id` - View/edit a specific prompt
- `/category/:id` - View prompts in a specific category
