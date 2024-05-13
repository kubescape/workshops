# Making Kubernetes Goat great again

This workshop is designed to help you understand how to secure a Kubernetes cluster by using Kubescape.

## Prerequisites

Make sure you have the following tools installed:
- git https://git-scm.com/downloads
- kubectl https://kubernetes.io/docs/tasks/tools/install-kubectl/
- minikube https://minikube.sigs.k8s.io/docs/start/
- helm https://helm.sh/docs/intro/install

### Starting a Kubernetes cluster

```shell
minikube start
```

### Setting up Kubernetes Goat

> [!TIP]
> If you’re running Windows please use Git Bash.

```shell
git clone https://github.com/madhuakula/kubernetes-goat.git
cd kubernetes-goat
chmod +x setup-kubernetes-goat.sh
bash setup-kubernetes-goat.sh
```

### Ensuring pods are running

```shell
kubectl get pods
```

```console
NAME                                               READY   STATUS    RESTARTS   AGE
batch-check-job-l8ht4                              1/1     Running   0          93s
build-code-deployment-6ff7b98f7c-jscbl             1/1     Running   0          93s
health-check-deployment-65d6ff7776-wxtkb           1/1     Running   0          92s
hidden-in-layers-wvzn6                             1/1     Running   0          90s
internal-proxy-deployment-646b4cfcd7-ndp5j         2/2     Running   0          91s
kubernetes-goat-home-deployment-7f8486f6c7-k8xcx   1/1     Running   0          91s
metadata-db-7fbf595cc5-8mflw                       1/1     Running   0          93s
poor-registry-deployment-877b55d89-lnhjs           1/1     Running   0          91s
system-monitor-deployment-5466d8b787-676jn         1/1     Running   0          91s
```

### Accessing Kubernetes Goat

```shell
bash access-kubernetes-goat.sh
```

```console
kubectl setup looks good.
Creating port forward for all the Kubernetes Goat resources to locally. We will be using 1230 to 1236 ports locally!
Visit http://127.0.0.1:1234 to get started with your Kubernetes Goat hacking!
```

You can now navigate to http://127.0.0.1:1234/ to access Kubernetes Goat.

### Running your first exploit

Let’s run the “Container escape to the host system” exploit, go to http://127.0.0.1:1233/ and enter the following commands:

```shell
mount
```

```console
...
sysfs on /host-system/sys type sysfs (ro,nosuid,nodev,noexec,relatime)
...
```

We can see that the sysfs is mounted on the host system. Let’s try to access the host system:

```shell
ls /host-system/
```

```console
CHANGELOG  Release.key  bin  boot  data  dev  docker.key  etc  home  kic.txt  kind  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  version.json
```

If this works we can try an escape to the host system:

```shell
chroot /host-system bash
docker images
ps aux
```

```console
root@system-monitor-deployment-5466d8b787-kscdh:/# chroot /host-system bash
root@system-monitor-deployment-5466d8b787-kscdh:/# docker images
REPOSITORY                                           TAG       IMAGE ID       CREATED         SIZE
nginx                                                1.25.5    2ac752d7aeb1   5 days ago      188MB
madhuakula/k8s-goat-info-app                         latest    57e24cd7eb27   4 months ago    66.3MB
madhuakula/k8s-goat-hunger-check                     latest    8a648053c00f   4 months ago    142MB
madhuakula/k8s-goat-hidden-in-layers                 latest    285cbdc185ff   4 months ago    7.34MB
madhuakula/k8s-goat-health-check                     latest    550e0029f6bd   4 months ago    952MB
madhuakula/k8s-goat-home                             latest    be8461449c23   4 months ago    43.4MB
madhuakula/k8s-goat-cache-store                      latest    aa2bf2205b2c   4 months ago    27.4MB
madhuakula/k8s-goat-batch-check                      latest    a79437e72bc1   4 months ago    148MB
madhuakula/k8s-goat-system-monitor                   latest    194bbb7b931a   4 months ago    138MB
...
root@system-monitor-deployment-5466d8b787-kscdh:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.5  0.0  22224 14592 ?        Ss   15:31   0:02 /sbin/init
root         108  0.0  0.0  23704 11520 ?        S<s  15:31   0:00 /lib/systemd/systemd-journald
root         161  0.0  0.0  15420  8960 ?        Ss   15:31   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root         600  0.6  0.4 1654636 67068 ?       Ssl  15:31   0:03 /usr/bin/containerd
...
```

Escape has worked successfully, you’re root on the host system!
Time to fix that...

## Installing Kubescape

Time to go back to your bash:

```shell
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
```

After the installation, a first scan is triggered automatically, you will see how NOT great the GOAT is!

