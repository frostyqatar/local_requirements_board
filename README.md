### **System Description: Local Requirements Board**

#### 1. Executive Summary

The Local Requirements Board is a standalone, single-file, serverless web application designed for agile project management and requirements gathering. It operates entirely within the user's web browser, requiring no installation, backend infrastructure, or user authentication. All data is persisted locally in the browser's `localStorage`.

The application provides a multi-project environment where users can create, manage, and switch between different projects. Each project features a four-column Kanban-style board representing different stages of requirement consensus: "Agreed," "Disagreed," "On-Hold," and "Unneeded." Users can manage requirements as draggable cards, which contain detailed metadata.

Key features include a sophisticated UI with a high-contrast dark mode, inline and modal-based card creation, keyboard shortcuts for efficiency, robust import/export functionality for both single projects and full backups, and an integrated AI Chat Assistant powered by Google Gemini to automate requirement generation from natural language.

#### 2. System Architecture

The application is architected as a **purely client-side single-page application (SPA)** contained within a single `.html` file. This design choice prioritizes portability, offline-first capability, and zero setup.

The architecture can be broken down into three logical layers within the file:

1.  **Presentation Layer (HTML & CSS):**
    *   **HTML:** A single, semantic HTML5 document structures the application into a header, a main board area, a chat section, and a modal dialog. Accessibility is considered with the use of appropriate tags and ARIA-friendly structures.
    *   **CSS:** Styling is primarily handled by **Tailwind CSS**, loaded via a CDN for rapid, utility-first development. This is augmented by a custom `<style>` block which defines:
        *   The advanced high-contrast dark mode theme and color palette.
        *   The "glassmorphism" effect for cards and sections.
        *   Column-specific background gradients.
        *   Animations and transition effects for user interactions (e.g., card dragging, hover states).

2.  **Logic Layer (Vanilla JavaScript):**
    *   This is the core of the application, written entirely in modern, framework-free JavaScript (ES6+). It operates as an event-driven system that manages application state, manipulates the DOM, and handles all user interactions.
    *   The script is self-contained and modular in function, handling state management, rendering, event listening, and API communication.

3.  **Data Layer (Browser `localStorage`):**
    *   All application data, including projects, cards, and user settings (theme, API key), is persisted in the browser's `localStorage`.
    *   Data is stored under a single key (`requirementsBoardData`) as a serialized JSON string, making it easy to back up and manage.

#### 3. Core Components & Functional Breakdown

##### 3.1. Project Management
*   **Multi-Project Support:** Users can create and manage an unlimited number of projects locally.
*   **Project Switching:** An interactive dropdown in the header allows for seamless switching between active projects, triggering a full re-render of the board.
*   **Project Creation:** A "New Project" button prompts the user for a name and initializes a new, empty project structure.
*   **Project Deletion:** A dedicated "Delete Project" button allows the user to permanently remove the currently active project after a confirmation prompt.

##### 3.2. Kanban Board & Requirement Cards
*   **Four-Column Layout:** The board is consistently divided into "Agreed Requirements," "Disagreed Requirements," "On-Hold Requirements," and "Unneeded Requirements."
*   **Draggable Cards:** Requirement cards can be moved smoothly between columns and reordered within a column using the native HTML5 Drag and Drop API. Visual cues (e.g., a dashed border on the target column) provide feedback during the drag operation.
*   **Card Metadata:** Each card is a rich object containing:
    *   Title, Description, Priority (Low, Medium, High, Critical), Owner, Due Date, and an array of Tags.
    *   System-generated fields: `id` (UUID), `createdAt`, `updatedAt`.
*   **Card Creation:**
    *   **Inline Add:** A quick-add text box appears in a column for rapidly entering a card's title. Saving is possible via click or `Ctrl+Enter`.
    *   **Modal Add/Edit:** A detailed modal allows for the creation or editing of all card metadata fields.
*   **Card Deletion:** A trash can icon appears on card hover, allowing for direct deletion after a confirmation prompt.
*   **Tag Management:** The UI intelligently handles tag overflow, displaying the first three tags and a `+X more` indicator to maintain a clean layout.

