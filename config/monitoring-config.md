# Monitoring configuration (region: us-east-1)

## Canary
| Setting | Value |
|---|---|
| Name | `internee-home` |
| Blueprint | Heartbeat monitoring |
| Runtime | syn-nodejs-puppeteer-18.0 |
| URL | https://internee.pk |
| Schedule | every 5 minutes |
| Browsers | Chrome, Firefox |
| Data retention | 7 days |

## Alarm: internee-success-low
| Setting | Value |
|---|---|
| Namespace / metric | CloudWatchSynthetics / SuccessPercent (By Canary: internee-home) |
| Statistic / period | Average / 5 minutes |
| Condition | Lower than 100 |
| Datapoints to alarm | 2 out of 2 |
| Action | In alarm -> SNS `internee-alerts` |

## Alarm: internee-duration-high
| Setting | Value |
|---|---|
| Namespace / metric | CloudWatchSynthetics / Duration (By Canary: internee-home) |
| Statistic / period | Average / 5 minutes |
| Condition | Greater than 45000 (ms) |
| Datapoints to alarm | 3 out of 3 |
| Action | In alarm -> SNS `internee-alerts` |

## Dashboard: internee-monitoring
- SuccessPercent (line)
- Duration (line)
- 4xx, 5xx, Failed requests (line)

## SNS
- Topic: `internee-alerts` (Standard), one email subscription

## Budget
- Monthly cost budget, $5, email alerts

## Alert test
- `internee-success-low` threshold temporarily set to 101 (21:01 UTC, Oct 4, 2026), alarm fired and email received, then reverted to 100.
