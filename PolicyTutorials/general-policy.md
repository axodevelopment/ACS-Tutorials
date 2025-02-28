# Overview of ACS Tutorial 

Example of image scanning with roxctl and seeing the impact of a bad manage, you can use the roxctl command line to scan images in pipelines etc  The app in /res has some known critical vulnerabilities will be used for this demo.

## OC LOGIN

```bash
oc login --token=sha256~NROpsTKJp9NE8rhNfCwFjq8dWei-GnlWSkUhscfCrfQ --server=https://api.cluster-qhktc.qhktc.sandbox2634.opentlc.com:6443 --insecure-skip-tls-verify
```

## How to login with roxctl

Get central route

```bash
oc get route central -o jsonpath="{.status.ingress[0].host}" -n stackrox

set ROX_ENDPOINT=https://central-stackrox.apps.cluster-qhktc.qhktc.sandbox2634.opentlc.com:443
echo %ROX_ENDPOINT%
```

Roxctl login...

```bash
roxctl central login
roxctl central login --insecure-skip-tls-verify
```

Now scan the image...

```bash
roxctl image check -e %ROX_ENDPOINT% --insecure-skip-tls-verify --image quay.io/cporter/springboot:1.0
```

## Show fixable severity

Fixable Severity

Build Inform and enforce

Exit code != 0

Change policy to not enforce

```bash
oc create -f app.yaml

oc delete -f app.yaml
```

enable policy deploy enforcement

Retest to see the policy get enforced.

## Resources

https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_security_for_kubernetes/4.1/html-single/roxctl_cli/index

https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_security_for_kubernetes/4.4/pdf/roxctl_cli/red_hat_advanced_cluster_security_for_kubernetes-4.4-roxctl_cli-en-us.pdf