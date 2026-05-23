# VM requirements

Canonical lab spec. Every CVE recipe assumes this baseline unless its `README.md` says otherwise.

## Mandatory

| Property | Value | Why |
|----------|-------|-----|
| OS | Ubuntu 22.04 LTS Server | The setup scripts use `apt`, `systemd`, journald, and Ubuntu package names. Other Debian-family distros likely work; Rocky/RHEL would need adaptation. |
| Architecture | x86_64 | Most published PoCs are x86_64. ARM64 works for Inner Warden itself but the exploits target x86-specific kernel paths in many cases. |
| Kernel | Vulnerable to the CVE under test (see per-CVE folder) | The whole point. If the kernel is patched, the exploit fails and you measure nothing. |
| Privilege | Root or passwordless sudo from your test user | `setup.sh` installs services, loads kernel modules, writes to `/sys/fs/bpf`. |
| Network | Host-only or fully isolated subnet | The exploits are local-only, but you must never run vulnerable kernels reachable from the internet. |
| Disk | 32 GiB minimum | Inner Warden's working state grows; 32 GiB gives you ~7 days of runtime headroom for repeated reproductions. |
| Memory | 4 GiB minimum | Inner Warden sensor + agent comfortably in 2 GiB; 4 GiB gives headroom for kernel page cache during the exploit. |
| vCPU | 2 minimum | One for the agent's tokio workers, one for the test. More is fine. |

## Recommended

| Property | Value | Why |
|----------|-------|-----|
| Snapshot capability | Native to your hypervisor | Each run should start from a clean baseline. Restoring a snapshot is faster than reinstalling. |
| `apt` cache enabled | Default Ubuntu config | `setup.sh` runs `apt-get install` , a warm cache makes restores fast. |
| `journald` configured for at least 3 days of retention | Default Ubuntu config is fine | `measure.sh` reads journal entries to verify the agent's behaviour. |

## Cloud or local , pick one

This lab is hypervisor-agnostic. Any of the following work; pick whichever matches your habits:

### Cloud VM (recommended for first-time reproduction)

| Provider | Suggested SKU | Approx cost |
|----------|---------------|------------|
| Azure | `Standard_B2s` (2 vCPU / 4 GiB) | ~$30/mo on-demand. Stop the VM between runs. |
| AWS | `t3.medium` (2 vCPU / 4 GiB) | ~$30/mo on-demand. |
| GCP | `e2-medium` (2 vCPU / 4 GiB) | ~$25/mo on-demand. |
| Hetzner | `CX22` (2 vCPU / 4 GiB) | ~€4/mo. Snapshots cost extra; budget for one snapshot per CVE. |
| Linode | `Linode 2GB` (1 vCPU / 2 GiB) | ~$12/mo. Tight on memory; works but may swap during the exploit. |

### Local hypervisor

| Hypervisor | Notes |
|------------|-------|
| QEMU / KVM | Native Linux virtualisation. Best performance. |
| UTM (macOS) | Apple Virtualization framework on Apple Silicon. Snapshot support is partial , confirm before relying on it. |
| VirtualBox | Cross-platform. Slow on Apple Silicon. |
| Multipass (Canonical) | Easiest if you're on Ubuntu/macOS already and just want `multipass launch 22.04`. |
| Vagrant | If you prefer Infra-as-Code. A `Vagrantfile` lives under [`infra/Vagrantfile`](infra/Vagrantfile) (planned). |

## Network isolation , non-negotiable

Run the lab VM with **no public IP** OR with SSH restricted to your own IP only. The exploits are local, so external network access is not required , and exposing a vulnerable-kernel VM to the public internet is irresponsible regardless of who else's CVE you're reproducing.

If you're using a cloud VM, the simplest pattern:

```
# Azure
az network nsg rule create -g <rg> --nsg-name <nsg> -n SSHFromMyIP \
 --priority 100 --source-address-prefixes $(curl -s https://api.ipify.org)/32 \
 --destination-port-ranges 22 --access Allow --protocol Tcp

# AWS
aws ec2 authorize-security-group-ingress --group-id <sg> \
 --protocol tcp --port 22 --cidr $(curl -s https://api.ipify.org)/32

# GCP
gcloud compute firewall-rules create lab-ssh-from-me \
 --direction=INGRESS --action=ALLOW --rules=tcp:22 \
 --source-ranges=$(curl -s https://api.ipify.org)/32
```

## After reproduction , cleanup

Most providers charge for stopped-but-allocated VMs. Run the right teardown for your hypervisor:

```bash
# Azure
az vm deallocate -g <rg> -n <vm> # stop billing for compute, keep disk
az group delete -n <rg> --yes --no-wait # nuke everything

# AWS
aws ec2 stop-instances --instance-ids <id>
aws ec2 terminate-instances --instance-ids <id>

# Local Vagrant
vagrant halt
vagrant destroy -f
```

The repository never assumes the lab VM is permanent , each CVE folder is self-contained and can be re-run on a fresh VM in ~10 minutes.
