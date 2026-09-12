# Security

Reading through [Wiz's writeup on GitHub PAT attacks](https://www.wiz.io/blog/github-attacks-pat-control-plane) was the wake-up call I needed to improve security, below is a overview of some of the fixes/tweaks I made.

## What I changed

**Dropped the PAT.** The KubeDiagrams workflow used `token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}`. The fallback meant a classic PAT was in play, and those aren't repo-scoped, so anything that exfiltrated it got access well beyond this repo. It's `GITHUB_TOKEN` only now, which is scoped to the repo and dies with the run.

**Pinned actions to SHAs.** Tags and branches are mutable, so `@v4` is a promise the action author can break or an attacker can rewrite. The tj-actions/changed-files compromise (CVE-2025-30066) is what that looks like in practice. Every action here is pinned to a commit SHA with the version in a trailing comment, including KubeDiagrams, which has no tagged releases.

**Explicit permissions.** Workflows declare `permissions:` at the top, read-only, with jobs widening it only where they need to write.

**Dependabot** runs weekly against the Actions ecosystem and groups updates into one PR, which keeps pinned SHAs from going stale.

**Gitleaks config** in `.gitleaks.toml`, extending the defaults with rules for GitHub PATs, AWS access keys, and private key headers.

## TODO items

**Kubeconfig in Actions secrets.** `generate-kubediagram.yml` pulls `secrets.KUBECONFIG` to talk to the cluster. It doesn't expire, rotating it means touching the workflow, and anything that reads it gets the cluster. The fix is a short-lived service account token, or OIDC federation so nothing long-lived is stored at all. Mitigating factor for now: the cluster is only reachable over Tailscale, so the kubeconfig alone isn't enough from an arbitrary host.

**Tailscale OAuth credentials** are in Actions secrets for the same reason and have the same problem.

**Gitleaks isn't wired into CI.** The config exists, but no workflow runs it, so right now it only helps if I run it locally. Needs a job on push plus GitHub push protection.

**No branch protection.** Solo repo, so I push to main. Worth turning on required status checks anyway, since CI catches broken kustomizations and I'd rather not rely on remembering.

**Secrets are still out-of-band.** SOPS or External Secrets Operator is on the TODO in the README, and it's the thing that would let cluster secrets live in Git honestly.

## Workflow authorization

Jobs that touch the cluster are gated on my GitHub user ID:

```yaml
if: (github.event.sender.id == '12449626')
```

That works but breaks the moment anyone else is involved, and a hardcoded ID is easy to get wrong. Checking `github.event.sender.association` against OWNER/MEMBER is the better form.

## Reporting something

If you find a problem here, email me rather than opening an issue.
