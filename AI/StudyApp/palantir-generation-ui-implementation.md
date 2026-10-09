# Palantir Foundry Workshop: Generation UI from Scratch

## 1. Goal and key requirement

Build a Workshop page for a selected **Study Analysis** object. The page displays a hierarchical list of report sections with checkboxes:

- **Generate** — whether the section should be generated.
- **Text** — whether text content should be included.
- **Tables** — whether tables should be included.
- Sections can be expanded and collapsed to show child sections.
- Existing checkbox values must be loaded from the selected **Study Analysis** object.
- When the user changes a checkbox, the change must remain a **draft in the Workshop UI**.
- Do **not** update the Study Analysis object while the user is clicking checkboxes.
- Update the Study Analysis object **only when the user clicks Submit Study Analysis**.

This guide starts from a clean slate. It does not assume that any of the previously suggested object types, actions, variables, or Workshop modules already exist.

---

## 2. Choose where the data lives

There are two different kinds of data in this UI:

1. **Section definitions** — stable information about the available sections, such as title, parent, display order, and whether Text or Tables are supported.
2. **Selections for one Study Analysis** — the current Generate, Text, and Tables values chosen for that particular Study Analysis.

Keep these concepts separate.

### Recommended beginner-friendly design

Use:

- The existing **Study Analysis** object type as the business object being edited.
- A **section catalogue** (a small Foundry dataset is sufficient) to define the hierarchy and capabilities of report sections.
- A property on Study Analysis, for example `generationSelectionJson`, to store the selected checkbox values for that Study Analysis.
- A Workshop draft variable to hold edits before submission.
- An action that writes the draft back to the **same Study Analysis object** when Submit Study Analysis is clicked.

This approach does **not** require creating a separate selection object for every section.

### Why use a JSON property?

Report sections are dynamic. A new section can be added later without adding a new property to the Study Analysis object each time. A JSON string property can store the selection state for all sections in one place.

If the section list is permanently fixed and very small, individual Boolean properties are possible, but they are harder to maintain when the section hierarchy changes. For a configurable report hierarchy, prefer a structured JSON payload.

> Important: a JSON property stores the values, but it does not automatically create a recursive tree UI. The catalogue and Workshop display logic still need to provide the section titles, hierarchy, and checkbox behavior.

---

## 3. Data model to create or verify

### 3.1 Study Analysis object type

Start by locating the existing **Study Analysis** object type in Ontology Manager. Do not create a duplicate if it already exists.

# Study Analysis dataset

| Study Number | Has Started CAR | Is Qualified for CAR | Qualification Reason | Generation Selection JSON | Generation Selection Updated At | Generation Selection Updated By | CAR Generation Status | CAR Generation Request ID |
|---|---|---|---|---|---|---|---|---|
| STUDY-2026-001 | false | false |  | `{ "sections": [] }` |  |  | Not Started |  |
| STUDY-2026-002 | true | true | Meets the criteria for CAR generation. | `{ "sections": [{ "sectionKey": "executiveSummary", "generate": true, "text": true, "tables": false }, { "sectionKey": "materials", "generate": true, "text": true, "tables": true }] }` | 2026-10-09T12:30:00Z | user@example.com | Completed | car-gen-8f3c2a1b |
| STUDY-2026-003 | true | false | Study does not meet CAR qualification criteria. | `{ "sections": [] }` | 2026-10-09T13:15:00Z | user@example.com | Not Started |  |


Verify that it has a stable identifier and the properties needed for this workflow. Add the following property if it does not already exist:

| Property | Type | Purpose |
|---|---|---|
| `generationSelectionJson` | String | Stores the saved Generate, Text, and Tables selections for all report sections |
| `generationSelectionUpdatedAt` | Timestamp, optional | Records when the selection was last submitted |
| `generationSelectionUpdatedBy` | String, optional | Records who submitted the selection, if your security and identity setup supports it |

