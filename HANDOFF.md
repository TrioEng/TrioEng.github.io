# Project Checklist — Handoff Notes

This file brings a new Claude session (or person) up to speed on the Project Checklist tool. It's meant to be read **along with `index.html`** in this repo, the current version of the tool.

**Keep this file current.** Update it in the same commit as any change to `index.html` that affects what's described here. It's served publicly by GitHub Pages along with the page, so never put project data or anything private in it.

---

## 1. What this is and why it exists

Engineering project managers asked for a project task checklist many times. Every past option went unused: Excel checklists, tables, SharePoint/Microsoft Lists, separate apps. The reasons:
- Too much content; trimming a long list was tedious.
- It looked bad.
- No easy place for notes or deadlines.
- **Opening a separate program at all is a barrier.** The tool has to live with the project, not in another system.
- People here strongly dislike Microsoft's SharePoint/Lists interface. Don't propose it.

**The solution:** a single, self-contained HTML page with three tabs. Project data is saved to a small JSON file in each project's folder.

1. **Choose tasks:** the full task library in collapsible categories (all collapsed to start). Click to check the tasks this project needs. Custom tasks can be added per category.
2. **Track progress:** only the chosen tasks. Each has a status (Not started / In progress / Submitted / Done), Who, Due date, and Notes. Filters: All / Open / Due soon / Overdue / Done, plus a filter by person. Group by phase or by due date.
3. **Journal:** shared project notes for the whole team. Author, label, date, pin to top, edit, delete, search, filter by label.

The original Excel file (`Project Checklist.xlsx`) is where the task library came from. It's no longer used.

---

## 2. Current status (as of 2026-10-19)

- **The tool lives in the company GitHub account:** repo `TrioEng/TrioEng.github.io`, file `index.html`, served by GitHub Pages at **https://trioeng.github.io/**. There are no more versioned file names (`Project Checklist v5.html`, etc.); git history is the version record.
- **The earlier repo, `MordorPeaceCorps/TrioProjects`** (a personal account, served at `mordorpeacecorps.github.io/TrioProjects/`), was the proof of concept and is retired. Any project folder linked from that address has a `Project Checklist.url` pointing there; delete it so the new page writes a fresh one.
- **`index.html` is the hosted version** (formerly "v7"). It works on GitHub Pages and still works when opened as a local file.
- Background on earlier rounds (files no longer used):
  - "v5" was the stable local-file version, reviewed and approved.
  - "v6" was v5 plus an attempt to reconnect on page load without a click. It **did not work** as a local file (`file://`).
- **Proof of concept passed:** v6 uploaded to GitHub Pages as `index.html`, then "Allow on every visit" plus installing the site as an app. After that, the page reconnected to its folder on its own; the yellow "Reconnect" bar was gone. This is why hosting is the plan.
- **Next step:** test the hosted page (section 7, steps 2–6).

---

## 3. How saving works (important background)

- The page uses the browser's **File System Access API**. It works only in **Chrome and Edge**; Firefox can use the Download / Load JSON buttons in the ••• menu.
- Data file: **`project-checklist.json`** in the project folder. The name is fixed; never rename it.
- **Linking** is once per project, per computer: **"Choose this project's folder."** The page looks for an existing `project-checklist.json` and merges with it, or creates one if there isn't one.
  - We deliberately do **not** use a "Save as" picker. Chrome/Edge empty an existing file as soon as it's picked in "Save as", which could wipe a coworker's data.
- The page remembers each project's link in IndexedDB and keeps a backup copy in localStorage.
  - Opened as a local file, it's keyed by the HTML file's path.
  - Hosted, it's keyed only by the `#project=` value in the link (`PAGE_KEY` in `index.html`), not by the page's path.
  - All GitHub Pages sites on the TrioEng account share one origin (`trioeng.github.io`, including any other TrioEng repo served at `trioeng.github.io/<repo>/`), so they share localStorage and IndexedDB. Keys all start with `pc:`; don't host another checklist-style page on the TrioEng account.
- **Browser permission behavior we learned:**
  - When opened as a local file, the browser requires one "Reconnect" click per browser session. "Allow on every visit" only removes the permission prompt, not the click. Trying to reconnect automatically on page load (v6) doesn't help.
  - On an https site (GitHub Pages), "Allow on every visit" works fully, especially once the site is installed as an app. No click is needed.
  - "Allow on every visit" sometimes took a few full browser restarts before it stuck.
