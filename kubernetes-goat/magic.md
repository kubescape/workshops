# Magic fix everything

Make sure you’re on the root folder of your cloned repo and run a local scan in JSON output:

```shell
kubescape scan . --format json --output output.json
```

```console
 ✅  Initialized scanner
 ✅  Loaded policies
 ✅  Loaded exceptions
 ✅  Loaded account configurations
 ✅  Done accessing local objects
Control: C-0061 100% |███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (50/50, 4 it/s)         
 ✅  Done scanning GitLocal
 ✅  Done aggregating results


Security posture overview for repo: '.'

Workload
┌─────────────────────────┬───────────┬──────────────────────────────────────┐
│ Control name            │ Resources │ View details                         │
├─────────────────────────┼───────────┼──────────────────────────────────────┤
│ Host PID/IPC privileges │     4     │ $ kubescape scan control C-0038 . -v │
│ HostNetwork access      │     1     │ $ kubescape scan control C-0041 . -v │
│ HostPath mount          │     6     │ $ kubescape scan control C-0048 . -v │
│ Non-root containers     │    16     │ $ kubescape scan control C-0013 . -v │
│ Privileged container    │     4     │ $ kubescape scan control C-0057 . -v │
└─────────────────────────┴───────────┴──────────────────────────────────────┘


Highest-stake workloads
───────────────────────

High-stakes workloads are defined as those which Kubescape estimates would have the highest impact if they were to be exploited.

1. name: kube-bench-node, kind: Job
   $ kubescape scan workload Job/kube-bench-node --file-path=/tmp/kubernetes-goat/scenarios/kube-bench-security/node-job.yaml
2. name: kube-bench-master, kind: Job
   $ kubescape scan workload Job/kube-bench-master --file-path=/tmp/kubernetes-goat/scenarios/kube-bench-security/master-job.yaml
3. name: batch-check-job, kind: Job
   $ kubescape scan workload Job/batch-check-job --file-path=/tmp/kubernetes-goat/scenarios/batch-check/job.yaml


What now?
─────────

* Run one of the suggested commands to learn more about a failed control failure
* Run a cluster scan: '$ kubescape scan'
* Scan a workload with '$ kubescape scan workload' to see vulnerability information
* Install Kubescape in your cluster for continuous monitoring and a full vulnerability report: https://kubescape.io/docs/install-operator/

 ✅  Scan results saved. filename: output.json

Overall compliance-score (100- Excellent, 0- All failed): 62
```

You have got a pretty big JSON file:

```shell
ls -lh output.json
```

```console
-rw-r--r-- 1 mbertschy mbertschy 346K Apr 22 18:01 output.json
```

Now run the magic command:

```shell
kubescape fix output.json
```

```console
 ℹ️   Reading report file...
 ℹ️   The following changes will be applied:
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/hunger-check/deployment.yaml
Resource: hunger-check-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/kube-bench-security/master-job.yaml
Resource: kube-bench-master
Kind: Job
Changes:
	1) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	2) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	3) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	4) spec.template.spec.containers[0].securityContext.privileged = false
	5) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/health-check/deployment.yaml
Resource: health-check-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.containers[0].volumeMounts[0].readOnly = true
	5) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	6) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/poor-registry/deployment.yaml
Resource: poor-registry-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	2) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	3) spec.template.spec.containers[0].securityContext.privileged = false
	4) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	5) spec.template.spec.containers[0].securityContext.runAsNonRoot = true

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/kubernetes-goat-home/deployment.yaml
Resource: kubernetes-goat-home-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	2) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	3) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	4) spec.template.spec.containers[0].securityContext.privileged = false
	5) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/kube-bench-security/node-job.yaml
Resource: kube-bench-node
Kind: Job
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/docker-bench-security/deployment.yaml
Resource: docker-bench-security
Kind: DaemonSet
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.securityContext.runAsUser = 1000
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/batch-check/job.yaml
Resource: batch-check-job
Kind: Job
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/hidden-in-layers/deployment.yaml
Resource: hidden-in-layers
Kind: Job
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/build-code/deployment.yaml
Resource: build-code-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[0].securityContext.privileged = false
	3) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/hunger-check/deployment.yaml
Resource: big-monolith-sa
Kind: ServiceAccount
Changes:
	1) automountServiceAccountToken = false

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/cache-store/deployment.yaml
Resource: cache-store-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	2) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	3) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	4) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	5) spec.template.spec.containers[0].securityContext.privileged = false

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/system-monitor/deployment.yaml
Resource: system-monitor-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.privileged = false
	2) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	3) spec.template.spec.containers[0].volumeMounts[0].readOnly = true
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	6) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/health-check/deployment-kind.yaml
Resource: health-check-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	2) spec.template.spec.containers[0].volumeMounts[0].readOnly = true
	3) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	4) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	5) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	6) spec.template.spec.containers[0].securityContext.privileged = false

------
File: /home/mbertschy/github.com/madhuakula/kubernetes-goat/scenarios/internal-proxy/deployment.yaml
Resource: internal-proxy-deployment
Kind: Deployment
Changes:
	1) spec.template.spec.containers[0].securityContext.allowPrivilegeEscalation = false
	2) spec.template.spec.containers[1].securityContext.readOnlyRootFilesystem = true
	3) spec.template.spec.containers[1].securityContext.runAsGroup = 1000
	4) spec.template.spec.containers[0].securityContext.runAsNonRoot = true
	5) spec.template.spec.containers[0].securityContext.runAsGroup = 1000
	6) spec.template.spec.containers[1].securityContext.allowPrivilegeEscalation = false
	7) spec.template.spec.containers[0].securityContext.privileged = false
	8) spec.template.spec.containers[0].securityContext.readOnlyRootFilesystem = true
	9) spec.template.spec.containers[1].securityContext.runAsNonRoot = true
	10) spec.template.spec.containers[1].securityContext.privileged = false

------

Would you like to apply the changes to the files above? [y|n]: 
y
 ℹ️   Fixed resources in 14 files.
```

