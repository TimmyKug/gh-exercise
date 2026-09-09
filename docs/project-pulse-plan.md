# Project Pulse Dashboard Implementation Plan

## Summary

Build Mona’s Project Pulse dashboard as a small, static frontend for contributors. The dashboard will present multiple projects in a clear card-based layout and make each project’s owner, status, recent activity, priority or risk, and contributor-friendly summary easy to scan.

The implementation will use the repository’s existing custom-agent workflow:

- **Orchestrator** coordinates phases, delegates non-overlapping file scopes, resolves dependencies, and performs final integration review.
- **Planner** has researched the repository and defines this implementation sequence.
- **Designer** owns the visual system, information hierarchy, accessibility, responsive behavior, and polished styling.
- **Coder** owns the static application implementation and runnable-app support.

The required deliverables are:

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

No framework or package installation is required. The app should remain dependency-free and run through Python’s built-in HTTP server. Serving the app rather than opening the HTML file directly is important because the page will load `project-data.json` at runtime.

## Repository Findings and Constraints

- The repository is an orchestration exercise rather than an existing application.
- The project brief is in `.github/project-pulse-brief.md`.
- Agent responsibilities and delegation rules are defined in `.github/agents/`.
- The repository expects the learner to use the Orchestrator, Planner, Designer, and Coder rather than handling all work as one undifferentiated task.
- Existing exercise validation checks require:
  - `app/index.html`, `app/styles.css`, and `app/project-data.json` to exist.
  - `.vscode/launch.json` to exist and parse as strict JSON.
  - The HTML to contain `Project Pulse`, reference `styles.css` and `project-data.json`, and include visible `project-card` markup.
  - The HTML to expose `status`, `recentActivity`, and `priority` in the rendered interface.
  - The stylesheet to include `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`.
  - The data file to contain a top-level `projects` array with `name`, `owner`, `status`, `recentActivity`, and `priority` on every project.
  - The launch configuration to be named `Run Project Pulse Dashboard` and target `index.html`.
- `.vscode/tasks.json` already supports the repository’s folder-open Copilot CLI workflow; it should not be modified for this feature.
- The repository has no existing application framework, build system, package manifest, test suite, or component library to preserve.

## Ordered Implementation Steps

### 1. Confirm the implementation contract

**Owner:** Orchestrator  
**File scope:** No application changes

The Orchestrator should review this plan and `.github/project-pulse-brief.md`, then convert the work into explicit specialist assignments. Before delegation, confirm:

- The app remains static and dependency-free.
- The four required files have distinct ownership.
- The JSON schema is agreed before HTML rendering logic is finalized.
- Designer and Coder do not edit the same file in the same phase.
- The launch configuration serves from `app/` and opens `index.html`, not the directory root.

**Dependency:** None.

---

### 2. Define the data model and dashboard content

**Owner:** Designer, with Coder review  
**File assignment:** `app/project-data.json`

Designer should define a representative set of multiple projects that makes the dashboard useful in a first view. The data should be contributor-friendly and internally consistent.

The file must contain:

```text
{
  "projects": [
    {
      "name": "...",
      "owner": "...",
      "status": "...",
      "recentActivity": "...",
      "priority": "..."
    }
  ]
}
```

Content decisions should include:

- At least several projects so the card layout can be evaluated meaningfully.
- A useful mixture of statuses, such as active, planning, blocked, or complete.
- Clear owner values.
- Recent activity written as concise, readable summaries rather than opaque identifiers.
- Priority or risk values that can be visually distinguished.
- Valid JSON only; no comments, trailing commas, or executable logic.

The data file should remain presentation-neutral. Status and priority values should be stable enough for the HTML and CSS to apply predictable labels or styles.

**Dependency:** The schema must be agreed before Coder finalizes the data-loading and rendering logic in `app/index.html`.

---

### 3. Establish the visual and accessibility direction

**Owner:** Designer  
**File assignment:** `app/styles.css`

Designer should implement the visual language for the dashboard, including:

- A clear page-level `.dashboard` layout.
- A strong Project Pulse heading and supporting context.
- A responsive grid or flexible layout for `.project-card` elements.
- Clear visual hierarchy between project name, owner, status, recent activity, and priority.
- Status badges with sufficient contrast and text labels that do not rely on color alone.
- Priority or risk treatment that is visually distinct but still readable.
- Rounded cards using `border-radius`.
- Subtle depth using `box-shadow`.
- Consistent spacing, readable typography, and a restrained color palette.
- Responsive behavior for narrow screens, including cards that collapse to a single-column layout.
- Visible keyboard focus states for interactive elements, if links or controls are introduced.
- Respect for reduced-motion preferences if transitions or animations are used.
- Sensible default styles for loading, empty, and error states.

