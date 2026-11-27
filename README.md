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
- **Same calibre, lighter weight** (plan Section 18). Every note opens with a real problem ("Why should you care?") and a real-life example. Every subject starts with an intro note. Class notes skip only the heavy practice parts: self-check quizzes, flashcards and Think deeper.
- **Same syntax as Sutra** (plan Section 8.5): KaTeX maths, Mermaid diagrams, Obsidian callouts, fenced code.
- **Same style guide** (plan Section 11): simple English, Q&A explanations, paraphrase, never copy textbooks, mark unverified chapter references `verify`.
- **Answers under about 120 words.** Keep each `###` answer short; prefer bullets and tables; split a long answer into two questions. Present mode turns each `###` into a slide, and short chunks read better on phones.
- **Exam questions live in Explained** (plan decision 48). There is no separate "Exam questions" section. Exam papers are generated from the `###` questions in Explained, so they should cover every question the exam can ask. End a heading with ` ⭐` for an important question (dictated in class) and with ` [2M]` for a 2-mark question (5 marks is the default). Q numbers never change once a note is `reviewed`.
- **Answer skeletons** (plan Section 11). Students copy "In one line", "Key points", the diagram, and the `==highlighted==` skeletons. In each exam-likely answer, highlight the definition and the key phrase that opens each bullet (usually 3–8 words; at most a definition plus 6 points). Never highlight a whole bullet or paragraph, and never inside "In one line", "Key points", "Diagram", "Why should you care?", analogies or `[!beyond]`.
- **Beyond the syllabus.** Anything not examined goes in a `> [!beyond] Beyond the syllabus` callout, never mixed into the main explanation. "Syllabus" means the course codes in the note's `syllabus` field, so every note sets it (codes below).
- **Alt text everywhere.** Every Mermaid diagram has `accTitle:` and `accDescr:` lines; every image is `![alt text](path)` with real alt text.
- **No icons or emojis in notes** (plan Sections 11 and 12), except the ` ⭐` that marks an important question. The engine adds icons and colours per section automatically.
- **Separate from the main site.** No `REGISTRY.md` rows, no career paths, no wikilinks to main-site notes. Link to a main-site note by URL only.
- **Feeds teacher-desk.** `teacher-desk` builds question papers from the `###` questions of **reviewed** notes here, using the notes' answers as the key (`TEACHER_DESK_PLAN.md`, Section 4). Its usage log stays private.
- **Delivery:** export PDFs from Obsidian. The Markdown is the only source; never keep a separate Word or PDF version.

**Syllabus codes** (for the `syllabus` field)

| Subject | Theory | Practical |
|---|---|---|
| `security-in-computing` | `UITVI-001` | `UITVI-002` |
| `embedded-systems` | `UITIV-005`, `UDSIV-005` | `UITIV-006`, `UDSIV-006` |

## Folder structure

```
content/
  security-in-computing/
    _index.md               ← intro first, then unit-wise topic list (IDs only, lecture order)
    <note-id>.md
  embedded-systems/
    _index.md
    <note-id>.md
  assets/img/<subject-id>/  ← screenshots (Packet Tracer, Proteus) and images that can't be Mermaid
templates/
  intro-note.md
  theory-note.md
  practical-note.md
```

## Writing a note

1. Open `_index.md` for the subject and pick the next topic. Its ID is already assigned. The intro note (`<subject-id>-intro`) comes first and opens the first lecture.
2. Create `content/<subject-id>/<that-id>.md` from the matching template.
3. Fill the front matter: `unit`, `type`, `syllabus`, `sources`. Keep `status: draft` until you have checked it.
4. After review, change `status` to `reviewed`. After it has gone to students, change it to `published`.

In Obsidian, the link in `_index.md` turns from unresolved (faded) to resolved as soon as the note exists, so the index doubles as a progress list.

**Note IDs** are lowercase kebab-case, unique across this repo, and never change once a PDF has gone out. Practicals are named `<topic>-practical` (e.g. `ospf-md5-authentication-practical`), carry `practical: <number>`, and are listed under `## Practicals` in the subject index (plan v2.3, Sections 8.2 and 18). Intro notes end with `-intro`.

**Template sections** (in order)

| Intro (`type: intro`, all required) | Theory (required unless marked) | Practical (required unless marked) |
|---|---|---|
| Why this subject matters | Why should you care? | Aim |
| A short story | In one line | Concepts used (optional) |
| Where it is used today | Explained (Q&A) | Setup (optional) |
| What you will be able to do | Analogy and real-life example | Code |
| Careers that use it | Diagram (optional) | Output |
| Map of the subject (Mermaid) | Key points | Explanation |
| Sources | Common mistakes (optional) | Try it (optional) |
| | Sources | Viva questions |
| | | Sources |

For practicals, **Setup** names the tool (Cisco Packet Tracer, Keil µVision, Proteus). **Code** is Cisco IOS / ASA commands (Security in Computing) or 8051 Embedded C in Keil (Embedded Systems).
**Output** is the real `show` command output, or a screenshot of the Proteus or Packet Tracer result saved in `content/assets/img/<subject-id>/`, with alt text.

## Workflow (plan Section 14.2)

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
