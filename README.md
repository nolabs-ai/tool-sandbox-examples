# nono Tool-Sandbox Demo Package

This package contains practical demos for nono command policy, brokered
credentials, endpoint policy, local approval policy, and approval webhooks.

The demos are intentionally close to real tool usage. They run the actual
`gh` and `kubectl` CLIs through nono, use local nono profiles, and keep real
credentials outside the sandboxed child processes.

## Contents

```text
approval-webhook-demo.py       shared local approval webhook used by Kubernetes
github-cli/                    GitHub CLI credential and policy demo
kubernetes/                    kubectl credential and policy demo
```

Each demo directory has its own README with full setup and troubleshooting.

## Prerequisites

Install or build:

```bash
nono --version
python3 --version
jq --version
```

For the scripted terminal demos, install `demonator`:

```bash
cargo install demonator
```

The GitHub CLI demo also needs:

```bash
gh --version
```

The Kubernetes demo also needs:

```bash
kubectl version --client
```

## GitHub CLI Demo

Read the full guide:

```bash
cd github-cli
less README.md
```

Run the macOS scripted demo:

```bash
cd github-cli
demonator -c ghcli-macos-demo.yml
```

Run the Linux scripted demo:

```bash
cd github-cli
demonator -c ghcli-linux-demo.yml
```

The GitHub CLI demo shows:

- phantom `GH_TOKEN` delivery to sandboxed `gh`
- real GitHub token loading by the parent nono proxy
- command approval through nono's local terminal approval policy
- endpoint approval for selected GitHub API routes
- invocation and L7 denial for issue comments and mutations

## Kubernetes Demo

Read the full guide:

```bash
cd kubernetes
less README.md
```

The Kubernetes demo shows:

- sandboxed `kubectl` using an exec credential helper
- phantom token delivery to the child process
- real ServiceAccount token injection by the nono proxy
- endpoint policy for Kubernetes API reads and dangerous mutations
- approval backend calls for selected `kubectl` operations

The Kubernetes demo generates local kubeconfig, CA, and kubectl cache files.
Those files are machine-specific and should not be committed.

## Generated Files

The root `.gitignore` excludes generated Kubernetes demo state:

```text
/kubernetes/staging-ca.pem
/kubernetes/staging-proxy.kubeconfig
/kubernetes/staging.yaml
/kubernetes/.kube/
```

If generated kubectl cache files were already committed, remove them from the
index without deleting local files:

```bash
git rm -r --cached kubernetes/.kube
git add .gitignore
```

Then amend or create a cleanup commit.

## Approval Webhook

`approval-webhook-demo.py` is a small local webhook used by the Kubernetes demo.
It receives nono approval requests and grants only the command, caller, route,
and argument prefixes configured at startup.

The Kubernetes README shows the exact command to run. Keep the webhook running
in a separate terminal while executing Kubernetes commands that require
approval.

## Safety Notes

- Review profiles before running them.
- Do not commit real tokens, kubeconfigs, generated CA files, or kubectl cache.
- Prefer short-lived credentials for demos.
- For production-like profiles, pin executables and helpers in immutable,
  trusted locations instead of writable user-managed paths.