You may already have a status property. If needed, add `generationSelectionStatus` as a String or configured enum. It is optional for the first version.

**Do not create one Boolean property per section** unless the list is fixed by design. Do not create a separate selection object per section for this version.

### 3.2 Section catalogue

Create a small dataset in Foundry for the report section catalogue, or use an approved existing source if your project already has one. The catalogue defines which sections are available; it is not the user's saved checkbox state.

Recommended columns:

| Column | Type | Example | Meaning |
|---|---|---|---|
| `sectionKey` | String | `1.1` | Stable unique key; do not use the display title as the key |
| `parentSectionKey` | String, nullable | `1.0` | Parent section key; blank for a root section |
| `title` | String | `Executive Summary` | Label shown in Workshop |
| `displayOrder` | Integer | `10` | Sort order among siblings |
| `sectionType` | String | `GROUP` or `CONTENT` | Group can contain children; Content is a leaf section |
| `supportsText` | Boolean | `true` | Whether Text can be selected |
| `supportsTables` | Boolean | `false` | Whether Tables can be selected |
| `active` | Boolean | `true` | Whether the section is currently available |
| `templateKey` | String, optional | `EXEC_SUMMARY` | Key used later by the generation service |

Use stable keys even if section titles change. Confirm the actual section names and whether each supports text or tables before loading production data.

### 3.3 Example catalogue rows

You can use these rows to test the hierarchy, then replace them with your actual report structure:

```csv
sectionKey,parentSectionKey,title,displayOrder,sectionType,supportsText,supportsTables,active,templateKey
1.0,,Report Summary,10,GROUP,true,true,true,REPORT_SUMMARY
1.1,1.0,Executive Summary,10,CONTENT,true,false,true,EXEC_SUMMARY
1.2,1.0,Key Findings,20,CONTENT,true,true,true,KEY_FINDINGS
1.3,1.0,Recommendations,30,CONTENT,true,false,true,RECOMMENDATIONS
2.0,,Fact-finding,20,GROUP,true,true,true,FACT_FINDING
2.1,2.0,Background,10,CONTENT,true,false,true,BACKGROUND
2.2,2.0,Evidence Tables,20,CONTENT,false,true,true,EVIDENCE_TABLES
3.0,,Cost Analysis,30,GROUP,true,true,true,COST_ANALYSIS
3.1,3.0,Materials,10,CONTENT,true,true,true,MATERIALS
3.2,3.0,Labor,20,CONTENT,true,true,true,LABOR
4.0,,Profit Analysis,40,CONTENT,true,true,true,PROFIT_ANALYSIS
5.0,,Price Analysis,50,CONTENT,true,true,true,PRICE_ANALYSIS
6.0,,Analysis Conclusion,60,CONTENT,true,false,true,ANALYSIS_CONCLUSION
```

Load the CSV as a Foundry dataset using your team's approved ingestion workflow. Do not recreate or insert catalogue rows every time a user opens the Workshop page.

---

## 4. Define the saved JSON structure on Study Analysis

The `generationSelectionJson` property contains one JSON document. Each item is keyed by the stable `sectionKey`.

Example value:

```json
{
  "version": 1,
  "sections": {
    "1.1": {
      "generate": true,
      "includeText": true,
      "includeTables": false
    },
    "1.2": {
      "generate": true,
      "includeText": true,
      "includeTables": true
    },
    "1.3": {
      "generate": false,
      "includeText": false,
      "includeTables": false
    }
  }
}
```

The example includes only a few sections to keep it readable. In the real saved value, include every section whose selection state must be persisted.

Rules:

- `sectionKey` identifies the section; it must match a catalogue key.
- `generate` controls whether the section is selected for generation.
- `includeText` controls the Text option.
- `includeTables` controls the Tables option.
- A value should not be true when the catalogue says the option is unsupported. Enforce this in the UI and validate it again when submitting.
- The JSON `version` allows you to evolve the format later.

### What if `generationSelectionJson` is empty?

