# buildah-unprivileged

Testing possible unprivileged buildah builds

## pipeline

> Steps copied from <https://docs.openshift.com/container-platform/4.12/cicd/pipelines/unprivileged-building-of-container-images-using-buildah.html>

```sh
helm upgrade -i pipeline helm/pipeline -n test --create-namespace
oc apply -f pipelinerun.yaml -n test
```

Failure with retch-repository pod

```text
[tbox@fedora buildah-unprivileged]$ oc get events -n test
LAST SEEN   TYPE      REASON                  OBJECT                                                      MESSAGE
53m         Normal    Scheduled               pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Successfully assigned test/pipelinerun-buildah-as-user-1000-fetch-repository-pod to crc-9ltqk-master-0
53m         Normal    AddedInterface          pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Add eth0 [10.217.0.219/23] from openshift-sdn
53m         Normal    Pulled                  pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Container image "registry.redhat.io/openshift-pipelines/pipelines-entrypoint-rhel8@sha256:3580541d0912cdba214343b79868d2adcfa1c60a6e5ee940c255fa76d4431f07" already present on machine
53m         Normal    Created                 pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Created container prepare
53m         Normal    Started                 pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Started container prepare
3m17s       Normal    Pulled                  pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Container image "registry.redhat.io/ubi8/ubi-minimal@sha256:c7b45019f4db32e536e69e102c4028b66bf5cde173cfff4ffd3281ccf7bb3863" already present on machine
51m         Warning   Failed                  pod/pipelinerun-buildah-as-user-1000-fetch-repository-pod   Error: container has runAsNonRoot and image will run as root (pod: "pipelinerun-buildah-as-user-1000-fetch-repository-pod_test(003ef27e-eeae-4024-8d46-86d939d0d6ad)", container: place-scripts)
53m         Normal    Started                 taskrun/pipelinerun-buildah-as-user-1000-fetch-repository   
53m         Normal    Pending                 taskrun/pipelinerun-buildah-as-user-1000-fetch-repository   Pending
53m         Normal    Pending                 taskrun/pipelinerun-buildah-as-user-1000-fetch-repository   pod status "Initialized":"False"; message: "containers with incomplete status: [prepare place-scripts]"
53m         Normal    Pending                 taskrun/pipelinerun-buildah-as-user-1000-fetch-repository   pod status "Initialized":"False"; message: "containers with incomplete status: [place-scripts]"
```

## standalone container

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