##### 3.3. AI Chat Assistant ("Chat")
*   **Toggleable Interface:** The chat interface is hidden by default and can be toggled via a button in the header.
*   **API Key Management:** The chat requires a Google Gemini API key, which is entered once by the user and saved securely in `localStorage`.
*   **Natural Language Processing:** Users can type requirements in plain English (e.g., "Create a user login page and a profile section").
*   **Automated Card Generation:** On submission (`Enter` key or button click), the input is sent to the Google Gemini API with a carefully engineered system prompt. This prompt instructs the AI to analyze the text and return a structured JSON array containing objects for each identified requirement, including a title, description, priority, and relevant "smart tags."
*   **Board Integration:** The application parses the AI's JSON response and automatically creates new cards in the "Agreed Requirements" column, providing immediate feedback to the user.

##### 3.4. Data Persistence & Management
*   **Local Storage:** The entire application state (`appData` object) is saved to `localStorage`, ensuring data persists between sessions. Saves are debounced to optimize performance during rapid operations like dragging.
*   **Intelligent Import:** The "Import" button can handle two types of files:
    1.  **Single Project File:** Adds a new project or prompts to overwrite an existing one with the same ID.
    2.  **Full Backup File:** Restores all projects and application settings from an "Export All" backup.
*   **Comprehensive Export:**
    *   **Export JSON:** Exports only the currently active project into a structured JSON file.
    *   **Export TXT:** Exports the current project as a human-readable, formatted text report grouped by column.
    *   **Export All:** Creates a full backup of the entire `appData` object, including all projects and settings, in a single JSON file.

##### 3.5. User Interface & Experience (UI/UX)
*   **Responsive Design:** The UI is fully responsive, adapting from a single-column layout on mobile devices to a four-column layout on desktops.
*   **High-Contrast Dark Mode:** A professionally designed dark theme provides excellent legibility and aesthetics, reducing eye strain. The user's theme preference is saved and automatically applied.
*   **Glassmorphism:** A subtle "frosted glass" effect is used on cards and sections to create a modern and layered interface.
*   **User Feedback:** The system provides constant feedback through toast notifications for actions (e.g., "Project Saved," "Card Deleted"), visual cues for drag-and-drop, and inline validation hints.
*   **Efficiency:** Keyboard shortcuts (`Enter` to send chat, `Ctrl+Enter` to add a card) and thoughtful workflows are implemented to enhance user productivity.

#### 4. Data Model

The application's state is managed through a single JavaScript object, `appData`, which is serialized to JSON for storage.

```json
{
  "projects": [
    {
      "id": "uuid-string",
      "name": "Project Name",
      "createdAt": "iso-datetime",
      "updatedAt": "iso-datetime",
      "columns": {
        "agreed": [ { "id": "...", "title": "Card 1" } ],
        "disagreed": [],
        "onHold": [],
        "unneeded": []
      }
    }
  ],
  "activeProjectId": "uuid-string",
  "theme": "light" | "dark",
  "apiKey": "user-supplied-google-api-key",
  "chatVisible": true | false
}
```
*   **Card Object Structure:**
    ```json
    {
      "id": "uuid-string",
      "title": "Requirement Title",
      "description": "Detailed description.",
      "priority": "Low" | "Medium" | "High" | "Critical",
      "owner": "User Name",
      "dueDate": "YYYY-MM-DD",
      "tags": ["ui", "auth"],
      "createdAt": "iso-datetime",
      "updatedAt": "iso-datetime"
    }
    ```

#### 5. Technology Stack

*   **Frontend:** HTML5, CSS3, Vanilla JavaScript (ES6+)
*   **Styling:** Tailwind CSS (loaded via CDN)
*   **AI Integration:** Google Gemini API (specifically, the `gemini-1.5-flash` model)
*   **Data Storage:** Browser `localStorage` API
*   **Runtime Environment:** Any modern web browser (Chrome, Firefox, Edge)

#### 6. Scope & Limitations

*   **No Backend:** The application is entirely client-side. There is no server database or user authentication system.
*   **No Real-time Collaboration:** As all data is stored locally, the application does not support simultaneous use by multiple users.
*   **Data Security:** All project data and settings are as secure as the user's local machine. The only external communication is with the Google Gemini API, which sends the user's chat prompts and API key.
