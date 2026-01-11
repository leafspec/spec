---
title: UI Requirements
version: 1.0.0-draft
last_updated: 2026-01-11
status: Draft
category: Frontend Specification
---

# UI Requirements

**Version:** 1.0.0-draft

This document defines the **frontend requirements** for LEAF spec implementations. It specifies **what** the UI must do and how users interact with it, not **how** to implement it.

> **Implementation Freedom:** Use any framework (React, Vue, Svelte, vanilla JS), any styling approach (CSS, Tailwind, styled-components), and any design system. These requirements define behavior and user experience, not technical implementation.

---

## Design Principles

1. **Clarity** - Users should always know what's happening (processing, thinking, streaming)
2. **Trust** - Citations and sources visible and accessible
3. **Responsiveness** - Fast feedback, optimistic updates where appropriate
4. **Accessibility** - Keyboard navigation, screen readers, proper semantics
5. **Simplicity** - Clear information architecture, no clutter

---

## Required Screens

### 1. Authentication Screens

#### 1.1 Registration Screen

**Elements:**
- Email input field (validated)
- Username input field (validated)
- Password input field (minimum 8 characters, type=password)
- Confirm password field (optional but recommended)
- "Register" / "Sign Up" button
- Link to login screen ("Already have an account?")
- Error message display area

**Behavior:**
- Form validation before submission
- Display specific error messages (email taken, password too short, etc.)
- Disable submit button during API call
- Redirect to main app on success
- Store authentication token securely

**Validation:**
- Email: Valid email format
- Username: 3-30 characters, alphanumeric + _ -
- Password: Minimum 8 characters
- Show validation errors inline or in summary

#### 1.2 Login Screen

**Elements:**
- Email input field
- Password input field
- "Login" / "Sign In" button
- Link to registration screen ("Need an account?")
- Error message display area

**Behavior:**
- Submit credentials to API
- Display error for invalid credentials
- Redirect to main app on success
- Store authentication token securely

---

### 2. Main Application Layout

#### 2.1 Navigation/Header

**Elements:**
- App logo/name
- Current user indicator (username or avatar)
- Primary navigation (Documents, Conversations, Search)
- User menu (Profile, Settings, Logout)

**Behavior:**
- Persistent across all screens
- Highlight active navigation item
- User menu opens on click/hover
- Mobile: Hamburger menu for navigation

#### 2.2 Responsive Layout

**Breakpoints:**
- Mobile: 320px - 767px (single column)
- Tablet: 768px - 1023px (collapsible sidebar)
- Desktop: 1024px+ (persistent sidebar)

**Behavior:**
- Sidebar navigation on desktop/tablet
- Bottom/hamburger navigation on mobile
- Content area adjusts to viewport
- Touch-friendly tap targets on mobile (44x44px minimum)

---

### 3. Document Library Screen

#### 3.1 Document List

**Elements:**
- Document grid or list view
- Each document shows:
  - Title
  - File type icon
  - File size
  - Upload date
  - Status indicator (processing/ready/failed)
  - Tags (if any)
- "Upload Document" button (prominent)
- Search/filter controls
- Sort controls (date, title, etc.)
- Pagination or infinite scroll

**Behavior:**
- Click document to view details
- Show processing status with loading indicator
- Failed documents show error message
- Empty state: "No documents yet. Upload your first document!"
- Loading state while fetching documents

#### 3.2 Document Upload

**Elements:**
- File picker or drag-and-drop area
- Supported formats listed (PDF, TXT, MD, etc.)
- File size limit displayed (50MB)
- Optional title input
- Optional tags input
- "Upload" button
- Progress indicator (during upload)

**Behavior:**
- Accept files via click or drag-drop
- Show file preview/name before upload
- Display upload progress (0-100%)
- Show success message with document ID
- Automatically navigate to document details or update list
- Show error if file too large or wrong type

#### 3.3 Document Details View