Choose a clear initialization policy:

1. When the page loads, read the selected Study Analysis object.
2. If `generationSelectionJson` contains valid saved JSON, use it as the initial checkbox state.
3. If it is empty, create an **in-memory default draft** from the active catalogue. For example, set all checkboxes to false.
4. Do not save these defaults to Study Analysis merely because the page was opened.
5. Persist them only when the user clicks Submit Study Analysis.

This keeps the requirement intact: **opening the page and changing checkboxes do not write to Study Analysis**.

If business rules require some sections to be selected by default, apply those defaults in the in-memory draft, not by silently updating the object on page load.

---

## 5. Create the Workshop page from scratch

### Step 1 — Create a Workshop module

1. Open the relevant Foundry project.
2. Create a new Workshop module/page.
3. Give it a name such as **Study Analysis — Generate Report**.
4. Add a way to select or open a particular Study Analysis object.
5. Ensure the page has access to the selected Study Analysis object and the section catalogue.

The page must work for the selected Study Analysis, not a hard-coded test object.

### Step 2 — Load the selected Study Analysis object

Configure the page to resolve the selected Study Analysis object from the appropriate object set, object view, or navigation context supported by your Workshop setup.

Use that object as the source for:

- The Study Analysis identifier.
- The current `generationSelectionJson` value.
- Any other Study Analysis details shown in the page.

Do not load checkbox values from a separate per-section selection object in this design. The saved values come from the selected Study Analysis object's `generationSelectionJson` property.

### Step 3 — Load the section catalogue

Load active rows from the catalogue dataset and sort them by `displayOrder` within their hierarchy.

The catalogue provides:

- Section titles.
- Parent-child relationships.
- Which checkboxes are supported.
- Display order.

It does **not** override saved checkbox values from Study Analysis.

### Step 4 — Create Workshop variables

Create variables with the equivalent purpose below. The exact variable configuration and available types depend on your Foundry Workshop version.

| Variable | Purpose |
|---|---|
| `selectedStudyAnalysis` | The selected Study Analysis object |
| `sectionCatalogue` | Active section definitions from the catalogue |
| `savedSelectionJson` | The saved JSON value read from Study Analysis |
| `draftSectionSelections` | In-memory working copy of checkbox values |
| `isDirty` | Whether the draft differs from the saved values |
| `submitResult` | Optional result/status shown after Submit |

The critical distinction is:

- `savedSelectionJson` is the last persisted value.
- `draftSectionSelections` is what the user is currently editing.
- Checkbox controls update **only** `draftSectionSelections`.
- Submit sends the draft to an action that updates Study Analysis.

Do not bind each checkbox directly to a write action or directly to an object property update.

### Step 5 — Initialize the draft

When the selected Study Analysis changes, initialize `draftSectionSelections` from that object's saved JSON.

Conceptually:

```text
on selected Study Analysis change:
    read generationSelectionJson from Study Analysis

    if valid saved JSON exists:
        draftSectionSelections = parse(saved JSON)
    else:
        draftSectionSelections = defaults generated in memory from active catalogue

    isDirty = false
```

Use the Workshop variable and action capabilities available in your environment to implement this flow. Avoid overwriting the user's current draft on every catalogue refresh or on every checkbox click. Reinitialize when the selected Study Analysis changes or when the user explicitly chooses to discard/reload changes.

### Step 6 — Build the section row UI

Create a reusable child module for a section row if your Workshop setup supports embedded modules. Otherwise, build the row using the available Workshop components.

Each row should display:

1. Expand/collapse control if the section has children.
2. Generate checkbox.
3. Section title.
4. Text checkbox, if `supportsText` is true.
5. Tables checkbox, if `supportsTables` is true.

Disable or hide Text/Tables controls when the catalogue says they are unsupported. Be consistent across all rows.

Use a **Loop layout** when you need to repeat the row module for each item in an array or object set. A Loop repeats the row UI; it does not automatically construct a recursive tree or provide parent/child checkbox semantics.

