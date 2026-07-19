# System Design Lab

My personal system design practice log. I pick a topic, write it up as a
problem statement, and design a solution — diagrammed in Excalidraw — so I
can look back on it later.

## How it works

1. Create a new folder at the repo root named after the subject (e.g.
   `rate-limiter/`, `url-shortener/`).
2. Add `topic.md` with the problem statement.
3. Design the solution and export it into `diagrams/`.
4. Add any takeaways or things to revisit in `notes.md`.

## Folder structure

Each subject gets its own folder at the root — no week numbers, no
grouping, just the topic name:

```
rate-limiter/
  topic.md              # problem statement
  notes.md               # takeaways, open questions, things to revisit
  diagrams/
    design.excalidraw
    design.png            # exported PNG so it previews on GitHub
```

`topic.md` should cover:
- Problem statement (what to design, and for what product)
- Functional requirements
- Non-functional requirements (scale, latency, consistency vs availability)
- Out of scope

Excalidraw `.excalidraw` files are JSON and are fully editable if you drag
them back into https://excalidraw.com. Always export and commit a `.png`
alongside it too, since GitHub can't preview `.excalidraw` files directly.

## Index

| Sr No | Topic | Design |
|-------|-------|--------|
|       |       |        |

Update this table each time a new subject is added.