**Elements:**
- Document title (editable)
- File type, size, upload date
- Status indicator
- Tags (add/remove)
- Processing status:
  - "Processing..." with spinner
  - "Ready" with checkmark
  - "Failed" with error details
- Chunk count (when ready)
- "Delete Document" button (with confirmation)
- "Download" or "View" button (if applicable)
- Content preview (for text documents)

**Behavior:**
- Editable title (save on blur or button)
- Add/remove tags
- Delete requires confirmation modal
- Show processing progress if available
- Refresh status automatically until ready/failed

#### 3.4 Text Document Creation

**Elements:**
- Modal or dedicated screen
- Title input field (required)
- Content textarea or markdown editor
- Tags input
- "Create" button
- "Cancel" button

**Behavior:**
- Validate title required
- Submit to API
- Show in document list immediately (processing status)
- Close modal/return to list on success

---

### 4. Conversation Screen

#### 4.1 Conversation List

**Elements:**
- List of conversations
- Each conversation shows:
  - Title
  - Last message preview
  - Timestamp
  - Message count
- "New Conversation" button
- Search/filter conversations
- Empty state: "Start your first conversation"

**Behavior:**
- Click conversation to open
- Sort by last updated (most recent first)
- Show unread indicator (optional)
- Delete conversation (with confirmation)

#### 4.2 Conversation Interface

**Layout:**
- Message thread (scrollable)
- Message input area (fixed at bottom)
- Sidebar or panel:
  - Conversation title (editable)
  - Scoped documents (if any)
  - Related documents (suggested)

**Elements:**

**Message Thread:**
- User messages (right-aligned or distinct style)
- Assistant messages (left-aligned)
- Timestamps
- Token usage (optional, collapsed)
- Loading indicator while AI responds

**Message Input:**
- Text input or textarea
- "Send" button
- Character counter (optional)
- File upload button (optional future feature)
- Microphone button (optional future feature)

**Behavior:**
- Auto-scroll to bottom on new message
- Preserve scroll position when loading history
- Show "typing..." or thinking indicator when AI processing
- Submit on Enter, Shift+Enter for new line (or vice versa)
- Disable input while waiting for response

#### 4.3 Streaming Response Display

**Requirements:**
- ✅ **CRITICAL:** Must display streaming responses in real-time
- Content appears token-by-token or in small chunks
- Cursor or loading indicator at end of streaming text
- Auto-scroll to keep new content visible
- Citations appear after content complete

**Behavior:**
- Connect to SSE or WebSocket stream
- Append text incrementally as received
- Handle stream interruption gracefully
- Show "Regenerate" button after complete (optional)

**Visual States:**
1. **Thinking:** "Assistant is thinking..."
2. **Streaming:** Text appearing with cursor
3. **Complete:** Full response with citations
4. **Error:** Error message with retry option

#### 4.4 Citation Display

**Requirements:**
- ✅ **CRITICAL:** Citations must be visible and interactive

**Elements:**
- Numbered citations in text [1][2] or superscript¹²
- Citation list at bottom of message
- Each citation shows:
  - Document title (clickable)
  - Excerpt (50-500 chars)
  - Page number (if available)
  - Relevance score (optional)

**Behavior:**
- Click citation number to scroll to citation details
- Click document title to open document
- Hover to show preview (optional)
- Highlight relevant passage in document (optional)

**Example Layout:**
```
Assistant: Based on your documents, neural networks
consist of layers¹ that process information through
weighted connections².

Sources:
[1] Neural Networks Basics (p. 12)
    "A neural network consists of an input layer..."

[2] Machine Learning Overview (p. 8)
    "Connections between neurons have weights that..."
```

#### 4.5 AI Reasoning Display (Thought Accordion)

**Purpose:** Show AI's reasoning process transparently to build user trust and make the system's behavior observable.

**Requirements:**
- ✅ **SHOULD:** Display reasoning steps before/during answer generation
- Show what the AI is doing at each stage of processing
- Make the AI's decision-making process visible and educational

