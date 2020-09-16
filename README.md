# opa-tekton-sar-demo

An initial prototype of using [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper) Rego based constructs 
to invoke Kubernetes Subject Access Review based authorization checks around [Tekton Pipeline](https://github.com/tektoncd/pipelines)
`TaskRun` references to `ClusterTask` objects.

The example Gateway yaml results in Gatekeeper employing:

- Watches on `TaskRuns` that meet certain matching constraints
- Use of Gatekeeper `Templates` that act as the implementation of a Kubernetes admission webhook
- An initial, though by no means complete, set of parameterization around the inputs into the Subject Access Review
request 

A simple Tekton `ClusterTask` and `TaskRun` is provided, as well as a `ClusterRoleBinding`, that utilizes 
an existing `ClusterRole` defined by Tekton Pipelines that control access to `ClusterTasks`, to declare that 
the `default` `ServiceAccount` in our samples's namespace has access to `ClusterTasks`.

To run this POC / Demo from with OpenShift Container Platform, the non-variable setup is:

- Deploy Tekton Pipelines and OPA Gatekeeper (this repo does not proscribe how you do that, as there are varying
approaches for each)
- run `oc new-project ggmtest`
- run `oc apply -f ./201-clusterrolebinding.yaml`
- run `oc apply -f ./template-sar.yaml`
- run `oc apply -f ./constraint-sar.yaml`
- run `oc apply -f ./simple-tekton-task.yaml`

Now, for actually creating the `TaskRun`, if the `simple-tekton-tasrun.yaml` file uses `default` for the `serviceAccountName`
the `TaskRun` will be allowed to be created.

If you switch to `builder` for `serviceAccountName` you'll get on `TaskRun` creation the a message like:

```bash
$ oc apply -f ../simple-tekton-taskrun.yaml 
Error from server ([denied by taskrun-serviceaccount-allowed-clustertask] ServiceAccount builder user system:serviceaccount:ggmtest:builder is not allowed to use ClusterTask {"kind": "ClusterTask", "name": "echo-hello-world"}, TaskRun: echo-hello-world-task-run): error when creating "../simple-tekton-taskrun.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by taskrun-serviceaccount-allowed-clustertask] ServiceAccount builder user system:serviceaccount:ggmtest:builder is not allowed to use ClusterTask {"kind": "ClusterTask", "name": "echo-hello-world"}, TaskRun: echo-hello-world-task-run
```