The stylesheet should use deterministic hooks required by the exercise, especially `.dashboard` and `.project-card`. It should not depend on an external CSS framework or remote font service.

**Dependency:** The layout must accommodate the fields in `app/project-data.json`. The class naming and structural assumptions should be communicated to Coder before HTML implementation.

**Can run in parallel:** This work can begin in parallel with the data-content definition if Designer documents the expected markup contract. It must be reconciled with the final HTML before integration.

---

### 4. Implement the semantic document and data rendering

**Owner:** Coder  
**File assignment:** `app/index.html`

Coder should build the complete static page structure and the small amount of inline browser logic needed to load and render `app/project-data.json`. No additional JavaScript file is required unless the Orchestrator explicitly expands the file scope.

The page should include:

- A valid HTML5 document structure.
- The exact title text **Project Pulse** in the visible page heading.
- A stylesheet reference to `styles.css`.
- A reference to `project-data.json`.
- Semantic landmarks such as `header`, `main`, and appropriate section headings.
- A project collection region with a meaningful accessible label.
- A reusable `project-card` structure for every project.
- Visible fields for:
  - project name
  - owner
  - status
  - recent activity
  - priority or risk
- Accessible status and priority text, not color-only indicators.
- Safe text rendering from JSON values; data must be inserted as text rather than interpreted as HTML.
- Loading, empty-data, and fetch-error messages.
- A rendering path that works when served from the `app/` directory and when the browser URL is `/index.html`.

The implementation should make the data relationship clear. The page may use a template element or create elements programmatically, but the final UI must contain visible project cards and not merely display raw JSON.

The Coder should avoid hard-coded project values in the presentation when those values already come from `project-data.json`. A small static heading or explanatory subtitle may remain in HTML.

**Dependencies:**

1. The JSON schema from Step 2 must be stable.
2. The CSS class contract from Step 3 must be known.
3. The Coder must account for the fact that `fetch()` generally requires an HTTP server rather than a `file://` URL.

**Must run sequentially:** Final HTML integration must follow agreement on the data fields and visual class names.

---

### 5. Create the runnable VS Code configuration

**Owner:** Coder  
**File assignment:** `.vscode/launch.json`

Coder should create `.vscode/launch.json` as strict JSON with no comments. The configuration should include:

- Configuration name: **Run Project Pulse Dashboard**
- A launch mechanism that runs:
  - `python3 -m http.server 5500`
- Working directory:
  - `${workspaceFolder}/app`
- A `serverReadyAction` that detects the server output and opens:
  - `http://localhost:%s/index.html`
- An external browser action or equivalent that opens the rendered dashboard.
- No directory-root URL that would show a server directory listing.

The configuration should use deterministic values and should not require package installation or additional extension-specific assumptions beyond the standard VS Code launch support expected by the exercise.

**Dependencies:**

- The app must use relative references to `styles.css` and `project-data.json`.
- The launch configuration must be tested against the actual server output pattern.
- The final URL must explicitly include `index.html`.

**Can run in parallel:** The initial JSON structure can be drafted while `app/index.html` is being implemented because the launch file has an independent scope. Final validation must wait until the app path and server behavior are confirmed.

---

### 6. Integrate and review all deliverables

**Owner:** Orchestrator, with Designer and Coder review  
**Files reviewed:** All four required files

The Orchestrator should verify that the four files work together:

- The HTML class names match the selectors in `app/styles.css`.
- Every required JSON field is rendered.
- Status and priority values have predictable visual treatment.
- Relative asset paths work from `app/index.html`.
- The launch configuration’s `cwd` points to `app/`.
- The launch URL opens `index.html`.
- The first browser view is the dashboard rather than a directory listing.
- No specialist has modified files outside the assigned scope.
- The implementation remains dependency-free and deterministic.

If changes are needed, assign them to the owning specialist rather than having multiple agents edit the same file concurrently.

**Dependency:** Steps 2–5 must be complete enough for an integrated review.

---

## File Assignment Matrix

