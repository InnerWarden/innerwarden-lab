# Inner Warden , CVE Reproduction Lab

A reproducible, isolated lab for verifying [Inner Warden's](https://github.com/InnerWarden/innerwarden) detection of publicly disclosed Linux kernel CVEs. Clone, follow the per-CVE recipe, watch the autonomous agent react in real time on your own VM.

> **What this is:** a defensive validation suite. Public CVEs, public PoCs, public patches. The lab provisions a vulnerable VM, runs the published exploit, and shows what Inner Warden caught (and didn't).
>
> **What this is not:** novel exploit research, weaponisable tooling, or anything that runs without an explicit isolated VM. See [`DISCLAIMER.md`](DISCLAIMER.md).

> ⭐ **If this lab is useful to you, [star Inner Warden](https://github.com/InnerWarden/innerwarden)** , it is the agent under test and the project that funds this work. See ["Build this with us"](#build-this-with-us) at the bottom for the three ways to contribute.

## Why this exists

Inner Warden ships a lot of detectors and makes a lot of claims. This lab puts those claims under the harshest possible light: real, public, patched-in-the-wild kernel CVEs with working PoCs. Each entry under `cve-*/` has:

- The repro recipe (kernel pin, vulnerable package versions, links to the original PoC).
- A `setup.sh` that provisions the lab state idempotently.
- A `baseline.sh` and `measure.sh` pair that records the agent's behaviour before and after the exploit, so the "did it catch it?" answer is data, not vibes.
- An `EXPECTED-DETECTION.md` capturing what Inner Warden actually did the last time we ran it , including the cases where the answer is "missed it, here's why, here's what we added in the next release".

If you reproduce a CVE and the detection differs from what's documented, [open an issue](https://github.com/InnerWarden/innerwarden-lab/issues/new) , that's what this repo is for.

## Available CVEs

| CVE | Class | Status |
|-----|-------|--------|
| [CVE-2026-31431 , Copy Fail](cve-2026-31431-copy-fail/README.md) | `algif_aead` → page cache write → LPE | ✓ Recipe + reproduction |
| CVE-2026-43284 , Dirty Frag (ESP) | esp4/esp6 → page cache write → LPE | _planned_ |
| CVE-2026-43500 , Dirty Frag (RxRPC) | RxRPC → page cache write → LPE | _planned_ |
| CVE-2026-46300 , Fragnesia | XFRM ESP-in-TCP → page cache write → LPE | _planned_ |

All four belong to the 2026 page-cache LPE family. They use different kernel subsystems for ingress but share the same primitive (foreign-owned scatterlist written into a setuid binary's page cache).

## Quick start

The shortest path from a fresh checkout to "I just watched Inner Warden catch a real CVE":

```bash
# 1. Provision an isolated VM (any cloud or local hypervisor, Ubuntu 22.04 LTS x86_64)
# See VM-REQUIREMENTS.md for the spec.

# 2. From your laptop, push the lab recipe to the VM:
scp -r cve-2026-31431-copy-fail/ <user>@<vm-ip>:/tmp/lab/

# 3. SSH into the VM and run the setup:
ssh <user>@<vm-ip>
bash /tmp/lab/setup.sh

# 4. Capture the "before" state:
sudo /tmp/lab/baseline.sh > /tmp/baseline.json

# 5. Fetch the published PoC from its original author (link in cve-*/SOURCES.md),
# record the start epoch, run it. See per-CVE README for the exact command.

# 6. Capture the "after" state:
POC_START=<the epoch you recorded>
sudo /tmp/lab/measure.sh /tmp/baseline.json $POC_START > /tmp/result.json

# 7. Read /tmp/result.json. Compare to cve-*/EXPECTED-DETECTION.md.
```

End-to-end runtime: ~10 minutes if the VM is already running. The exploit itself is sub-second.

## VM requirements

See [`VM-REQUIREMENTS.md`](VM-REQUIREMENTS.md) for the canonical spec. Summary:

- **Ubuntu 22.04 LTS Server** (x86_64) , other distros may work but this is what we test against.
- **Kernel:** must be vulnerable to the CVE you're testing. Most distros backport patches quickly, so you may need to pin an older kernel , instructions in each CVE folder.
- **Network:** host-only / no public exposure. The exploit itself is local-only, but you should never run vulnerable kernels reachable from the internet.
- **Size:** 2 vCPU / 4 GiB RAM / 32 GiB disk is enough for all four CVEs.
- **Snapshot capability:** strongly recommended. Each run should start from a clean baseline; restoring a snapshot is faster than reinstalling.

## What gets measured

`measure.sh` produces a JSON document with the operationally-honest answer:

- **`counters.privesc_detector_fired`** , did the post-exploit `commit_creds` hook fire?
- **`counters.kill_chain_detector_fired`** , did the cross-layer chain detector match?
- **`counters.lsm_blocked_events`** , did the kernel LSM hook deny anything synchronously?
- **`counters.incidents_since_poc`** , total incidents Inner Warden wrote since the exploit started.
- **`first_incident_latency_seconds`** , time from POC start to first relevant incident (lower is better).
- **`suid_target_su_binary.mismatch_means_page_cache_corrupted`** , ground truth that the exploit actually executed.
- **`verdict_hints`** , the four-question rollup that the per-CVE `EXPECTED-DETECTION.md` references.

The `EXPECTED-DETECTION.md` for each CVE has the verdict matrix Inner Warden has historically produced, so you have a reference point.

## How to use this without Inner Warden

The lab is structured so the measurement scripts run against any reasonable Linux security agent , they read `journalctl`, `iptables`, the relevant `/var/lib/<agent>/` artefacts. Currently the queries are wired for Inner Warden's SQLite store, but the structure is intentionally generic so you can fork and adapt for other agents.

If you do, please open an issue or PR with what you learned , comparing detection profiles across agents on the same well-defined CVE is the kind of public artefact this whole project is here to enable.

## Repo conventions

- **One CVE per folder.** `cve-YYYY-NNNNN-<short-name>/`. Lowercase, hyphens, no version drift.
- **Setup scripts are idempotent.** Re-running `setup.sh` on an already-configured VM must be a no-op.
- **No exploit code in the repo.** Every `SOURCES.md` links to the original author / vendor / academic publication. If the original source goes dark, the entry gets marked deprecated, not re-hosted.
- **`EXPECTED-DETECTION.md` is regenerated, not hand-edited.** It is the verbatim narrative of the most recent measured run, with the result JSON inline. If you reproduce and the numbers differ, that's a signal worth filing.
- **License:** Apache-2.0, matching the main Inner Warden repo.

## Related repos

- [`InnerWarden/innerwarden`](https://github.com/InnerWarden/innerwarden) , the agent under test.
- [`InnerWarden/innerwarden-test`](https://github.com/InnerWarden/innerwarden-test) , Docker-based attack lab (functional + chain testing in containers). This repo is the VM-level companion focused on real kernel CVEs.
- Inner Warden docs (configuration, deployment, operations): [the wiki](https://github.com/InnerWarden/innerwarden/wiki).

## Contributing

New CVE walkthrough? Bug in a `setup.sh`? Mismatch between `EXPECTED-DETECTION.md` and what you reproduced? See [`CONTRIBUTING.md`](CONTRIBUTING.md). PRs welcome under the conventions above , most importantly, no exploit code commits and no novel research.

---

## Build this with us

This lab is one half of a larger open project: an autonomous Linux defence agent built in the open, on real CVEs, with the gaps documented honestly. The first CVE in this repo (`copy-fail`) is the one that the agent caught 5 seconds after the exploit fired. The next three slots are open , and the agent will miss things. That's the point.

If you've made it this far, three ways to help:

- **[Star Inner Warden on GitHub](https://github.com/InnerWarden/innerwarden)** , the agent under test. Stars are how this project finds the contributors who care about getting Linux runtime defence right.
- **[Contribute code](https://github.com/InnerWarden/innerwarden/blob/main/CONTRIBUTING.md)** , detectors, collectors, response skills, dashboard surfaces. The codebase is Rust, BUSL-1.1 / Apache-2.0 mixed, and there are [good first issues](https://github.com/InnerWarden/innerwarden/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22) tagged.
- **Reproduce a CVE in this lab and file the result.** Whether the agent caught it or missed it, that data is the whole reason this repo exists. Open an issue with your `result.json` and we'll turn the misses into the next set of detectors.

More context: [innerwarden.com](https://innerwarden.com) , the project, the philosophy, and the roadmap.
