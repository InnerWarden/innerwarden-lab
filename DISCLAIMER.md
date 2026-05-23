# Disclaimer

This repository exists to validate the [Inner Warden](https://github.com/InnerWarden/innerwarden) security agent's behaviour against **publicly disclosed and patched Linux kernel CVEs**. Read this before cloning, reproducing, or extending.

## What this lab is

A reproducer for known kernel vulnerabilities, on isolated VMs, under operator control. Each CVE entry points at the original public PoC (authored and published by independent security researchers, with disclosure timelines respected by the upstream maintainers) and shows what an autonomous host-based defence agent observes during and after the exploit.

The intended audience is:

- Defenders evaluating whether Inner Warden , or any host-based defence , actually catches what it claims to catch.
- Security engineers calibrating detection rules against a known-good attack baseline.
- Curious technical readers who want to see how a public CVE actually behaves on a real host.

## What this lab is NOT

- **Not novel exploit research.** Every primitive used here was independently discovered, disclosed, and patched by their respective maintainers (Linux kernel team, distribution security teams). This repo contributes nothing new to offensive capability.
- **Not a vehicle for distributing exploit code.** No proof-of-concept source lives in this repo. Each CVE folder's `SOURCES.md` links to the original author's publication. If the original goes dark, the entry is marked deprecated, not re-hosted.
- **Not for running on production systems.** Every recipe assumes an isolated VM with no public exposure and no production data. Running the recipes on shared infrastructure is your responsibility and outside the scope of what this lab is designed for.
- **Not a substitute for patching.** The correct primary mitigation for any of these CVEs is the upstream kernel patch. This lab demonstrates what behavioural defence catches; it does not replace the patch.

## Acceptable use

You may use this lab to:

1. Reproduce a CVE on a personal or organisation-owned isolated VM, where you have explicit authorisation.
2. Compare detection profiles across security agents on the same well-defined attack.
3. Educate yourself or your team about the operational signatures of a class of kernel vulnerabilities.
4. Contribute new CVE walkthroughs (publicly disclosed, patched) following the conventions in [`CONTRIBUTING.md`](CONTRIBUTING.md).

You may NOT use this lab to:

1. Attack any system you do not own or have explicit written authorisation to test against.
2. Distribute or republish the original PoC source code under the cover of this lab.
3. Demonstrate the exploits in contexts that imply Inner Warden is an offensive tool. Inner Warden is a defensive agent; this lab measures what it catches.

## Patches respected

Each CVE folder documents the upstream patch commit (and the relevant distribution security advisories) so a reader can both (a) understand what got fixed and (b) verify their production systems are patched. We deliberately ship recipes that require operators to pin an older, vulnerable kernel , because we are NOT in the business of leaving running systems exposed.

## License

Apache-2.0. See [`LICENSE`](LICENSE).

## Reporting concerns

If you believe this repository contains material that violates a maintainer's disclosure policy, includes inappropriately-licensed content, or otherwise lowers the safety bar of the open security ecosystem, [open an issue](https://github.com/InnerWarden/innerwarden-lab/issues) or email `security@innerwarden.com`. We will reply within seven days.
