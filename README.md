# Hello world on Argo CD

A one-line Python script running as a Kubernetes Job. No custom image build,
dependencies, Helm chart, Service, or ingress required. It prints `hello world!`
once and exits successfully. A completed Pod is expected, not a running server.

## Files

- `hello.py`: the script.
- `job.yaml`: runs the script using the public `python:3.13-alpine` image.
- `kustomization.yaml`: packages the script in a ConfigMap and includes the Job.
- `argocd/application.yaml`: template to paste into Argo CD's application editor.

## Deploy using only web UIs

Argo CD cannot upload/run a local Python file or execute it inside its backend.
It reads Kubernetes manifests from Git and deploys them to a connected cluster.
The Python process runs in that cluster, not in the Argo CD server.

1. Upload these files to a Git repository, keeping their paths. You can use your
   Git provider's web UI; no local Git, Docker, or Kubernetes CLI is required.
   The example assumes the files are at the repository root on branch `main`.
2. In Argo CD, open **Settings > Repositories > Connect Repo** if needed,
   especially for a private repository. Configure credentials there, not in YAML.
3. Open **Applications > New App > Edit as YAML** and paste
   `argocd/application.yaml`. Replace:
   - `spec.source.repoURL` with your actual Git repository URL.
   - `spec.source.targetRevision` if your branch is not `main`.
   - `metadata.namespace` if Argo CD is installed outside `argocd`.
   - `spec.project` if your installation requires a project other than `default`.
   - `spec.destination.server` with the API server URL of the connected target
     cluster, as listed under **Settings > Clusters**. The supplied value selects
     the cluster hosting Argo CD, not automatically another connected cluster.
   - `spec.destination.namespace` with an existing namespace you may deploy into.
4. Click **Create**, then **Sync > Synchronize**.
5. In the application's resource tree, select the Pod under the `hello-world`
   Job and open **Logs**. Expected output:

   ```text
   hello world!
   ```

Alternatively, fill out **New App** with the same values: application name
`hello-world`, project `default`, manual sync, your repository URL, revision
`main`, path `.`, target cluster, and namespace `default`. Argo CD detects
Kustomize automatically.

Your Argo CD account/project must allow application creation, syncing, and
Job/ConfigMap deployment to the selected destination. The cluster must be able
to pull the Python image from Docker Hub. Organization-specific policies may
require an approved image, security settings, or different resource limits.

## Run again or update the script

Jobs run once. Syncing an unchanged completed Job does not rerun it. To rerun,
delete only the `hello-world` **Job** from the Argo CD resource tree, then sync
again. Deleting the Job also deletes its Pod and logs, so read the logs first.
Keep the default manual sync policy for this workflow.

After changing `hello.py` in Git, refresh the application, delete the old Job,
and sync again. Kubernetes does not allow changing an existing Job's Pod
template; Kustomize updates the ConfigMap reference when the script changes.

## Local smoke test (optional)

```sh
python3 hello.py
```
