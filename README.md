<br />
<p align="center">
    <img src="assets/logo.png"/>
    <h3 align="center">Orion Bootstrap Wizard</h3>
</p>
A guided, terminal-based installer for Juno Orion. It walks you through a short series of screens, then either installs a lightweight Kubernetes (k3s) on the local machine and deploys Juno onto it, or deploys Juno onto a Kubernetes cluster you already have.

<br/>

> **Beta:** This installer is currently in beta. You may encounter rough edges, and behaviour may change between releases. Please report any problems to support (see the Support section below).

## Before you start

**Choose your install type**

| Install type | What it does | Requirements |
|---|---|---|
| New k3s deployment | Installs k3s on this machine, then deploys Orion | Linux host, must be run as `root` |
| Existing Cluster | Deploys Orion onto a cluster you already run | `kubeconfig` for the cluster, network access to it |

**For an Existing Cluster**

- Your active kubeconfig context must point at the target cluster (`KUBECONFIG` is respected).
- At least one node must carry the Juno label:

  ```
  kubectl label node <NODE NAME> juno-innovations.com/service=true
  ```

- For AWS Marketplace deployments, AWS credentials for the target account must be available in your environment (for example via `AWS_PROFILE` or `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`), and a region must be set (`AWS_REGION`). The credentials need permission to manage IAM roles and policies and to list EKS clusters.
- Make sure the kube context you have active is the cluster you select in the AWS step. The installer checks this and will stop with a clear message if they differ.

**For an Offline (air-gapped) install**

- Have the Genesis and ingress-nginx chart sources available at locations reachable from this machine (an internal chart repo or git repo).
- For a k3s install, have internal registry mirrors ready for `docker.io`, `registry.k8s.io`, `nvcr.io`, `quay.io` and `ghcr.io`.

## Running the installer

Copy the binary to the target machine and run it in a terminal:

```
chmod +x orion-bootstrap-<arch>
./orion-bootstrap-<arch>
```

For a k3s install, run it as root:

```
sudo ./orion-bootstrap-<arch>
```

## Controls

| Key | Action |
|---|---|
| Up / Down | Move between options |
| Space | Select or toggle the highlighted item |
| Enter | Confirm and continue |
| Ctrl+a | Show or hide advanced settings (first screen) |
| Ctrl+n | Go back to the previous screen |
| Ctrl+c | Save your progress and quit |

## Walkthrough

### 1. Deployment method

- Pick **New k3s deployment** or **Existing Cluster** and press Space to select it (a tick appears), then press Enter.
- **Deployment is offline**: tick this for an air-gapped install. An extra screen collects the offline details.
- **Advanced settings** (Ctrl+A): lets you set a custom **Namespace** for Juno. Leave it empty to use the default (`argocd`). Names must be lowercase letters, numbers or `-`, and cannot be a reserved name (the installer lists them if you pick one).
- **Previous config found**: if the installer detects settings from an earlier run, an extra tick box appears at the top. Tick it to delete the old settings and start completely fresh. Otherwise your earlier answers are pre-filled.

### 2. Offline installation details (offline installs only)

Fill in each field and press Enter to move on. Example values are shown in grey and disappear when you start typing.

- **Genesis** and **ingress-nginx**: chart URL, whether the source is a git repo (and if so the chart path inside it), and the chart version.
- **Registry mirrors** (k3s installs): a mirror URL for each of the five registries listed above.
- **Registry username / password** (k3s installs): only needed if your mirrors require authentication. Leave blank otherwise.

Move to **[ Continue ]** at the bottom, press Enter, review the summary, then press Enter to proceed or `ctrl+n` to go back and edit.

### 3a. New k3s deployment

- Confirm you are happy to proceed by checking the box (space) and pressing Enter.
- The installer shows live progress while k3s is installed and Orion is deployed. This can take a while, especially while images are loaded.
- On success the screen shows the address for the Orion Admin setup wizard.

### 3b. Existing Cluster

- **AWS Marketplace**: you are asked whether the deployment is facilitated by AWS Marketplace. Press `Y` or `N`, then Enter.
  - If yes, pick your EKS cluster (Space to select, Enter to confirm). A checklist then shows each AWS setup step (connectivity, license manager roles, IAM roles and service accounts) as it completes. Press any key when it finishes.
  - If no, you go straight to the next step.
- The installer checks your local environment and cluster connectivity, then asks you to confirm.
- It installs [ArgoCD](https://argo-cd.readthedocs.io/en/stable/), then Orion. Progress is shown with a spinner.

## Finishing up

When the install completes you will see:

```
https://<IP/HOSTNAME of your target cluster>/orion/setup/
```

Open that address in a web browser and complete the Orion Admin setup wizard.

## Saved settings

Your answers are saved to:

```
/tmp/juno_bootstrap_chart_values.yaml
```

They are saved when you quit with Ctrl+C or when an install step starts, and reloaded the next time you run the installer. If something goes wrong you can fix the problem and run the installer again to carry on where you left off. This file can contain registry credentials, so treat it as sensitive. Because it lives in `/tmp` it may be removed on reboot.

## Troubleshooting

- **"application is not running as root"**: a k3s install needs root. Re-run with `sudo ./orion-bootstrap-<arch>`.
- **"failed to contact kubernetes"**: set `KUBECONFIG` to a valid kubeconfig and check the cluster is reachable.
- **"failed to locate Juno node label"**: label at least one node as shown in "Before you start".
- **Active kube context does not match the selected cluster**: switch context, for example `aws eks update-kubeconfig --name <cluster-name>`, and run again.
- **Permission denied writing the settings file**: an old `/tmp/juno_bootstrap_chart_values.yaml` owned by another user is in the way. Remove it or run as the same user.
- **Install failed**: the error screen shows the reason and, for k3s installs, the location of the install log. Include it when contacting support.
- **Verbose logging**: run with `DEBUG=yes ./orion-bootstrap-<arch>` to write detailed logs to `debug.log` in the current directory.

## Support

- Support: https://www.juno-innovations.com/contact/support
- Documentation: https://docs.juno-innovations.com/Orion-Documentation/latest/