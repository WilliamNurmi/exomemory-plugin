---
name: exomemory
description: Search and read the user's own Exomem memory archive (their chats, notes and voice memos) with the Exomem tools. Use when the user asks about something they said, wrote, planned or discussed before, wants dated evidence from their own records, or asks what their Exomem contains.
---

# Exomem

Exomem is the user's personal memory archive: chat histories, notes and voice-memo
transcripts they exported and imported into an Exomem running on their own
computer. The Exomem tools reach it through exomem.io. Every tool is read-only
except `log_feedback`.

If the user has not set up their own Exomem yet, the tools answer from a demo
workspace with synthetic memories of an invented person, and every result starts
with a demo notice. Say so plainly, never present demo memories as the user's
own, and point to https://exomem.io/setup when they want their own data.

## Workflow

1. **Orient once per conversation.** Call `capabilities` to learn the source keys,
   record types, collections and date coverage this archive actually has. Use
   those keys in filters instead of guessing.
2. **Search.** Call `search` with the user's question as `query` and a one-sentence
   `queryInstruction` of the form "Retrieve <what> from <which part of the
   archive>", for example "Retrieve passages from personal notes that answer the
   question." Put hard constraints in `filter`: `sourceKeys`, `occurredAt` for
   time, `people` for who was involved. Resolve a person's name with
   `resolve_person` first when the name is ambiguous.
3. **Read before answering.** Open the most promising hits with `get` (pass the
   hit's `passageRef`). Long records come in sections; follow `sectionCursor`
   only as far as the question needs.
4. **Widen when needed.** Use `expand` on a record to see its neighbours: the
   surrounding chat messages, a note's outline parents and children, linked notes.
5. **Browse instead of search** when the user asks what exists ("my notes about
   X", "everything from March"): `list_collections`, then `list_records` with
   filters, newest first.

## Search tactics

- Keep `topK` modest (about 10) and page with `cursor` instead of asking for
  hundreds of passages. Make calls one after another: they are served by the
  user's own computer.
- When an episodic framing misses ("what did I write after the trip"), search
  again with the remembered content itself as the query.
- Use `mode: "lexical"` for exact names, titles or phrases; the default hybrid
  mode covers most questions.
- People often write in more than one language. Search in the language the
  memory was most likely written in, or try both.
- If a source the user mentions is not in `capabilities`, say it is not in their
  archive (or is kept local-only on their computer) instead of guessing.

## Answering

- Lead with the answer, then the evidence: date, source and who said or wrote
  it, with short quotes. Use the record links the tools return.
- Separate what the records say from your own inference. If nothing relevant was
  found, say what you searched and suggest a better query or source.
- Memories mention other people. Quote only what the question needs.
- Do not copy memory content into other tools, files or services unless the user
  asks for that in this conversation.

## Problems

- Messages such as "Your Exomem device ... is offline" or "did not answer within
  60 seconds" come from exomem.io. Pass them on as they are; the fix is on the
  user's computer (`pnpm link:status` there).
- When an Exomem tool misbehaves (an error, an obviously wrong result, a missing
  capability), you may record one line with `log_feedback`: the kind (bug,
  friction, gap, idea or win), the tool or area, and what happened. exomem.io
  keeps the note. Use only made-up examples or limited details the user has
  agreed to share, and tell the user you sent it.
