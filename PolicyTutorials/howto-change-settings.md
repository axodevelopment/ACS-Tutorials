# ACS how to change settings

There can be a need to manage the ACS System Config outside of RHACS.  The usual way is to store some resource into something like github and then use Argo or an equivalent to sync those changes into ACS.

One difficulty here is that there is no yaml for this configuration, but we do have access to the api's that RHACS exposes.

Lets explore the API's relevant `/v1/config`

Lets pull some data to see what we get from here...

```bash
curl -k -X GET -H "Authorization: Bearer $ROX_API_TOKEN" \
     -H "Content-Type: application/json" \
     "https://$ROX_CENTRAL_ADDRESS/v1/config" > config.json
```

Now that we have the config.json we can spend a moment to explore the options here.

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


After we make some updates we can can post to the same API and see our changes reflected in the UI.

What you will need to note here is the PUT API endpoint needs a slight change

```bash
curl -k -X PUT -H "Authorization: Bearer $ROX_API_TOKEN" \
     -H "Content-Type: application/json" \
     -d @config.json \
     "https://$ROX_CENTRAL_ADDRESS/v1/config"
```

We will need to make some slight changes to the config.json please note the new structure.

```bash
{
    "config": {
      "privateConfig": {
        "alertConfig": {
          "resolvedDeployRetentionDurationDays": 6,
          "deletedRuntimeRetentionDurationDays": "8",
          "allRuntimeRetentionDurationDays": 30,
          "attemptedDeployRetentionDurationDays": 6,
          "attemptedRuntimeRetentionDurationDays": 6
        },
        "imageRetentionDurationDays": 6,
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
      },
      "publicConfig": {
        "header": {
          "color": "#000000",
          "backgroundColor": "#FFFFFF",
          "text": "",
          "enabled": false,
          "size": "UNSET"
        },
        "footer": {
          "color": "#000000",
          "backgroundColor": "#FFFFFF",
          "text": "",
          "enabled": false,
          "size": "UNSET"
        },
        "loginNotice": {
          "text": "",
          "enabled": false
        },
        "telemetry": {
          "enabled": true
        }
      }
    }
}
```

After the PUT we see the configuration changes in RHACS UI.

---

As for a CI/CD approach we could achieve this with a direct approach of keeping the config in a `ConfigMap`

Once in a ConfigMap we can create a Tekton task which executes our same curl command with whatever changes you make the ConfigMap.  I'll expand this out more in a future update but for now let me show you the core ideas.


The `ConfigMap` would look something along the lines of:

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: rhacs-config
data:
  config.json: |
    {
      "publicConfig": {
        "loginNotice": {"enabled": false, "text": ""},
        "header": {"enabled": false, "text": "", "size": "UNSET", "color": "#000000", "backgroundColor": "#FFFFFF"},
        "footer": {"enabled": false, "text": "", "size": "UNSET", "color": "#000000", "backgroundColor": "#FFFFFF"},
        "telemetry": {"enabled": true}
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
          "ignoreClusterLabels": {}
        },
        "reportRetentionConfig": {
          "historyRetentionDurationDays": 7,
          "downloadableReportRetentionDays": 7,
          "downloadableReportGlobalRetentionBytes": 524288000
        },
        "vulnerabilityExceptionConfig": {
          "expiryOptions": {
            "dayOptions": [
              {"numDays": 14, "enabled": true},
              {"numDays": 30, "enabled": true},
              {"numDays": 60, "enabled": true},
              {"numDays": 90, "enabled": true}
            ],
            "fixableCveOptions": {"allFixable": true, "anyFixable": true},
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

We would want a `Secret` for our Token that we generated and have been using in this quick tutorial:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: rhacs-api-token
type: Opaque
data:
  token: ${ROX_API_TOKEN_BASE64}
```

We would need a tekton `Task` (or a similar task for your preferred pipeline):

```bash
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: update-rhacs-config
spec:
  params:
    - name: rhacs-central-address
      type: string
      description: "The RHACS Central address"
  steps:
    - name: update-config
      image: registry.access.redhat.com/ubi8/ubi-minimal:latest
      workingDir: /workspace
      env:
        - name: ROX_API_TOKEN
          valueFrom:
            secretKeyRef:
              name: rhacs-api-token
              key: token
        - name: ROX_CENTRAL_ADDRESS
          value: $(params.rhacs-central-address)
      script: |
        #!/bin/sh
        microdnf install -y curl
        
        cp /mnt/config/config.json .

        curl -k -X PUT -H "Authorization: Bearer $ROX_API_TOKEN" \
            -H "Content-Type: application/json" \
            -d @config.json \
            "https://$ROX_CENTRAL_ADDRESS/v1/config"

      volumeMounts:
        - name: config-volume
          mountPath: /mnt/config
  volumes:
    - name: config-volume
      configMap:
        name: rhacs-config
```

With these steps it should give you an idea of how to build into a pipeline with your repo.  Furthermore, we will also include OCP Gitops and use that structure of infrastructure-as-code because we have a `ConfigMap` that can be monitored outside in a github repo.  I'll show that and similar the execution of the pipeline in this case.