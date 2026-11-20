# class-notes

Lecture notes for subjects I teach that are **outside the scope of Sutra**, the main AI/ML/Data Science notes site (`ai-ds-notes` repo).
Rules come from `NOTES_PROJECT_PLAN.md`, Section 18.

| Subject ID | Subject | Class |
|---|---|---|
| `security-in-computing` | Security in Computing (theory + practical) | Sem 6 BSc IT |
| `embedded-systems` | Embedded System (theory + practical) | Sem 4 BSc IT and BSc DS |

## Rules

- **Just in time.** Write notes lecture by lecture, a few days before class. Never pre-write a whole syllabus.
- **One topic per note.** The file name is the note ID: `content/<subject-id>/<note-id>.md`.
- **Same syntax as Sutra** (plan Section 8.5): KaTeX maths, Mermaid diagrams, Obsidian callouts, fenced code.
- **Same style guide** (plan Section 11): simple English, Q&A explanations, paraphrase, never copy textbooks, mark unverified chapter references `verify`.
- **No icons or emojis in notes** (plan Sections 11 and 12). When this repo is published with the main engine, icons and colours are added per section automatically.
- **Write each `##` section to fit on one slide.** The engine's present mode (plan Section 12) turns every `##` section into a slide, so these notes become lecture slides without a separate PPT. Split a long section into Q&A items rather than writing one long block.
- **Separate from the main site.** No `REGISTRY.md` rows, no career paths, no wikilinks to main-site notes. Link to a main-site note by URL only.
- **Separate from the exam question bank** (`qp-bank`, see `QP_GENERATOR_PLAN.md`). Students see the "Exam questions" in these notes, so a QP-BANK chat for these subjects must write new questions, never copy them from here.
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

## Workflow (plan Section 14.1)

- **Chats push directly to GitHub.** Each CLASS-NOTES chat pulls before starting and pushes to `main` when a piece of work is done. No zips, no uploads.
- **Each chat touches only `content/<its subject>/`** (plus that subject's images in `content/assets/img/<subject-id>/`).
- **Safe pushing:** small commits with clear messages; pull and rebase before every push; never force-push.

## Obsidian setup (one time)

1. Clone this repository and open the folder as a vault.
2. Settings → Community plugins → install **Obsidian Git**. It pulls new notes automatically and commits your edits back.
3. Settings → Core plugins → turn on **Templates**. Set the template folder to `templates`.
4. New note → Ctrl+P → "Templates: Insert template".

## Exporting PDFs

Ctrl+P → "Export to PDF". Callouts, maths and Mermaid render. For printed copies to be filed, set a wider margin in the export dialog for binding.

## Starter block for a subject chat

Name the chat `CLASS-NOTES · <subject-id>` (plan Section 7.5), paste the matching block, and change the last line.

```text
CLASS-NOTES CHAT
Subject ID: security-in-computing
Subject: Security in Computing, Sem 6 IT
Follow NOTES_PROJECT_PLAN.md Section 18 (lighter template).
Repo: Prathameshv07/class-notes. Pull before work; push to main when done.
This subject is NOT part of the main notes site: no registry rows, no career paths,
no wikilinks to main-site notes. Use the note IDs in
content/security-in-computing/_index.md.
Today's task: notes for Unit 1, lectures 1–3
```

```text
CLASS-NOTES CHAT
Subject ID: embedded-systems
Subject: Embedded System, Sem 4 IT/DS
Follow NOTES_PROJECT_PLAN.md Section 18 (lighter template).
Repo: Prathameshv07/class-notes. Pull before work; push to main when done.
This subject is NOT part of the main notes site: no registry rows, no career paths,
no wikilinks to main-site notes. Use the note IDs in
content/embedded-systems/_index.md.
Today's task: notes for Unit 1, lectures 1–3
```

## Licence

Notes are licensed CC BY-NC-SA 4.0, the same as Sutra. See `LICENSE`.
