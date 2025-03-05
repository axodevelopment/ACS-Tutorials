# ACS Tutorials
Policy as code is a strategy we will look at in this tutorial.

## Overview of RH ACS

First things first, I highly suggest this other tutorial @
https://www.stb.id.au/blog/rhacs-policy-as-code.

This tutorial will will leverage where this one leaves off.

A common question we get are how could we implement policy as code with the ideas of managing policies both in lifecycle, how they migrate from their initial development and migrate to production.  This is the concept i'll be primarily focusing on.

## Corrections

This tutorial deviates abit from the tutorial it was built ontop of we do have some corrections to continue on this tutorial.  This may mean that some of the actions you have either already completed will look a bit different in this tutorial in this setup, i'll attempt to cover the changes and map the items correctly before we continue.

### Pre req

I am current using the following:
Openshift 4.17.x
RHACS 4.6x
RHGitops 1.15.x

### ACS Central location

When a policy in yaml is deployed it should be deployed into the same location as `central` deployment is.

For a quick search

```bash
oc get pod -A | grep central
```

Here is an example yaml we will look at for clarity.

```bash
kind: SecurityPolicy
apiVersion: config.stackrox.io/v1alpha1
metadata:
  name: insecure-http-registry
spec:
  policyName: Insecure HTTP Registry
  categories:
  - DevOps Best Practices
  - Security Best Practices
  description: Detect deployments using container images from insecure HTTP registries
  disabled: true
  remediation: Use only HTTPS-enabled registries for all container images. Update image references to use secure registries or enable HTTPS on your registry server
  lifecycleStages:
  - BUILD
  - DEPLOY
  enforcementActions:
  - FAIL_BUILD_ENFORCEMENT
  - SCALE_TO_ZERO_ENFORCEMENT
  severity: HIGH_SEVERITY
  policySections:
  - sectionName: Check Repo Protocol
    policyGroups:
    - fieldName: Image Registry
      booleanOperator: OR
      negate: false
      values:
      - value: ^http://
  rationale: Using HTTP instead of HTTPS for container registries exposes your infrastructure to man-in-the-middle attacks, credential theft, and tampering with container images during pull operations
```

When this yaml is deployed it is similar to importing a .json format of a policy that you can either do from the API or from the UI.

Lastly if you are using a default install with my version, the namespace is `stackrox`

---

### GitOps Notes

The `Application` yamls covered today do expect a repo already configured.  If you are having access issues there are really three types of access you need.

One you need access to the github repo:
I recommend at least a PAT with your user name.

Second, the application-controller needs access to the destination project
```bash
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n stackrox
```

While you could grant admin access if you want to be more granular RBAC is the way to go:

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: stackrox-policies-admin
rules:
- apiGroups: ["config.stackrox.io"]
  resources: ["securitypolicies"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-stackrox-policies-admin
subjects:
- kind: ServiceAccount
  name: openshift-gitops-argocd-application-controller
  namespace: openshift-gitops
roleRef:
  kind: ClusterRole
  name: stackrox-policies-admin
  apiGroup: rbac.authorization.k8s.io
```

and then you'll likely want to do grant serviceaccount / group access in Argo

```bash
oc get configmap argocd-rbac-cm -n openshift-gitops -o yaml
```

You could update it to something similar to:

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: openshift-gitops
data:
  policy.csv: |
    p, role:admin, applications, *, */*, allow
    p, role:admin, clusters, *, *, allow
    p, role:admin, repositories, *, *, allow
    p, role:admin, logs, *, *, allow
    p, role:admin, exec, *, *, allow
    p, rh-ee-michwils, repositories, create, *, allow
    p, rh-ee-michwils, repositories, get, *, allow
    p, rh-ee-michwils, repositories, update, *, allow
    p, rh-ee-michwils, repositories, delete, *, allow
  policy.default: role:readonly
```