You can see several files have been modified. You can now check the changes in the files:

```shell
git status
```

```console
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   scenarios/batch-check/job.yaml
        modified:   scenarios/build-code/deployment.yaml
        modified:   scenarios/cache-store/deployment.yaml
        modified:   scenarios/docker-bench-security/deployment.yaml
        modified:   scenarios/health-check/deployment-kind.yaml
        modified:   scenarios/health-check/deployment.yaml
        modified:   scenarios/hidden-in-layers/deployment.yaml
        modified:   scenarios/hunger-check/deployment.yaml
        modified:   scenarios/internal-proxy/deployment.yaml
        modified:   scenarios/kube-bench-security/master-job.yaml
        modified:   scenarios/kube-bench-security/node-job.yaml
        modified:   scenarios/kubernetes-goat-home/deployment.yaml
        modified:   scenarios/poor-registry/deployment.yaml
        modified:   scenarios/system-monitor/deployment.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        output.json

no changes added to commit (use "git add" and/or "git commit -a")
```

We can now run a new scan to see if the changes have been applied correctly:

```shell
kubescape scan .
```

```console
 ✅  Initialized scanner
 ✅  Loaded policies
 ✅  Loaded exceptions
 ✅  Loaded account configurations
 ✅  Done accessing local objects
Control: C-0049 100% |███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| (50/50, 4 it/s)         
 ✅  Done scanning GitLocal
 ✅  Done aggregating results


Security posture overview for repo: '.'

Workload
┌─────────────────────────┬───────────┬──────────────────────────────────────┐
│ Control name            │ Resources │ View details                         │
├─────────────────────────┼───────────┼──────────────────────────────────────┤
│ Host PID/IPC privileges │     2     │ $ kubescape scan control C-0038 . -v │
│ HostPath mount          │     2     │ $ kubescape scan control C-0048 . -v │
│ Non-root containers     │     2     │ $ kubescape scan control C-0013 . -v │
└─────────────────────────┴───────────┴──────────────────────────────────────┘


Highest-stake workloads
───────────────────────

High-stakes workloads are defined as those which Kubescape estimates would have the highest impact if they were to be exploited.

1. name: kube-bench-node, kind: Job
   $ kubescape scan workload Job/kube-bench-node --file-path=/tmp/kubernetes-goat/scenarios/kube-bench-security/node-job.yaml
2. name: kube-bench-master, kind: Job
   $ kubescape scan workload Job/kube-bench-master --file-path=/tmp/kubernetes-goat/scenarios/kube-bench-security/master-job.yaml
3. name: -metadata-db-test-connection, kind: Pod
   $ kubescape scan workload Pod/-metadata-db-test-connection --chart-path=/tmp/kubernetes-goat/scenarios/metadata-db --file-path=/tmp/kubernetes-goat/scenarios/metadata-db/templates/tests/test-connection.yaml
```

Almost all controls have been fixed, only few are left, and this is because we're lacking runtime information for these:
- C-0038: Host PID/IPC privileges
- C-0048: HostPath mount

Please also note that templates are not modified, only the actual resources are. This is why the `metadata-db` scenario is still failing.
