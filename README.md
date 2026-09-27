# Specification Phase Exercise

A little exercise to get started with the specification phase of the software development lifecycle. In this exercise, your team specifies a set of improvements and new features for [The Slide Machine](https://theslidemachine.com) — see the [instructions](instructions.md) for detail, and the [background](background.md) for an introduction to the software product you are tasked with extending.

## Team members

- [Tracy Wang](https://github.com/TracyWang0904)
- [Emma Ao](https://github.com/emma6594)
- [Jingjing Wang](https://github.com/JingjingWang129)
- [Uuriintuya Ganzorig](https://github.com/Uuriii1003)

## Review of the Current Application

Findings from the team's independent use of [theslidemachine.com](https://theslidemachine.com).

### Strengths

- Speech recognition is clear and converts spoken language into consistent, natural-sounding text.
- A dedicated discovery/browse interface lets users search for colleagues' and other relevant finished slide decks to use as references.
- Supports uploading a previously created presentation and adding new, explanatory slides in the same format based on further voice input — an instructor can reuse a deck someone else made and personalize it while teaching their first class, saving prep time.
- The AI read-aloud (narration) feature gives coherent explanations rather than simply reading the slide text verbatim, which is friendly to self-study — a student who finds a useful deck online can upload it and listen to the AI's "lecture," which benefits auditory learners.

### Weaknesses

- Cannot generate an introduction/opening slide without voice input.
- When spoken input is short, transcription is unreliable and drops parts of the speech.
- Some slide themes make text hard to read because text and visual elements compete for the same space.
- Text editing is limited: existing text can be changed, but no new text box can be added, which limits how much information can be inserted.
- Text formatting is very limited: the editor lets users change text content but provides no controls for basic formatting such as font style or size.
- Changing the layout/style of one slide changes the whole presentation's style, rather than only that slide.
- If an instructor later adds more to a topic mentioned earlier, the new slide cannot be placed near the other slides on that topic — slides are ordered strictly by when they were spoken.
- Quiz answers tend to be the most specific/longest option, because quiz questions are generated directly from slide text and pull answers verbatim from it.
- The distinction between "add slide" and "add whiteboard" is unclear from the button labels alone; a first-time user has to experiment with both to learn when to use each.
- The delete-slide action is hard to discover during editing — the functionality exists but is not visible without extra searching.

### Gaps

- No way to provide a URL/website as seed material.
- Refinement changes apply immediately, with no save/confirm step before they take effect.
- No undo button.
- Limited ability to add custom visual elements such as shapes, icons, graphics, or photos.
- No support for understanding/transcribing other languages.
- No apparent mechanism for an instructor to mark or revise a generated slide after verbally correcting or updating what they said (needs re-testing to confirm).
- No live draft control: no mechanism to temporarily hold, discard, or revise AI-generated slide content created from a spoken idea that wasn't yet finished.
- No mechanism to mark certain spoken content as non-lecture material so it doesn't influence generated slides (needs re-testing to confirm).
- No mechanism for an instructor to signal they are returning to or continuing an earlier topic during live generation.

## Prior Art & Originality

Our proposal is **Real-Time Editing for Instructors and Students** — one coherent theme with two halves that share the same moment in time: the live lecture, while it is still being captured. The Slide Machine currently treats a lecture as a linear stream generated for an audience of one (the instructor's own screen), decided phrase-by-phrase in strict speaking order, with no way to correct that structure while capture is running and no way for a student to see any of it until the deck is saved and shared afterward. Our proposal gives both sides a live foothold in that process:

- **Instructor side — structural control.** The instructor gets control over where generated content belongs in the lecture's conceptual structure as it happens: branching a spoken tangent off the main line, redirecting the AI's current generation target, merging/splitting/moving/reattaching slides, retroactively reclassifying already-generated content, explicit topic-transition cues, and undo/recovery for all of the above. The guiding principle is that **speaking order does not always equal presentation structure**.
- **Student side — live access and private annotation.** A student can open a link the instructor shares while the session is still recording, watch the same deck update live (read-only — students cannot change slide content), and attach a private comment to any slide that only they can see, which stays attached to that slide after the lecture ends and the deck is saved.

This is not a general-purpose slide editor (fonts, colors, themes, and image placement are explicitly out of scope) — every capability here is about *structure and live access*, not visual design. Several directions trace directly to gaps and weaknesses our own team found while using the live app (see [Review of the Current Application](#review-of-the-current-application) above) — most directly, "no undo button," "no live draft control," "no mechanism to signal returning to an earlier topic," and "a new slide can't be placed near earlier slides on the same topic."

**What we checked:** the SDD's Future Work (§18) and Open Questions (§19), the Delivery Roadmap (including its risks & cut-line section and Phase 3 outstanding list), and the repository's open GitHub issues — none address live topic/deck structure or live audience access, and none propose branching, redirection, retroactive reclassification, structural undo, topic-transition cues, or a live/private student view during capture.

- **SDD Future Work (§18)** — the deferred items (local AI models, real-time translation, extracting the STT→generation pipeline into its own service, an MCP agent server, **multi-user collaborative editing of a single deck**, seat-based billing, richer analytics/recommendation, a faculty setup guide) are all unrelated to our proposal — including the collaborative-editing item, which is about several people jointly *editing the same shared content*, the opposite of a read-only live view with **private**, per-student annotations nobody else can see or change.
- **SDD Open Questions (§19)** — none of the still-open questions (plan pricing, pilot exemptions, student roster source, latency targets, image licensing enforcement, image disambiguation depth, Slides export fidelity, coverage-gate scope, preflight concept-set limits, MCP auth/scope, AI-imagery accuracy) touch on deck structure, live correction, or live audience access.
- **Delivery Roadmap** — the risks & cut-line section and the Phase 3 outstanding list (`CAP-5` live captions, `EDIT-8` duplicate slide, `PLAY-4`/`PLAY-5`, `PREP-1..4` preflight, `IMG-4` AI imagery, hardening) include nothing about structural control, branching, live correction, or live student access.
- **Open GitHub issues** — none propose structural control or live audience access during capture. The closest related item, issue #27 ("Required Transcript Viewing"), proposes forcing linear, unskippable playback — the opposite concern from ours — and remains unresolved as of this check.

**Adjacent existing/shipped features we checked each proposed capability against, to avoid re-proposing them:**

- **Topic branching** — live generation only ever decides, per spoken phrase, "update the current slide" or "start a new slide" (`GEN-8`), a strictly linear choice with no concept of a tangent to set aside and rejoin. Nothing like a branch exists anywhere in the app.
- **Active generation target / redirect** — live generation shows only a generic "something is generating" cue (`GEN-5`'s activity indicator); there is no display of *which topic* is the AI's current target, and no way to redirect new content to a different one. The post-lecture Refine job does narrate which slide it is working on, but that is a background batch job's progress readout, not a live, redirectable target.
- **Structural correction (merge / split / move / reattach)** — merging and splitting slides already exist, but only inside `GEN-4`'s post-lecture "Reformat with AI" pass, where the **AI** decides whether and how to merge or split as part of a holistic, opt-in, post-lecture rewrite — the instructor cannot trigger either action live or control the result. Moving a slide already exists (`EDIT-1` includes manual slide reordering), but only **after** the lecture ends, by hand, with no live or automatic grouping. "Reattach" (moving a branch to a different parent) has no counterpart, since branching itself doesn't exist.
- **Retroactive adjustment** — no mechanism anywhere lets an instructor reclassify already-generated content (e.g., turn an existing slide into a branch after realizing the discussion became a tangent); this depends on the branching concept above, which is new.
- **Topic transitions** — `CAP-4` already ships a fixed voice-command vocabulary (start/stop/pause/resume/rewind/fast-forward), and `GEN-8`'s opt-in manual new-slide mode already supports an explicit "next slide" cue to force a slide boundary. None of these carry topic-level meaning, though — there is no "continue previous topic" or "return to main topic" cue; the existing commands only mark *that* a boundary should occur, never *why*.
- **Undo / quick recovery** — an undo/redo control already exists, but it is scoped entirely to whiteboard pen strokes (`EDIT-5`, "Undo / redo, per slide"); there is no undo for slide content edits, merges, splits, moves, or any other structural change, which matches our own team's review finding of "no undo button" for the editing experience generally.
- **Live/shared student access** — `SHARE-1` only covers a **saved** deck's permalink, shared after the fact; there is nothing that lets a student open a link and watch a deck update **while it is still being generated**.
- **Private per-slide comments** — no comment/annotation surface exists anywhere in the app, for any user, private or otherwise.

**What is new:** live, instructor-driven control over the deck's conceptual structure while capture is still running — topic branching with a one-action return to the main thread; a visible, redirectable generation target; manual merge/split/move/reattach of slides during (not only after) the lecture; retroactive reclassification of already-generated slides; explicit topic-transition cues beyond a bare slide boundary; undo/recovery that covers structural changes, not just whiteboard strokes; a branch slide that stays attached to its main-line slide even across export into different folders — **and**, on the student side, a live, read-only, shareable view of the deck while it is still generating, with private per-slide comments that persist after the deck is saved. None of this appears in the current application, the SDD, the Roadmap, or any open issue or pull request we reviewed.

## Stakeholders

### Instructors

- **Baohua** — Differential Geometry TA
- **Yuchen** — Calculus III TA

**Goals / needs:**

1. Keep unscripted, off-topic Q&A during a review session out of the official generated slides, so slides stay focused on useful instructional content rather than every spoken exchange.
2. Support a workflow built around professor-assigned quizzes and pre-prepared examples, rather than fully on-the-fly generation.
3. Reliable speech recognition for math/technical vocabulary, including tolerance for non-standard pronunciation and disambiguation of similar-sounding terms across disciplines.
4. Preserve useful explanations and examples as organized notes that students can review after the session.
5. Accurately represent mathematical content — equations, symbols, and graphics — that speech-to-text alone cannot reliably capture.

**Problems / frustrations:**

1. Real-time slide generation itself creates pressure, since the TA has to teach while also worrying about messy or tangential Q&A being recorded as "official" material.
2. Math TAs generally prefer the blackboard; the format doesn't obviously fit how they already like to teach, especially for handwritten equations, symbols, diagrams, and graphs.
3. When the AI misjudges a topic shift and files content under the wrong slide, the only fix today is reorganizing the deck by hand after the session ends.
4. Review sessions are highly non-linear — unexpected questions, tangents, and fragmented conversations — which should not automatically become slides.
5. Speech recognition may misinterpret specialized terminology, particularly similar-sounding technical terms or non-standard pronunciation.

### Students

- **Heidi L.**
- **Jocelyn Y.**
- **Judy Y.**
- **Urangoo C.**
- **Riko E.**

**Goals / needs:**

1. A visual marker distinguishing branch/tangent slides from main-line content, so it's clear what's supplementary versus core lecture material when reviewing.
2. Any live correction the instructor makes (e.g., reclassifying a slide's topic, or merging it back into the main line) should show up clearly in the final shared deck, rather than leaving it ambiguous what changed.
3. Slides that accurately reflect what the instructor explained during the lecture, including important examples and explanations.
4. An easy way to review topics that were difficult to understand, without having to go through the entire presentation again.
5. Lecture materials organized so it's easy to follow the order and connection between different topics.

**Problems / frustrations:**

1. May miss important information when the instructor moves quickly through a topic or changes topics during the lecture.
2. May have difficulty remembering where a specific topic or explanation appeared when reviewing the lecture later.
3. May have difficulty finding important information when a lecture contains a large number of slides or a lot of content.
4. Worried that content seen live during the lecture may not match the final shared deck, since the instructor can reorganize, merge, or reclassify slides afterward, leaving their own notes/screenshots out of sync with the official version.

## Product Vision Statement

See instructions. Delete this line and place your Product Vision Statement here — one sentence describing the improvements and new features your team is proposing for The Slide Machine.

## User Requirements

### Instructors

1. As an instructor, I want to mark a spoken aside as a tangent while I'm still talking, so that it becomes a branch off the current slide.
2. As an instructor, I want to see which topic the AI is currently adding new content to, so that I can redirect it to a different slide if it's about to file something in the wrong place.
3. As an instructor, I want to reclassify an already-generated slide as a branch after I realize the discussion turned into a tangent, so that the deck's structure still reflects what actually happened even though I didn't catch it at the moment.
4. As an instructor, I want to merge a branch slide back into its main-line slide, so that content that turned out not to be a real tangent doesn't stay separated for no reason.
5. As an instructor, I want to delete a branch slide I no longer need, so that irrelevant tangents don't clutter the final deck.
6. As an instructor, I want to undo my last structural action (branch, merge, split, or reclassification) immediately after making it, so that I can recover quickly if the system misunderstood my intent.
7. As an instructor, I want to be notified when marking a tangent fails due to a network or recognition error, so that I know the content was captured as a normal slide instead and isn't lost.
8. As an instructor, I want to move a branch slide to attach it to a different main-line slide, so that I can correct it if it ended up associated with the wrong topic.
9. As an instructor, I want to explicitly signal "continue previous topic" to rejoin an earlier main-line slide instead of starting a new one, so that returning to something I already covered doesn't fragment the deck.
10. As an instructor, I want to split an already-generated slide into two once I realize it covers two separate topics, so that each topic gets its own slide without waiting for a post-lecture reformat.
11. As an instructor, I want to generate a shareable link to the deck while a session is still live, so that students can follow along on their own devices in real time.
12. As an instructor, I want a branch slide to stay attached to its main-line slide when exporting the deck, even if the two are organized into different folders, so that a branch is never separated from the topic it belongs to.
13. As a mathematics instructor, I want to quickly correct a misrecognized technical term, spoken mathematical expression, or diagram/graph the system couldn't capture from speech alone, without stopping the lecture, so that later generated slides use the correct terminology and notation without requiring me to stop speaking.

### Students

1. As a student, I want to see when the instructor corrected or reclassified a slide during the lecture, so that the shared deck doesn't leave me confused about which version of a slide is the final one.
2. As a student, I want to see supplementary branches connected to a lecture topic, so that I can explore related material without losing the main lecture structure.
3. As a student, I want to see questions other students have already asked and the professor's answers, so that I can find answers to my own questions without having to ask the professor again.
4. As a student, I want to open a supplementary branch from the point where it appears in the lecture, so that I can understand how the related material connects to the main topic.
5. As a student, I want to return to the main lecture from a supplementary branch, so that I can continue following the lecture without losing my place.
6. As a student, I want to skip a supplementary branch, so that I can stay focused on the main lecture when the related material is not relevant to me.
7. As a student, I want to filter branch/tangent slides out of the deck when studying for the quiz, so that I only review the main-line content the lecture was actually about.
8. As a student, I want to see which slides were reorganized after the live session ended, so that notes I took during the original lecture don't reference a structure that no longer exists.
9. As a student, I want to open a live-shared link to the deck while the instructor is still generating it, so that I can follow the lecture on my own device as it happens.
10. As a student, I want to add a private comment to a slide while viewing the live-shared deck, so that I can record my own thoughts or questions without anyone else seeing them.
11. As a student, I want my private comments to remain attached to their slide after the session ends and the deck is saved, so that reviewing later feels like looking at my own personal copy.
12. As a student, I want to edit or delete a private comment I made, so that I can correct or remove notes I no longer need.
13. As a student, I want the live-shared deck to update automatically as the instructor generates or restructures slides, so that I'm always seeing the current state of the lecture without needing to refresh or re-open the link.

## Activity Diagrams

See instructions. Delete this line and place images of your UML Activity diagrams here, each with the text of the user story it illustrates.

## Wireframes

See instructions. Delete this line and place your wireframe diagrams here, covering every new screen and every existing screen your proposal changes, for every type of user.

## Clickable Prototype

See instructions. Delete this line and place a publicly-accessible link to your clickable prototype here.

## Stakeholder Demo

See instructions. Delete this line and place a link to the deck The Slide Machine generated during your presentation here, after you have presented.

## Exit Ticket

See instructions. Delete this line and place a link to the exit-ticket quiz you generated from your demo deck and distributed to the class, along with a short note on what — if anything — you had to correct in the generated questions before publishing.
