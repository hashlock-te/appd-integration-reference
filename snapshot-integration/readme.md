# ThousandEyes Snapshot Integration
Create a ThousandEyes snapshot when an AppDynamics Event / Alert occurs.

This uses AppDyamics' HTTP Request Template webhook integration feature, similar to ThousandEyes alert webhook notification. 

For each ThousandEyes test associated with a given AppD application (or tier / node) we will create a separate HTTP request to trigger a ThousandEyes snapshot via the ThousandEyes API. 

We will create an HTTP Request template so that the ThousandEyes snapshot time window and TestID can be provided dynamically.

## Create HTTP Request Template
1. Under `Alert and Respond` select `HTTP Request Template`.
2. Click "+" (New)
3. Setup HTTP Request Template to trigger ThousandEyes Snapshot:

### Name
Name the HTTP Request `ThousandEyes Snapshot`

### Set Request URL

Set the request URL to `POST https://api.thousandeyes.com/v6/snapshot.json`

[httprequest-url.png]

### Set Authentication

Set authentication to `Basic`. Use your ThousandEyes username and API token as password.


### Set Payload

Note - using Apache VTL to format time string in request.

In ThousandEyes, the “adcapital” test ID is 1246117.

```
#set( $String = '' )
#set( $to = $String.format('%1$tY-%1$tm-%1$tdT%1$tH:%1$tM:%1$tS ', $latestEvent.eventTime))
#set( $tohour = $String.format('%1$tH', $latestEvent.eventTime))
#set($tohourint = 0)
#set( $fromhourint = $tohourint.parseInt($tohour) - 1)
#set( $fromhour = $fromhourint.toString())
#set( $from = $String.format('%1$tY-%1$tm-%1$tdT%2$s:%1$tM:%1$tS ', $latestEvent.eventTime, $fromhour))
{
    "testId": ${testid},
    "displayName": "Snapshot from AppDynamics - ${testname} - ${to}",
    "from": "${from}",
    "to": "${to}",
    "isPublic": 1
}
```

### Save HTTP Request
[httprequest-save.png]


### Setup Alert Policy to Trigger HTTP Request
See [thousandeyes-snapshot-template.json](thousandeyes-snapshot-template.json) as an example of referencing the HTTP Request Template from an Alert Policy.

```
{
  "name": "ThousandEyes Snapshot Template",
  "description": null,
  "version": 1,
  "healthRuleMembers": [],
  "actionMembers": [
    {
      "model": {
        "id": 3884,
        "actionType": "HTTP_REQUEST",
        "name": "ThousndEyes Snapshot",
        "httpRequestTemplateName": "ThousandEyes Snapshot",
        "customTemplateVariables": [
          {
            "key": "Date",
            "value": "2/5/2020"
          },
          {
            "key": "Test",
            "value": "customerapp"
          }
        ]
      },
      "memberType": "ACTION"
    }
  ],
  "scheduleMembers": [],
  "actionSuppressionMembers": [],
  "emailDigestMembers": [],
  "policyMembers": []
}
```

[httprequest-action.png]

