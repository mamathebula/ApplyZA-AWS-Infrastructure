# ApplyZA — AWS Infrastructure

CloudFormation template for deploying the ApplyZA static website on AWS with S3, CloudFront, and a custom domain.

## Architecture

```
applyza.co.za → Cloudflare DNS → CloudFront (CDN + HTTPS) → S3 Bucket (static files)
```

## What Gets Created

| Resource | Purpose |
|----------|---------|
| S3 Bucket | Stores your site files (private, no public access) |
| Origin Access Control (OAC) | Lets CloudFront read from S3 securely |
| S3 Bucket Policy | Grants CloudFront permission to access the bucket |
| CloudFront Distribution | CDN, HTTPS, caching, custom domain support |

## Prerequisites

- An AWS account
- Domain `applyza.co.za` managed on Cloudflare
- An ACM certificate in **us-east-1** for `applyza.co.za` and `*.applyza.co.za` (see guides below)

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `BucketName` | `applyza-site` | S3 bucket name (must be globally unique) |
| `DomainName` | `applyza.co.za` | Your custom domain |
| `CertificateArn` | *(required)* | ACM certificate ARN from us-east-1 |

## Outputs

| Output | Description |
|--------|-------------|
| `WebsiteURL` | `https://applyza.co.za` |
| `CloudFrontURL` | CloudFront distribution URL |
| `CloudFrontDomainName` | Use this as CNAME target in Cloudflare |
| `S3BucketName` | Bucket name for uploading files |
| `DistributionId` | For cache invalidation |

## Files in This Folder

| File | Description |
|------|-------------|
| `cloudformation.yaml` | CloudFormation template (automated deployment) |
| `CFT-EXPLAINED.md` | Explains every resource in the template and why it exists |
| `SETUP-GUIDE.md` | Step-by-step guide using the CloudFormation template |
| `MANUAL-CONSOLE-GUIDE.md` | Step-by-step guide doing everything manually in the AWS Console (no CloudFormation) |
| `COST-ESTIMATE.md` | Detailed cost breakdown for S3 + CloudFront hosting |
| `SCALING.md` | How the architecture scales and when to scale operations |
| `README.md` | This file |

## Quick Start

1. Request an ACM certificate in us-east-1 → validate via Cloudflare DNS
2. Deploy `cloudformation.yaml` via CloudFormation Console → paste the certificate ARN
3. Upload site files to S3
4. Add CNAME records in Cloudflare (proxy OFF)
5. Invalidate CloudFront cache

Full details in [SETUP-GUIDE.md](SETUP-GUIDE.md).

Prefer doing it manually without CloudFormation? See [MANUAL-CONSOLE-GUIDE.md](MANUAL-CONSOLE-GUIDE.md).

## Files to Upload to S3

```
index.html
admin.html
staff.html
track.html
styles.css
app.js
manifest.json
sw.js
_headers
assets/
  applyza_logo.png
  icon-192.png
  icon-512.png
```

Do **not** upload: `docs/`, `aws/`, `.git/`, `.vscode/`, `cloudformation.yaml`, `.gitignore`, `applyza-tampermonkey.user.js`, `README.md`, `resume_mandla/`

## External Services

| Service | Purpose | Config Location |
|---------|---------|-----------------|
| Google Sheets (Apps Script v4) | Application data storage & sync | URL in `app.js`, `admin.html`, `staff.html`, `track.html` |
| PayFast (LIVE) | Payment processing | Merchant ID/key in `app.js` |
| FormSubmit | Contact form emails → info@applyza.co.za | Endpoint in `index.html` |
| Cloudflare | DNS + domain management | Cloudflare dashboard |

## Current Version

- Service Worker cache: `applyza-v24.1`
- Google Apps Script: Version 4
- Site: `https://applyza.co.za`
- Launch date: 15 March 2026

## Google Sheets Setup

The site uses Google Sheets as the primary database via Apps Script (Version 4). See `docs/GOOGLE-APPS-SCRIPT.js` for the full script to deploy. The Apps Script URL must be updated in 4 files if redeployed:
- `app.js` (GOOGLE_SHEETS_URL)
- `admin.html` (GOOGLE_SHEETS_URL)
- `staff.html` (GOOGLE_SHEETS_URL)
- `track.html` (GOOGLE_SHEETS_READ_URL)

Sheet tab must be named "Applications" with headers in Row 1 matching the column order in the Apps Script.

## Updating the Site

1. Upload changed files to S3 (overwrite existing)
2. Create a CloudFront invalidation: `/*`
3. Wait 1–2 minutes

## Cost Estimate

| Service | Cost |
|---------|------|
| S3 | ~$0.02/month (minimal storage) |
| CloudFront | Free tier covers 1TB/month + 10M requests |
| ACM Certificate | Free |
| **Total** | **~$0–1/month** for a low-traffic site |
