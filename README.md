# Team Inbox / social-outgoing

Queue folder for the daily Facebook + LinkedIn social-posting flow, mirroring
the `Team Inbox/slack-outgoing/` convention. One file per scheduled post day.
Written by [[SOP-019-weekly-podcast-social-package]] (§5), drained by
[[SOP-020-daily-social-review-and-posting]].

**This folder is mirrored to a private GitHub repo,**
`https://github.com/danlaby-del/mypka-social-outgoing` **— that mirror, not
this local folder, is what SOP-020's cloud-side routines actually read and
write against** (they have no access to this local vault). Sync is one-way
push (local -> GitHub, via SOP-019 §5 step 7) plus one-way-per-field pull
(GitHub -> local, via a launchd job), handled by
`Team Knowledge/SOPs/scripts/social_outgoing_sync.py`. See SOP-020 §0 for
the full contract.

**Known upstream fragility (as of 2026-08-26, diagnosis corrected same
day):** the real root cause is the two CronCreate cloud routines behind
SOP-020 (`trig_01JjLgsExJQrmP91vkUxW6kB` morning,
`trig_01MHVojPidA74UX8MuNqPvDc` afternoon, see SOP-020 §0). They run
inline, self-contained natural-language instructions that compose the YAML
frontmatter text themselves before writing it to the GitHub mirror, and
have been observed writing an unquoted colon inside a scalar value
(`review_email.subject`), which is invalid YAML and used to crash the local
`--pull` job. This is NOT a bug in the Zapier code action
`code_action_githubcliapi__create_or_update_file_reliable` (see SOP-020
§0/§3 step 4). That action is a generic plain-text GitHub file writer
(`repo`, `path`, `content`, `message`, `branch`, `sha`) that never builds
YAML itself; an earlier same-day diagnosis blaming that action was wrong
and has been corrected. `social_outgoing_sync.py` v1.2.0 fixed the
local-side symptom: a single malformed mirror file no longer aborts the
whole pull run (every other in-scope file still gets processed), and the
pull now attempts a best-effort repair of that specific malformed-value
pattern before giving up on a file. The real fix, an explicit mandatory
quoting rule for the cloud routines' YAML composition step, is now
documented in SOP-020 §3 and §4 (added 2026-08-26); pushing that rule into
the routines' actual stored instructions via `RemoteTrigger update` is
still an outstanding action item flagged in SOP-020 §0 (Larry has the
live-routine-editing permission needed). See `Team Knowledge/log.md`
2026-08-26 entries for the full incident, the wrong diagnosis, and the
correction. The now-superseded apply guide at
`Team Knowledge/SOPs/scripts/2026-08-26-zapier-yaml-fix-apply-guide.md`
carries a correction notice at the top; do not apply it, no Zapier code
action needs editing.

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
facebook_image_url: null      # public raw.githubusercontent.com URL — populated by social_outgoing_sync.py --push, see "Image hosting" section below
linkedin_text: |
  Post copy for LinkedIn — same base copy as Facebook, lightly re-framed if needed
  (SOP-019 reuses one drafting pass per GL-... convention; don't re-draft from scratch).
linkedin_image_path: "/Users/.../Dropbox/.../tuesday.png"
linkedin_image_url: null      # public raw.githubusercontent.com URL — populated by social_outgoing_sync.py --push, see "Image hosting" section below
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

## Image hosting — resolved 2026-08-11

`facebook_image_path` / `linkedin_image_path` point at local Dropbox files,
which the cloud-side Claude session that executes the Facebook/LinkedIn
Zapier calls (see [[SOP-020-daily-social-review-and-posting]]) cannot read
directly. This is now solved: the mirror repo,
`https://github.com/danlaby-del/mypka-social-outgoing`, is a **public**
repo (Daniel confirmed 2026-08-11), so
`https://raw.githubusercontent.com/danlaby-del/mypka-social-outgoing/main/<path>`
URLs against it are genuinely public and fetchable by Facebook/LinkedIn's
servers without auth.

Mechanism: `social_outgoing_sync.py --push` (run from SOP-019 §5 step 7)
now does two things for each queue file it pushes, not just one:

1. Copies the queue file's markdown into the mirror repo, as before.
2. Reads `facebook_image_path` / `linkedin_image_path`, copies that image
   file into the mirror repo's `images/` subfolder (named after the queue
   file, e.g. `images/2026-08-18-tuesday.png` — Facebook and LinkedIn
   almost always share one source file, so normally only one copy is
   pushed and both URL fields point at it), and writes the resulting
   `raw.githubusercontent.com` URL into `facebook_image_url` /
   `linkedin_image_url` — both in the pushed mirror copy and back into the
   local vault file, so the URL persists locally too.

SOP-020 §4 step 5 reads `facebook_image_url`/`linkedin_image_url` when
present and passes them as Facebook's `source` param and LinkedIn's
`content__submitted_image_url` param. If a queue file's URL is still
`null` (an older or malformed file, or a missing/unreadable source image),
the posting step falls back to text-only for that platform rather than
erroring — see SOP-020 §4 step 5 for the exact fallback logic.

The mirror repo's scope stays deliberately narrow even with images added:
it only ever contains queue markdown files, `README.md`, and the specific
images those queue files reference via `*_image_path` — never a Dropbox
directory scan, never anything else from the vault. See
`social_outgoing_sync.py`'s module docstring, constraint #1, for the exact
scope-enforcement rule.

## References

- [[SOP-019-weekly-podcast-social-package]] — writes these files, one per
  weekday, at the end of the weekly package run.
- [[SOP-020-daily-social-review-and-posting]] — reads/updates these files:
  morning review email, afternoon approve-and-post.
- `Team Inbox/slack-outgoing/` — the sibling convention this folder mirrors.
