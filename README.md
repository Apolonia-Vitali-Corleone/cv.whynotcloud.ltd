# AWS SAA Labs – Resume Site

A compact bilingual resume site that serves static HTML from Amazon S3 behind CloudFront. A lightweight AWS serverless backend captures contact form submissions via API Gateway, stores them in DynamoDB, and relays notifications through Amazon SES.

## Project Structure

| Path | Description |
| --- | --- |
| `site/` | Static assets for the Chinese (`cn/`) and English (`index.html`) resume pages plus the shared profile image. |
| `cloudformation/s3-cf-api-ddb-ses.yml` | Infrastructure-as-code template provisioning S3, CloudFront, API Gateway, Lambda, DynamoDB, and SES resources required for the site and contact form. |
| `runbook.md` | Operational procedures for deploying and maintaining the stack. |

## Prerequisites

* AWS CLI v2 configured with credentials that can create IAM, S3, CloudFront, API Gateway, Lambda, DynamoDB, and SES resources.
* An AWS Region where Amazon SES is available and sandbox restrictions have been lifted (or the notification email is verified).
* Node.js or Python is **not** required locally—the static site can be edited directly and uploaded.

## Deployment Overview

1. Review and customize the parameters in `cloudformation/s3-cf-api-ddb-ses.yml` (bucket name, notification email, optional tags).
2. Deploy the stack:
   ```bash
   aws cloudformation deploy \
     --template-file cloudformation/s3-cf-api-ddb-ses.yml \
     --stack-name cv-resume \
     --capabilities CAPABILITY_NAMED_IAM \
     --parameter-overrides \
       SiteBucketName=my-unique-bucket \
       NotificationEmail=you@example.com
   ```
3. Upload the contents of the `site/` folder to the provisioned bucket:
   ```bash
   aws s3 sync site/ s3://my-unique-bucket/
   ```
4. Wait for the CloudFront distribution to deploy and test the resume via the distribution domain name output by the stack.

Additional operational details—such as handling content updates, invalidations, and troubleshooting—are documented in `runbook.md`.

## Local Preview

Open `site/index.html` or `site/cn/index.html` directly in a browser to preview the static content. Because the contact form relies on the deployed API Gateway endpoint, form submissions are only fully functional in the deployed environment.

## License

This repository is maintained as a personal lab project and does not currently define a formal license. Reach out before reusing code or assets.