**Pattern: Collapsible "Thought Process" Section**

When the AI processes a query, display a collapsible section that shows:
1. **Retrieval step** - Searching knowledge base
2. **Document selection** - Which documents were found
3. **Generation step** - Synthesizing answer

**Visual Design (Recommended):**

```
[▼] Thinking...  ← Collapsible header, auto-collapses when answer starts
  ✓ Searching knowledge base... (found 12 chunks)
  ✓ Reading "annual_report.pdf", "Q3_summary.pdf"
  → Synthesizing answer...

[AI response streams here]

Sources:
[1] annual_report.pdf (p. 23)
[2] Q3_summary.pdf (p. 5)
```

**States:**

**1. Loading State (Before retrieval complete):**
```
[▼] Thinking...
  → Searching knowledge base...
```

**2. Retrieval Complete:**
```
[▼] Thinking...
  ✓ Searching knowledge base... (found 12 chunks)
  → Reading "annual_report.pdf", "Q3_summary.pdf", "budget_2024.pdf"
```

**3. Generation Started:**
```
[▼] Thinking...
  ✓ Searching knowledge base... (found 12 chunks)
  ✓ Reading 3 documents
  → Synthesizing answer...

[Text begins streaming...]
```

**4. Complete (Auto-collapsed):**
```
[▶] Thought Process  ← User can expand to review steps

[Full AI response here with citations]
```

**Elements (Required):**
- Collapsible container (starts expanded, auto-collapses when answer begins)
- Step indicators (checkmark ✓ for complete, arrow → for in-progress)
- Document count ("found X chunks")
- Document names (up to 3, then "+ N more")
- Loading/progress indicators

**Elements (Optional but Recommended):**
- Similarity scores for retrieved chunks
- Token count estimation
- Context management strategy indicator (if map-reduce used)
- Estimated cost (tokens × price)

**Behavior:**
- MUST update in real-time as steps complete
- SHOULD show BEFORE answer starts streaming
- SHOULD collapse automatically when answer begins
- MUST remain expandable after collapse
- MAY show timing information (e.g., "Retrieved in 340ms")
- MAY show confidence scores

**Why This Matters:**

1. **Trust Building** - Users see the AI isn't making things up
2. **Debugging** - Helps users understand poor results
   - "Oh, it only found 2 chunks - I need to upload more documents"
   - "It's reading the wrong document - I need to refine my query"
3. **Education** - Shows how RAG systems work
4. **Transparency** - Makes AI decision-making observable
5. **Compliance** - Shows provenance for regulated industries

**Inspiration Examples:**
- Perplexity.ai's "Sources" panel (shows docs while thinking)
- ChatGPT's "Searching..." indicator (minimal version)
- Claude's "Thinking..." state (shows reasoning in progress)

**Implementation Notes:**
- Can be implemented with any frontend framework
- Use SSE/WebSocket events to update steps in real-time
- Backend should emit events: `retrieval_started`, `retrieval_complete`, `generation_started`
- CSS transitions for smooth expand/collapse

**Accessibility:**
- Collapsible region must have `aria-expanded` attribute
- Screen readers should announce state changes
- Keyboard accessible (Space/Enter to expand/collapse)

---

### 5. Search Screen

#### 5.1 Search Interface

**Elements:**
- Search input field (prominent)
- Search button or search-on-type
- Filter controls:
  - Document filter (dropdown or multi-select)
  - Minimum relevance slider (optional)
- Results list
- Result count ("23 results")

**Behavior:**
- Search as you type (debounced) or on submit
- Show loading state during search
- Display results instantly
- Clear button to reset search

#### 5.2 Search Results

**Elements:**
- Each result shows:
  - Document title
  - Excerpt with query highlighted
  - Relevance score (bar or percentage)
  - Page number / section
  - "View Document" link
- Pagination or load more
- Empty state: "No results found for 'query'"

**Behavior:**
- Highlight search terms in excerpts
- Click result to view in document
- Sort by relevance (default)
- Show relevance score visually

