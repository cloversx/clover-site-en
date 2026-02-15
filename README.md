* * *

Clover Site — Canonical Archive
===============================

This repository maintains a canonical archive of writing and explorations across technology, society, and human experience — including essays and creative work.

It is not built for rapid content cycles or algorithmic feeds.

It exists to preserve the evolution of thought and making.

* * *

Canonical Principle
-------------------

This site is the canonical source.

Other platforms (social, newsletters, mirrors) are treated as distribution paths and pointers back to the canonical URLs.

The site’s **display naming** (e.g., “Library”) may change over time, but the **URL structure remains stable**.

* * *

Structure
---------

Each piece lives at:

```
/0/<dir-name>/
```

This URL always represents the latest version (current).

Previous versions are preserved at:

```
/0/<dir-name>/v1/
/0/<dir-name>/v2/
...
```

Archived versions are immutable and **self-canonical**.

* * *

Identifiers: `dir-name` vs `content_id`
---------------------------------------

*   `dir-name` is the **stable URL identifier** (derived from the folder path).
*   `content_id` is a **logical identifier in front matter** and is **not used to generate URLs**.

Recommended practice:

*   Keep `content_id` equal to `<dir-name>` to avoid operational confusion.
*   Do not change `content_id` when archiving under `/vN/`.

* * *

Version Philosophy
------------------

Version numbers represent intellectual (or creative) stages.

### Minor updates (no version bump)

*   Update `last_updated`
*   Do not increment `version`
*   Record the change under the current Version’s minor updates (in the content’s Changelog section)

### Major updates (version bump)

*   Archive the current state to `/vN/`
*   Increment `version` in the current page
*   Preserve both states as layered history

Conceptually distinct works (Revisited) are published under a new `<dir-name>` and start again at Version 1.

History is layered, not erased.

* * *

Dates
-----

*   `first_published` marks when the work first entered the archive (fixed per `<dir-name>`).
*   `last_updated` marks when the current form was last shaped.
*   Archived versions remain fixed and are never edited.

* * *

Canonical, hreflang, and front matter
-------------------------------------

*   Canonical URLs are usually generated automatically from `page.url`.
*   `canonical_url` is only set manually for exceptional cases (migration, consolidation, special SEO needs).
*   `alt_url` is meaningful for current pages (language linking) and is not emitted for archives.
    *   When archiving, `alt_url` is typically left unchanged to keep the workflow lightweight.

* * *

Design Principle
----------------

This is not a content stream.

It is a structured archive for preserving thought over time —  
written not to conclude, but to continue thinking.

* * *
