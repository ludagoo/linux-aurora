# SmartRoom carry on linux-aurora

`smartroom/main` is the kernel SmartRoom Firmware ships: an omacom
`integration/aurora` commit, plus the patches listed below, plus one
non-kernel commit (this file and `.github/workflows/smartroom-sync.yml`).
That commit is ours only and is never sent upstream. Packages are built by
[ludagoo/smartroom-pkgs](https://github.com/ludagoo/smartroom-pkgs).

## Base

| | |
| --- | --- |
| Base commit | `b798f2b0c767e3d780896cff2f57e19e412bf954` ("Merge pull request #352 … fix/t8103-sio-alias", 2026-11-04) |
| Why this one | omacom/omarchy-pkgs pins it for `linux-aurora 7.2.11.aurora15-1` (omarchy-pkgs #2203, 2026-11-05; Omarchy edge). rc and stable still ship aurora14. |
| Kernel changes | none (Phase 0 builds upstream Aurora unchanged) |

## Carried patches

None yet.

| Subject | Why we carry it | Upstream PR / status | Drop when |
| --- | --- | --- | --- |
| | | | |

## Branches in this fork

- `integration/aurora`: an exact mirror of omacom's, fast-forward only. Never commit to it.
- `smartroom/main`: this carry. Rebased, never merged. Before force-updating it, tag the old tip (`sr/<pkgver>-<pkgrel>` for a published build, else `sr/prev-<date>`), so every shipped package can be rebuilt (GPL source). Never delete `sr/*` tags.
- `sr/<topic>`: one upstream topic each, cut from `integration/aurora`, opened as a PR to omacom/linux-aurora, cherry-picked here until it merges.
- `fix/*`, `local/*` and the other existing branches are Lucas's own work and PR heads. Never delete or rewrite them.

## Rules for patches

- **Patches written by an AI agent go only to omacom** (linux-aurora or omarchy-pkgs), never to AsahiLinux, m1n1 or asahi-installer: Asahi's LLM policy forbids generative-AI contributions. Only work Lucas writes himself goes to Asahi.
- **Agents never add `Signed-off-by`.** They add `Assisted-by: AGENT_NAME:MODEL_VERSION` (for example `Assisted-by: Claude:claude-opus-5-5`), per `Documentation/process/coding-assistants.rst`.
- **Lucas reviews each patch and adds his own `Signed-off-by`** (`git rebase --signoff`) before a PR is opened. omacom's PR CI blocks commits without it.
- Prefer config (through omacom's in-tree `arch/arm64/configs/asahi.config`) or userspace changes over kernel patches. Keep this list short and upstream early.

## Rebasing onto a new base

`smartroom-sync.yml` opens the issue "smartroom/main needs a rebase" when omarchy-pkgs pins a commit that `smartroom/main` does not contain.

```sh
git fetch origin integration/aurora smartroom/main
old_base=$(git merge-base origin/smartroom/main origin/integration/aurora)
new_base=<the new omarchy-pkgs _commit>
git tag sr/prev-$(date +%F) origin/smartroom/main && git push origin sr/prev-$(date +%F)
git switch -C smartroom/main origin/smartroom/main
git rebase --onto "$new_base" "$old_base"
git range-diff "$old_base"..origin/smartroom/main "$new_base"..smartroom/main   # review what moved or dropped
git cherry "$new_base" smartroom/main                                          # patches still carried
git push --force-with-lease=smartroom/main:origin/smartroom/main origin smartroom/main
```

Then update the Base table above and rebuild in smartroom-pkgs. A conflict
resolution that changes a carried patch is Lucas's decision before it ships to
rooms.

## Sync workflow

`.github/workflows/smartroom-sync.yml` (daily, on push to `smartroom/main`, or by hand):

1. fast-forwards `integration/aurora` to omacom's (never merges, never forces);
2. disables any inherited upstream workflow (`kernel-ci*.yml` and new ones), so upstream CI never runs here;
3. checks that `smartroom/main` contains omarchy-pkgs' current `_commit`, opening, updating or closing an issue.

It does not build the kernel. GitHub runs schedules only from the default
branch, so the daily run is dormant while the default branch is
`integration/aurora`; run it by hand with
`gh workflow run smartroom-sync.yml -R ludagoo/linux-aurora --ref smartroom/main`.
The Actions token cannot fast-forward onto upstream commits that change
`.github/workflows`; then the workflow opens "integration/aurora mirror sync
failed" and Lucas syncs once with
`gh api -X POST repos/ludagoo/linux-aurora/merge-upstream -f branch=integration/aurora`
(or adds a `SMARTROOM_SYNC_TOKEN` secret: fine-grained, contents + workflows
write on this repository only).
