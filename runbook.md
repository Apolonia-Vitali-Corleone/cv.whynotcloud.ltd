# Resume Platform Runbook

This runbook documents the operational steps required to deploy, operate, and troubleshoot the resume site and contact form stack defined in `cloudformation/s3-cf-api-ddb-ses.yml`.

## 1. Stack Deployment / Update

1. **Prepare parameters**
   * `SiteBucketName` must be globally unique (e.g., `cv-yourname-prod`).
   * `NotificationEmail` must be verified in Amazon SES (the template creates the identity, but you must confirm the verification email).
   * Optionally adjust the `ProjectTag` parameter to match your tagging standard.
2. **Deploy or update**
   ```bash
   aws cloudformation deploy \
     --template-file cloudformation/s3-cf-api-ddb-ses.yml \
     --stack-name cv-resume \
     --capabilities CAPABILITY_NAMED_IAM \
     --parameter-overrides \
       SiteBucketName=cv-yourname-prod \
       NotificationEmail=you@example.com \
       ProjectTag=resume
   ```
3. Wait for the stack to finish. Confirm outputs:
   * `CloudFrontDomain` – public URL to the resume.
   * `ContactApiEndpoint` – HTTPS endpoint for the contact form.
   * `ResumeBucket` – name of the S3 bucket hosting the static assets.

## 2. Verifying Amazon SES Identity

1. After the first deployment you will receive a verification email at `NotificationEmail`.
2. Click the verification link to activate the SES identity.
3. Once verified, move the SES configuration set from sandbox (if necessary) and request production access if you need to email unverified recipients.

## 3. Publishing Site Content

1. Build or edit HTML in the `site/` directory.
2. Sync the files to S3 (this preserves existing objects and uploads changes):
   ```bash
   aws s3 sync site/ s3://cv-yourname-prod/ --delete
   ```
3. Invalidate the CloudFront cache to propagate changes quickly:
   ```bash
   aws cloudfront create-invalidation \
     --distribution-id <DistributionId> \
     --paths '/*'
   ```
   The distribution ID is available in the CloudFormation console under the distribution resource.

## 4. Monitoring & Logs

* **CloudFront** – Check the CloudFront console for error rates and request metrics.
* **Lambda & API Gateway** – Use CloudWatch Logs (`/aws/lambda/<function-name>`) for request/response details. CloudWatch metrics can alert on high 5XX error counts.
* **DynamoDB** – Monitor consumed capacity if the contact form begins receiving significant traffic.
* **SES** – Review delivery reports and bounces to ensure the notification email remains healthy.

## 5. Backup & Data Export

* Enable DynamoDB point-in-time recovery if long-term retention is required (not enabled by default in the template).
* To export submissions, run:
  ```bash
  TABLE_NAME=$(aws cloudformation describe-stacks \
    --stack-name cv-resume \
    --query 'Stacks[0].Outputs[?OutputKey==`ContactTableName`].OutputValue' \
    --output text)
  aws dynamodb scan \
    --table-name "$TABLE_NAME" \
    --region <region> \
    --output json > submissions.json
  ```

## 6. Troubleshooting

| Symptom | Checks |
| --- | --- |
| Static site returns 403 | Ensure files are uploaded to the correct bucket prefix and the CloudFront invalidation has completed. |
| Contact form fails with 500 | Inspect the Lambda logs for stack traces; verify DynamoDB and SES IAM permissions are intact. |
| SES email not delivered | Confirm the identity is verified and the region has exited sandbox mode; review suppression list. |
| CloudFormation rollback | Review the Events tab for the failing resource. Common issues include bucket name collisions or SES verification pending. |

## 7. Teardown

1. Empty the S3 bucket:
   ```bash
   aws s3 rm s3://cv-yourname-prod --recursive
   ```
2. Delete the CloudFormation stack:
   ```bash
   aws cloudformation delete-stack --stack-name cv-resume
   ```
3. Optionally remove the SES identity manually if you no longer need to send emails from it.

Keep this document updated whenever operational procedures change.