### Step 7 — Display a hierarchy

For a shallow, known hierarchy, you may use nested row modules or nested loops if the Workshop capabilities and data shape support them.

For a deeper or configurable hierarchy, prepare a flattened list for display, with fields such as:

```text
sectionKey
parentSectionKey
title
depth
hasChildren
isExpanded
displayOrder
```

Filter the visible list according to which parent rows are expanded. Indent each row according to `depth`.

If your current Workshop components cannot support a genuinely nested hierarchy or a parent checkbox with an indeterminate state, build a simpler first version with a flat ordered list, or use an approved custom component. Do not assume Loop alone solves these behaviors.

### Step 8 — Define checkbox behavior

All checkbox handlers update the in-memory draft only.

**Generate**
- Clicking a leaf section updates only that section's `generate` value.
- Clicking a group section can select or clear its descendants if that is the chosen business rule.
- If a parent has a mixture of selected and unselected descendants, show an indeterminate state only if the chosen Workshop control supports it. Otherwise, define a clear alternative visual rule.
- Decide whether selecting Generate automatically selects Text or Tables. Do not assume it should; document and implement the rule explicitly.

**Text**
- Updates only that section's `includeText` draft value.
- Only enabled if `supportsText` is true.

**Tables**
- Updates only that section's `includeTables` draft value.
- Only enabled if `supportsTables` is true.

After any checkbox change:

```text
draftSectionSelections = updated draft
isDirty = compare draft with saved selection
```

There must be **no call to the update Study Analysis action** in these checkbox handlers.

### Step 9 — Add Submit Study Analysis

Add a button named **Submit Study Analysis**.

When clicked:

1. Validate the draft against the catalogue.
2. Build the complete JSON payload, including all sections that should be persisted.
3. Call the action that updates the selected Study Analysis object.
4. In that action, set `generationSelectionJson` to the submitted payload.
5. Optionally set `generationSelectionUpdatedAt`, `generationSelectionUpdatedBy`, and a status property.
6. Wait for the action result.
7. Only after success, update the UI's saved-state snapshot and set `isDirty = false`.
8. Show a success message. If the action fails, keep the draft so the user can correct or retry it.

Conceptually:

```text
on Submit Study Analysis:
    validate draftSectionSelections

    payload = serialize({
        version: 1,
        sections: draftSectionSelections
    })

    call Update Study Analysis action with:
        target Study Analysis = selectedStudyAnalysis
        generationSelectionJson = payload
        generationSelectionUpdatedAt = current timestamp (if supported)
        generationSelectionUpdatedBy = current user (if supported)

    if action succeeds:
        savedSelectionJson = payload
        isDirty = false
        show success
    else:
        keep draft unchanged
        show error
```

The action must target the **selected existing Study Analysis object**. Do not create a new Study Analysis object on each submit unless creating a new business object is explicitly part of the workflow.

---

## 6. Configure the action that updates Study Analysis

Create an action type in Ontology Manager for updating the existing Study Analysis object. The exact steps and field names vary by Foundry configuration, but the action should have this shape:

**Action name:** `Submit Study Analysis Selection`

**Inputs:**
- Target Study Analysis object or its required primary key.
- `generationSelectionJson` payload.
- Optional updated-at and updated-by values, if your environment permits these to be supplied safely.

**Effects:**
- Modify the existing Study Analysis object's `generationSelectionJson`.
- Optionally update audit/status properties.
- Do not create section-selection objects in this design.
- Do not create another Study Analysis object.

Configure permissions and validation according to your project's access model. Prefer server-side validation where possible; do not rely only on client-side checkbox disabling.

### Validation before writing

Validate at least the following:

- The target Study Analysis exists and the user is authorized to modify it.
- The payload is valid JSON and uses a supported schema version.
- Every submitted `sectionKey` exists in the active catalogue, or is explicitly allowed by your business rules.
- `includeText` is not true for a section that does not support Text.
- `includeTables` is not true for a section that does not support Tables.
- Parent/child Generate rules are consistent with your agreed business rules.