- **Multiple users:** confirmed working with two browsers on the same file. Not yet tested on two separate computers through OneDrive sync.
  - The page re-reads the file every 20 seconds, when the window regains focus, and before every save. It **merges** item by item using last-modified times (`m` fields).
  - Deletions are recorded in a `deleted` map, so deleted items don't come back.
  - The header fields (name, number, client, PM) merge one at a time.
  - Redraws wait while someone is typing, and wait for any click in progress to land.

---

## 4. Hosted design

- **Site home page** (no `#project=`): enter a project number to open it, plus a "Recently opened on this computer" list.
- **Project page:** `…/#project=24-118`. The project number is filled in from the link.
  - **Why `#` and not `?`:** the part of a link after `#` never leaves the browser, so GitHub never receives project numbers (its server logs only see `/`). With `?project=` every page load sent the number to GitHub. Only the page itself is downloaded from GitHub; the page makes no other web requests.
  - Old `?project=` links still work: the page reads the number and rewrites the address to the `#` form without reloading. That first load still sends the number to GitHub, so replace old shortcuts when convenient.
  - Changing the `#` part of an open page (home page form, recent list, typing a new link) reloads the page, since it's set up for one project per load.
- **When a project folder is linked, the page writes `Project Checklist.url`** (a Windows internet shortcut) into that folder, pointing to the project's page. It never overwrites an existing shortcut. Team members double-click the shortcut.
- ••• menu → **Copy link to this project**.
- **The shortcut stores the full page address** (e.g. `https://trioeng.github.io/#project=24-118`), and the page never overwrites an existing shortcut. **This URL is now locked.** Don't rename the repo, rename `index.html`, move it into a folder, or add a custom domain: every existing shortcut would break and have to be deleted by hand so the page can write a new one. `TrioEng.github.io` is the organization's root Pages site, which is why the address has no repo name in it.
- **Opened as a local file, the page behaves like the old v5** (one copy of the file per project folder). Local copies don't get updates from the repo.
- **Project data never goes to GitHub.** GitHub only serves the page; everything is read from and written to the project folders on the user's computer.
- GitHub Pages sites are **public** unless the organization has GitHub Enterprise. The page code and task list are visible to anyone with the URL; project data is not. The user is aware of this.
- Benefit: **update once, everyone gets it.** Editing `index.html` in the repo updates every project.
- The flip side: **every commit to `main` reaches every PM within about a minute.** There's no separate test copy. Test changes first, on a branch merged when ready or on a `beta/index.html` page (`https://trioeng.github.io/beta/`). A beta page on the same site shares project links and saved data with the live page (storage is keyed by `#project=` only). That's fine for testing on real folders, but be aware of it. A beta page also writes a shortcut pointing to the beta URL if the folder doesn't already have one.
- Possible future feature: a **cross-project dashboard** ("what's due this week across all my projects"). The user would grant the site access to the parent Projects folder once, and the page would read every subfolder's `project-checklist.json`. Not requested yet.

---

## 5. Decisions already made (don't re-ask)

- **Status column stays before the task name** in Track progress. It works like a checkbox on the left. It can be swapped if the PMs push back.
- **Filters don't update live.** A row marked Done stays visible until the filter changes, so it doesn't vanish while someone is about to type a note.
- **No "% complete" number.** PMs would want it weighted by effort, which is a rabbit hole. The progress bar stays (passive). The "X of Y done" counts stay; the user may revisit.
- **Repeated task names** across categories (e.g., "Site Plan" in Concept, Prelim and Final) are intentional. The category headings, and the category label in the by-due-date view, tell them apart.
- **Saving needs the minimum steps the browser allows**; the user accepts that.
- **People names come from the project itself** (anything typed in Who, plus journal authors), not a master list. The user liked this.
  - We use a custom suggestion dropdown, not the browser's built-in `<datalist>`. The built-in one only shows names matching what's already in the box, so a filled box offered only its own name.
- The **setup dialog was simplified at the user's request**: "Current location:" with path, a "Copy to clipboard" link, one button "Choose this project's folder / Creates save file or links to existing", and a small "Open a save file stored somewhere else…" link. Users don't read long text; keep wording minimal.
- **No versioned file names.** The page is always `index.html`. Each round of changes is a commit with a clear message; earlier versions can be recovered from git history.
- **Renaming a local copy of the HTML file** is fine; it only needs a one-time re-link. Best to rename before first use. Don't rename `index.html` in the repo; that changes the site URL (see section 4).

---

## 6. Where things are in the HTML file

Everything is in one file: HTML, CSS and JavaScript.

