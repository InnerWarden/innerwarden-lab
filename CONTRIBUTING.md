# Contributing

Thanks for considering a contribution. This lab grows by adding well-scoped CVE walkthroughs and by reporting reproductions where Inner Warden's behaviour differs from what's documented.

## What we accept

- **New CVE walkthroughs.** Any CVE that meets ALL of:
 - Already publicly disclosed (CVE record exists, advisories published).
 - Already patched upstream (the fix is shipping in current kernel / package releases).
 - The original PoC is published by a recognised security researcher / lab / vendor and is referenced (not re-hosted) from the walkthrough.
 - The walkthrough teaches DEFENDERS something about detection , it is not framed as a how-to-exploit guide.
- **Reproduction reports.** You ran one of the existing recipes and your `result.json` differs from `EXPECTED-DETECTION.md`. Open an issue with both files attached and the VM details you used.
- **Detection improvements.** You ran a recipe and found a behaviour Inner Warden misses that should be caught. Open an issue describing the gap; if you have a detector idea, sketch the design in the issue or open a PR against [`InnerWarden/innerwarden`](https://github.com/InnerWarden/innerwarden) itself.
- **Setup script improvements.** `setup.sh` is wrong for your distro / kernel / cloud. PR welcome , keep the script idempotent and well-commented.
- **Documentation fixes.** Typos, clarifications, dead links , straightforward PRs.

## What we don't accept

- **Novel exploit code.** This is not a 0-day research repo. Every PoC referenced here is already publicly disclosed and patched. Submissions that depend on undisclosed primitives are out of scope and will be closed.
- **Re-hosted exploit source.** Always link to the original author's publication. If you maintain a fork of an upstream PoC, link to that fork; do not vendor the source here.
- **Lab recipes that run on hosts other than isolated VMs.** Every setup.sh assumes a throwaway, network-isolated host. If your setup needs internet access, requires production credentials, or otherwise compromises the isolation model, rework it before submitting.
- **Detection adversarial techniques.** This lab measures detection, not evasion. Submissions designed to demonstrate how to bypass Inner Warden are best discussed privately with the maintainers first; reach out via `security@innerwarden.com`.
- **Anything that lowers the safety bar.** If you have to think about whether your contribution is responsible, the answer is usually that it isn't.

## How to add a new CVE walkthrough

1. Create `cve-YYYY-NNNNN-<short-name>/`. Lowercase, hyphens.
2. Mirror the structure of `cve-2026-31431-copy-fail/`:
 - `README.md` , the recipe
 - `setup.sh` , provisioning
 - `baseline.sh` + `measure.sh` , the measurement pair (copy from Copy Fail, adapt the SUID target / detector-of-interest queries)
 - `SOURCES.md` , pointers only, no source
 - `EXPECTED-DETECTION.md` , what Inner Warden produced on YOUR run (the canonical reference future readers compare against)
 - `teardown.sh` , clean up after testing
3. Add a row to the main [`README.md`](README.md) "Available CVEs" table.
4. Open a PR with:
 - The VM you used (cloud, image, kernel version pinned).
 - The exact PoC reference + SHA-256.
 - A copy of YOUR `baseline.json` + `result.json` from the run that produced the `EXPECTED-DETECTION.md`.
 - A brief note on what worked, what didn't, what surprised you.

A maintainer will reproduce on a second VM and (typically) merge within a week.

## How to report a divergence

If you ran a recipe and your `result.json` differs from what `EXPECTED-DETECTION.md` claims:

1. [Open an issue](https://github.com/InnerWarden/innerwarden-lab/issues/new).
2. Include:
 - Which CVE folder you reproduced.
 - Inner Warden version (`/usr/local/bin/innerwarden-sensor --version`).
 - VM image + kernel (`uname -a` + `cat /etc/os-release`).
 - Both your `baseline.json` and `result.json` (gist or paste).
 - What you expected to see vs what you actually saw.
3. The maintainers will either reproduce the divergence (and update `EXPECTED-DETECTION.md`) or explain what changed in the agent that produced your result. Either outcome is useful.

## Code style

- Bash scripts: `set -euo pipefail`, comments explaining non-obvious decisions, idempotent.
- Markdown: GitHub-flavoured, sentence case, code blocks language-tagged.
- Commit messages: English. `<type>(<scope>): <what>`. Body explains WHY when the diff doesn't make it obvious.

## License

Contributions are accepted under Apache-2.0, the project license. By submitting a PR you confirm you have the right to license your contribution under those terms.
