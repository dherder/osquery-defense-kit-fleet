# Chainguard Queries
Originally forked from https://github.com/chainguard-dev/osquery-defense-kit. 

[Ben Heater](https://benheater.com/) wrote a [Powershell conversion scritpt](https://github.com/0xBEN/osquery-defense-kit
) to take the Chainguard sql and convert it to .yml format that can be consumed by fleetctl. 

I took Ben's .yml files for IR and Detection and just added references to using teams with the queries. 

## Adopt a "teams first" approach to testing 
The teams feature is a Premium licensed feature in Fleet, but it allows the admin to segment and target deployment of groups of queries (amongst other use cases). 

### Create teams
This can be done in the Fleet UI or progrmammatically via [REST API](https://fleetdm.com/docs/rest-api/rest-api#create-team) or [`fleetctl`](https://fleetdm.com/docs/using-fleet/fleetctl-cli#fleetctl-cli). Your testing scenarios may differ, but a starting approach would be to create a single team to transfer hosts into when running DART activities:

- [x] create team 'Chainguard' 

### Import the queries to teams in Fleet
The .yml files have been formatted to use the above team names. If you would like to use different team names, please modify the .yml files appropriately. 

- [x] Using [`fleetctl`](https://fleetdm.com/docs/using-fleet/fleetctl-cli#fleetctl-cli), apply the following commands:

`fleetctl apply --context <context> -f chainguard-detection-queries.yml`
`fleetctl apply --context <context> -f chainguard-IR-queries.yml`

### Enable process events (evented table)
https://fleetdm.com/guides/osquery-evented-tables-overview

I had created a separate team (Chainguard) in the above step. Either enable evented tables as per the above article on the same "Chainguard" team, or if you'd like to only enable on a case by case basis, create a different team. In my test scenario, I just enabled events in the Chainguard team.

#### Agent options
```
command_line_flags:
  events_max: 50000
  events_expiry: 3600
  disable_events: false
  events_optimize: true
  enable_file_events: true
  disable_endpointsecurity: false
  ```

### Select queries and define frequency of execution
Done either through the UI or through modifying the .yml directly. 

## Configure AWS Role for Fleet Cloud to sts:AssumeRole into
### Permissions
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "kinesis:PutRecord",
                "kinesis:PutRecords",
                "kinesis:DescribeStream"
            ],
            "Resource": "arn:aws:kinesis:*:8888888xxxxxx:stream/*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "kinesis:ListStreams",
                "kinesis:DescribeStream"
            ],
            "Resource": "*"
        }
    ]
}
```
### Trust Relationship
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::99999999xxxxxx:role/demo1-role"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

## Configure Splunk to access Kinesis data stream
https://docs.splunk.com/Documentation/AddOns/released/AWS/Setuptheadd-on


## Additional articles
https://unfinished.bike/behavioral-detection-of-macos-malware-using-osquery


