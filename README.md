# Google Shopping Feed Pipeline (AWS + Playwright)

This project automates the creation of a Google Shopping-compatible XML product feed for CustomGear using AWS Fargate, Docker, and Playwright.

---

## 📦 What It Does
- Scrapes product data from SAGE-powered product URLs
- Extracts title, description, image, and pricing
- Generates a valid Google Shopping XML feed
- Uploads the feed to an S3 bucket
- Fully automated weekly via Amazon EventBridge

---

## 🔧 Technologies Used

| Component     | Purpose                            |
|---------------|------------------------------------|
| Python        | Core scraping + XML generation     |
| Playwright    | JavaScript-rendered scraping       |
| Docker        | Containerized execution environment|
| AWS Fargate   | Serverless container hosting       |
| AWS S3        | XML feed storage                   |
| EventBridge   | Weekly task scheduler              |
| IAM Roles     | Fine-grained permissions           |

---

## 🗂️ Project Structure

```
.
├── products.csv                # Input: List of product URLs
├── run_pipeline_scrape_only.py # Main pipeline logic (no SAGE API)
├── Dockerfile                  # Container setup
├── .env                        # Optional environment config
```

---

## 🚀 Deployment Overview

1. Build Docker image:
   ```bash
   docker build -t google-feed .
   ```

2. Push to ECR:
   ```bash
   aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin <ECR_URI>
   docker push <ECR_URI>
   ```

3. Create Fargate task:
   - Cluster: `google-feed-cluster-2`
   - Task definition: `google-feed-task`
   - Role: `fargate-task-role-google-feed`

4. Set up EventBridge:
   - Schedule: `cron(0 0 ? * 1 *)` (Sundays at 00:00 UTC)
   - Target: ECS → Run task
   - VPC config: Use `fargate-vpc`, two public subnets
   - Auto-assign public IP: `ENABLED`

---

## 🌐 Output Feed URL

```
https://googleshoppingfeed.s3.us-west-1.amazonaws.com/feeds/google_shopping_feed.xml
```

---

## 🔒 Security & Access
- Bucket uses `BucketOwnerEnforced` ACL mode
- Public access is granted via S3 **bucket policy** on `/feeds/*`

---

## 🖼 Infographic
See: `pipeline_flow.png` for visual reference of the pipeline flow.

---

## 📅 Schedule & Maintenance
- Runs **every Sunday @ midnight UTC**
- Output is overwritten weekly
- Can be scaled with retries, Slack/email alerts, or feed validators

---

## 🙌 Credits
Built by CustomGear | Designed for hands-free Google Shopping feed automation using modern AWS architecture.
