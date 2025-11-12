# EHR-form-builder

This repository contains an **ER/class-model proposal** for a dynamic electronic health record (EHR) forms system. It was designed with **Java** in mind, but it’s fully adaptable to any language that embraces inheritance and OOP.

You’ll find a PlantUML diagram describing the core entities for building "Google-Forms-style" questionnaires per medical specialty, with immutable versions and patient history. The model focuses on letting you create form templates, publish frozen versions, collect typed answers per appointment, and keep every record consistent over time.

**PlantUML**

* Diagram file: `docs/er-ehr-forms.puml`
* View it online by pasting the file here: [https://www.plantuml.com/plantuml/](https://www.plantuml.com/plantuml/)

---

## What the model is about 

Think of **FormTemplate** as the idea of a form (e.g., “Cardiology”). Each time you **publish**, you get a **FormVersion**. A version has **FormSection** groups, which contain **FormField**s (with a FieldType, required, displayOrder, and a constraintsJson for flexible rules like min/max, regex, etc.). Choice fields point to **OptionItem**; visibility and interactivity can be controlled with **ConditionalRule** (simple expressions like `field('smokes') == 'yes'` with actions such as SHOW/HIDE/ENABLE/DISABLE).

When a form is used during an appointment, we create a **FormSubmission** linked to the **FormVersion** that was active at that moment. The actual answers are **Register**s. Each Register references the specific FormField of that same version, so your data always points back to the exact field definition it answered.

---

## How a form version gets created

In this section we highlight a few practical rules when shaping how versions are created:

You start by creating **FormTemplate**. From there, you create a **FormVersion** in draft mode, add **Sections**, define **Fields** (type, label, required, order), include **Options** when needed, and attach **ConditionalRules** for show/hide/enable/disable behavior. When everything looks right, you **publish** the version.

Once published:

* A **FormVersion is immutable**. To change a field, you publish a new version.
* **(formVersion_id, code)** should be unique for fields, but each published version creates **new field IDs**, so Registers can safely point to the exact field they answered.

---

## How answers are collected and validated 

When an appointment begins, the system chooses which **FormVersion** to use. The frontend fetches that version and renders **sections → fields** in order.

On submit (e.g., `POST /form-submissions`), the backend:

1. Ensures all `fieldId`s exist within the same FormVersion.
2. Verifies the type matches (`Register.type == FormField.type`).
3. Applies required rules only if the field is visible/enabled after considering conditionals.
4. Verirfy constraints (length, range, regex, max files/MB, max items, etc.).
5. Confirms options are valid for choice fields.
6. Persists the **FormSubmission** with its **Register**.

---

## Entities

* **User**: the authenticated person using the system (ADMIN, CLINICIAN, RECEPTION).
* **Patient**: patient identifiers.
* **Specialty**: a medical specialty.
* **Appointment**: a clinical encounter with timestamps and status.
* **FormTemplate**: the conceptual form (name, description, optional specialty).
* **FormVersion**: a frozen snapshot of the template at publish time; holds **Sections** and **Fields**.
* **FormSection**: a visual/logical group of fields, displayed in order.
* **FormField**: the actual field definition (type, label, required, order, constraints).

  * **OptionItem**: choices for RADIO/SELECT/MULTISELECT.
  * **ConditionalRule**: expressions that control visibility or enabled state.
* **FormSubmission**: a user-filled instance tied to a specific `FormVersion` and `Appointment`.
* **Register** (abstract): one answer per field, with concrete subclasses by type.

  * **Attachment**: file metadata referenced by `FileRegister`.

---

## Files and layout 

```
.
├─ docs/
│  └─ er-ehr-forms.puml       # PlantUML diagram
│  └─ er-ehr-forms.png       # PNG print diagram
└─ README.md
```

