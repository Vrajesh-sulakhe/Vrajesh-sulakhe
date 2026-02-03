# NexRepo - Technical Design Specification

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌──────────────────────────────────────────────────────┐
│                    USER LAYER                        │
│             (Web Browser Interface)                  │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│                 FRONTEND LAYER                       │
│              (React Application)                     │
│                                                      │
│  ┌──────────┐ ┌───────────┐ ┌──────┐ ┌───────────┐   │
│  │  Upload  │ │ Dashboard │ │ Chat │ │   Code    │   │
│  │    UI    │ │     UI    │ │  UI  │ │  Viewer   │   │
│  └──────────┘ └───────────┘ └──────┘ └───────────┘   │
└────────────────────┬─────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────┐
│            CODE PROCESSING LAYER                     │
│           (Client-Side JavaScript)                   │
│                                                      │
│  ┌─────────────┐ ┌──────────────┐ ┌──────────────┐   │
│  │    File     │ │   Pattern    │ │  Dependency  │   │
│  │   Parser    │ │  Recognizer  │ │    Mapper    │   │
│  │(tree-sitter)│ │  (Regex+AST) │ │ (Import Ana.)│   │
│  └─────────────┘ └──────────────┘ └──────────────┘   │
└──────────┬──────────────────────┬────────────────────┘
           │                      │
           ▼                      ▼
┌─────────────────────┐  ┌─────────────────────────────┐
│    AI ENGINE        │  │    DATA STORAGE             │
│      (LLM)          │  │  (Client-side persistence)  │
│                     │  │                             │
│  • Onboarding paths │  │  • Parsed codebase          │
│  • Q&A responses    │  │  • User progress            │
│  • Explanations     │  │  • Chat history             │
└─────────────────────┘  └─────────────────────────────┘
```

### 1.2 Architecture Principles

1. **Client-First Architecture**: All processing happens in the browser (no backend server for MVP)
2. **Progressive Enhancement**: Core features work offline after initial load
3. **Modular Design**: Components are independent, easy to test and replace
4. **Privacy by Design**: Code never leaves user's browser unless explicitly shared
5. **Performance First**: Optimize for perceived performance (streaming, lazy loading)

### 1.3 Key Differentiator

Unlike generic AI chat tools, NexRepo:
- Understands the structure of the entire codebase
- Adapts explanations to the user's learning progress
- Focuses on onboarding, not code generation

---

## 2. Component Design

### 2.1 Frontend Components

#### 2.1.1 Upload Component

**Purpose**: Handle codebase upload via GitHub URL or ZIP file

**Design Considerations**:
- Drag-and-drop for ZIP files (better UX)
- Validate GitHub URL format before processing
- Show file extraction progress (especially for large ZIPs)
- Handle network errors gracefully (GitHub API rate limits)

---

#### 2.1.2 Dashboard Component

**Purpose**: Display onboarding plan and track progress

**Design Considerations**:
- Visual hierarchy: Day → Module → Completion
- Clear CTAs ("Start Learning", "Continue", "Mark Complete")
- Celebration animation when module completed
- Estimated time remaining calculation

---

#### 2.1.3 Chat Component

**Purpose**: Conversational interface for asking questions

**Design Considerations**:
- Streaming responses (show text as it's generated, not all at once)
- Clickable code references (open file in Code Viewer)
- Markdown formatting (code blocks, lists, bold, links)
- Context awareness (remember what module user is on)
- Suggested questions to reduce "blank canvas" problem

---

#### 2.1.4 Code Viewer Component

**Purpose**: Display code with syntax highlighting and navigation

**Design Considerations**:
- Lazy loading (don't load all files at once)
- Virtual scrolling for large files (performance)
- Breadcrumb navigation (show current file path)
- Search with fuzzy matching (like VS Code)
- Keyboard shortcuts (Cmd+P for file search)

---

#### 2.1.5 Flow Visualization Component

**Purpose**: Show interactive diagrams of code execution flows

**Design Considerations**:
- Use a graph visualization library for interactive node graphs
- Color coding by node type (start=green, process=blue, decision=yellow, end=red)
- Smooth animations between steps (not jarring)
- Zoom and pan controls for complex flows
- Generate flows dynamically based on codebase analysis

---

### 2.2 Processing Layer

#### 2.2.1 File Parser

**Purpose**: Parse code files into Abstract Syntax Trees (AST)

**Technology**: tree-sitter (WASM build)

**Supported Languages**: JavaScript, TypeScript, Python, Java (MVP)

**Design Considerations**:
- Parse files in batches (don't block UI)
- Use Web Workers for parsing (keep UI responsive)
- Handle parsing errors gracefully (skip unparseable files)

---

### 2.2.2 Structural Analysis
**Purpose**: Identify patterns and relationships between files to support onboarding and Q&A.


### 2.3 AI Integration Layer

#### 2.3.1 Onboarding Plan Generator

**Purpose**: Generate personalized learning curriculum

---

#### 2.3.2 Conversational Q&A

**Purpose**: Answer questions with codebase-specific context

---

### 2.4 Data Storage Layer

#### 2.4.1 Client-side Persistence

**Structure**:
Client-side persistence is used to store:
- Parsed metadata
- Onboarding plan
- User progress
- Chat history

---

## 3. Data Flow Diagrams

### 3.1 Upload & Analysis Flow

```
[User uploads codebase]
       ↓
