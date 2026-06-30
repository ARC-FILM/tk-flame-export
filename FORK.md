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
5. `python/export_utils/export_preset.py` — inject `<startFrame>1001</startFrame>`
   and `<frameIndex>3</frameIndex>`; flip `use_timecode_as_frame_number` default
   to `False`. VERIFY on each re-apply — most drift-prone since v1.11.1.
6. `python/export_utils/segment.py` — add `frame_offset` property
   (`sourceIn - startFrame`); subtract it in `cut_in`/`cut_out`/`head_in`/`tail_out`
   to normalise absolute frames to sequence-relative (1001, not 12029320).
   Pairs with patch 5. VERIFY on re-apply.
7. `app.py` + new `arc_batch_patcher.py` — `fix_batch()` runs when
   `batch_version_number == 0`; rewrites batch setup + writefile node XML
   (snapshot library naming, iteration, version-follow, output names/type) and
   backs up originals. Logic lives in the module; `app.py` is reduced to a
   one-line call.

Config-side, NOT in this fork:

4. `hooks/settings.py` "10 bit DPX" → "12 bit DPX" (bitsPerChannel 10→12,
   removed `<publishLinked>` lines). Relocated to `{config}/flame_export_settings.py`
   so it overrides without forking the hook. Apply this one first, before the
   fork patches.

Recommended re-apply order: #4 (config) → #1–3 → #7 → #5/#6 (with verification).

## Tagging & descriptors

- Tag fork releases `v1.11.4-arc.N`. Push tags to origin explicitly:
  `git push origin v1.11.4-arc.1` (branch pushes don't carry tags).
- Production config: `git` descriptor pinned to the tag (immutable).
- While iterating: `dev` descriptor pointing at a local checkout.

## Updating onto a newer upstream release

    git fetch upstream --tags
    git rebase --onto <new-tag> <old-tag> arc-main
    # resolve conflicts per-commit; pay attention to patches 5 and 6
    git tag v<new>-arc.1
    git push origin arc-main --force-with-lease
    git push origin v<new>-arc.1
    # then bump the descriptor version in the studio config

## info.yml field-spec syntax (reference)

Bare name = mandatory · `[name]` = optional · `*` = allow others.
Upstream mandates both `version` and `segment_name` on every
render/quicktime/batch-render template; only `shot_clip_template` and
`batch_template` omit them.
