# Overview of App Baseline

A powerful capability of ACS is to baseline the activity and behaviors of your applications running in ACS.

oc debug -n ssl-test-broker --image=axodevelopment/artemistools:v1.0.0.3

## Notes

For my case I am going to create a new project called 'acs-demos' and for an application i will create a debug pod that uses ubi-minimal for this.  If you have deployment already you can either oc exec $pod -- bash

```bash
oc new-project acs-demos
```

When I refer to my app or pod I mean the following

```bash
oc debug --image=registry.access.redhat.com/ubi8/ubi-minimal -n acs-demos
```

## Begin Tutorial - Logging in with OC

oc login

Deploy your app and get an bash into your pod for this case
```bash
oc debug --image=registry.access.redhat.com/ubi8/ubi-minimal -n acs-demos
```

If you are following the tutorial you will need to know the pod name

```bash
oc get pods
```

With your pod deployed you will need to login into ACS

Navigate from the main ui to the 'Risk' section

If you just deployed your app or used the oc debug command to quickly find the deployment by sorting by Created which is a date / time of when the deployment was created.

You'll notice a Priority number on the side, this is the 'percieved' risk value the pod.  You'll likely have a fairly high number here.  The lower the number here the 'higher' the risk.

Because I am using a debug pod, my deployment name is 'image-debug-abcde'.

Note that there is a circle next to the name.  This is actually color coded based upon violations it may or may not be having.  Yours is probably clear when you first deploy.

Click on your deployment name.

On the right pane, you'll see Risk Indicators, Deployment Details, Process Discovery.  For now we will be focusing on Process Discovery...

Click on Process Discovery.