---

### 6. Summarization Interface

#### 6.1 Single Document Summary

**Location:** Document details screen or modal

**Elements:**
- "Generate Summary" button
- Length selector (Short / Medium / Long)
- Focus selector (General / Key Points / Technical / Conclusions)
- Summary display area
- Token usage display (optional)

**Behavior:**
- Click to generate (async)
- Show loading state
- Display summary when ready
- Option to copy or regenerate

#### 6.2 Multi-Document Summary

**Location:** Separate screen or modal

**Elements:**
- Document selector (multi-select)
- Optional query input ("Focus on...")
- Length and focus selectors
- "Generate Summary" button
- Summary display with citations

**Behavior:**
- Select 2+ documents
- Optional focus query
- Generate asynchronously
- Display with citations to source docs

---

### 7. User Profile / Settings

#### 7.1 Profile Screen

**Elements:**
- Username (editable)
- Email (display only or editable)
- Usage statistics:
  - Total documents
  - Total conversations
  - Tokens used
- Memory/Facts section (display learned facts)

**Behavior:**
- Edit username
- View usage metrics
- See what system remembers about user

#### 7.2 Settings/Preferences

**Elements:**
- Default document scope (All / Recent / Tagged)
- Default summary length
- Enable/disable streaming
- Theme (Light / Dark) - optional
- "Save" button

**Behavior:**
- Update preferences via API
- Apply immediately or on save
- Show success confirmation

---

## Interactive Patterns

### Loading States

**Requirements:**
- Show loading indicator for async operations
- Differentiate between:
  - Page loading (full screen spinner)
  - Component loading (skeleton screens)
  - Action loading (button spinner)
- Never show blank screen without indicator

**Examples:**
- Document list: Skeleton cards
- Message response: Typing indicator → streaming
- Upload: Progress bar

### Error Handling

**Requirements:**
- Display user-friendly error messages
- Provide actionable guidance ("Try again" button)
- Show specific errors when available
- Never show raw error objects or stack traces

**Error Types:**
- Network errors: "Connection lost. Check your internet."
- Validation errors: "Title is required"
- Authorization errors: "Session expired. Please log in again."
- Server errors: "Something went wrong. Try again."

### Empty States

**Requirements:**
- Every list/collection must have empty state
- Empty state includes:
  - Icon or illustration
  - Descriptive message
  - Call to action button

**Examples:**
- "No documents yet. Upload your first document!"
- "No conversations. Start chatting with your documents!"
- "No search results. Try different keywords."

### Confirmations

**Requirements:**
- Destructive actions require confirmation
- Confirmation modal or dialog with:
  - Clear warning message
  - "Confirm" and "Cancel" buttons
  - Confirm button styled as danger

**Examples:**
- Delete document
- Delete conversation
- Logout (optional)

---

## Accessibility Requirements

### Keyboard Navigation

**Requirements:**
- ✅ All interactive elements focusable with Tab
- ✅ Focus visible (outline or highlight)
- ✅ Enter/Space activate buttons
- ✅ Escape closes modals
- ✅ Arrow keys navigate lists (optional but recommended)

**Critical Flows:**
- Tab through registration form
- Navigate conversations with keyboard
- Send message with Enter
- Close modals with Escape

### Screen Reader Support

**Requirements:**
- ✅ Semantic HTML (header, nav, main, article, etc.)
- ✅ ARIA labels on interactive elements
- ✅ Alt text on images/icons
- ✅ Announcements for dynamic content
- ✅ Skip to main content link

**Examples:**
```html
<button aria-label="Upload document">
  <UploadIcon />
</button>

<div role="status" aria-live="polite">
  Document uploaded successfully
</div>
```

### Color & Contrast

**Requirements:**
- ✅ Text contrast ratio 4.5:1 minimum (WCAG AA)
- ✅ Don't rely solely on color for information
- ✅ Focus indicators clearly visible
- ✅ Support high contrast mode (optional)

