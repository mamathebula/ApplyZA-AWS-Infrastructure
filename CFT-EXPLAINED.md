# CloudFormation Template — Why Each Resource Exists

This explains every resource in `cloudformation.yaml` and why it's needed for applyza.co.za to work.

---

## What is CloudFormation (CFT)?

CloudFormation is AWS's infrastructure-as-code tool. Instead of clicking through the AWS Console to create each resource one by one, the CFT file describes everything your site needs in one file. You can deploy it once and AWS creates everything automatically, wired together correctly.

Think of it as a blueprint for your house — you could build each wall by hand, or hand the blueprint to a builder and let them do it all at once.

You already set everything up manually via the Console, so this CFT serves as a **backup blueprint**. If you ever need to recreate the infrastructure (new AWS account, disaster recovery, or helping someone replicate the setup), you deploy this one file and everything is back.

---

## Resources Breakdown

### 1. S3 Bucket (`S3Bucket`)

**What it is:** A storage container on AWS where all your website files live (index.html, admin.html, staff.html, track.html, app.js, styles.css, images, etc.).

**Why it's needed:** Your site has to be stored somewhere. S3 is where the actual files sit. When someone visits applyza.co.za, CloudFront fetches the files from this bucket and serves them.

**Key settings:**
- `BlockPublicAcls: true` / `BlockPublicPolicy: true` — Nobody can access the bucket directly. Only CloudFront can read from it. This prevents anyone from bypassing your domain and accessing files through a raw S3 URL.
- `IndexDocument: index.html` — When someone visits the root URL, serve index.html.
- `ErrorDocument: index.html` — If a file isn't found, serve index.html instead of an ugly error page (important for single-page app behavior).

---

### 2. Bucket Policy (`BucketPolicy`)

**What it is:** A permission rule attached to the S3 bucket that says "only CloudFront is allowed to read files from here."

**Why it's needed:** The bucket is locked down (all public access blocked). Without this policy, even CloudFront couldn't read your files. This policy creates a secure tunnel: CloudFront → S3, and nothing else.

**How it works:** It checks that the request comes from your specific CloudFront distribution using the `AWS:SourceArn` condition. Even other CloudFront distributions can't access your bucket — only yours.

---

### 3. Origin Access Control (`OriginAccessControl`)

**What it is:** A security mechanism that tells CloudFront how to authenticate when requesting files from S3.

**Why it's needed:** This is the modern replacement for the older "Origin Access Identity" (OAI). It uses AWS Signature Version 4 (sigv4) to sign every request CloudFront makes to S3. Without this, CloudFront can't prove to S3 that it's authorized, and S3 would reject the requests.

**In simple terms:** It's the ID badge that CloudFront shows to S3 to prove it's allowed in.

---

### 4. CloudFront Distribution (`CloudFrontDistribution`)

**What it is:** A CDN (Content Delivery Network) that sits in front of S3 and serves your site to visitors worldwide with low latency and HTTPS.

**Why it's needed:** S3 alone can host a static site, but:
- S3 website hosting doesn't support HTTPS with a custom domain
- S3 serves from one region — slow for users far away
- No caching, compression, or HTTP/2 support

CloudFront solves all of this. It caches your files at edge locations around the world, serves everything over HTTPS, compresses files for faster loading, and supports HTTP/2 and HTTP/3.

**Key settings:**

| Setting | Why |
|---------|-----|
| `DefaultRootObject: index.html` | Visiting applyza.co.za loads index.html |
| `HttpVersion: http2and3` | Faster page loads with modern protocols |
| `PriceClass_100` | Uses only North America + Europe edge locations (cheapest tier, still fast for SA users via European nodes) |
| `Aliases: applyza.co.za, www.applyza.co.za` | Tells CloudFront to respond to your custom domain |
| `ViewerCertificate` | Attaches your SSL certificate so HTTPS works |
| `ViewerProtocolPolicy: redirect-to-https` | Forces all HTTP traffic to HTTPS |
| `Compress: true` | Gzip/Brotli compression — smaller files, faster loads |
| `CachePolicyId: 658327ea...` | Uses AWS's "CachingOptimized" managed policy — good defaults for static sites |
| `CustomErrorResponses (403/404 → index.html)` | If someone hits a URL that doesn't exist as a file (like /admin), serve index.html instead of a CloudFront error page |

---

## Parameters (Inputs You Provide)

| Parameter | What It Is | Your Value |
|-----------|-----------|------------|
| `BucketName` | Name of the S3 bucket | `applyza-site` |
| `DomainName` | Your custom domain | `applyza.co.za` |
| `CertificateArn` | ARN of your SSL certificate from ACM (must be in us-east-1) | Your ACM cert ARN |

---

## Outputs (What You Get Back)

| Output | What It Tells You |
|--------|------------------|
| `WebsiteURL` | Your live site URL: `https://applyza.co.za` |
| `CloudFrontURL` | The CloudFront URL (e.g., `d1234abcdef.cloudfront.net`) |
| `CloudFrontDomainName` | The value you put as a CNAME in Cloudflare DNS |
| `S3BucketName` | The bucket name for uploading files |
| `DistributionId` | The CloudFront ID you use when invalidating cache after updates |

---

## How It All Connects

```
Visitor types applyza.co.za
        │
        ▼
Cloudflare DNS → CNAME points to CloudFront domain
        │
        ▼
CloudFront (CDN)
  ├── Has SSL cert from ACM (HTTPS)
  ├── Caches files at edge locations
  ├── Compresses responses
  └── Uses OAC to authenticate with S3
        │
        ▼
S3 Bucket (private)
  ├── Bucket Policy only allows CloudFront
  └── Stores all site files
```

---

## Do You Need This File?

You already built everything manually through the AWS Console. This CFT is your **infrastructure backup**. You don't need to deploy it unless:

- You're setting up a fresh AWS account
- Something breaks and you need to recreate everything
- You want to replicate the setup for another project
- Someone else needs to understand exactly what AWS resources power the site

It costs nothing to keep. It's just a YAML file in your repo.
