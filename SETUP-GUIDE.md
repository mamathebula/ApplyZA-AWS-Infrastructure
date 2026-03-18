# ApplyZA — AWS + Cloudflare Custom Domain Setup

## Overview

This deploys your site on S3 + CloudFront with your custom domain `applyza.co.za` using Cloudflare for DNS.

---

## Step 1: Request SSL Certificate (do this FIRST)

1. Go to **ACM Console** → [https://console.aws.amazon.com/acm/home?region=us-east-1](https://console.aws.amazon.com/acm/home?region=us-east-1)
   - **Must be us-east-1 (N. Virginia)** — CloudFront only accepts certificates from this region
2. Click **"Request a certificate"** → **"Request a public certificate"**
3. Add domain names:
   - `applyza.co.za`
   - `*.applyza.co.za`
4. Select **DNS validation** → Click **"Request"**
5. You'll see a CNAME record to add — something like:

| Type  | Name                          | Value                                    |
|-------|-------------------------------|------------------------------------------|
| CNAME | `_abc123.applyza.co.za`       | `_xyz789.acm-validations.aws.`           |

6. Go to **Cloudflare Dashboard** → `applyza.co.za` → **DNS** → **Records**
7. Add the CNAME record from ACM
8. **Important:** Click the orange cloud to turn it **grey (DNS only)** — proxy must be OFF
9. Wait for ACM status to change to **"Issued"** (5–30 minutes)
10. Copy the **Certificate ARN** — you'll need it for the next step

---

## Step 2: Deploy the CloudFormation Stack

1. Go to **CloudFormation Console** → [https://console.aws.amazon.com/cloudformation](https://console.aws.amazon.com/cloudformation)
2. Click **"Create stack"** → **"With new resources"**
3. Upload `cloudformation.yaml` from this folder
4. Fill in the parameters:
   - **BucketName:** `applyza-site` (or your preferred name)
   - **DomainName:** `applyza.co.za`
   - **CertificateArn:** paste the ARN from Step 1
5. Click through and **"Submit"**
6. Wait for status: **CREATE_COMPLETE** (5–10 minutes)

---

## Step 3: Upload Site Files to S3

1. Go to **S3 Console** → [https://s3.console.aws.amazon.com/s3/buckets](https://s3.console.aws.amazon.com/s3/buckets)
2. Click on your bucket (`applyza-site`)
3. Click **"Upload"**
4. Drag and drop these files/folders:
   - `index.html`
   - `admin.html`
   - `staff.html`
   - `track.html`
   - `styles.css`
   - `app.js`
   - `manifest.json`
   - `sw.js`
   - `_headers`
   - `assets/` folder
5. Click **"Upload"**

**Do NOT upload:** `docs/`, `aws/`, `.git/`, `.vscode/`, `applyza-tampermonkey.user.js`, `README.md`, `resume_mandla/`

---

## Step 4: Point Domain to CloudFront (Cloudflare DNS)

1. Go to **CloudFormation Console** → your stack → **Outputs** tab
2. Copy the **CloudFrontDomainName** value (e.g. `d1abc2xyz.cloudfront.net`)
3. Go to **Cloudflare Dashboard** → `applyza.co.za` → **DNS** → **Records**
4. Add these records:

| Type  | Name  | Content                        | Proxy Status       |
|-------|-------|--------------------------------|--------------------|
| CNAME | `@`   | `d1abc2xyz.cloudfront.net`     | **Grey (DNS only)** |
| CNAME | `www` | `d1abc2xyz.cloudfront.net`     | **Grey (DNS only)** |

5. **Both records must have the orange cloud turned OFF (grey = DNS only)**

---

## Step 5: Cloudflare SSL Settings

1. In Cloudflare → **SSL/TLS** → **Overview**
2. Set encryption mode to **"Full (strict)"**
   - Or just leave proxy off (grey cloud) which avoids any SSL conflicts entirely

---

## Step 6: Invalidate CloudFront Cache

1. Go to **CloudFront Console** → your distribution
2. Click **"Invalidations"** tab
3. Click **"Create invalidation"**
4. Enter `/*`
5. Click **"Create invalidation"**

---

## Done!

After 5–15 minutes, your site should be live at:
- **https://applyza.co.za**
- **https://www.applyza.co.za**

---

## Updating Your Site

Whenever you change files:

1. Upload the changed files to S3 (overwrite existing)
2. Create a CloudFront invalidation with `/*`
3. Wait 1–2 minutes

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Access Denied (403) | S3 bucket is empty — upload your files |
| Certificate not showing | Make sure it's in **us-east-1** and status is **"Issued"** |
| Redirect loops | Turn OFF Cloudflare proxy (grey cloud on both DNS records) |
| Site not loading on domain | DNS propagation — wait 15 minutes, clear browser cache |
| Old content showing | Create a CloudFront invalidation with `/*` |
