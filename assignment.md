# Kubectl Debug Commands 

This page describes some useful `kubectl` commands to debug your Kubernetes pods and containers.

## Kubectl Get Pods

To list information about the status of all available pods,
use the `kubectl get pods` command.

By default, `kubectl get pods` lists all pods in the current namespace.
To list pods for a different namespace, use the `--namespace` flag.

For example, this command lists all pods in the `kube-system` namespace:

```shell
kubectl get pods --namespace kube-system
```

The command returns a table in the following format:

```tsv
NAME                                      READY   STATUS             RESTARTS        AGE
calico-kube-controllers-7bb4b4d4d-8q2r8   0/1     CrashLoopBackOff   6 (30s ago)     22d
canal-rgkd5                               2/2     Running            2 (72m ago)     22d
```

## Kubectl Logs

To print the logs for a container in a pod, use the [`kubectl logs`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/) command.

If the pod has multiple containers, specify the container with the `--container` flag.
If the pod has only one container, this flag is optional.

For example, this command prints the logs for the container in the `etcd-controlplane` pod:

```shell
kubectl logs etcd-controlplane
```

The preceding command prints logs for the `ectd-controlplane` pod in the following structure:

```json
{"level":"info","ts":"2025-11-11T00:34:21.083521Z","caller":"mvcc/hash.go:157","msg":"storing new hash","hash":1608398330,"revision":6809,"compact-revision":6419}
{"level":"warn","ts":"2025-11-11T00:41:37.990443Z","caller":"embed/config_logging.go:188","msg":"rejected connection on client endpoint","remote-addr":"127.0.0.1:42944","server-name":"","error":"EOF"}

```

Note that the logs have different severity levels.
When debugging, pay special attention to `warn` and `error` level logs. 

## Kubectl Exec

To execute commands in a container, use the [`kubectl exec`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/) command.

When you run `kubectl exec`, separate the `kubectl` options from the shell command with two dashes, `--`.
If you don't specify a container, `kubectl` runs the command in the pod's first container.

For example, this command runs `ps aux` to list the processes running in the first container of the `canal` pod:

```shell
kubectl exec canal -- ps aux
```

If the container does not have the specified executable, the `kubectl exec` command fails.
For example, if the `canal` pod does not have the `ps` command, the preceding command returns an error such as the following:

```tsv
exec failed: unable to start container process: exec: "ps": executable file not found in $PATH: unknown
```

In these cases, you can create a debugging container using [`kubectl debug`](#kubectl-debug).

## Kubectl Debug

To create interactive debugging containers, use the [`kubectl debug`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/) command.
With this command, you can clone pods to experiment with changes or create ephemeral containers in the pods that are already running.

When you clone a pod, use  `--copy-to` flag to specify a destination pod. Use the `--image` flag to specify the image to use for the debug container.

For example, this command clones the `canal` pod to `canal-debugger` and opens an interactive shell:


```shell
kubectl debug canal  -it --image=busybox --copy-to=canal-debugger
```

On success, the output prints something like this before starting the interactive shell:

```shell
Defaulting debug container name to debugger-z8f4x.
```

You can also use `kubectl` to create ephemeral containers in existing pods.
For example, this command opens a debug shell in the `canal` pod:

```shell
kubectl debug canal -it --image=busybox
```


