# ApplyZA — AWS Hosting Cost Estimate

Breakdown of what it costs to run applyza.co.za on AWS (S3 + CloudFront).

---

## Free Tier (First 12 Months)

AWS gives you a free tier for the first year of a new account.

| Service | Free Tier Allowance |
|---------|-------------------|
| CloudFront | 1 TB data transfer out + 10,000,000 HTTP/HTTPS requests per month |
| S3 | 5 GB storage + 20,000 GET requests + 2,000 PUT requests per month |
| ACM (SSL Certificate) | Always free |

**For a low-traffic static site like ApplyZA, you'll likely pay R0/month during the free tier period.**

---

## After Free Tier (or High Traffic)

### S3 Storage

| Item | Cost |
|------|------|
| Storage | $0.023 per GB/month |
| PUT/POST requests (uploads) | $0.005 per 1,000 requests |
| GET requests (downloads) | $0.0004 per 1,000 requests |

**ApplyZA site size is ~1 MB. Monthly storage cost: ~$0.01 (basically nothing).**

### CloudFront (CDN)

| Item | Cost |
|------|------|
| Data transfer to internet | $0.085 per GB (first 10 TB) |
| HTTP/HTTPS requests | $0.0075 per 10,000 requests |

### ACM (SSL Certificate)

| Item | Cost |
|------|------|
| Public certificate | **Always free** |

---

## Monthly Cost Scenarios

### Scenario 1: Low Traffic (starting out)
*~1,000 visitors/month, ~5,000 page views*

| Service | Usage | Cost (USD) | Cost (ZAR ~R18.50) |
|---------|-------|-----------|-------------------|
| S3 Storage | ~1 MB | $0.01 | R0.19 |
| S3 Requests | ~5,000 GET | $0.002 | R0.04 |
| CloudFront Data | ~500 MB | $0.04 | R0.74 |
| CloudFront Requests | ~5,000 | $0.004 | R0.07 |
| ACM Certificate | 1 | Free | Free |
| **Total** | | **~$0.06** | **~R1.04** |

Effectively free — especially during the free tier year.

---

### Scenario 2: Moderate Traffic (growing)
*~5,000 visitors/month, ~25,000 page views*

| Service | Usage | Cost (USD) | Cost (ZAR ~R18.50) |
|---------|-------|-----------|-------------------|
| S3 Storage | ~1 MB | $0.01 | R0.19 |
| S3 Requests | ~25,000 GET | $0.01 | R0.19 |
| CloudFront Data | ~2.5 GB | $0.21 | R3.89 |
| CloudFront Requests | ~25,000 | $0.02 | R0.37 |
| ACM Certificate | 1 | Free | Free |
| **Total** | | **~$0.25** | **~R4.63** |

---

### Scenario 3: High Traffic (peak application season)
*~20,000 visitors/month, ~100,000 page views*

| Service | Usage | Cost (USD) | Cost (ZAR ~R18.50) |
|---------|-------|-----------|-------------------|
| S3 Storage | ~1 MB | $0.01 | R0.19 |
| S3 Requests | ~100,000 GET | $0.04 | R0.74 |
| CloudFront Data | ~10 GB | $0.85 | R15.73 |
| CloudFront Requests | ~100,000 | $0.08 | R1.48 |
| ACM Certificate | 1 | Free | Free |
| **Total** | | **~$0.98** | **~R18.13** |

---

### Scenario 4: Very High Traffic (viral / national reach)
*~100,000 visitors/month, ~500,000 page views*

| Service | Usage | Cost (USD) | Cost (ZAR ~R18.50) |
|---------|-------|-----------|-------------------|
| S3 Storage | ~1 MB | $0.01 | R0.19 |
| S3 Requests | ~500,000 GET | $0.20 | R3.70 |
| CloudFront Data | ~50 GB | $4.25 | R78.63 |
| CloudFront Requests | ~500,000 | $0.38 | R7.03 |
| ACM Certificate | 1 | Free | Free |
| **Total** | | **~$4.84** | **~R89.54** |

---

## Comparison: AWS vs Cloudflare Pages (Current)

| | Cloudflare Pages | AWS (S3 + CloudFront) |
|---|---|---|
| Hosting cost | Free | R0–R5/month (low traffic) |
| SSL certificate | Free | Free (ACM) |
| CDN | Included | CloudFront (global) |
| Custom domain | Free | Free (you configure DNS) |
| Bandwidth limit | Unlimited | Pay per GB after free tier |
| Build/deploy | Auto from GitHub | Manual upload to S3 |
| Complexity | Very simple | More setup required |
| Control | Limited | Full control over caching, headers, etc. |

**Bottom line:** Cloudflare Pages is simpler and free. AWS gives you more control and scales better, but costs a few rands per month. For ApplyZA's current stage, either works perfectly.

---

## What You'll Never Pay For

- ✅ ACM SSL certificates — always free
- ✅ S3 bucket creation — free
- ✅ CloudFront distribution creation — free
- ✅ DNS on Cloudflare — free
- ✅ GitHub repository — free
- ✅ Google Sheets + Apps Script (database) — free
- ✅ FormSubmit (contact form emails) — free

## What Could Increase Costs

- ❗ Large file uploads (videos, PDFs) stored in S3
- ❗ Extremely high traffic (100K+ visitors/month)
- ❗ Multiple CloudFront invalidations (first 1,000/month are free, then $0.005 each)
- ❗ Enabling CloudFront logging to S3 (adds storage costs)

---

## Tips to Keep Costs Low

1. **Enable CloudFront compression** — already set in the template, reduces data transfer
2. **Use long cache TTLs** — less requests to S3
3. **Don't store large files in S3** — host videos on YouTube/TikTok, not S3
4. **Limit invalidations** — only invalidate when you actually update the site
5. **Monitor with AWS Cost Explorer** — set a billing alert at $1 and $5

### Set a Billing Alert

1. Go to **AWS Billing Console** → [https://console.aws.amazon.com/billing](https://console.aws.amazon.com/billing)
2. Click **"Budgets"** → **"Create budget"**
3. Choose **"Zero spend budget"** (alerts you on any charge) or set a custom amount like $5
4. Add your email for notifications
5. You'll get an email if costs exceed your threshold

---

## Summary

| Traffic Level | Monthly Cost (ZAR) |
|--------------|-------------------|
| Starting out (1K visitors) | **~R1** |
| Growing (5K visitors) | **~R5** |
| Peak season (20K visitors) | **~R18** |
| Viral (100K visitors) | **~R90** |

**ApplyZA on AWS is essentially free for a small to medium site.** Even at peak application season with 20,000 visitors, you're looking at less than R20/month.
