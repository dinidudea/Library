## **Manual for Processing Sri Lankan Legal Documents into JSON Format**

This is a free and open repository of Sri Lankan legislation. Laws are processed with strict JSON schema that will allow for correct nesting of sub-sections, and correct display marginal notes. Each JSON must contain a metadata section. If adding a new law, it should also be recorded in the manifest.json file in order to be displayed in the dropdown-list.

This is maintained by Dinidu de Alwis. All content including the schema and the render logic is released to the public domain.

### **1. Objective**

This document provides a set of rules and a detailed schema for converting Sri Lankan legislation into a structured JSON format. Adherence to these instructions is mandatory to ensure data consistency and accuracy.

### **2. General Principles**

*   **Complete Conversion:** The entire content of the source legal document must be captured in the JSON output.
*   **Strict Schema Adherence:** The final output must be a single, valid JSON object that strictly follows the structure and field names defined in this guide. Do not add any fields not specified in the schema.
*   **Exact Text Replication:** All text, including punctuation, spacing, and capitalization, must be copied exactly as it appears in the source document.
*   **No External Commentary:** Do not add any personal notes, explanations, or comments within the JSON output.

### **3. JSON Schema Overview**

The JSON structure consists of two main parts: `metadata` for document information and `body` for the legal text.

```json
{
  "metadata": { ... },
  "body": [ ... ]
}
```

---

### **4. Detailed Field Instructions: `metadata` Object**

The `metadata` object contains key information about the document as a whole.

| Field | Instruction | Example(s) |
| :--- | :--- | :--- |
| **`title`** | Extract the official, primary title of the document. | `"APPROPRIATION ACT, No. 24 OF 2016"`<br/>`"The Constitution of the Democratic Socialist Republic of Sri Lanka"` |
| **`document_type`** | Select one specific value from the following list: `"Constitution"`, `"Act"`, `"Amendment"`, `"Bill"`, `"Ordinance"`. | `"Act"` |
| **`document_subtype`** | (Optional) Use only for specific classifications. | For an Appropriation Act, use `"Appropriation"`. |
| **`act_number`** | (Optional) Extract the Act number and year. | For "Act, No. 24 OF 2016", the value is `"24 of 2016"`. |
| **`ordinance_number`** | (Optional) Extract the Ordinance number and year. | `"1 of 1889"` |
| **`publication_date`** | (Optional) The date the document was published. **Format: YYYY-MM-DD**. | `"2017-09-19"` |
| **`enactment_date`** | (Optional) The date the law was certified or came into force. **Format: YYYY-MM-DD**. | `"2016-11-16"` |
| **`amended_up_to`** | (Optional) For revised editions, this is the cutoff date for included amendments. **Format: YYYY-MM-DD**. | `"2021-12-31"` |
| **`description`** | (Optional) Use for the long-form description of an Act, typically starting with "AN ACT TO...". | `"AN ACT TO PROVIDE FOR THE SERVICE OF THE FINANCIAL YEAR 2017;..."` |
| **`preamble_text`** | (Optional) Use for the full text of a formal preamble, such as the "SVASTI..." section in the Constitution. | `"The PEOPLE OF SRI LANKA having, by their Mandate freely expressed..."` |
| **`amends`** | (Optional) If the document is an `Amendment`, state the full title of the document it modifies. | `"THE CONSTITUTION OF THE DEMOCRATIC SOCIALIST REPUBLIC OF SRI LANKA"` |
| **`interpretations`** | An array of objects. Populate this **only if** the document has a dedicated "Interpretation" or "Definitions" section. For each defined term, create one object with the fields `term`, `definition`, and `source_section`. | `[ { "term": "public officer", "definition": "means a person who holds any paid office under the Republic...", "source_section": "170" } ]` |

---

### **5. Detailed Field Instructions: `body` Array**

The `body` is an array that holds the main structural divisions of the document (Chapters, Parts, or Schedules).

#### **5.1. Structural Divisions (Chapters, Parts)**

For each "Chapter" or "Part" in the document, create a corresponding object in the `body` array.

