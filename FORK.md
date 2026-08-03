# tk-flame-export — Arc Film studio fork

Studio fork of [`shotgunsoftware/tk-flame-export`](https://github.com/shotgunsoftware/tk-flame-export).
Maintained because we carry real code changes across several files, not just
config tweaks. If `info.yml` template validation fails, the app silently fails
to load and its menu items vanish from Flame — so these changes cannot live as
loose patches on a host.

## Baseline & remotes

- `origin`   → `<yourorg>/tk-flame-export` (this fork)
- `upstream` → `shotgunsoftware/tk-flame-export`
- Working branch: `arc-main`, branched from the upstream release tag **v1.11.4**
  (branched from the tag, never from `master`).

Each logical change is applied as **one commit** so future rebases isolate any
conflict to a single patch.

## Patch inventory

Fork-side patches (live on `arc-main`):

1. `info.yml` — `plate_presets > template` fields: drop `version`
   (bracketed as `[version]` to keep it optional). Version is already carried in
   the segment name, e.g. `RND_001_0270_src01_v001`.
2. `info.yml` — `batch_render_template` fields: drop `segment_name`. The comp is
   one versioned output per shot, no segment.
3. `info.yml` — `frame_handles` default `10` → `8`.
5. `python/export_utils/export_preset.py` — `<frameIndex>3</frameIndex>`; VERIFY on each re-apply — most drift-prone since v1.11.1.
6. `python/export_utils/segment.py` — Added `_source_frame_offset()` property and `_normalise()` function to manage cut
   info to Flow being calculated from timecode-absolute frames rather than sequence/render relative (ie, start at 1001,
   not 4939493949). Added to properties for `cut_in_frame()`, `cut_out_frame()`, `head_in_frame()` and `tail_out_frame()`
7. Add line to `export_preset.py` - Added `<mode>Follow Iteration</mode>` to preset. Nixed arc_batch_patcher logic.
8. Re-implement flame_batch_patcher.py functionality via arc-utils to fix the batch iteration name after publish.
   Can't do via export templates/regular means.

Config-side, NOT in this fork:

4. `hooks/settings.py` "10 bit DPX" → "12 bit DPX" (bitsPerChannel 10→12,
   removed `<publishLinked>` lines). Relocated to `{config}/flame_export_settings.py`
   so it overrides without forking the hook. Apply this one first, before the
   fork patches.
5. Renamed configs in order of preference (EXR first), enabled hard link logic.

Recommended re-apply order: #4 (config) → #1–3 → #7 → #5/#6 (with verification).

## Tagging & descriptors

- Tag fork releases `v1.11.4-arc.N`.
- Production config: `git` descriptor pinned to the tag (immutable).
- May actually just point at cloned repo, but we'll see.
- While iterating: `dev` descriptor pointing at a local checkout.

## info.yml field-spec syntax (reference)

Bare name = mandatory · `[name]` = optional · `*` = allow others.
Upstream mandates both `version` and `segment_name` on every
render/quicktime/batch-render template; only `shot_clip_template` and
`batch_template` omit them.