If the action framework does not support a particular validation directly, validate in the approved backend/function layer before performing the write.

---

## 7. Test the required save behavior

Use a test Study Analysis object and verify each scenario.

| Test | Expected result |
|---|---|
| Open the page | Checkboxes show the values saved on Study Analysis |
| Open a Study Analysis with empty JSON | Default draft appears; object remains unchanged |
| Toggle Generate | Only the in-memory draft changes |
| Toggle Text or Tables | Only the in-memory draft changes |
| Navigate away without submitting | No checkbox changes have been saved to Study Analysis |
| Click Submit Study Analysis | One update action writes the draft to the selected Study Analysis object |
| Submit succeeds | Saved values match the draft and the UI is no longer dirty |
| Submit fails | Draft is retained; error is shown; persisted object is not reported as updated |
| Switch to another Study Analysis | Draft is initialized from the other object's saved JSON |
| Return to the first Study Analysis | Its last submitted values are shown |
| Add a new catalogue section | It can be displayed without adding a new Boolean property to Study Analysis |

---

## 8. Recommended build order

Implement and verify in small milestones:

1. **Study Analysis property:** confirm `generationSelectionJson` exists.
2. **Section catalogue:** load a few test sections with parent keys and capability flags.
3. **Read-only page:** display the selected Study Analysis and section titles.
4. **Initial checkbox state:** parse the JSON from Study Analysis, or create defaults in memory if empty.
5. **Draft behavior:** confirm toggles change only the Workshop draft.
6. **Submit action:** update the existing Study Analysis object only after clicking Submit.
7. **Hierarchy:** add expand/collapse and parent-child selection behavior.
8. **Validation and error handling:** test unsupported options and failed submissions.
9. **Generation workflow:** only after saving selections works, connect a separate Generate action or process if required.

Do not start by building every nested checkbox behavior at once. First prove the read → draft → submit → read-back lifecycle for one Study Analysis and two or three sections.

---

## 9. Common mistakes to avoid

- **Writing on every click:** checkbox handlers must not call the Study Analysis update action.
- **Using a Loop as a complete tree engine:** Loop repeats UI; hierarchy, expansion, and parent selection need separate logic.
- **Hard-coding one property per section:** this becomes difficult to maintain as the catalogue changes.
- **Creating a new Study Analysis on submit:** the action should modify the selected existing object.
- **Treating catalogue definitions as saved selections:** catalogue rows define labels and capabilities; Study Analysis JSON stores the user's last submitted values.
- **Overwriting the draft during refresh:** load saved values when selecting a different Study Analysis or when explicitly discarding edits, not after each checkbox click.
- **Reporting success too early:** show success only after the update action confirms completion.
- **Assuming exact Workshop features:** component names, JSON parsing support, nested loops, indeterminate checkboxes, and action inputs vary by Foundry version and project configuration. Validate them in your environment.

---

## 10. Official documentation

- [Workshop Loop layouts](https://www.palantir.com/docs/foundry/workshop/loop-layouts)
- [Workshop variables](https://www.palantir.com/docs/foundry/workshop/concepts-variables)
- [Workshop layouts](https://www.palantir.com/docs/foundry/workshop/concepts-layouts)
- [Object and link types](https://www.palantir.com/docs/foundry/object-link-types/type-reference)
- [Create an object type](https://www.palantir.com/docs/foundry/object-link-types/create-object-type/index.html)
- [Action types overview](https://www.palantir.com/docs/foundry/action-types/overview/index.html)
- [Action rules](https://www.palantir.com/docs/foundry/action-types/rules)

## Final design rule

**Read checkbox state from the selected Study Analysis object → edit an in-memory Workshop draft → write the draft back to that same Study Analysis object only when Submit Study Analysis is clicked.**
