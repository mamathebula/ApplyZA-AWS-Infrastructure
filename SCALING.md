# ApplyZA — Scaling & Auto-Scaling

How your AWS setup handles growth — from 100 visitors to 1,000,000.

---

## You're Already Auto-Scaled

Your architecture (S3 + CloudFront) is serverless and auto-scales by default. There are no servers to manage, no capacity to plan, and nothing to configure.

| Component | Scales Automatically? | Limit |
|-----------|----------------------|-------|
| S3 (file storage) | ✅ Yes | Unlimited — handles any number of requests |
| CloudFront (CDN) | ✅ Yes | Built for millions of requests per second |
| ACM (SSL) | ✅ Yes | No limit |
| Cloudflare DNS | ✅ Yes | No limit |

**There is no auto-scaling setting to turn on — it's already on by design.**

---

## Why This Architecture Scales

### S3 (Storage)
- Designed to handle thousands of requests per second per prefix
- No server to overload — files are served directly from AWS infrastructure
- Automatically distributes load across multiple servers behind the scenes

### CloudFront (CDN)
- 450+ edge locations worldwide
- Caches your files at locations closest to your users
- Users in Johannesburg get served from a nearby edge, not from a single server
- Handles traffic spikes automatically — no warm-up needed
- Netflix, Amazon, and Spotify use CloudFront

### Static Site (HTML/CSS/JS)
- No backend server that can crash
- No database that can run out of connections
- Every request is just serving a cached file
- This is the most scalable web architecture that exists

---

## Traffic Scenarios

| Visitors/Month | What Happens | Action Needed |
|---------------|-------------|---------------|
| 100 | Runs perfectly | Nothing |
| 1,000 | Runs perfectly | Nothing |
| 10,000 | Runs perfectly | Nothing |
| 100,000 | Runs perfectly | Nothing |
| 1,000,000 | Runs perfectly | Nothing — just a bigger bill (~R900/month) |

---

## What WILL Need Scaling (Not Infrastructure)

Your website scales automatically. But your business operations don't. Here's what to plan for:

### 1. You (Processing Applications)

| Clients/Month | Can You Handle It Alone? | What to Do |
|--------------|------------------------|------------|
| 1–50 | ✅ Yes | You're good |
| 50–200 | ⚠️ Getting busy | Consider a part-time assistant |
| 200–500 | ❌ Too much for one person | Hire 1–2 assistants |
| 500+ | ❌ Need a team | Build a small team, consider a dashboard/CRM |

### 2. Email (FormSubmit.co)

| Volume | Status | Action |
|--------|--------|--------|
| 1–100 emails/month | ✅ Fine | Free tier works |
| 100–500 emails/month | ⚠️ May hit limits | Monitor for delivery issues |
| 500+ emails/month | ❌ Need upgrade | Move to AWS SES ($0.10 per 1,000 emails) |

### 3. PayFast (Payments)

| Volume | Status | Action |
|--------|--------|--------|
| Any amount | ✅ Fine | PayFast scales on their side — no action needed |

### 4. WhatsApp (Communication)

| Clients/Month | Status | Action |
|--------------|--------|--------|
| 1–100 | ✅ Personal WhatsApp works | Keep using 083 775 1937 |
| 100–500 | ⚠️ Getting overwhelming | Consider WhatsApp Business with quick replies and labels |
| 500+ | ❌ Too many chats | Consider WhatsApp Business API with automated responses |

---

## Future Scaling Options (When You're Ready)

These are things you'd only consider when the business is much bigger. Not needed now.

### Level 1: Simple Improvements (50–200 clients)
- **WhatsApp Business app** — labels, quick replies, away messages (free)
- **Google Sheets** — already in use as primary database via Apps Script
- **Admin dashboard** — already built (`admin.html`) with stats, status tracking, vault, staff management
- **Staff portal** — already built (`staff.html`) for team members to manage applications
- **Tracking page** — already built (`track.html`) for clients to check application status
- **Email templates** — pre-written responses for common questions

### Level 2: Systems & Tools (200–500 clients)
- **AWS SES** — reliable email at scale ($0.10 per 1,000 emails)
- **Advanced CRM** — upgrade beyond Google Sheets if needed
- **Hire 1–2 part-time assistants** — train them on the application process (they can use `staff.html`)
- **Google Workspace** — shared email inbox (info@applyza.co.za) for the team

### Level 3: Full Operations (500+ clients)
- **Custom admin dashboard** — built on AWS (Lambda + DynamoDB + API Gateway)
- **Automated workflows** — payment received → auto-email client → create task
- **WhatsApp Business API** — automated messages, chatbot for FAQs
- **Team of 3–5 people** — dedicated application processors
- **Accounting software** — track revenue, expenses, PayFast payouts

---

## Cost at Scale

| Visitors/Month | AWS Cost (ZAR) | Notes |
|---------------|----------------|-------|
| 1,000 | ~R1 | Free tier covers this |
| 5,000 | ~R5 | Still basically free |
| 20,000 | ~R18 | Peak application season |
| 100,000 | ~R90 | Going viral |
| 500,000 | ~R450 | National reach |
| 1,000,000 | ~R900 | Massive scale |

Even at 1 million visitors, your hosting costs less than one month's rent. The real costs at scale are people, not servers.

---

## Summary

- ✅ Your website auto-scales — no action needed, ever
- ✅ S3 and CloudFront handle unlimited traffic
- ✅ Google Sheets + Apps Script handles data storage (already live)
- ✅ Admin, Staff, and Tracking portals already built
- ⚠️ Your bottleneck is people, not infrastructure
- 📋 Plan for hiring help at 200+ clients/month
- 📧 Move to AWS SES if email volume exceeds 500/month
- 💬 Upgrade to WhatsApp Business when chats get overwhelming

**You built the right architecture from day one. Focus on getting clients — the tech will keep up.**
