# Cleanup (do this after submission, in us-east-1)

1. CloudWatch > Synthetics Canaries > `internee-home` > Actions > Delete (also delete its Lambda function and layers).
2. Route 53 > Health checks > delete any health check. If the Free plan error appears, retry later or temporarily upgrade the plan.
3. CloudWatch > Alarms > delete `internee-success-low`, `internee-duration-high`, and any `Synthetics-Alarm-...` or Route 53 alarms.
4. CloudWatch > Dashboards > delete `internee-monitoring`.
5. S3 > bucket `cw-syn-results-...`: empty it, then delete it.
6. CloudWatch > Logs > log groups: delete `/aws/lambda/cwsyn-internee-home-...`.
7. IAM > Roles: delete `CloudWatchSyntheticsRole-internee-home-...`.
8. SNS > delete the `internee-alerts` subscription and topic.
9. Billing > Budgets: delete the budget after a few days.
10. Check Billing / Cost Explorer a day or two later to confirm nothing is still charging.
