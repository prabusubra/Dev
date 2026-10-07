# Create Study: Ontology to Form (Step by Step)

Build the **Create Study** form with study number, two toggles and **section checkboxes**, from the Ontology objects through to a Workshop button.

```
Create Study
Study number *        [ ABC-123        ]
Already started       [ ◯ ]
Audit required        [ ◯ ]
Sections *
  ☐ Cover Page
  ☐ Report Summary
  ☐ Profit Analysis
  ☐ Analysis Conclusion
                              [Submit]
```

> Foundry menu labels change between versions. If a button name differs slightly, pick the closest match.

---

## Step 1: Section config data

1. Create `section.csv` on your computer:

   ```csv
   sectionCode,name,displayOrder
   COVER_PAGE,Cover Page,1
   REPORT_SUMMARY,Report Summary,2
   PROFIT_ANALYSIS,Profit Analysis,3
   ANALYSIS_CONCLUSION,Analysis Conclusion,4
   ```

2. Drag it into your project's `data` folder. It becomes a dataset.
3. Open the dataset → **Edit schema** → set `displayOrder` to **Integer** → **Save and apply**.

---

## Step 2: Section object type

1. **Ontology Manager → New → Object type**.
2. Datasource: **Use existing** → `section`.
3. Name `Section`, API name `Section`.
4. Primary key: `sectionCode`. Title: `name`.
5. **Create → Save** → confirm.

---

## Step 3: Study object type

1. **New → Object type** → datasource **Create new** (saved in `data`).
2. Name `Study`, API name `Study`.
3. **+ Add property** for each:

   | API name | Type | Setting |
   |---|---|---|
   | studyId | String | **Primary key** |
   | studyNumber | String | **Title** |
   | alreadyStarted | Boolean | |
   | auditRequired | Boolean | |
   | selectedSections | String | **Array: on** |
   | createdAt | Timestamp | |

4. **Datasources / Capabilities:** turn on **Allow edits**.
5. **Create → Save** → confirm.
6. Check: **Object Explorer → Section** shows 4 objects.

---

## Step 4: Function

1. **New → Code repository → TypeScript v2 Functions** (in the `functions` folder).
2. **Resource imports → Ontology** → add **Section** and **Study**.
3. Create `src/createStudy.ts`:

   ```ts
   import { Client } from "@osdk/client";
   import { createEditBatch, Edits } from "@osdk/functions";
   import { Study, Section } from "@ontology/sdk";
   import { randomUUID } from "crypto";

   type OntologyEdit = Edits.Object<Study>;

   export default async function createStudy(
     client: Client,
     studyNumber: string,
     alreadyStarted: boolean,
     auditRequired: boolean,
     sectionCodes: string[],
   ): Promise<OntologyEdit[]> {
     // Study number
     const sn = studyNumber.trim().toUpperCase();
     if (!/^[A-Z0-9-]{3,20}$/.test(sn)) {
       throw new Error("Study number must be 3–20 letters, numbers or hyphens.");
     }
     const existing = await client(Study)
       .where({ studyNumber: { $eq: sn } })
       .fetchPage({ $pageSize: 1 });
     if (existing.data.length > 0) throw new Error(`Study ${sn} already exists.`);

     // Sections
     const codes = [...new Set(sectionCodes)];
     if (codes.length === 0) throw new Error("Select at least one section.");
     const valid = (await client(Section).fetchPage({ $pageSize: 100 })).data.map(s => s.sectionCode);
     const invalid = codes.filter(c => !valid.includes(c));
     if (invalid.length > 0) throw new Error(`Unknown section: ${invalid.join(", ")}`);

     // Save
     const batch = createEditBatch<OntologyEdit>(client);
     batch.create(Study, {
       studyId: randomUUID(),
       studyNumber: sn,
       alreadyStarted,
       auditRequired,
       selectedSections: codes,
       createdAt: new Date().toISOString(),
     });
     return batch.getEdits();
   }
   ```

4. Test it in **Live preview**, e.g. `"abc-123"`, `true`, `false`, `["COVER_PAGE","REPORT_SUMMARY"]`.
5. **Commit** → checks pass → **Tag version** `1.0.0`.