```console
 ✅  Initialized scanner
 ✅  Loaded policies
 ✅  Loaded exceptions
 ✅  Loaded account configurations
 ✅  Accessed Kubernetes objects
 ✅  Collected RBAC resources
Control: C-0057 100% |█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (47/47, 37 it/s)        
 ✅  Done scanning. Cluster: minikube
 ✅  Done aggregating results
...
Control plane
┌────┬─────────────────────────────────────┬────────────────────────────────────┐
│    │ Control name                        │ Docs                               │
├────┼─────────────────────────────────────┼────────────────────────────────────┤
│ ✅ │ API server insecure port is enabled │ https://hub.armosec.io/docs/c-0005 │
│ ❌ │ Anonymous access enabled            │ https://hub.armosec.io/docs/c-0262 │
│ ❌ │ Audit logs enabled                  │ https://hub.armosec.io/docs/c-0067 │
│ ✅ │ RBAC enabled                        │ https://hub.armosec.io/docs/c-0088 │
│ ❌ │ Secret/etcd encryption enabled      │ https://hub.armosec.io/docs/c-0066 │
└────┴─────────────────────────────────────┴────────────────────────────────────┘

Access control
┌────────────────────────────────────────────────────┬───────────┬────────────────────────────────────┐
│ Control name                                       │ Resources │ View details                       │
├────────────────────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Administrative Roles                               │     0     │ $ kubescape scan control C-0035 -v │
│ List Kubernetes secrets                            │     0     │ $ kubescape scan control C-0015 -v │
│ Minimize access to create pods                     │     0     │ $ kubescape scan control C-0188 -v │
│ Minimize wildcard use in Roles and ClusterRoles    │     0     │ $ kubescape scan control C-0187 -v │
│ Portforwarding privileges                          │     0     │ $ kubescape scan control C-0063 -v │
│ Prevent containers from allowing command execution │     0     │ $ kubescape scan control C-0002 -v │
│ Roles with delete capabilities                     │     0     │ $ kubescape scan control C-0007 -v │
│ Validate admission controller (mutating)           │     0     │ $ kubescape scan control C-0039 -v │
│ Validate admission controller (validating)         │     0     │ $ kubescape scan control C-0036 -v │
└────────────────────────────────────────────────────┴───────────┴────────────────────────────────────┘

Secrets
┌─────────────────────────────────────────────────┬───────────┬────────────────────────────────────┐
│ Control name                                    │ Resources │ View details                       │
├─────────────────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Applications credentials in configuration files │     0     │ $ kubescape scan control C-0012 -v │
└─────────────────────────────────────────────────┴───────────┴────────────────────────────────────┘

Network
┌────────────────────────┬───────────┬────────────────────────────────────┐
│ Control name           │ Resources │ View details                       │
├────────────────────────┼───────────┼────────────────────────────────────┤
│ Missing network policy │    13     │ $ kubescape scan control C-0260 -v │
└────────────────────────┴───────────┴────────────────────────────────────┘

Workload
┌─────────────────────────┬───────────┬────────────────────────────────────┐
│ Control name            │ Resources │ View details                       │
├─────────────────────────┼───────────┼────────────────────────────────────┤
│ Host PID/IPC privileges │     1     │ $ kubescape scan control C-0038 -v │
│ HostNetwork access      │     0     │ $ kubescape scan control C-0041 -v │
│ HostPath mount          │     2     │ $ kubescape scan control C-0048 -v │
│ Non-root containers     │    12     │ $ kubescape scan control C-0013 -v │
│ Privileged container    │     2     │ $ kubescape scan control C-0057 -v │
└─────────────────────────┴───────────┴────────────────────────────────────┘


...
Compliance Score
────────────────

The compliance score is calculated by multiplying control failures by the number of failures against supported compliance frameworks. Remediate controls, or configure your cluster baseline with exceptions, to improve this score.

* MITRE: 77.91%
* NSA: 61.96%
```

This is a very poor score. The first scan gives a high-level overview of the cluster posture and suggests follow-up scans to dig deeper.

Let’s run one of them:

```shell
kubescape scan control C-0038 -v
```

```console
...
################################################################################
ApiVersion: apps/v1
Kind: Deployment
Name: system-monitor-deployment
Namespace: default

Controls: 1 (Failed: 1, action required: 0)

┌──────────┬─────────────────────────┬────────────────────────────────────┬────────────────────────────┐
│ Severity │ Control name            │ Docs                               │ Assisted remediation       │
├──────────┼─────────────────────────┼────────────────────────────────────┼────────────────────────────┤
│ High     │ Host PID/IPC privileges │ https://hub.armosec.io/docs/c-0038 │ spec.template.spec.hostIPC │
│          │                         │                                    │ spec.template.spec.hostPID │
└──────────┴─────────────────────────┴────────────────────────────────────┴────────────────────────────┘
```

Now we can start applying the suggested fix:

```shell
kubectl edit deployment system-monitor-deployment
```

The goal is to fix all the issues and get a 100% compliance score.
