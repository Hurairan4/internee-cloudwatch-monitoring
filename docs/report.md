# Monitoring Report: Internee.pk

## 1. Task
Monitor the Internee.pk system for performance and security using AWS CloudWatch: track CPU usage, response time and error rates, and set up alerts for system failures.

## 2. Constraints and approach
Internee.pk is a live platform and no source code, repository or server access was provided. I used external (black-box) monitoring: a CloudWatch Synthetics canary requests the public page on a schedule, and CloudWatch records the outcome. All resources are in my own AWS account in us-east-1. Only light, visitor-like requests were made, with no scanning or load testing.

## 3. Implementation
1. **Budget:** $5 monthly cost budget with email alerts.
2. **SNS:** standard topic `internee-alerts`, email subscription confirmed, test message delivered.
3. **Canary:** `internee-home`, Heartbeat blueprint, target `https://internee.pk`, every 5 minutes, runs in Chrome and Firefox.
4. **Alarms:** `internee-success-low` and `internee-duration-high` (see `config/monitoring-config.md`), both notifying `internee-alerts`.
5. **Dashboard:** `internee-monitoring` with SuccessPercent, Duration and 4xx/5xx/Failed-requests widgets.
6. **Alert test:** raised the success threshold to 101 to force a breach, confirmed the alarm and email, then reverted to 100.

## 4. Results (12-hour window, Oct 5, 2026, 00:37-12:37 UTC)
| Metric | Result |
|---|---|
| Canary runs | 144 |
| Success rate | 100% |
| Duration range | about 25.7-32.8 s per run |
| 5xx responses | none observed |
| Failed requests | intermittent, 0-3 per period |
| Alarm state after test | both OK |

**Notes**
- Duration is the full canary execution (browser startup, load, screenshots), so it is much higher than real page-load time.
- Failed requests were non-zero while the success rate stayed at 100%. This suggests individual sub-resources (images, scripts, third-party calls) failing, not an outage. I did not inspect the HAR data to confirm.
- The alert test fired at 21:01 UTC on Oct 4, 2026 and the email arrived with the reason, timestamp and metric details.

## 5. Mapping to the task
| Requirement | How it was covered |
|---|---|
| CPU usage | Not possible externally. Needs server or cloud-account access. |
| Response time | Canary Duration metric (full run); per-run screenshots and HAR available |
| Error rates | 4xx, 5xx and Failed requests metrics on the dashboard |
| Alerts for failures | Success-rate and duration alarms with SNS email |
| Security | Passive only: availability and HTTP errors. No scanning without authorization. |

## 6. Limitations
- No CPU, memory or disk visibility.
- Public pages only; login-protected flows not covered.
- Measured from us-east-1 (US), not from Pakistan.
- Route 53 health checks unavailable on a Free plan account.
- A 12-hour window is enough to validate the setup but not to establish long-term baselines.

## 7. Recommendations with more access
1. Read-only CloudWatch access or server logs for CPU, memory and backend error rates.
2. A second canary for the login page using a dedicated test account stored in Secrets Manager.
3. An SSL-expiry check and a deeper security-header review, with written approval.
4. A canary in a region closer to users.

## 8. Cleanup
All resources are deleted after submission. See `CLEANUP.md`.