| File | Primary owner | Supporting owner | Responsibility |
|---|---|---|---|
| `app/index.html` | Coder | Designer | Semantic page structure, Project Pulse heading, data loading, project-card rendering, loading/empty/error states, accessible field labels |
| `app/styles.css` | Designer | Coder | Dashboard layout, responsive behavior, card styling, badges, spacing, contrast, focus states, `.dashboard`, `.project-card`, `border-radius`, and `box-shadow` |
| `app/project-data.json` | Designer | Coder | Top-level `projects` array and representative project records with `name`, `owner`, `status`, `recentActivity`, and `priority` |
| `.vscode/launch.json` | Coder | Orchestrator | Strict JSON launch configuration named `Run Project Pulse Dashboard`, Python server command, `app/` working directory, and `index.html` browser target |

The Orchestrator owns coordination and review but should not directly implement application code.

## Designer Responsibilities

The Designer is responsible for:

1. Defining the information hierarchy for contributors scanning project health.
2. Choosing the card layout and responsive behavior.
3. Establishing status and priority visual conventions.
4. Ensuring labels remain understandable without color.
5. Implementing `app/styles.css` within the assigned scope.
6. Supplying or approving representative project content in `app/project-data.json`.
7. Checking readable spacing, contrast, typography, focus visibility, and mobile behavior.
8. Communicating the expected HTML class and structural contract to the Coder.
9. Reviewing the integrated result for visual clarity rather than only checking whether selectors exist.

The Designer should not modify `app/index.html` or `.vscode/launch.json` unless the Orchestrator explicitly creates a later, non-overlapping review assignment.

## Coder Responsibilities

The Coder is responsible for:

1. Implementing `app/index.html` within the assigned scope.
2. Referencing `styles.css` and `project-data.json` correctly.
3. Loading and safely rendering project data.
4. Ensuring every project card displays the required fields.
5. Providing loading, empty, and fetch-error handling.
6. Creating `.vscode/launch.json` as strict JSON.
7. Configuring the Python server command, `cwd`, server-ready detection, and `index.html` URL.
8. Validating JSON syntax and the runnable preview.
9. Reporting changed files, checks performed, and any remaining browser or tooling risk.
10. Avoiding changes to design-only files unless explicitly assigned.

## Dependencies

The dependency graph is:

1. **Repository and brief review**
   - Required before specialist delegation.
2. **Data schema and content**
   - Required for reliable rendering in `app/index.html`.
3. **Visual contract**
   - Required for the Coder to use the correct dashboard and card class names.
4. **HTML rendering**
   - Depends on the data schema and visual contract.
5. **Launch configuration**
   - Can be drafted independently, but final validation depends on the actual HTML path and server behavior.
6. **Integrated browser review**
   - Depends on all four deliverables.
7. **Final validation and handoff**
   - Must follow integration fixes.

No external runtime library, package manager, build step, API, database, or network service is required.

## Parallel Work Decisions

### Work that can run in parallel

The following can run concurrently after Step 1:

- **Designer:** Define and implement `app/styles.css`.
- **Designer/Coder collaboration:** Define representative records for `app/project-data.json`, provided the schema is written down first.
- **Coder:** Draft `.vscode/launch.json`, because it has an independent file scope.
- **Coder:** Prepare the semantic HTML skeleton while Designer documents the expected classes and data fields.

These parallel tasks are safe only when each agent has a non-overlapping file assignment and shares interface decisions before final integration.

### Work that must run sequentially

The following must remain sequential:

1. Repository and brief review before delegation.
2. Data schema agreement before finalizing the JSON rendering logic.
3. CSS/class contract agreement before final HTML integration.
4. HTML and data completion before browser verification.
5. Launch configuration verification after confirming the served path and server output.
6. Integrated accessibility, responsive, and content review after all files exist.
7. Final validation after all corrections are applied.

No two agents should simultaneously edit `app/index.html`, `app/styles.css`, `app/project-data.json`, or `.vscode/launch.json`.

## Edge Cases and Error States

The implementation should account for:

