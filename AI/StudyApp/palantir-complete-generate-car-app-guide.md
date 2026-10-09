# Palantir Foundry — Build the Complete Generate CAR App from Scratch

**Scope:** Ontology, object types, properties, section catalogue, action type, Workshop page, draft state, checkbox hierarchy, submit flow, validation, and testing.

**Excluded for now:** Uploads, file drop zones, upload categories, file storage, and file-to-CAR associations.

This guide assumes a fresh implementation. You may already have a `Study Analysis` object type in your Foundry project; inspect it first and reuse it if it exists. Do not create a duplicate object type.

---

# 1. What you are building

Recreate the non-upload parts of the supplied CAR form:

1. Study Number
2. Have you started this CAR?
3. Is this a qualified CAR?
4. Why is this qualified?
5. Generation section tree/table with Generate, Text, and Table(s)
6. **Generate CAR** submit button

The essential requirement is that the selected Study Analysis object supplies the saved values shown on the page. User edits remain a Workshop draft until they click **Generate CAR**. On submit, update that same Study Analysis object with both the form values and all section selections.

## Architecture

```text
Ontology
  └── Study Analysis object type
        ├── CAR form properties
        ├── generationSelectionJson
        └── generation status properties

Section catalogue dataset
  └── Defines section keys, labels, parent hierarchy, order, capabilities

Workshop: Generate CAR
  ├── Reads selected Study Analysis
  ├── Reads section catalogue
  ├── Initializes form draft and section-selection draft
  ├── Displays fields and checkbox rows
  └── Generate CAR button
        └── Submit Generate CAR action
              ├── Validates inputs
              ├── Updates the same Study Analysis object
              └── Starts/queues generation only if that workflow is connected
```

## Chosen data design

Use **one Study Analysis object** to store both the form fields and the user's last-submitted section selection JSON. Do not create a separate Section Selection object type for this version.

Keep a separate section catalogue dataset because section definitions (title, parent, order, supported options) are configuration. The catalogue does not store user selections.

---

# 2. Plan the Foundry resources

Create or configure resources in this order:

