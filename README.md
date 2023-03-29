# buildah-unprivileged

Testing possible unprivileged buildah builds

```sh
helm upgrade -i test helm/buildah-unprivileged -n test --create-namespace
```

logs...

```sh
[tbox@fedora buildah-unprivileged]$ oc logs deploy/test-buildah-unprivileged -n test
time="2023-03-29T23:33:29Z" level=warning msg="Error running newgidmap: exit status 1: newgidmap: write to gid_map failed: Operation not permitted\n"
time="2023-03-29T23:33:29Z" level=warning msg="Falling back to single mapping"
time="2023-03-29T23:33:29Z" level=warning msg="Error running newuidmap: exit status 1: newuidmap: write to uid_map failed: Operation not permitted\n"
time="2023-03-29T23:33:29Z" level=warning msg="Falling back to single mapping"
STEP 1/2: FROM registry.access.redhat.com/ubi9
Trying to pull registry.access.redhat.com/ubi9:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob sha256:2a625e4afab51b49edb0e5f4ff37d8afbb20ec644ed1e68641358a6305557de3
Copying blob sha256:2a625e4afab51b49edb0e5f4ff37d8afbb20ec644ed1e68641358a6305557de3
time="2023-03-29T23:33:37Z" level=error msg="While applying layer: ApplyLayer exit status 1 stdout:  stderr: potentially insufficient UIDs or GIDs available in user namespace (requested 0:5 for /usr/bin/write): Check /etc/subuid and /etc/subgid if configured locally: lchown /usr/bin/write: invalid argument"
error creating build container: writing blob: adding layer with blob "sha256:2a625e4afab51b49edb0e5f4ff37d8afbb20ec644ed1e68641358a6305557de3": ApplyLayer exit status 1 stdout:  stderr: potentially insufficient UIDs or GIDs available in user namespace (requested 0:5 for /usr/bin/write): Check /etc/subuid and /etc/subgid if configured locally: lchown /usr/bin/write: invalid argument
time="2023-03-29T23:33:37Z" level=error msg="exit status 125"
```

However if you deploy as privileged by changing the scc in the ClusterRole it works...

```sh
helm upgrade -i test helm/buildah-unprivileged --set privileged="true" -n test --create-namespace
```

restart the pod...

```sh
oc rollout restart deploy/test-buildah-unprivileged -n test
```

logs (working)...

```sh
[tbox@fedora buildah-unprivileged]$ oc logs deploy/test-buildah-unprivileged -n test
STEP 1/2: FROM registry.access.redhat.com/ubi9
Trying to pull registry.access.redhat.com/ubi9:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob sha256:2a625e4afab51b49edb0e5f4ff37d8afbb20ec644ed1e68641358a6305557de3
Copying blob sha256:2a625e4afab51b49edb0e5f4ff37d8afbb20ec644ed1e68641358a6305557de3
Copying config sha256:9877f06ecc6f0d76ab8bfba1495a6dd1fd00aa3df13b1c1434c9ae65443f0feb
Writing manifest to image destination
Storing signatures
STEP 2/2: RUN dnf install bind-utils -y
Updating Subscription Management repositories.
Unable to read consumer identity
Subscription Manager is operating in container mode.
...
```
