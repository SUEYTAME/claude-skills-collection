---
name: "capacitate-course-kb"
description: "Build a knowledge base from a capacitateparaelempleo.org course (PDFs + video transcripts) and answer its lesson exams from it. Use when Jarvis asks for help with a Capacítate course or its exams."
---

# Capacítate course KB + exam answering

Goal: answer exam questions using the course's own wording, not general knowledge. Never watch videos: use the official VTT subtitles. Keep it under ~15 tool calls.

## 1. Extract (Claude in Chrome, on the course page /cursos/view/<id>, user logged in)
The cloud sandbox cannot reach the CDN (403), so everything runs in the page.

1. Collect resources from Angular state (javascript_tool):
   - Walk `el.__ngContext__` arrays of all DOM elements (WeakSet for seen, depth <= 14, skip Nodes/window).
   - Levels = objects with `lections` array (+ `order`); each lesson has `resources`.
   - Resource types: 1 video, 2 actividad (PDF), 3 infografía (PDF), 6 video + JobAid.
   - PDF url: `r.storage.urlFile`; JobAid: `r.storageJobAid.urlFile`; subtitles: `r.video.subtitles.find(s=>s.languageId==1).urlSubtitles` (Spanish).
2. Transcripts: fetch each VTT, drop `WEBVTT`, cue numbers and `-->` lines, dedupe consecutive lines, join.
3. PDFs: inject pdf.js from cdnjs (`https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js`, set `GlobalWorkerOptions.workerSrc` to the matching worker), then `getTextContent()` per page. Run fetches with `Promise.all`; long awaits in one call can fail, so split into steps if needed.
4. Build one markdown string ordered Nivel > Lección > recurso.
5. Read it out: javascript_tool output truncates around 1k chars, so instead replace `document.body` with an `<article style=white-space:pre-wrap>` holding the markdown and call `get_page_text` (50k chars per read; slice for the rest). Reload the course page afterwards.
6. Do NOT pull exam answer keys from the API; build only from lesson content.

## 2. Write the KB
Save `/mnt/user-data/outputs/KB_<curso>.md`: condensed but keep the course's literal wording for definitions, lists, steps, formulas, numbers and thresholds. End with a "Puntos confusos" section listing near-synonyms and numeric traps. If a PDF is image-only (little text), screenshot it and transcribe.

## 3. Answer exams
- For each question, locate the KB section and pick the option matching the course wording; if course and general knowledge disagree, the course wins.
- Show calculations with the course formula.
- Output: question number -> answer, plus a one-line source (Nivel/Lección/recurso). Mark anything not covered as "inferida".
- Concise, no em dashes.