### Responsive Text

**Requirements:**
- ✅ Text resizable up to 200%
- ✅ No horizontal scrolling at 320px width
- ✅ Touch targets at least 44x44px on mobile

---

## Performance Requirements

### Perceived Performance

**Requirements:**
- Optimistic updates where possible
- Skeleton screens instead of spinners
- Progressive enhancement
- Lazy load images/documents
- Virtual scrolling for long lists (optional)

**Examples:**
- Message appears immediately, sends in background
- Document shows in list instantly, processes async
- Conversations load initial messages, fetch history on scroll

### Network Efficiency

**Requirements:**
- Debounce search inputs (300-500ms)
- Paginate long lists
- Cache documents/conversations locally (optional)
- Compress large payloads

---

## Mobile-Specific Considerations

### Touch Interactions

**Requirements:**
- Swipe to delete conversations (optional)
- Pull to refresh document list (optional)
- Touch-friendly spacing (44x44px minimum)
- No hover-dependent interactions

### Mobile Navigation

**Requirements:**
- Bottom navigation bar or hamburger menu
- Full-screen modals for forms
- Back button support (browser history)

### Mobile Optimizations

**Requirements:**
- Smaller font sizes where appropriate
- Collapsible sections for citations/details
- Simplified layouts (single column)

---

## Visual Design Guidelines

While specific design is implementation-specific, these principles apply:

### Hierarchy

- Clear visual hierarchy (headings, spacing, size)
- Primary actions prominent (Upload, Send, New)
- Secondary actions subtle (Delete, Edit)
- Destructive actions styled differently (red, warning)

### Consistency

- Consistent spacing system (4px, 8px, 16px, 24px, etc.)
- Consistent color palette
- Consistent typography
- Consistent button styles

### Feedback

- Hover states on interactive elements
- Active states on buttons
- Disabled states clearly distinguishable
- Success/error states with color + icon

---

## Testing Frontend Compliance

### Functional Tests
- ✅ Can register and login
- ✅ Can upload documents
- ✅ Can create conversations
- ✅ Can send messages and see responses
- ✅ Can see citations and click them
- ✅ Can search documents
- ✅ Can generate summaries

### Accessibility Tests
- ✅ Keyboard navigation works
- ✅ Screen reader compatible
- ✅ Color contrast passes WCAG AA
- ✅ Touch targets adequate on mobile

### Responsive Tests
- ✅ Works at 320px, 768px, 1024px, 1920px
- ✅ No horizontal scroll at any size
- ✅ Touch targets adequate on mobile
- ✅ Text readable without zoom

### Performance Tests
- ✅ First contentful paint < 2s
- ✅ Time to interactive < 4s
- ✅ Streaming responses appear smoothly
- ✅ Search results appear < 1s

---

## Reference Implementations

While not prescriptive, these examples show compliant patterns:

**Streaming UI:**
- Similar to ChatGPT, Claude, or Perplexity
- Token-by-token or chunked display
- Cursor/indicator during generation

**Citation UI:**
- Numbered citations like Wikipedia
- Source cards like Perplexity
- Inline references like academic papers

**Document Library:**
- Grid like Google Drive
- List like Notion
- Cards like Evernote

---

## Summary Checklist

A frontend implementation is **compliant** when it:

- ✅ Supports registration and login
- ✅ Displays document library with upload
- ✅ Shows document processing status
- ✅ Provides conversation interface
- ✅ **Streams AI responses in real-time**
- ✅ **Displays citations prominently**
- ✅ Shows search interface with results
- ✅ Supports summary generation
- ✅ Handles errors gracefully
- ✅ Shows appropriate loading states
- ✅ Keyboard navigable
- ✅ Screen reader accessible
- ✅ Works on mobile (320px+)
- ✅ Passes color contrast requirements
- ✅ Responsive at 768px and 1024px breakpoints

---

**Implementation Complete!** You now have a full LEAF specification. Proceed to build with any tech stack.
