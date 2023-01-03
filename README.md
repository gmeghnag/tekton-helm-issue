# 

# How to run the reproducer
```
$ oc new-project tekton-helm-issue
$ git clone https://github.com/gmeghnag/tekton-helm-issue
$ helm install tekton-helm-issue ./tekton-helm-issue
$ tkn pipeline start tekton-helm-issue-pipeline
```

# Logs
```
$ tkn pipeline logs
task first-task has failed: "step-build-and-push-image" exited with code 125 (image: "registry.redhat.io/rhel8/buildah@sha256:7678ad61e06e442b0093ab73faa73ce536721ae523015dd942f9196c4699a31d"); for logs run: kubectl -n tekton-helm-issue logs tekton-helm-issue-pipeline-run-djhc5-first-task-pod -c step-build-and-push-image

STEP 1/1: FROM registry.access.redhat.com/ubi8/ubi
Trying to pull registry.access.redhat.com/ubi8/ubi:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob sha256:0e0c4af1097a36c0f18495632a030a0e90cbbcfd056ea724eeecffe27386a1d1
Copying config sha256:b2276c479c6016fc5f3f87899e43a32578781feb6043fade2e132e97a75ba2c9
Writing manifest to image destination
Storing signatures
COMMIT image-registry.openshift-image-registry.svc:5000/tekton-helm-issue/test-image:latest
--> b2276c479c6
Successfully tagged image-registry.openshift-image-registry.svc:5000/tekton-helm-issue/test-image:latest
Successfully tagged registry.access.redhat.com/ubi8/ubi:latest
b2276c479c6016fc5f3f87899e43a32578781feb6043fade2e132e97a75ba2c9
Getting image source signatures
Copying blob sha256:9e80c5c3649ab000c99b6dce8e80d8b9a4566ac9a188fb5df6146235128c8169
error pushing image "image-registry.openshift-image-registry.svc:5000/tekton-helm-issue/test-image:latest" to "docker://image-registry.openshift-image-registry.svc:5000/tekton-helm-issue/test-image:latest": trying to reuse blob sha256:9e80c5c3649ab000c99b6dce8e80d8b9a4566ac9a188fb5df6146235128c8169 at destination: pinging container registry image-registry.openshift-image-registry.svc:5000: Get "https://image-registry.openshift-image-registry.svc:5000/v2/": x509: certificate signed by unknown authority

Tasks Completed: 1 (Failed: 1, Cancelled 0), Skipped: 0
```

# Observation
The issue is happening most probably because of the `.metadata.labels."app.kubernetes.io/managed-by"` label equal to `Helm` while it should be `tekton-pipelines`:
```
oc get pipeline tekton-helm-issue-pipeline -n tekton-helm-issue -o json | jq '.metadata.labels."app.kubernetes.io/managed-by"' 
"Helm"
```

# Additional info
```
$ tkn version                      
Client version: 0.22.0
Pipeline version: v0.37.5
Triggers version: v0.20.2

$ oc version                       
Client Version: 4.8.0-202108312109.p0.git.0d10c3f.assembly.stream-0d10c3f
Server Version: 4.11.21
Kubernetes Version: v1.24.6+5658434

$ oc get csv -n openshift-pipelines
NAME                                     DISPLAY                       VERSION   REPLACES   PHASE
openshift-pipelines-operator-rh.v1.8.2   Red Hat OpenShift Pipelines   1.8.2                Succeeded
```
