# ACS has different EnforcementActions

we can see there are 8 different types of enforcements.

```bash
enum EnforcementAction {
  UNSET_ENFORCEMENT = 0;
  SCALE_TO_ZERO_ENFORCEMENT = 1;
  UNSATISFIABLE_NODE_CONSTRAINT_ENFORCEMENT = 2;
  KILL_POD_ENFORCEMENT = 3;
  FAIL_BUILD_ENFORCEMENT = 4;
  // FAIL_KUBE_REQUEST_ENFORCEMENT takes effect only if admission control webhook is enabled to listen on exec and port-forward events.
  FAIL_KUBE_REQUEST_ENFORCEMENT = 5;
  // FAIL_DEPLOYMENT_CREATE_ENFORCEMENT takes effect only if admission control webhook is configured to enforce on object creates.
  FAIL_DEPLOYMENT_CREATE_ENFORCEMENT = 6;
  // FAIL_DEPLOYMENT_UPDATE_ENFORCEMENT takes effect only if admission control webhook is configured to enforce on object updates.
  FAIL_DEPLOYMENT_UPDATE_ENFORCEMENT = 7;
}
```

We can see that while these stages may be self informing lets check the code:

```bash
var enforcementToLifecycle = map[storage.EnforcementAction]storage.LifecycleStage{
	storage.EnforcementAction_FAIL_BUILD_ENFORCEMENT:                    storage.LifecycleStage_BUILD,
	storage.EnforcementAction_SCALE_TO_ZERO_ENFORCEMENT:                 storage.LifecycleStage_DEPLOY,
	storage.EnforcementAction_UNSATISFIABLE_NODE_CONSTRAINT_ENFORCEMENT: storage.LifecycleStage_DEPLOY,
	storage.EnforcementAction_KILL_POD_ENFORCEMENT:                      storage.LifecycleStage_RUNTIME,
	storage.EnforcementAction_FAIL_KUBE_REQUEST_ENFORCEMENT:             storage.LifecycleStage_RUNTIME,
}
```

With the following code we get the clue we need on how some of these DEPLOYMENT enforcment actions should behave:

```bash
		// Update enforcement for deploy time policy enforcements.
		if op == admission.Create {
			alert.GetDeployment().Inactive = true
			alert.Enforcement = &storage.Alert_Enforcement{
				Action:  storage.EnforcementAction_FAIL_DEPLOYMENT_CREATE_ENFORCEMENT,
				Message: "Failed deployment create in response to this policy violation.",
			}
		} else if op == admission.Update {
			alert.Enforcement = &storage.Alert_Enforcement{
				Action:  storage.EnforcementAction_FAIL_DEPLOYMENT_UPDATE_ENFORCEMENT,
				Message: "Failed deployment update in response to this policy violation.",
			}
		}
```

We can tell from here that the yaml it is looking at is Deployments resource as it changes but there are some things to note, when you create an admissionwebhook you attach something akin to a filter to determine when the admissionwebhook is called.  Here is the configuration for stackrox:

```bash
oc get validatingwebhookconfiguration -n stackrox
```

On my cluster it is called `stackrox` in the `stackrox` namespace...

```bash
oc get validatingwebhookconfiguration stackrox -o yaml
```

```bash
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  annotations:
    email: support@stackrox.com
    meta.helm.sh/release-name: stackrox-secured-cluster-services
    meta.helm.sh/release-namespace: stackrox
    operator-sdk/primary-resource: stackrox/stackrox-secured-cluster-services
    operator-sdk/primary-resource-type: SecuredCluster.platform.stackrox.io
    owner: stackrox
  creationTimestamp: "2025-02-28T00:35:31Z"
  generation: 1
  labels:
    app.kubernetes.io/component: admission-control
    app.kubernetes.io/instance: stackrox-secured-cluster-services
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: stackrox
    app.kubernetes.io/part-of: stackrox-secured-cluster-services
    app.kubernetes.io/version: 4.6.2
    app.stackrox.io/managed-by: operator
    auto-upgrade.stackrox.io/component: sensor
    helm.sh/chart: stackrox-secured-cluster-services-400.6.2
  name: stackrox
  resourceVersion: "49707404"
  uid: 83806d59-97ae-4a94-8c17-856034de192f
webhooks:
- admissionReviewVersions:
  - v1
  - v1beta1
  clientConfig:
    caBundle: ...
    service:
      name: admission-control
      namespace: stackrox
      path: /validate
      port: 443
  failurePolicy: Ignore
  matchPolicy: Equivalent
  name: policyeval.stackrox.io
  namespaceSelector:
    matchExpressions:
    - key: namespace.metadata.stackrox.io/name
      operator: NotIn
      values:
      - stackrox
      - kube-system
      - kube-public
      - istio-system
  objectSelector: {}
  rules:
  - apiGroups:
    - '*'
    apiVersions:
    - '*'
    operations:
    - CREATE
    - UPDATE
    resources:
    - pods
    - deployments
    - deployments/scale
    - replicasets
    - replicationcontrollers
    - statefulsets
    - daemonsets
    - cronjobs
    - jobs
    - deploymentconfigs
    scope: '*'
  sideEffects: NoneOnDryRun
  timeoutSeconds: 12
```

Lets take a look at a couple elements:
Here in this webhook we can see which operations are passed to this webhook for admission

```bash
    operations:
    - CREATE
    - UPDATE
    resources:
    - pods
    - deployments
    - deployments/scale
    - replicasets
    - replicationcontrollers
    - statefulsets
    - daemonsets
    - cronjobs
    - jobs
    - deploymentconfigs
```

One thing to note is `PATCH` is not available this would mean that any alterations that are done to resources via a `PATCH` will not be processed by RHACS such as doing a scale up within the UI of OCP.

From the above information we should be able to predict what the behavior should be when we enforce an action.

|Action|Stage|Expected Result|
|---|---|---|
| FAIL_BUILD_ENFORCEMENT | BUILD | Fail builds during continuous integration |
| SCALE_TO_ZERO_ENFORCEMENT | DEPLOY | Scale to Zero Replicas |
| UNSATISFIABLE_NODE_CONSTRAINT_ENFORCEMENT | DEPLOY | Add an Unsatisfiable Node Constraint |
| FAIL_DEPLOYMENT_CREATE_ENFORCEMENT | DEPLOY* | Block Deployment Create |
| FAIL_DEPLOYMENT_UPDATE_ENFORCEMENT | DEPLOY* | Block Deployment Update |
| FAIL_KUBE_REQUEST_ENFORCEMENT | RUNTIME | Fail Kubernetes API Request |
| KILL_POD_ENFORCEMENT | RUNTIME | Kill Pod |



---

## Behaviors Expected vs Actuals

Ok now that we have an idea of what types of enforcmenets can happen and when they may be applied... Lets see it in action.

To do these tests I have two argo apps, one for when we update the Policy and one for the project, I'll set them to manual update for now.

They are located at `PolicyTutorials/res/app-dep/*`

With both app and policy deployed we can begin our tests...

We can update enforcementActions, push to our repo that change then we can run the app changes.

|Action|Stage|Expected Result|
|---|---|---|
| FAIL_DEPLOYMENT_CREATE_ENFORCEMENT | DEPLOY* | Block Deployment Create |
| FAIL_DEPLOYMENT_UPDATE_ENFORCEMENT | DEPLOY* | Block Deployment Update |


For this as we mentioned before some irregular behaviors for example when you go to the OCP UI