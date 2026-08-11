# Team Inbox / social-outgoing

Queue folder for the daily Facebook + LinkedIn social-posting flow, mirroring
the `Team Inbox/slack-outgoing/` convention. One file per scheduled post day.
Written by [[SOP-019-weekly-podcast-social-package]] (§5), drained by
[[SOP-020-daily-social-review-and-posting]].

## Filename convention

`YYYY-MM-DD-<weekday>.md` — e.g. `2026-08-18-tuesday.md`. One file per
posting day (Tue/Wed/Thu/Fri only — SOP-019 does not generate Mon/Sat/Sun
posts).

## Frontmatter contract

```yaml
---
date: 2026-08-18
weekday: tuesday
status: pending-review        # pending-review | approved | posted | skipped-no-reply
source_sop: SOP-019
episode_slug: coaches-vision-weekly           # matches the Dropbox episode folder slug
facebook_text: |
  Post copy for Facebook, as drafted by SOP-019 Step 2.
facebook_image_path: "/Users/.../Dropbox/For Chloe/YOUTUBE:POD/<episode>/tuesday.png"
facebook_image_url: null      # public URL — NOT YET SOLVED, see gap note below
linkedin_text: |
  Post copy for LinkedIn — same base copy as Facebook, lightly re-framed if needed
  (SOP-019 reuses one drafting pass per GL-... convention; don't re-draft from scratch).
linkedin_image_path: "/Users/.../Dropbox/.../tuesday.png"
linkedin_image_url: null      # public URL — NOT YET SOLVED, see gap note below
instagram_caption: |
  Caption text only — Instagram is manual-post-only, never auto-posted.
review_email:
  sent_at: null
  message_id: null            # Gmail Message-ID of the review email Mack/Larry sent
  thread_id: null              # Gmail thread ID — used to search for Daniel's reply
  subject: null                 # exact subject line used, so the reply search is exact
edits:                          # populated only if Daniel's reply changed the copy
  facebook_text: null
  linkedin_text: null
  instagram_caption: null
posted_at: null
posted_results:
  facebook_post_id: null
  linkedin_share_id: null
---
Optional freeform notes (e.g. anything unusual about this day's post, decisions made).
```

## Status lifecycle

1. **pending-review** — written by SOP-019 §5. Nothing has happened yet.
2. **approved** — Daniel replied APPROVE (or with edited text) to the review
   email. Edits, if any, are copied into the `edits` block and treated as the
   authoritative copy to post.
3. **posted** — Facebook + LinkedIn posts executed successfully. `posted_at`
   and `posted_results` filled in.
4. **skipped-no-reply** — no reply arrived by the afternoon check. This is
   the safe default; nothing gets posted without an explicit approval.

Files are **not** moved between subfolders on status change — the frontmatter
`status` field is the single source of truth, exactly as specified by
Daniel. `.posted/` and `.skipped/` exist only as an optional manual-archive
target if the folder gets noisy; SOP-020 does not move files automatically.

## Known gap — image hosting (flagged, not solved)

`facebook_image_path` / `linkedin_image_path` point at local Dropbox files.
Both the Facebook and LinkedIn Zapier actions are executed by a cloud-side
Claude session (see [[SOP-020-daily-social-review-and-posting]]), which
cannot read Daniel's local filesystem or his Dropbox — so a local path alone
is not postable. Until a public-image-hosting step exists, SOP-020 posts
**text-only** to both Facebook and LinkedIn (no `source` / no
`content__submitted_image_url`). The `_image_url` fields exist in this
frontmatter so a future hosting step (e.g. upload to a public Dropbox share
link, S3 bucket, or sportsvision.nyc media folder) has somewhere to write
its output without a schema change. This is an open decision for Daniel —
see SOP-020 for the full flag.

## References

- [[SOP-019-weekly-podcast-social-package]] — writes these files, one per
  weekday, at the end of the weekly package run.
- [[SOP-020-daily-social-review-and-posting]] — reads/updates these files:
  morning review email, afternoon approve-and-post.
- `Team Inbox/slack-outgoing/` — the sibling convention this folder mirrors.
cloud-side content