- **Missing data file:** Show a useful error message instead of leaving an empty dashboard.
- **Malformed JSON:** Fail gracefully and identify that project data could not be loaded.
- **Missing `projects` key:** Treat the response as invalid data.
- **Empty `projects` array:** Show an explicit “No projects available” state.
- **Incomplete project record:** Use a readable fallback such as “Not provided” rather than rendering `undefined`.
- **Unexpected status or priority value:** Preserve the text and use a neutral visual treatment instead of relying on a missing CSS class.
- **Long project names or activity summaries:** Allow wrapping without horizontal overflow.
- **Many projects:** Keep the grid usable as the number of cards grows.
- **Small screens:** Collapse the layout and preserve readable spacing.
- **Keyboard navigation:** Ensure any interactive elements have visible focus indicators.
- **Reduced motion:** Avoid making transitions necessary to understand status or priority.
- **Color vision differences:** Pair colors with text, labels, or icons only as supplementary cues.
- **Direct file opening:** Explain that `fetch()` may fail from `file://`; use the provided launch configuration or another local HTTP server.
- **Port conflict:** If port `5500` is already occupied, report the conflict and use the repository’s expected configuration consistently rather than silently changing the launch contract.
- **Directory listing risk:** Ensure `serverReadyAction` opens `/index.html`, not only the server root.
- **Unsafe data insertion:** Render JSON values as text and do not interpret project data as arbitrary HTML.

## Validation Expectations

### Automated file and syntax validation

Run the repository’s existing validation where applicable:

```bash
bash scripts/validate-exercise.sh
```

Also validate the newly created JSON files directly:

```bash
python3 -m json.tool app/project-data.json
python3 -m json.tool .vscode/launch.json
```

The validation should confirm:

- All four required files exist.
- `app/project-data.json` is valid JSON.
- `.vscode/launch.json` is valid strict JSON with no comments.
- The data has a top-level `projects` array.
- Every project includes `name`, `owner`, `status`, `recentActivity`, and `priority`.
- `app/index.html` contains the exact `Project Pulse` title or heading.
- `app/index.html` references `styles.css` and `project-data.json`.
- `app/index.html` includes or generates `project-card` elements.
- `status`, `recentActivity`, and `priority` are visible in the UI path.
- `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`.
- `.vscode/launch.json` contains `Run Project Pulse Dashboard` and `index.html`.

### Manual browser validation

Use VS Code’s **Run and Debug** view:

1. Select **Run Project Pulse Dashboard**.
2. Start the configuration.
3. Confirm the server runs with the `app/` working directory.
4. Confirm the browser opens `http://localhost:5500/index.html` or the equivalent configured URL.
5. Confirm the first view is the Project Pulse dashboard, not a directory listing.
6. Confirm multiple project cards are visible.
7. Confirm every card displays project name, owner, status, recent activity, and priority.
8. Confirm loading, empty, and error states are reasonable where they can be exercised.
9. Resize the viewport to a narrow width and verify the layout remains readable.
10. Navigate with the keyboard and verify visible focus treatment.
11. Inspect contrast and confirm status or priority is not communicated by color alone.
12. Stop the preview server after testing.

### Integration review

The Orchestrator should review the final result against the brief and confirm:

- The dashboard is polished rather than a bare HTML page.
- The page is contributor-friendly and scannable.
- The Designer’s visual decisions are reflected in the actual rendered page.
- The Coder’s data-loading and launch behavior are deterministic.
- The four assigned files are the only required implementation outputs.
- No unresolved errors or hidden dependency on an unavailable service remains.

## Open Questions and Assumptions

- **Exact visual theme:** The brief does not prescribe brand colors, so Designer may choose a restrained Project Pulse palette as long as contrast and readability remain strong.
- **Status vocabulary:** The brief does not define an exact enum. Use a small, consistent set of human-readable values and provide a neutral fallback for unknown values.
- **Priority vocabulary:** The brief allows priority or risk level. The implementation should choose one consistent representation, such as `Low`, `Medium`, `High`, or `At risk`, and use it consistently in the data and UI.
- **JavaScript file scope:** No separate JavaScript file is required by the brief. The plan assumes the small data-loading/rendering behavior will be contained in `app/index.html`. If the Orchestrator decides a separate script is necessary, that is a scope expansion and should be explicitly approved.
- **Browser launch support:** The plan assumes the learner’s VS Code environment supports the configured launch type and `serverReadyAction`. If the browser does not open automatically, the server URL should still be tested manually while preserving the required launch name, command, working directory, and `index.html` target.
- **Port ownership:** The expected launch command uses port `5500`. A port conflict should be reported rather than silently changing the exercise’s deterministic launch contract.
- **No persistent backend:** Project data is intentionally static JSON; live updates, authentication, filtering APIs, and persistence are outside the current scope.