---

## Step 5: Action and form

1. **Ontology Manager → Study → Action types → + New action type**.
2. Choose **Function** → `createStudy` → version `1.0.0`.
3. Name: `Create Study`.
4. Open the **Form** tab and click each parameter:

   | Parameter | Display name | Settings |
   |---|---|---|
   | studyNumber | Study number | Required, placeholder "e.g. ABC-123" |
   | alreadyStarted | Already started | Display **Toggle**, default **false** |
   | auditRequired | Audit required | Display **Toggle**, default **false** |
   | sectionCodes | Sections | Required. **Allowed values → Multiple choice** (see below). **Display → Checkboxes** |

   Allowed values for `sectionCodes`:

   | Value (saved) | Label (shown) |
   |---|---|
   | `COVER_PAGE` | Cover Page |
   | `REPORT_SUMMARY` | Report Summary |
   | `PROFIT_ANALYSIS` | Profit Analysis |
   | `ANALYSIS_CONCLUSION` | Analysis Conclusion |

5. **Order:** drag the parameters into the order above.
6. Optional **form sections**:
   - "Study details": studyNumber, alreadyStarted, auditRequired
   - "Report sections": sectionCodes
7. **Security / Submission criteria:** *Current user is member of* → `Study Editors`.
8. **Save**.
9. Test with **Test run** (or **Preview form**). You should see:

   ```
   Study number *     [            ]
   Already started    [ ◯ ]
   Audit required     [ ◯ ]
   Sections *
     ☐ Cover Page
     ☐ Report Summary
     ☐ Profit Analysis
     ☐ Analysis Conclusion
   ```

> **No Checkboxes option?** The parameter must be a **String list** (from `sectionCodes: string[]`) with a **static Multiple choice** list. If it shows a single String, republish the function and select the new version. If Checkboxes still isn't offered, use **Multi-select**.

---

## Step 6: Workshop button

1. **New → Workshop module** → `Study Documents`.
2. Add a **Button Group** → button "New Study" → **On click: Action** → **Create Study**.
3. Add an **Object Table** of **Study** (columns: studyNumber, alreadyStarted, auditRequired, selectedSections).
4. **Preview:** click **New Study**, fill the form, tick sections, **Submit**. The new study appears in the table.
5. **Save → Publish**.

---

## Checklist

- [ ] Form shows all 4 fields, with sections as checkboxes
- [ ] Submit with no section ticked → "Select at least one section."
- [ ] `abc-123` is saved as `ABC-123`
- [ ] The same number twice → "already exists"
- [ ] The table shows the new study with the ticked sections

---

## Next: Edit Study

Create **Edit Study** the same way with an `editStudy` function (parameters: `study`, `alreadyStarted`, `auditRequired`, `sectionCodes`):

| Parameter | Settings |
|---|---|
| study | Object, **hidden** (prefilled from Workshop with `selectedStudy`) |
| alreadyStarted | Toggle, **default = `study.alreadyStarted`** |
| auditRequired | Toggle, **default = `study.auditRequired`** |
| sectionCodes | Checkboxes, **default = `study.selectedSections`** |

The defaults make the form open **pre-filled** with the current values. In Workshop, add an "Edit" button that runs **Edit Study** with `study` = `selectedStudy`.


sectionId,studyNumber,title,parentSectionId,level,sortOrder,generate,includeText,includeTables,tablesApplicable
S001-COVER,S001,Cover page,,1,0,true,true,false,false
S001-1.0,S001,1.0 Report Summary,,1,1,true,true,true,true
S001-1.1,S001,1.1 Background,S001-1.0,2,1,true,true,false,false
S001-2.0,S001,2.0 Fact-finding,,1,2,true,true,false,false
S001-3.0,S001,3.0 Cost Analysis,,1,3,true,true,true,true
S001-4.0,S001,4.0 Profit Analysis,,1,4,true,true,true,true
S001-5.0,S001,5.0 Price Analysis,,1,5,true,true,true,true
S001-6.0,S001,6.0 Analysis Conclusion,,1,6,true,true,false,false
