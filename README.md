# college-notes

Lecture notes for subjects I teach that are **outside the scope of the main AI/ML/Data Science notes site**.
Rules come from `NOTES_PROJECT_PLAN.md`, Section 18.

| Subject ID | Subject | Class |
|---|---|---|
| `security-in-computing` | Security in Computing (theory + practical) | Sem 6 BSc IT |
| `embedded-systems` | Embedded System (theory + practical) | Sem 4 BSc IT and BSc DS |

## Rules

- **Just in time.** Write notes lecture by lecture, a few days before class. Never pre-write a whole syllabus.
- **One topic per note.** The file name is the note ID: `content/<subject-id>/<note-id>.md`.
- **Same syntax as the main site** (plan Section 8.5): KaTeX maths, Mermaid diagrams, Obsidian callouts, fenced code.
- **Same style guide** (plan Section 11): simple English, Q&A explanations, paraphrase, never copy textbooks, mark unverified chapter references `verify`.
- **Separate from the main site.** No `REGISTRY.md` rows, no career paths, no wikilinks to main-site notes. Link to a main-site note by URL only.
- **Delivery:** export PDFs from Obsidian. The Markdown is the only source; never keep a separate Word or PDF version.

## Folder structure

```
content/
  security-in-computing/
    _index.md               ← unit-wise topic list (IDs only, lecture order)
    <note-id>.md
  embedded-systems/
    _index.md
    <note-id>.md
  assets/img/<subject-id>/  ← screenshots (Packet Tracer, Proteus) and images that can't be Mermaid
templates/
  theory-note.md
  practical-note.md
```

## Writing a note

1. Open `_index.md` for the subject and pick the next topic. Its ID is already assigned.
2. Create `content/<subject-id>/<that-id>.md` from the matching template.
3. Fill the front matter: `unit`, `type`, `sources`. Keep `status: draft` until you have checked it.
4. After review, change `status` to `reviewed`. After it has gone to students, change it to `published`.

In Obsidian, the link in `_index.md` turns from unresolved (faded) to resolved as soon as the note exists, so the index doubles as a progress list.

**Note IDs** are lowercase kebab-case, unique across this repo, and never change once a PDF has gone out. Practicals start with `prac-`.

**Template sections**

| Theory (required unless marked) | Practical (required unless marked) |
|---|---|
| In one line | Aim |
| Explained (Q&A) | Concepts used (optional) |
| Diagram (optional) | Setup (optional) |
| Key points | Code |
| Exam questions (short + long) | Output |
| Sources | Explanation |
| | Try it (optional) |
| | Viva questions |
| | Sources |

For practicals, **Code** is Cisco IOS / ASA commands (Security in Computing) or 8051 Embedded C in Keil (Embedded Systems).
**Output** is the real `show` command output, or a screenshot of the Proteus or Packet Tracer result saved in `content/assets/img/<subject-id>/`.

## Obsidian setup (one time)

1. Open this repository folder as a vault.
2. Settings → Core plugins → turn on **Templates**. Set the template folder to `templates`.
3. New note → Ctrl+P → "Templates: Insert template".

## Exporting PDFs

Ctrl+P → "Export to PDF". Callouts, maths and Mermaid render. For printed copies to be filed, set a wider margin in the export dialog for binding.

## Starter block for a subject chat

Paste one of these into a new chat in the notes project, and change the last line.

```text
COLLEGE-NOTES CHAT
Subject ID: security-in-computing
Subject: Security in Computing, Sem 6 IT
Follow NOTES_PROJECT_PLAN.md Section 18 (lighter template). This subject is
NOT part of the main notes site: no registry rows, no career paths,
no wikilinks to main-site notes. Use the note IDs in
content/security-in-computing/_index.md.
Today's task: notes for Unit 1, lectures 1–3
```

```text
COLLEGE-NOTES CHAT
Subject ID: embedded-systems
Subject: Embedded System, Sem 4 IT/DS
Follow NOTES_PROJECT_PLAN.md Section 18 (lighter template). This subject is
NOT part of the main notes site: no registry rows, no career paths,
no wikilinks to main-site notes. Use the note IDs in
content/embedded-systems/_index.md.
Today's task: notes for Unit 1, lectures 1–3
```

## Licence

Notes are licensed CC BY-NC-SA 4.0, the same as the main site. See `LICENSE`.