| Field | Instruction | Example |
| :--- | :--- | :--- |
| **`type`** | Use `"Chapter"` or `"Part"`. | `"Chapter"` |
| **`division_number`** | (Optional) The Roman or Arabic numeral of the division. | `"I"`, `"XXII"` |
| **`division_title`** | (Optional) The title of the division. | `"THE PEOPLE, THE STATE AND SOVEREIGNTY"` |
| **`sections`** | An array containing all the Section objects within that division. (See section 6 below). | `[ { "section_number": "1", ... }, { "section_number": "2", ... } ]` |

**Handling documents without formal chapters:**
*   **Amendment:** Use a single structural element with `type: "AmendmentBody"`.
*   **Simple Act/Bill:** Use a single structural element: `type: "Part"`, `division_number: "I"`, `division_title: "Main Body"`.

#### **5.2. Schedules**

Place all Schedules as distinct objects within the main `body` array. For detailed instructions, see Section 8 below.

---

### **6. Detailed Field Instructions: `Section` Object**

A Section object represents a single Article or Section of the law.

| Field | Instruction | Example(s) |
| :--- | :--- | :--- |
| **`section_number`** | The exact section number as a string. | `"4"`, `"14A"`, `"154G"` |
| **`marginal_note`** | The short, descriptive side-note or bolded heading for the section. | `"Right of access to information"` |
| **`note`** | (Optional) Capture any footnote attached to the section indicating its amendment history. | `"Inserted by the Nineteenth Amendment to the Constitution Sec. 2."` |
| **`content`** | An array of `ContentBlock` objects that form the text of the section. (See section 7 below). | `[ { "type": "paragraph", ... }, { "type": "subsection", ... } ]` |

---

### **7. Detailed Field Instructions: `ContentBlock` Object (for textual content)**

This object is used to capture all textual content within a section. It has a recursive structure to handle nested lists.

| Field | Instruction | Example |
| :--- | :--- | :--- |
| **`type`** | Use `"paragraph"` for a regular block of text. Use `"subsection"` for an enumerated or indented item. | `"subsection"` |
| **`identifier`** | (Optional) The enumeration marker for a `subsection`. For a `paragraph`, this should be `null`. | `"(a)"`, `"(1)"`, `"(i)"` |
| **`text`** | The full text of the paragraph or subsection. | `"Every citizen shall have the right of access to any information..."` |
| **`children`** | If a paragraph or subsection is followed by a nested list (e.g., (a) is followed by (i), (ii)), place those nested `ContentBlock` objects inside this array. | `[ { "type": "subsection", "identifier": "(i)", "text": "..." } ]` |

---

### **8. Detailed Field Instructions: `Schedule` Object**

Place each Schedule as a separate object within the top-level `body` array.

| Field | Instruction | Example |
| :--- | :--- | :--- |
| **`type`** | Always `"Schedule"`. | `"Schedule"` |
| **`schedule_number`** | The formal name of the schedule. | `"First Schedule"`, `"Ninth Schedule"` |
| **`schedule_title`** | (Optional) The descriptive title of the schedule. | `"Names of Administrative Districts"` |
| **`schedule_reference`** | (Optional) The article that refers to the schedule, if mentioned. | `"ARTICLE 5"` |
| **`content`** | An array of `SchedulePart` objects. If a schedule has no internal parts (e.g., "LIST I"), create a single `SchedulePart` with `part_title: null`. Each `SchedulePart` contains `items`, which is an array of `ContentBlock` objects. | `[ { "part_title": "LIST I", "items": [ ... ] } ]` |

#### **8.1. Handling Tables within Schedules**

For tabular data inside a schedule:
1.  Represent each row of the table as a separate `ContentBlock` of type `"paragraph"`.
2.  The first `ContentBlock` should represent the header row.
3.  Within the `text` field of each `ContentBlock`, separate the content of each cell/column with a pipe delimiter (`|`).

**Example of a table row:**
```json
{
  "type": "paragraph",
  "identifier": null,
  "text": "1. | Colombo",
  "children": []
}
```