# Tickets

One file per stage: `stage-00.md`, `stage-01.md`, ... Tickets are written
**just before** you start a stage, not all at once, because what you learn
in one stage changes the next.

The map of all stages is `../booking-platform-requirements.md`.

## Ticket format

```
### S<stage>-<number> Title = a clear outcome, not an activity
**Why:** one line.
**Do:** short list of the work.
**Acceptance:** 2-5 checkable statements. If you cannot check it, rewrite it.
**Test first:** the test you write BEFORE the code (when it applies).
**Learn:** the concept you must be able to explain afterwards.
```

## Working rules

-   One ticket = one branch = one pull request. Branch name:
    `s1-03-tenants-table`. Commit style: `feat(db): add tenants table`.
-   The PR description lists the acceptance items and ticks them.
-   A ticket should fit in one sitting or one day. If not, split it.
-   Mark a ticket done only when its acceptance items are verified, not
    when the code "looks right".
-   When a stage is done, add a short entry to `docs/learning-log.md`:
    what I built, what broke, what I would do differently.

## Index

-   [Stage 0 --- Project setup](stage-00.md)
-   [Stage 1 --- Schema foundations and migrations](stage-01.md)