[Validate input (URL or ZIP)]
       ↓
[Extract files]
   ├─ GitHub URL → Fetch via API → Get file contents
   └─ ZIP file → Unzip in browser → Extract files
       ↓
[Parse files with tree-sitter]
   ├─ JavaScript → Parse with JS parser
   ├─ Python → Parse with Python parser
   └─ Java → Parse with Java parser
       ↓
[Extract metadata]
   ├─ Functions, classes, imports
   ├─ File relationships
   └─ Complexity metrics
       ↓
[Detect patterns]
   ├─ Authentication
   ├─ API routes
   ├─ Database operations
   └─ State management
       ↓
[Build dependency graph]
   ├─ Import relationships
   └─ Identify critical files
       ↓
[Generate onboarding plan] (Large Language Model (LLM))
   ├─ Prioritize modules
   ├─ Estimate time
   └─ Create curriculum
       ↓
[Save to Client-side persistence]
   ├─ Codebase structure
   ├─ Onboarding plan
   └─ Initial progress
       ↓
[Display dashboard]
```

### 3.2 Question & Answer Flow

```
[User asks question in chat]
       ↓
[Extract keywords from question]
       ↓
[Search relevant files]
   ├─ Match keywords in file paths
   ├─ Match in function names
   └─ Match in file content
       ↓
[Retrieve top 3 relevant files]
       ↓
[Build context]
   ├─ User's current module
   ├─ Completed modules
   ├─ Conversation history (last 5)
   └─ Relevant code snippets
       ↓
[Construct prompt for Large Language Model (LLM)]
   ├─ System prompt (codebase info)
   ├─ User context
   ├─ Code snippets
   └─ Question
       ↓
[Call Large Language Model (LLM) (streaming)]
       ↓
[Stream response to UI]
   ├─ Show text as it's generated
   └─ Parse for code references
       ↓
[Extract code references]
   ├─ File paths
   └─ Line numbers
       ↓
[Make references clickable]
       ↓
[Save message to chat history]
       ↓
[Update Client-side persistence]
```

### 3.3 Progress Tracking Flow

```
[User completes module]
       ↓
[Mark module as complete]
       ↓
[Update progress state]
   ├─ Add to completedModules[]
   ├─ Increment timeSpent
   └─ Set completedAt timestamp
       ↓
[Calculate new progress percentage]
       ↓
[Determine next module]
   ├─ Get next incomplete module
   └─ Update currentModule
       ↓
[Show celebration animation]
   └─ Confetti or checkmark
       ↓
[Save to Client-side persistence]
       ↓
[Update dashboard UI]
   ├─ Update progress bar
   ├─ Check off completed module
   └─ Highlight next module
       ↓
[Generate suggested next steps]
   └─ "Ready for: Database Schema"
```

---

## 4. Security Considerations

### 4.1 API Key Protection

- API keys are managed via environment variables.
- For MVP, client-side usage is acceptable with clear limitations.


### 4.2 Code Privacy

**User code never sent to our servers**:
- All parsing happens in browser
- Only small code snippets sent to Large Language Model (LLM)
- No logging or storage of user code on our end

### 4.3 XSS Prevention

- User-generated content is sanitized before rendering to prevent XSS.

---

## 5. Error Handling

- Errors are surfaced with clear, user-friendly messages and safe fallbacks.

---

## 6. Design Decisions & Rationale

### 6.1 Why Client-Side Architecture?

**Decision**: Build entirely in browser (no backend for MVP)

**Rationale**:
1. **Speed**: Faster development (10 days for hackathon)
2. **Cost**: Zero infrastructure costs
3. **Privacy**: User code never leaves their browser
4. **Simplicity**: No server management, scaling concerns

**Trade-offs**:
- ❌ Can't sync across devices
- ❌ API key security less robust
- ❌ Limited by browser storage (5MB)

**Future**: Add optional backend for teams, sync, better security

---

### 6.2 Why tree-sitter Over Other Parsers?

**Decision**: Use tree-sitter for code parsing

**Alternatives Considered**:
- Babel (JavaScript only)
- Esprima (JavaScript only)
- Python ast module (Python only, requires backend)

**Rationale**:
1. **Multi-language**: Supports 40+ languages
2. **Performance**: Fast, incremental parsing
3. **Browser Support**: WASM build works in browser
4. **Accuracy**: Used by GitHub, Neovim (battle-tested)

**Trade-offs**:
- ❌ Larger bundle size (~500KB for parsers)
- ❌ WASM learning curve

---

**End of Design Specification**