1. Confirm the source dataset for Study Analysis.
2. Create or update the Study Analysis object type in Ontology Manager.
3. Create and load the section catalogue dataset.
4. Add/map the section catalogue as a readable source for Workshop (or expose it through the project's approved object/dataset access pattern).
5. Create the `Submit Generate CAR Form` action type.
6. Create the Workshop module and variables.
7. Build and test the form fields.
8. Build and test the section table and checkbox draft behavior.
9. Connect **Generate CAR** to the submit action.
10. Test persistence and read-back.
11. Connect the actual CAR generation backend/workflow if available.

Foundry menu labels and exact configuration screens vary by tenant, permissions, and product version. Follow the equivalent current workflow in your environment.

---

# 3. Ontology: create or reuse Study Analysis

## 3.1 Locate the existing source

1. Open **Ontology Manager**.
2. Search for `Study Analysis`.
3. If the object type already exists, open it and inspect:
   - Its backing dataset/source.
   - Its primary key.
   - Existing properties and types.
   - Whether the properties are editable through actions.
4. Reuse it if it represents the correct business record.
5. If no appropriate object type exists, identify the approved Study Analysis source dataset first, then create the object type from that source using your project's standard Foundry workflow.

Do not create an empty duplicate object type just because this guide uses example names. An object type normally needs a backing source and a stable primary key.

## 3.2 Object identity

| Setting | Recommendation | Example |
|---|---|---|
| Object type display name | `Study Analysis` | Study Analysis |
| Primary key | Existing stable source ID | `SA-000123` |
| Business identifier | `studyNumber` | `CY-00-000` |
| Display title | Use Study Number if appropriate for the current ontology configuration | `CY-00-000` |

A Study Number may be a business identifier, but do not use it as the primary key unless it is guaranteed unique and immutable. Keep the existing primary key if the object type already has one.

---

# 4. Study Analysis object type: final properties

In Ontology Manager, add only the properties that are missing. Reuse equivalent existing properties rather than creating duplicates.

| Property API name | Suggested type | Required? | Example | Purpose |
|---|---|---:|---|---|
| `studyNumber` | String | Yes | `CY-00-000` | Study Number shown in the form |
| `hasStartedCar` | String or configured enum | Yes | `IN_PROGRESS` | Selects in-progress or blank CAR template |
| `isQualifiedCar` | Boolean | Yes | `true` | Qualified CAR Yes/No |
| `qualificationReason` | String or configured enum, nullable | Conditional | `PENDING_TINA_SUBMISSIONS` | Selected reason |
| `generationSelectionJson` | String | Yes for this design | JSON string | Last-submitted section selections |
| `generationSelectionUpdatedAt` | Timestamp | Optional | `2026-10-09T10:30:00Z` | Last successful submit time |
| `generationSelectionUpdatedBy` | String | Optional | `example-user` | Audit information, if supported |
| `carGenerationStatus` | String or configured enum | Recommended | `NOT_STARTED` | Tracks the actual generation lifecycle |
| `carGenerationRequestId` | String, nullable | Optional | `CAR-REQ-000123` | Correlates with an asynchronous job |
| `lastGenerationMessage` | String, nullable | Optional | `Ready to generate` | User-friendly status |

### Fixed-choice values

`hasStartedCar`:

| Stored value | UI label |
|---|---|
| `IN_PROGRESS` | Yes, generate using my in-progress CAR template |
| `BLANK` | No, generate from the blank CAR Template |

`isQualifiedCar`:

| Stored value | UI label |
|---|---|
| `true` | Yes, it is qualified |
| `false` | No, it's not qualified |

`qualificationReason` examples (confirm the approved list with the business owner):

| Stored value | UI label |
|---|---|
| `PENDING_TINA_SUBMISSIONS` | Pending the receipt of outstanding Truth in Negotiations Act (TINA) submissions by the supplier |
| `PENDING_DCAA_DCMA_AUDIT` | Pending the receipt of the DCAA/DCMA Assist Audit for Direct Labor Rates and Overhead costs |
| `PENDING_AUDIT_AND_TINA` | Pending the audit and outstanding TINA submissions |

When `isQualifiedCar` is false, clear the draft reason or ignore it during validation according to the agreed business rule. Do not accidentally resubmit a stale reason.

### Generation status values

Suggested lifecycle:

- `NOT_STARTED` — no generation has been requested.
- `QUEUED` — request accepted and queued.
- `PROCESSING` — generation is running.
- `SUCCEEDED` — generation completed successfully.
- `FAILED` — generation failed.

Do not set `SUCCEEDED` just because the form-save action succeeded. If no generation backend is connected yet, save the form and use a status such as `NOT_STARTED` or a separate agreed `READY_TO_GENERATE` value.

---

# 5. Section selections stored on Study Analysis

## 5.1 Property strategy

Store all submitted section selections in the `generationSelectionJson` String property on Study Analysis.

Do **not** create individual properties such as `coverGenerate`, `section1Generate`, `section1Text`, or `section1Tables`. That approach makes the object schema grow whenever the report hierarchy changes.

Example of the readable JSON document:

```json
{
  "version": 1,
  "sections": {
    "COVER": {
      "generate": true,
      "includeText": true,
      "includeTables": false
    },
    "1.0": {
      "generate": true,
      "includeText": true,
      "includeTables": true
    },
    "1.1": {
      "generate": false,
      "includeText": false,
      "includeTables": true
    },
    "1.2": {
      "generate": false,
      "includeText": true,
      "includeTables": false
    },
    "2.0": {
      "generate": true,
      "includeText": true,
      "includeTables": false
    }
  }
}
```

The example is abbreviated. Store every section's state when submitting the complete form. Each key must match a `sectionKey` in the catalogue.

**Foundry type note:** if `generationSelectionJson` is a String property, save serialized JSON text, not an arbitrary nested object. The actual stored String is escaped, for example:

```json
"{\"version\":1,\"sections\":{\"COVER\":{\"generate\":true,\"includeText\":true,\"includeTables\":false}}}"
```

Workshop/backend must serialize and parse this string using capabilities supported in your environment. If your project's approved implementation uses a structured property or backend-managed schema instead, follow that standard.

## 5.2 One combined Study Analysis record

Conceptual example, expanded for readability:

```json
{
  "studyAnalysisId": "SA-000123",
  "studyNumber": "CY-00-000",
  "hasStartedCar": "IN_PROGRESS",
  "isQualifiedCar": true,
  "qualificationReason": "PENDING_TINA_SUBMISSIONS",
  "generationSelectionJson": {
    "version": 1,
    "sections": {
      "COVER": {
        "generate": true,
        "includeText": true,
        "includeTables": false
      },
      "1.0": {
        "generate": true,
        "includeText": true,
        "includeTables": true
      },
      "1.1": {
        "generate": false,
        "includeText": false,
        "includeTables": true
      }
    }
  },
  "generationSelectionUpdatedAt": "2026-10-09T10:30:00Z",
  "generationSelectionUpdatedBy": "example-user",
  "carGenerationStatus": "NOT_STARTED",
  "carGenerationRequestId": null,
  "lastGenerationMessage": null
}
```

This is a conceptual record, not necessarily the literal format of a Foundry object export. If the property is String-typed, `generationSelectionJson` contains serialized JSON text.

---

# 6. Create the section catalogue dataset

This dataset defines what the Workshop displays. It does not store the user's checkbox choices.

## 6.1 Columns

| Column | Type | Example | Purpose |
|---|---|---|---|
| `sectionKey` | String | `1.1` | Stable unique key |
| `parentSectionKey` | String, nullable | `1.0` | Parent key; empty for a root |
| `title` | String | `Summary by CLIN/Total Price` | Row label |
| `displayOrder` | Integer | `10` | Order among siblings |
| `sectionType` | String | `GROUP` / `CONTENT` | Parent group or leaf/content |
| `supportsText` | Boolean | `false` | Whether Text is available |
| `supportsTables` | Boolean | `true` | Whether Table(s) is available |
| `active` | Boolean | `true` | Whether to display the section |
| `templateKey` | String | `SUMMARY_CLIN_TOTAL_PRICE` | Optional generation mapping |

Use your approved Foundry dataset creation/import workflow. Load the catalogue once as configuration; do not insert rows each time the Workshop opens.

## 6.2 Starter CSV

This is example data based on visible screenshot rows, not a complete approved CAR template. Validate titles, ordering, hierarchy, and capability flags with the business owner.

```csv
sectionKey,parentSectionKey,title,displayOrder,sectionType,supportsText,supportsTables,active,templateKey
COVER,,Cover page,0,CONTENT,true,false,true,COVER_PAGE
1.0,,Report Summary,10,GROUP,true,true,true,REPORT_SUMMARY
1.1,1.0,Summary by CLIN/Total Price,10,CONTENT,false,true,true,SUMMARY_CLIN_TOTAL_PRICE
1.2,1.0,Description of Proposed Effort,20,CONTENT,true,false,true,PROPOSED_EFFORT
1.3,1.0,Background Information,30,CONTENT,true,false,true,BACKGROUND_INFORMATION
2.0,,Fact-finding,20,GROUP,true,true,true,FACT_FINDING
2.1,2.0,Date and Location,10,CONTENT,true,false,true,DATE_LOCATION
2.2,2.0,Fact-Finding Participants,20,CONTENT,true,false,true,FACT_FINDING_PARTICIPANTS
2.3,2.0,Data Provided,30,CONTENT,true,false,true,DATA_PROVIDED
2.4,2.0,CAS and Accounting System Adequacy,40,CONTENT,true,false,true,CAS_ACCOUNTING_SYSTEM
2.5,2.0,Technical Analysis,50,CONTENT,true,false,true,TECHNICAL_ANALYSIS
3.0,,Cost Analysis,30,GROUP,true,true,true,COST_ANALYSIS
3.1,3.0,Summary by Cost Element,10,CONTENT,true,true,true,SUMMARY_COST_ELEMENT
```

## 6.3 Make the catalogue available to Workshop

Use the approved way in your Foundry project to query/read this dataset from Workshop. Depending on the setup, that may involve a backing object type for catalogue rows or an existing dataset-backed variable/query pattern. Do not create a new Ontology object type solely because this guide mentions a dataset; choose the supported access pattern available in your tenant.

---

# 7. Create the submit Action Type

The action saves the draft to the existing Study Analysis object. It should not create another Study Analysis object.

## 7.1 Create the action

1. Open **Ontology Manager** and navigate to **Action types** (or the equivalent action configuration area).
2. Create an action type named `Submit Generate CAR Form`.
3. Choose the operation/effect that modifies an existing object.
4. Select `Study Analysis` as the target object type.
5. Configure the action to target the existing selected object using its stable primary key/object reference.
6. Add inputs for the fields that are to be submitted.
7. Map each input to the corresponding Study Analysis property.
8. Configure validation rules and permissions according to your environment.
9. Save, then test the action against a non-production record.

Exact UI steps vary between action builder versions. Follow the current action editor's required configuration; do not assume a field or effect exists if it is not available in your tenant.

## 7.2 Action inputs and property mappings

| Action input | Maps to Study Analysis property |
|---|---|
| `targetStudyAnalysis` or target primary key | Existing Study Analysis object to modify |
| `studyNumber` | `studyNumber` |
| `hasStartedCar` | `hasStartedCar` |
| `isQualifiedCar` | `isQualifiedCar` |
| `qualificationReason` | `qualificationReason` |
| `generationSelectionJson` | `generationSelectionJson` |
| `generationSelectionUpdatedAt` | `generationSelectionUpdatedAt`, if used |
| `generationSelectionUpdatedBy` | `generationSelectionUpdatedBy`, if supported |
| `carGenerationStatus` | `carGenerationStatus`, only with defined status semantics |

Use the actual target-object input pattern supported by the action builder. Do not pass an untrusted arbitrary ID without verifying that the user may modify that object.

## 7.3 Validation rules

Validate server-side, not only in Workshop:

- The target Study Analysis exists and the user has permission to modify it.
- Required Study Number and CAR choices are present.
- The qualification reason is valid for the selected qualified state.
- The selection JSON parses and has a supported `version`.
- Every section key exists in the active catalogue.
- `includeText` is not true when `supportsText` is false.
- `includeTables` is not true when `supportsTables` is false.
- Parent/child Generate rules follow the business agreement.
- Duplicate submissions or concurrent edits are handled according to project policy.

If the action editor cannot perform a particular validation, use the approved function/backend layer before the object update. Do not rely solely on client-side validation for data integrity.

## 7.4 Atomicity and generation

Where possible, save the form fields and selection JSON in the same action so the record represents one submission. If the generation process is a separate asynchronous service, start/queue it only after the save succeeds. If saving and queueing require different systems, define retry/idempotency behavior so a retry does not create duplicate CAR jobs.

---

# 8. Create the Workshop module

## 8.1 Create the page

1. Open Workshop and create a new module named **Generate CAR**.
2. Configure how the page receives the selected Study Analysis:
   - from navigation/object context, if opened from a Study Analysis object page; or
   - from an object selector/search component, if the user starts from a general page.
3. Ensure the page is working with an existing Study Analysis object.
4. Add a header and the form sections in the same order as the screenshot.
5. Do not add upload controls in this phase.

## 8.2 Create Workshop variables

Use the variable types and data sources supported by your Workshop version. Suggested logical variables:

| Variable | Content |
|---|---|
| `selectedStudyAnalysis` | Existing Study Analysis object |
| `sectionCatalogue` | Active ordered section definitions |
| `savedSelectionJson` | Saved selection String from Study Analysis |
| `draftForm` | Unsaved values for Study Number, started CAR, qualification, reason |
| `draftSectionSelections` | Unsaved Generate/Text/Table(s) map keyed by sectionKey |
| `isDirty` | Whether the draft differs from the last saved values |
| `isSubmitting` | Prevents duplicate submits |
| `submitStatus` | Idle, submitting, success, or failure |
| `submitError` | User-friendly validation/action error |

Keep the saved object state separate from the draft. Input and checkbox changes must update only the draft variables.

## 8.3 Initialize the page

Conceptual initialization logic:

```text
when selectedStudyAnalysis changes:
    draftForm = copy the form properties from selectedStudyAnalysis
    savedSelectionJson = selectedStudyAnalysis.generationSelectionJson

    if savedSelectionJson is valid and supported:
        draftSectionSelections = parse(savedSelectionJson)
    else:
        draftSectionSelections = default state for active catalogue sections

    isDirty = false
    submitStatus = IDLE
```

If the saved JSON is empty, choose a documented business default (for example, all Generate values false). Build those defaults in memory only. Do not write them to Study Analysis just because the page opened.

If the user switches to another Study Analysis, initialize from that object's saved values. Avoid replacing a dirty draft without warning or explicit discard behavior.

---

# 9. Build the form fields

## 9.1 Study Number

- Add a text input labeled `Study Number*`.
- Initial value comes from `selectedStudyAnalysis.studyNumber`.
- Bind the editable value to `draftForm.studyNumber`.
- Add the agreed format/required validation.
- Do not directly update the object when the user types.

## 9.2 Have you started this CAR?

Add a radio group with:

- **Yes, generate using my in-progress CAR template** → `IN_PROGRESS`
- **No, generate from the blank CAR Template** → `BLANK`

Bind the control to `draftForm.hasStartedCar`.

## 9.3 Is this a qualified CAR?

Add a radio group with:

- **Yes, it is qualified** → `true`
- **No, it's not qualified** → `false`

Bind it to `draftForm.isQualifiedCar`.

## 9.4 Why is this qualified?

Add a radio group or dropdown populated with the approved qualification reasons. Bind it to `draftForm.qualificationReason`.

- Show/enable it when `draftForm.isQualifiedCar` is true.
- When false, clear the draft reason or ignore it during validation, based on the agreed rule.
- Do not save until Generate CAR is clicked.

---

# 10. Build the Generation section table

## 10.1 Row content

Each visible row should contain:

- Expand/collapse control when the section has children.
- Section title.
- Generate toggle.
- Text checkbox if supported.
- Table(s) checkbox if supported.

Use `sectionKey` as the stable key, not the displayed title.

## 10.2 Loop layout

A Workshop Loop layout can repeat a child row module for each item in an array/object set. Configure it to iterate over the ordered section data or the flattened visible rows prepared for the UI.

Important: Loop repeats a row/module; it does not automatically build an arbitrary recursive tree or implement parent/child checkbox rules.

For a fixed shallow hierarchy, nested loops/modules may be sufficient if your Workshop version supports the required structure. For a dynamic hierarchy, prepare a flattened visible-row list with fields such as:

```text
sectionKey
parentSectionKey
title
depth
hasChildren
isExpanded
displayOrder
supportsText
supportsTables
```

Use `depth` for indentation and `isExpanded` to control visibility. If the available Workshop components cannot show arbitrary nesting or a mixed/indeterminate parent checkbox, start with an ordered flat list or use an approved custom component.

## 10.3 Checkbox rules

Define these rules before building the handlers:

- A leaf Generate toggle changes only that leaf's draft value.
- A group Generate toggle selects or clears descendants if that is the approved business rule.
- If child values are mixed, show an indeterminate parent state only if the selected component supports it. Otherwise choose a clear documented alternative.
- Text and Table(s) are independent of Generate unless the business rule says otherwise.
- Hide or disable Text/Table(s) if the catalogue says the option is unsupported.
- Do not assume a parent Generate toggle automatically updates all children; implement and test that behavior explicitly.

All row handlers update only `draftSectionSelections`. They must not call the action or write to the object.

---

# 11. Connect the Generate CAR button

The button is the submission point for the complete form draft.

## 11.1 Button behavior

1. Add a button labeled **Generate CAR**.
2. Disable it while `isSubmitting` is true.
3. On click, validate the draft fields.
4. Validate the section selection map against the catalogue.
5. Serialize the full selection map into the String format required by `generationSelectionJson`.
6. Set `isSubmitting = true` and clear old errors.
7. Invoke `Submit Generate CAR Form` against the selected existing Study Analysis object.
8. Wait for the action result.
9. On confirmed success:
   - refresh/read the Study Analysis object;
   - synchronize saved and draft variables with the successful values;
   - set `isDirty = false`;
   - show a success message;
   - show the real generation status.
10. On failure:
    - retain the user's draft;
    - show the validation/action error;
    - do not display a success message;
    - allow a corrected retry.

Pseudocode:

```text
on Generate CAR click:
    if isSubmitting:
        stop

    validate draftForm
    validate draftSectionSelections against sectionCatalogue
    if validation fails:
        show errors
        stop

    serializedSelections = serialize({
        version: 1,
        sections: draftSectionSelections
    })

    isSubmitting = true
    result = call Submit Generate CAR Form(
        target = selectedStudyAnalysis,
        studyNumber = draftForm.studyNumber,
        hasStartedCar = draftForm.hasStartedCar,
        isQualifiedCar = draftForm.isQualifiedCar,
        qualificationReason = draftForm.qualificationReason,
        generationSelectionJson = serializedSelections
    )

    if result succeeds:
        refresh selectedStudyAnalysis
        copy saved values back into drafts
        isDirty = false
        show confirmation
    else:
        preserve drafts
        show error

    isSubmitting = false
```

Implement the equivalent using the supported Workshop action interaction and variable update mechanisms in your tenant; this pseudocode describes behavior, not a copy-paste Foundry expression.

## 11.2 Save versus generate

The screenshot's former **Edit CAR** button is now **Generate CAR** and is treated here as the submit button, as requested. Decide with the business owner whether the click only saves/submits the request or also starts generation.

- If it only saves/submits, do not claim that the CAR has already been generated.
- If it starts an asynchronous generation process, set `QUEUED`/`PROCESSING` after the request is accepted and update to `SUCCEEDED`/`FAILED` from the real process result.
- If a separate **Save Progress** button is needed, define it as a distinct action that saves a draft without starting generation. Do not accidentally wire it to Generate CAR.

---

# 12. Test the complete app

Use a non-production Study Analysis record.

| Test | Expected result |
|---|---|
| Open an existing Study Analysis | Form fields show its saved values |
| Open with saved selection JSON | Checkboxes reflect that JSON |
| Open with empty selection JSON | Default choices appear only in draft |
| Change a field | Object remains unchanged before submit |
| Change a checkbox | Object remains unchanged before submit |
| Expand/collapse a section | Visibility changes without saving object data |
| Toggle a group Generate | Descendant behavior matches agreed rules |
| Select unsupported Text/Table(s) | UI prevents it; submit validation also rejects it |
| Submit invalid form | Error appears; no object update/generation |
| Click Generate CAR successfully | Same Study Analysis object is updated |
| Submit action fails | Draft remains and no false success is shown |
| Reopen the page | Last successfully submitted values reappear |
| Switch to another Study Analysis | Draft is initialized from the other object |
| Add a catalogue section | It appears without adding a new property to Study Analysis |
| Double-click submit | Duplicate submissions are prevented or handled idempotently |
| User lacks edit permission | Action is rejected safely |

---

# 13. Recommended build milestones

Complete and verify each milestone before moving on.

### Milestone 1 — Ontology

- [ ] Find/reuse the existing Study Analysis object type.
- [ ] Verify its source and primary key.
- [ ] Add only missing properties.
- [ ] Confirm a test record has the expected fields.

### Milestone 2 — Section catalogue

- [ ] Create/load the catalogue dataset.
- [ ] Validate keys, hierarchy, order, and capability flags.
- [ ] Make the catalogue readable from Workshop.
- [ ] Confirm rows are sorted correctly.

### Milestone 3 — Read-only Workshop

- [ ] Create Generate CAR module.
- [ ] Resolve the selected Study Analysis.
- [ ] Display Study Number and saved radio values.
- [ ] Parse saved selection JSON and display the checkboxes.
- [ ] Confirm the page does not modify the object.

### Milestone 4 — Draft interactions

- [ ] Bind inputs to draft variables only.
- [ ] Implement expand/collapse.
- [ ] Implement Generate/Text/Table(s) behavior.
- [ ] Confirm no action runs on each input change.

### Milestone 5 — Action and submit

- [ ] Create Submit Generate CAR Form action.
- [ ] Target the existing Study Analysis object.
- [ ] Map all submitted fields and selection JSON.
- [ ] Add validation and permissions.
- [ ] Connect Generate CAR button.
- [ ] Test success and failure.

### Milestone 6 — Real generation integration

- [ ] Connect the approved CAR generation workflow.
- [ ] Track request ID and real processing status if required.
- [ ] Handle retry, duplicate request, and failure paths.
- [ ] Keep Uploads out of scope until the rest of the app works.

---

# 14. Common mistakes to avoid

1. **Creating a second Study Analysis object on submit.** Modify the selected existing object.
2. **Saving on every checkbox click.** Checkbox changes stay in Workshop draft state.
3. **Adding one property per section.** Store the selection map in `generationSelectionJson`.
4. **Mixing the catalogue and user state.** The catalogue defines sections; Study Analysis stores submitted choices.
5. **Assuming Loop is a recursive tree.** Loop repeats UI for supplied rows; hierarchy logic must be built explicitly.
6. **Trusting client validation only.** Validate again in the action/backend.
7. **Marking generation complete when only the form was saved.** Reflect the real workflow status.
8. **Overwriting a dirty draft when switching objects.** Prompt or explicitly discard/reload.
9. **Assuming native JSON support on a String property.** Serialize before writing and parse after reading.
10. **Implementing uploads too early.** Keep them excluded until the non-upload form and submit flow are correct.

---

# 15. Official documentation

- [Workshop Loop layouts](https://www.palantir.com/docs/foundry/workshop/loop-layouts)
- [Workshop variables](https://www.palantir.com/docs/foundry/workshop/concepts-variables)
- [Workshop layouts](https://www.palantir.com/docs/foundry/workshop/concepts-layouts)
- [Object and link types](https://www.palantir.com/docs/foundry/object-link-types/type-reference)
- [Create an object type](https://www.palantir.com/docs/foundry/object-link-types/create-object-type/index.html)
- [Action types overview](https://www.palantir.com/docs/foundry/action-types/overview/index.html)
- [Action rules](https://www.palantir.com/docs/foundry/action-types/rules)

Foundry capabilities and labels can vary by version, tenant, permissions, and project setup. Verify exact property types, action effects, JSON parsing support, and Workshop component behavior in your environment.

---

# Final rule

**One existing Study Analysis object stores the CAR form values and the last-submitted section selections. The section catalogue defines what sections exist. Workshop holds unsaved edits in draft variables. Clicking Generate CAR validates and submits the whole draft to the same Study Analysis object. Uploads are excluded from this implementation.**