- **Top `<script>` block = editable settings:**
  - `LIBRARY`: categories and tasks. Edit here to change the task list. Task identity is category + name, so **renaming a task disconnects its saved notes**; adding and removing tasks is safe. **Because the page is hosted, a rename breaks that task in every project at once.** Don't rename tasks without a plan (e.g., an alias map from old ID to new ID; not built yet).
  - `JOURNAL_TAGS`: journal labels.
  - `STATUSES`, `SAVE_FILE_NAME`, `SHORTCUT_NAME`, `PROJECT_FIELDS`.
- **Main script sections:** helpers → state/`normalize()`/`merge()`/`canon()`/`same()` → dates → storage (localStorage, IndexedDB, file read/write with merge-before-write, `checkRemote` polling, `softRender`, `afterPointer`) → connect (`connectFolder`, `saveShortcut`, `connectOpen`, `reconnect`) → export/import (full replace re-stamps times; "copy selections from another project" asks first) → Choose view → Track view (person filter, row "⋯" menu with Remove from project) → Journal (`jEdit` draft object) → views/tabs → name suggestions (`#suggest`) → menus/print (Print menu: task list or journal) → boot.
- **Data file shape (`project-checklist.json`, version 2):**
  ```json
  {
    "app": "project-checklist", "version": 2,
    "project": { "name": "", "number": "", "client": "", "pm": "", "m": 0, "mf": { "name": 0 } },
    "tasks":   { "<cat>.<slug>": { "sel": true, "status": "todo|prog|sub|done", "due": "YYYY-MM-DD", "who": "", "notes": "", "m": 0 } },
    "custom":  [ { "id": "<cat>.custom-<base36>", "cat": "<cat>", "name": "" } ],
    "journal": [ { "id": "j-…", "created": "ISO", "date": "YYYY-MM-DD", "author": "", "tag": "", "text": "", "pinned": false, "edited": null, "m": 0 } ],
    "deleted": { "<id>": 0 },
    "updated": "ISO"
  }
  ```
  Older v1-format files (no `m`, `mf`, `journal` or `deleted`) load fine.

---

## 7. Publishing and testing

1. Changes are committed to `index.html` on `main` of `TrioEng/TrioEng.github.io`; GitHub Pages updates in about a minute. (Done for the hosted version.) The work computer blocks GitHub CLI and connectors, so commits come from github.com in a browser or a Claude Code session working on the repo. Upload files with their full name (`index.html`, not `index`), or Pages won't serve them.
   - Open the site at **https://trioeng.github.io/**.
2. Open the site. On the home page, enter a test project number → **Open project**.
3. Choose **"Choose this project's folder"** and pick a test project folder. Choose **Allow on every visit** if asked.
4. Confirm:
   - `project-checklist.json` is created or found, and existing data appears.
   - **`Project Checklist.url` appears in the folder.** Not yet tested; folder access couldn't be tested in the preview.
   - Double-clicking the shortcut opens the right project (check the address still ends in `#project=…`; Windows shortcuts should keep the `#` part, but this hasn't been tested), connected, with no yellow bar. The shortcut opens a normal browser tab, not the installed app window; permission should still hold for the site.
5. Fully restart the browser and reopen via the shortcut. It should still connect with no click.
6. When available: test two people on two different computers on the same OneDrive project folder.

**Rollout notes for PMs:**
- Use Chrome or Edge.
- Choose "Allow on every visit"; it may take a restart or two to stick.
- The site can be installed via the browser menu → "Install page as app".
- New project: open the site home page (https://trioeng.github.io/) → enter the number → link the folder. The shortcut is created automatically.

---

## 8. Testing approach used so far

- **Merge logic:** extract `normalize`/`merge`/`same` from the page script and run assertion tests in Node. Tests cover: newest edit wins; items on only one side are kept; journal union minus deletions; header fields merge separately; old-format header; merge is symmetric; merge is idempotent. All pass on the current `index.html`.
- **Syntax:** pull the `<script>` contents out of the page and run `node --check` on them.
- **UI:** an in-app browser preview.
  - It can't access real files: there's no File System Access as a local file, so saving and linking were always tested by the user in real Chrome/Edge.
  - A hidden preview window doesn't fire focus/blur events, so those were simulated.
  - The hosted behavior (home page, `#project=`, recent list) was tested on a temporary local web server (`python -m http.server`).

---

## 9. Ideas not yet requested (only if the user asks)

- Preset project types (e.g., "Residential subdivision", "Commercial site", "Survey only") that pre-check common tasks. Needs PM input on what each type includes.
- Cross-project dashboard (see section 4).
- A web app manifest and icon, so the installed app has a proper name and icon.
- Per-phase hour/cost estimates (the original Excel had this). The user leans toward keeping things simple.
