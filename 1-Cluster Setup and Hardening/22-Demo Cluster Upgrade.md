> ## Documentation Index
> Fetch the complete documentation index at: https://notes.kodekloud.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Demo Cluster Upgrade

> This lesson demonstrates upgrading a Kubernetes cluster from version 1.28 to 1.29 using kubeadm, following official documentation procedures.

In this lesson, we'll demonstrate upgrading a Kubernetes cluster from version 1.28 to 1.29 using kubeadm. The procedure follows the official Kubernetes documentation under "Tasks → Administer Cluster → Administration with KubeADM → Upgrading a KubeADM Cluster." Although the documentation provides upgrade paths for various version transitions, this demo focuses on moving from v1.28 to the latest v1.29 release.

<Frame>
  ![The image shows a webpage from Kubernetes documentation about upgrading kubeadm clusters, detailing version upgrade paths and linking to additional resources.](https://kodekloud.com/kk-media/image/upload/v1752871356/notes-assets/images/Certified-Kubernetes-Security-Specialist-CKS-Demo-Cluster-Upgrade/frame_50.jpg)
</Frame>

Select the upgrade path that best suits your environment and follow the provided commands. The process remains largely consistent regardless of the version specifics.

***

## Preliminary Steps: Updating Package Repositories

Before starting the upgrade, scroll down the official documentation until you find the important note about changing the package repository. Previously, Kubernetes packages were hosted at `app.kubernetes.io` and `yum.kubernetes.io`, but these repositories have been deprecated. You must now use `packages.k8s.io` to download the latest versions of tools such as kubectl and kubeadm.

<Frame>
  ![The image is a webpage from Kubernetes documentation about changing the package repository, highlighting the deprecation of legacy repositories and recommending new ones.](https://kodekloud.com/kk-media/image/upload/v1752871358/notes-assets/images/Certified-Kubernetes-Security-Specialist-CKS-Demo-Cluster-Upgrade/frame_80.jpg)
</Frame>

<Callout icon="lightbulb" color="#1CB2FE">
  Before proceeding, ensure that you update the package repository configuration on every cluster node.
</Callout>

### Determine Your Operating System Distribution

To see which operating system you are using, run:

```bash  theme={null}
kubectl get nodes
```

On a two-node cluster (one control-plane and one worker node), you can verify your distribution with:

```bash  theme={null}
cat /etc/*release
```

If the output indicates Ubuntu 20.04 or another Debian-based distribution, follow the Debian/Ubuntu instructions in the documentation and update your package repository accordingly.

For instance, initially run a command like this:

```bash  theme={null}
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.23/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core/stable/v1.28/deb/Release.key | sudo apt-key add -
```

To upgrade to v1.29, adjust the version in the command as follows:

```bash  theme={null}
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core/stable/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

After modifying the repository configuration on both control-plane and worker nodes, refresh the package list:

```bash  theme={null}
sudo apt-get update
```

***

## Determining the Target Version

To identify the latest available version in the 1.29 series, use these commands:

```bash  theme={null}
sudo apt update
sudo apt-cache madison kubeadm
```

You should see output resembling:

```plaintext  theme={null}
kubeadm | 1.29.3-1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
kubeadm | 1.29.2-1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
kubeadm | 1.29.1-1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
kubeadm | 1.29.0-1 | https://pkgs.k8s.io/core/stable/v1.29/deb Packages
```

Select the highest version available (for example, 1.29.3-1) and note it for subsequent steps.

***

## Upgrading the Control Plane Node

### Step 1: Upgrade the kubeadm Tool

First, update kubeadm on the control plane node to prepare for the upgrade. Replace the version string with the latest version (e.g., 1.29.3-1.1):

```bash  theme={null}
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm='1.29.3-1.1' && \
sudo apt-mark hold kubeadm
```

Verify the upgrade by checking the version:

```bash  theme={null}
kubeadm version
```

Expected output:

```plaintext  theme={null}
kubeadm version: &version.Info{Major:"1", Minor:"29", GitVersion:"v1.29.3", GitCommit:"6813625b7cd706db5c7388921be03071e14a294", GitTreeState:"clean", BuildDate:"2024-03-15T00:06:16Z", GoVersion:"go1.21.8", Compiler:"gc", Platform:"linux/amd64"}
```

### Step 2: Run the Upgrade Plan

Before applying the upgrade, conduct a dry run to see the available upgrade options. This command will outline which components upgrade automatically and those requiring manual updates (e.g., kubelet):

```bash  theme={null}
sudo kubeadm upgrade plan
```

The output should indicate that your cluster is currently at v1.28.0 with the target control plane version of v1.29.3. Typically, kubeadm upgrades essential components such as the API server, controller manager, scheduler, and kube-proxy automatically, while kubelet must be updated separately.

### Step 3: Apply the Upgrade

Initiate the upgrade process for the control plane:

```bash  theme={null}
sudo kubeadm upgrade apply v1.29.3
```

Monitor the progress as the system renews certificates, updates static pod manifests, and restarts components. After a successful upgrade, you may see messages similar to:

```plaintext  theme={null}
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.29.3". Enjoy!
[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

Keep in mind that when you verify node versions via `kubectl get nodes`, the displayed version corresponds to the kubelet, which will still show v1.28.0 until its upgrade is completed.

***

## Upgrading kubelet and kubectl on the Control Plane Node

### Drain the Control Plane Node

Before upgrading kubelet (which runs outside the control plane pods), drain the control plane node to ensure safe maintenance:

```bash  theme={null}
kubectl drain controlplane --ignore-daemonsets
```

### Upgrade kubelet and kubectl

Next, update kubelet and kubectl on the control plane node by specifying the target version:

```bash  theme={null}
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && \
sudo apt-mark hold kubelet kubectl
```

Restart the kubelet service to apply the changes:

```bash  theme={null}
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Finally, confirm the node version:

```bash  theme={null}
kubectl get nodes
```

Since the control plane node might still be marked as “SchedulingDisabled” due to the drain operation, uncordon it to allow scheduling:

```bash  theme={null}
kubectl uncordon controlplane
```

***

## Upgrading Worker Nodes

Follow a similar process to upgrade each worker node.

### Step 1: Upgrade kubeadm on the Worker Node

On each worker node, upgrade kubeadm with:

```bash  theme={null}
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm='1.29.3-1.1' && \
sudo apt-mark hold kubeadm
```

Then, from a control plane node, initiate the upgrade for the worker node:

```bash  theme={null}
sudo kubeadm upgrade node
```

### Step 2: Drain the Worker Node

Drain the worker node to safely upgrade kubelet. Replace `<node-name>` with the actual name of your worker node:

```bash  theme={null}
kubectl drain <node-name> --ignore-daemonsets
```

If you encounter issues related to DaemonSet-managed pods, ensure that the `--ignore-daemonsets` flag is used correctly. Refer to `kubectl drain --help` for further details if needed.

### Step 3: Upgrade kubelet and kubectl on the Worker Node

Upgrade the kubelet and kubectl packages with:

```bash  theme={null}
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1' && \
sudo apt-mark hold kubelet kubectl
```

Restart the kubelet service:

```bash  theme={null}
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

After the upgrade, uncordon the worker node to resume scheduling:

```bash  theme={null}
kubectl uncordon <node-name>
```

Finally, verify that all nodes are upgraded:

```bash  theme={null}
kubectl get nodes
```

Both control-plane and worker nodes should now display version v1.29.3.

***

## Summary

This guide has detailed the process of upgrading a Kubernetes cluster using kubeadm. The critical steps include:

1. Updating the package repository to `packages.k8s.io`.
2. Determining the target version available from the repository.
3. Upgrading the control plane by updating kubeadm, applying the upgrade, and then updating kubelet and kubectl.
4. Draining nodes before performing kubelet upgrades and uncordoning them afterwards.
5. Repeating the process on each worker node.

By following these steps, you ensure that every component of your Kubernetes cluster is updated properly, maintaining compatibility with the newer version. Happy upgrading!

<CardGroup>
  <Card title="Watch Video" icon="video" cta="Learn more" href="https://learn.kodekloud.com/user/courses/certified-kubernetes-security-specialist-cks/module/eac6dac8-4481-4138-96ef-a2135f20e05e/lesson/28d5ca34-88c2-4e74-b380-cf3667420462" />

  <Card title="Practice Lab" icon="installation" cta="Learn more" href="https://learn.kodekloud.com/user/courses/certified-kubernetes-security-specialist-cks/module/eac6dac8-4481-4138-96ef-a2135f20e05e/lesson/f797f920-c9d5-420c-b2b4-3c8904f42b47" />
</CardGroup>


Built with [Mintlify](https://mintlify.com).