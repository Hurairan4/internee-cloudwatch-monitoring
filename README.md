# Internee.pk Monitoring with AWS CloudWatch

External (black-box) monitoring of the live Internee.pk platform for availability, response time and HTTP errors, with email alerting. Built for an internship task, with **no access to Internee.pk source code or servers**.

## Objective
Monitor the Internee.pk system for performance and security using AWS CloudWatch: track key metrics, and set up alerts for system failures.

## Approach
Because only the public URL was available, the site is monitored the way a visitor experiences it. Everything lives in my own AWS account (us-east-1) and only sends ordinary page requests to `https://internee.pk`.

```
CloudWatch Synthetics canary  -->  CloudWatch metrics  -->  CloudWatch alarms  -->  SNS topic  -->  Email
 (Chrome + Firefox, every 5 min)     (success, duration,       (2 alarms)           internee-alerts
                                      2xx/4xx/5xx, failed)
                                              |
                                      CloudWatch dashboard
```

## What was built
| Component | Name | Purpose |
|---|---|---|
| Synthetics canary | `internee-home` | Heartbeat check of `https://internee.pk` every 5 min in Chrome and Firefox |
| Alarm | `internee-success-low` | SuccessPercent < 100 for 2 of 2 periods (availability) |
| Alarm | `internee-duration-high` | Duration > 45,000 ms for 3 of 3 periods (slow runs) |
| SNS topic | `internee-alerts` | Emails alarm state changes |
| Dashboard | `internee-monitoring` | SuccessPercent, Duration, 4xx / 5xx / Failed requests |
| Budget | $5 monthly | Cost safety net |

## Results
- **144 runs over 12 hours (Oct 5, 2026, 00:37-12:37 UTC), 100% success rate.**
- Canary Duration stayed between about 25.7 s and 32.8 s. This is the whole canary run (browser launch, page load, screenshots), not page-load time alone.
- No 5xx responses observed. "Failed requests" showed small intermittent counts (0-3 per period) while SuccessPercent stayed at 100%, which points to individual sub-resources failing rather than the site being down (not investigated further).
- **Alert test:** I temporarily raised the `internee-success-low` threshold to 101 to force a breach. The alarm went OK -> ALARM at 21:01 UTC on Oct 4 and the email arrived, then the threshold was set back to 100 and the alarm returned to OK.

The recorded demo shows a 3-hour window; the full monitoring window was about 12 hours.

## Evidence
| File | Shows |
|---|---|
| `screenshots/01-dashboard-12h.png` | Dashboard, 12-hour range |
| `screenshots/02-canary-12h-summary.png` | 144 runs, 100% success |
| `screenshots/03-alarms-ok.png` | Both alarms OK after the test |
| `screenshots/04-alarm-test-in-alarm.png` | Alarm in ALARM state during the test |
| `screenshots/05-alert-email.png` | SNS alert email (account ID redacted) |
| `screenshots/06-canary-3h-summary.png` | 3-hour summary used in the demo video |

## Limitations
- No CPU, memory or disk metrics: those need server or cloud-account access.
- Only public pages are covered. Login-protected pages would need a test account.
- Measurements come from AWS us-east-1, so latency reflects the US-to-Pakistan path.
- Route 53 health checks were not usable on my Free plan account, so availability relies on the canary.
- Security monitoring is limited to availability and HTTP errors. No scanning or load testing was done, since that needs written authorization on a live production site.

## With more access
Read-only CloudWatch access or server logs would unlock real CPU/memory metrics, backend error rates and log-based alarms. A staging environment would allow agent-based monitoring.

## Docs
- [`docs/report.md`](docs/report.md): full report
- [`config/monitoring-config.md`](config/monitoring-config.md): exact settings
- [`CLEANUP.md`](CLEANUP.md): teardown steps
