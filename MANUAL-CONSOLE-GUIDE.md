# ApplyZA — Manual AWS Console Setup (No CloudFormation)

Set up S3 + CloudFront + Custom Domain entirely through the AWS Console.

---

## Step 1: Request SSL Certificate (ACM)

1. Go to **ACM Console** → [https://console.aws.amazon.com/acm/home?region=us-east-1](https://console.aws.amazon.com/acm/home?region=us-east-1)
   - **Must be us-east-1 (N. Virginia)** — CloudFront only works with certs from this region
2. Click **"Request a certificate"** → **"Request a public certificate"** → **Next**
3. Add domain names:
   - `applyza.co.za`
   - `*.applyza.co.za`
4. Validation method: **DNS validation**
5. Click **"Request"**
6. Click into the certificate → you'll see a CNAME record needed for validation
7. Go to **Cloudflare Dashboard** → `applyza.co.za` → **DNS** → **Records**
8. Add the CNAME record ACM gave you
9. **Turn the orange cloud OFF (grey = DNS only)**
10. Wait for ACM status to show **"Issued"** (5–30 minutes)

---

## Step 2: Create S3 Bucket

1. Go to **S3 Console** → [https://s3.console.aws.amazon.com/s3](https://s3.console.aws.amazon.com/s3)
2. Click **"Create bucket"**
3. Settings:
   - **Bucket name:** `applyza-site` (must be globally unique)
   - **Region:** pick one close to your users (e.g. `eu-west-1` or `af-south-1` if available)
   - **Block all public access:** ✅ Keep this ON (CloudFront will access it, not the public)
   - Leave everything else as default
4. Click **"Create bucket"**

### Enable Static Website Hosting (optional but recommended)

1. Click into your bucket → **Properties** tab
2. Scroll to **Static website hosting** → Click **Edit**
3. Enable it
4. Index document: `index.html`
5. Error document: `index.html`
6. Click **Save changes**

---

## Step 3: Upload Site Files

1. Click into your bucket → **Objects** tab
2. Click **"Upload"**
3. Drag and drop:
   - `index.html`
   - `admin.html`
   - `staff.html`
   - `track.html`
   - `styles.css`
   - `app.js`
   - `manifest.json`
   - `sw.js`
   - `_headers`
   - `assets/` folder (with all images inside)
4. Click **"Upload"**

**Do NOT upload:** `docs/`, `aws/`, `.git/`, `.vscode/`, `applyza-tampermonkey.user.js`, `README.md`, `resume_mandla/`

---

## Step 4: Create CloudFront Distribution

1. Go to **CloudFront Console** → [https://console.aws.amazon.com/cloudfront](https://console.aws.amazon.com/cloudfront)
2. Click **"Create distribution"**

### Origin Settings

3. **Origin domain:** Click the field and select your S3 bucket (`applyza-site.s3.amazonaws.com`)
4. A banner will appear: **"Use Origin Access Control"** → Click **"Yes, update the bucket policy"**
5. Under **Origin access:**
   - Select **"Origin access control settings"**
   - Click **"Create new OAC"**
   - Name: `applyza-site-oac`
   - Signing behavior: **Sign requests**
   - Click **"Create"**

### Default Cache Behavior

6. **Viewer protocol policy:** Redirect HTTP to HTTPS
7. **Cache policy:** `CachingOptimized` (select from dropdown)
8. **Compress objects automatically:** Yes

### Custom Domain & SSL

9. **Alternate domain names (CNAMEs):** Add both:
   - `applyza.co.za`
   - `www.applyza.co.za`
10. **Custom SSL certificate:** Select the certificate you created in Step 1
11. **Minimum SSL protocol:** TLSv1.2_2021

### Other Settings

12. **Default root object:** `index.html`
13. **Price class:** Use only North America and Europe (cheapest) or All edge locations
14. **HTTP version:** HTTP/2 and HTTP/3
15. Click **"Create distribution"**

### Update S3 Bucket Policy

After creating the distribution, CloudFront will show a banner saying the bucket policy needs updating.

16. Click **"Copy policy"**
17. Go to **S3 Console** → your bucket → **Permissions** tab → **Bucket policy** → **Edit**
18. Paste the policy CloudFront gave you
19. Click **"Save changes"**

---

## Step 5: Add Custom Error Responses

1. In CloudFront → your distribution → **Error pages** tab
2. Click **"Create custom error response"**
3. Add these two:

**Error 1:**
- HTTP error code: `403`
- Customize error response: Yes
- Response page path: `/index.html`
- HTTP response code: `200`
- Error caching minimum TTL: `10`

**Error 2:**
- HTTP error code: `404`
- Customize error response: Yes
- Response page path: `/index.html`
- HTTP response code: `200`
- Error caching minimum TTL: `10`

---

## Step 6: Point Domain to CloudFront (Cloudflare DNS)

1. In CloudFront → your distribution → copy the **Distribution domain name** (e.g. `d1abc2xyz.cloudfront.net`)
2. Go to **Cloudflare Dashboard** → `applyza.co.za` → **DNS** → **Records**
3. Add these records:

| Type  | Name  | Content                        | Proxy Status        |
|-------|-------|--------------------------------|---------------------|
| CNAME | `@`   | `d1abc2xyz.cloudfront.net`     | **Grey (DNS only)** |
| CNAME | `www` | `d1abc2xyz.cloudfront.net`     | **Grey (DNS only)** |

4. **Both must have the orange cloud turned OFF (grey)**

### Cloudflare SSL Setting

5. Go to **SSL/TLS** → **Overview**
6. Set mode to **Full (strict)** — or just keep proxy off which avoids conflicts

---

## Step 7: Invalidate Cache

1. In CloudFront → your distribution → **Invalidations** tab
2. Click **"Create invalidation"**
3. Path: `/*`
4. Click **"Create invalidation"**

---

## Done!

Wait 5–15 minutes. Your site should be live at:
- **https://applyza.co.za**
- **https://www.applyza.co.za**

---

## Updating Your Site Later

1. Go to S3 → your bucket → upload changed files (overwrite)
2. Go to CloudFront → Invalidations → create `/*`
3. Wait 1–2 minutes

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Access Denied (403) | Bucket policy not updated — copy it from CloudFront and paste in S3 |
| Certificate not in dropdown | Must be in **us-east-1** and status must be **"Issued"** |
| Redirect loops | Turn OFF Cloudflare proxy (grey cloud on DNS records) |
| Site not loading | DNS propagation — wait 15 min, try incognito browser |
| Old content showing | Create CloudFront invalidation `/*` |
| Bucket policy error | Make sure "Block all public access" is ON — CloudFront uses OAC, not public access |
