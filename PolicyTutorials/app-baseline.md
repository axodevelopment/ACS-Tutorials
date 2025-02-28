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
oc debug --image=axodevelopment/artemistools:v1.0.0.3 -n acs-demos
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

## Lets start looking at ACS

With your pod deployed you will need to login into ACS

Navigate from the main ui to the 'Risk' section

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/risks.jpg)


If you just deployed your app or used the oc debug command to quickly find the deployment by sorting by Created which is a date / time of when the deployment was created.

You'll notice a Priority number on the side, this is the 'percieved' risk value the pod.  You'll likely have a fairly high number here.  The lower the number here the 'higher' the risk.

Because I am using a debug pod, my deployment name is 'image-debug-abcde'.

Note that there is a circle next to the name.  This is actually color coded based upon violations it may or may not be having.  Yours is probably clear when you first deploy.

Click on your deployment name.

On the right pane, you'll see Risk Indicators, Deployment Details, Process Discovery.  For now we will be focusing on Process Discovery...

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/process-discovery.jpg)

Click on Process Discovery.

Under event timeline you'll see View Graph.

For the moment explore this section.

Note here there is a History of Running Processes.  ACS will monitor what processes are running and generate a history / timeline of these activities.

Below that is the Spec Container Baseline.  This is the current known process that "MAY" become a baseline.  Right now you'll notice that this is currently an unlocked baseline you can see that buy clicking the lock buttons on and off, before you continue leave it unlocked atm.

Inside your pod, from earlier do the following a few times.  If your app doesn't use microdnf use whatever is appropriate...

```bash
microdnf install openssl
```
In most images this won't actually do anything because you are not usually running as a root user ie you shouldnt have USER 0 typically...

This process will take a few seconds to get captured and processed by ACS.

What we want to do is tell ACS what is abnormal about this and create a baseline for ACS to compare to, in this case you want to click the [-] on the /usr/bin/microdnf from the baseline.

Then you want to LOCK the baseline.  Locking the baseline means that ACS will actually evaluate the process.  IT MUST BE LOCKED.  To continue.

If you have done it correctly it should look like...

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/baseline-locked.jpg)

## Lets create a policy

First we should note it is best practice to clone a policy instead of editing a System policy (System policies are ones that are pre defined and come with ACS).

To see policies we should navigate from Platform Configuration -> Policy Management.

What we want to do is to react to the anomalous activity that we are generating at runtime.

The filters on this page have many options but for now select policy and then fine this policy "Unauthorized Process Execution"

On the right side of this you see 3 dot notation for actions, lets 'Clone' the policy.  Use all default values for now.

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/clone-policy.jpg)

For now, lets click on the clone policy and then in the 'Actions' we will 'Edit policy'

For the 'Details' explore these items and edit these as you see fit except keep the Categories as is.

There are two areas I really want to highlight the first is the Lifecycle

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/lifecycle.jpg)

You'll see 3 stages here Build, Deploy, Runtime...

In general how this works is at which stage do you want ACS to evaluate and react to this.  In this particular process, the policy can really on be done at runtime.  But why, well it has to do with what these stages mean.

"Build" stage is triggered before deployment, meaning in your CI.  You can use the roxctl to integrate ACS to activities outside of something running in OCP think of a build pipeline hence the name.

"Deploy" stage is triggered when a new deployment is deployed to the cluster.  Behind the scenes ACS uses an admissionwebhook to trigger the scan as the workload is being deployed.

"Runtime" stage is triggered by the live monitoring of workloads in the cluster.

These stages allow you to expand the 'safety net' behind to just whats in your secured clusters.

The next section to look at is the actions section.

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/actions.jpg)

Enforcement is how do you want the policy to react, do you want to just inform when that policy is triggered or do you want it to enforce as well.  In this case for runtime anomalous behaviors when set to enforce it will terminate any pod violating that.

Lets set the Enformcement to Informat and Enforce, lets set the policy to be Enabled and then hit the radio button of the Runtime under Confirgure enforcement behavior to be toggled on.

# With that lets see the enforcement.

If you go back into your pod and then attempt the microdnf instal openssl command again, since it is marked specifically as not in the baseline of the pod and the baseline is locked the pod will be evicted.

As a cleanup process you will want to revert any changes into a safe state for your ACS pre tutorial.
