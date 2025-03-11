# ACS how to change settings

There can be a need to manage the ACS System Config outside of RHACS.  The usual way is to store some resource into something like github and then use Argo or an equivalent to sync those changes into ACS.

One difficulty here is that there is no yaml for this configuration, but we do have access to the api's that RHACS exposes.

Lets explore the API's relevant `/v1/config`

Lets pull some data to see what we get from here...

```bash


```

```bash
{
    "publicConfig": {
        "loginNotice": {
            "enabled": false,
            "text": ""
        },
        "header": {
            "enabled": false,
            "text": "",
            "size": "UNSET",
            "color": "#000000",
            "backgroundColor": "#FFFFFF"
        },
        "footer": {
            "enabled": false,
            "text": "",
            "size": "UNSET",
            "color": "#000000",
            "backgroundColor": "#FFFFFF"
        },
        "telemetry": {
            "enabled": true,
            "lastSetTime": null
        }
    },
    "privateConfig": {
        "alertConfig": {
            "resolvedDeployRetentionDurationDays": 7,
            "deletedRuntimeRetentionDurationDays": 8,
            "allRuntimeRetentionDurationDays": 30,
            "attemptedDeployRetentionDurationDays": 7,
            "attemptedRuntimeRetentionDurationDays": 7
        },
        "imageRetentionDurationDays": 7,
        "expiredVulnReqRetentionDurationDays": 90,
        "decommissionedClusterRetention": {
            "retentionDurationDays": 0,
            "ignoreClusterLabels": {},
            "lastUpdated": "2025-02-28T00:08:01.581393180Z",
            "createdAt": "2025-02-28T00:08:01.581392414Z"
        },
        "reportRetentionConfig": {
            "historyRetentionDurationDays": 7,
            "downloadableReportRetentionDays": 7,
            "downloadableReportGlobalRetentionBytes": 524288000
        },
        "vulnerabilityExceptionConfig": {
            "expiryOptions": {
                "dayOptions": [
                    {
                        "numDays": 14,
                        "enabled": true
                    },
                    {
                        "numDays": 30,
                        "enabled": true
                    },
                    {
                        "numDays": 60,
                        "enabled": true
                    },
                    {
                        "numDays": 90,
                        "enabled": true
                    }
                ],
                "fixableCveOptions": {
                    "allFixable": true,
                    "anyFixable": true
                },
                "customDate": false,
                "indefinite": false
            }
        },
        "administrationEventsConfig": {
            "retentionDurationDays": 4
        }
    }
}
```


After we make some updates we can 