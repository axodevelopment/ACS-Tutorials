# ACS Tutorials
In this tutorial we will explore the api of RHACS


## The API

The API has a ton of different api calls that allow you to collect most elements of data within ACS.

## Requirements

To make an api call you'll need a couple of things.

`apitoken` - Gives you authorization into the api.
`routetoacs` - Route to ACS

## Exploring the API Reference

By looking at the following image you can actually explore and look at the api definitions for all the accessible apis...

![ApiView](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/api-view.jpg)


## Creating the API Token

To create the `apitoken` you'll need to create an API token by navigating to `Platform Configuration -> Integrations -> API Token`

![ApiView](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/api-token.jpg)

Once you create a token, much like other providers you'll need to copy that locally somewhere for your need, I usually create a temp directory that I have auto clean at some point in the future, I stored mine in a file called acs_token.token and then setup my export as such.

```bash
export ROX_API_TOKEN=$(cat acs_token.token)
```

## lets get the route

```bash
oc get route -n stackrox | grep central

export ROX_CENTRAL_ADDRESS=central-stackrox.apps....
```

Lets test the client and check an image

```bash
roxctl --insecure-skip-tls-verify image check --endpoint $ROX_CENTRAL_ADDRESS:443 --image quay.io/centos7/httpd-24-centos7:centos7
```

Now we should be able to start calling some apis.

You may have noticed earlier when you looked at the API that you can get the route and path for a particular api (see the image below)

![ApiView](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/api-paths.jpg)

---

If you want to just do a a quick ping you can curl the route as such:

```bash
curl -k -X GET "https://$ROX_CENTRAL_ADDRESS/v1/ping" \
  -H "Authorization: Bearer $ROX_API_TOKEN"
```

Here is an additional

```bash
"https://$ROX_CENTRAL_ADDRESS/v1/export/images 
```

The `jq` tool can be used here to get some interesting data.  

With this we can collect the image data and reduce the total size of data.  Lets get the images we've scanned, list the cvecount and get the highest vcss and the list of the vulnerabilities.

```bash
curl -k -X GET "https://$ROX_CENTRAL_ADDRESS/v1/export/images" \
  -H "Authorization: Bearer $ROX_API_TOKEN" | \
  jq -r '.result.image | {
    imageName: .name.fullName,
    cveCount: .cves,
    topCvss: .topCvss,
    vulnerabilities: [.scan.components[].vulns[] | select(.cve != null) | {
      cve: .cve,
      cvss: .cvss,
      severity: .severity
    }] | sort_by(-.cvss) | unique_by(.cve) | .[0:10] 
  }'
```

## Resources

API docs:
https://developers.redhat.com/api-catalog/api/rhacs-service-fleet-manager#content-operations