# ACS Reportint Details 

Lets cover reporting, there are two main ways of reporting through the UI and through the API.  

## Reports

To Create a report you can `Vulnerability Management -> Vulnerability Reporting`

![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/vuln-reporting.jpg)

Reports will reference a `collection` deployments that you will need to create.  The report will reference the `collection` you make. `Platform Configurations -> Collections`.  You can create one through `Create` button:


![Risk](https://github.com/axodevelopment/ACS-Tutorials/blob/main/images/collections.jpg)

Here you'll want to setup a notifier in the report when creating the `Report`.  From this UI you can Generate reorts and they will by default go to the configuration ie by email or to be able to be downloaded within the UI.

---

## API

You can also do the same through the API.  I would look at `exploring-api.md` for some basics.

In short we need to `id` of a report, an id will have this shape... [{`5c1d75bd-6668-4c82-a279-4f1e438d9bf2`}]...

One way is to look at the URL when clicking on the report you are interested in running:

https://central-stackrox.apps..../main/vulnerabilities/reports/`5c1d75bd-6668-4c82-a279-4f1e438d9bf2`


Here is an api call that will get a list of all reports and there names and id's:

Here is an example call...

```bash
curl -k -X GET "https://$ROX_CENTRAL_ADDRESS/v2/reports/configurations?query=&pagination.offset=0&pagination.limit=10&pagination.sortOption.field=Report%20Name&pagination.sortOption.reversed=false" \
  -H "Authorization: Bearer $ROX_API_TOKEN" 
```

And this is what we got back:

```bash
{"reportConfigs":[{"id":"5c1d75bd-6668-4c82-a279-4f1e438d9bf2","name":"test-report","description":"","type":"VULNERABILITY","vulnReportFilters":{"fixability":"FIXABLE","severities":["CRITICAL_VULNERABILITY_SEVERITY","IMPORTANT_VULNERABILITY_SEVERITY"],"imageTypes":["DEPLOYED","WATCHED"],"allVuln":true,"includeNvdCvss":false},"schedule":{"intervalType":"WEEKLY","hour":0,"minute":0,"daysOfWeek":{"days":[1]}},"resourceScope":{"collectionScope":{"collectionId":"fb660447-3f56-4b27-9ac8-f9992747b673","collectionName":"mycoll"}},"notifiers":[{"emailConfig":{"notifierId":"c2021e9e-f7e0-4b1d-b38b-830c3028cc0c","mailingLists":["..."],"customSubject":"","customBody":""},"notifierName":"Home Lab RHACS Notifier"}]}]}
```

You can then use the id in the above response to trigger a run of the report, to do so we will need the `reportConfigId`.

```bash
curl -k -X POST "https://$ROX_CENTRAL_ADDRESS/v2/reports/run" \
  -H "Authorization: Bearer $ROX_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"reportConfigId": "5c1d75bd-6668-4c82-a279-4f1e438d9bf2", "reportNotificationMethod": "DOWNLOAD"}'
```

The response from the API will be a reportId, this will link to the report to the requestor.

```bash
{"reportConfigId":"5c1d75bd-6668-4c82-a279-4f1e438d9bf2","reportId":"19e3be8a-373a-4425-b82f-7c607c38e90b"}%
```

The reportId here can be sent to the report job function to trigger a download.  If you set `reportNotificationMethod` to `EMAIL` the process would attempt to send an email instead.

```bash
curl -k -X GET "https://$ROX_CENTRAL_ADDRESS/api/reports/jobs/download?id=19e3be8a-373a-4425-b82f-7c607c38e90b" \
  -H "Authorization: Bearer $ROX_API_TOKEN" 

```
