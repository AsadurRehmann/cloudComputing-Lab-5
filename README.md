#  Cloud Computing Lab 05

## 1. Project Overview

A multi-page personal portfolio website (Home, About, Projects) redeployed from the Lab 04 EC2 + Apache setup to an Amazon S3 static website. It uses a shared stylesheet, consistent navigation on every page, and two images.

**Live endpoint:** `http://cs471-f24605093-portfolio-2026.s3-website-us-east-1.amazonaws.com`
**Bucket name:** `cs471-f24605093-portfolio-2026`
**Region:** `US East (N. Virginia) us-east-1`

## 2. Architecture

```
Website files (HTML/CSS/images) -> S3 bucket -> S3 website endpoint -> Browser
```

- Visitors request the S3 **website endpoint** over HTTP.
- S3 serves `index.html` as the index document and returns the other objects by key.
- No server is managed. There is no EC2 instance, OS, or Apache.

### Project structure

```
Lab-05/
  index.html
  about.html
  projects.html
  style.css
  images/
    profile.png
    profile2.png
  README.md
```

## 3. Deployment Steps

1. **Prepared the site locally.** Created `index.html`, `about.html`, `projects.html`, `style.css`, and an `images/` folder. All links use relative paths, and every page has the same navigation and `<link rel="stylesheet" href="style.css">`. Tested all links and images locally.
2. **Created the S3 bucket.** S3 console -> Create bucket -> unique name, instructor-specified region, Object Ownership set to ACLs disabled (recommended). Account-level settings were not changed.
3. **Uploaded the objects.** Uploaded the HTML files and `style.css` with **Add files**, and the `images` folder with **Add folder** to preserve the `images/` prefix. Inspected an object's key, size, and last-modified time, and downloaded one to compare with the local copy.
4. **Enabled static website hosting.** Bucket -> Properties -> Static website hosting -> Edit -> Enable -> Host a static website -> Index document `index.html`. Copied the website endpoint.
5. **Configured approved public read access.** With instructor approval, adjusted **bucket-level** Block Public Access only, and applied this bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadForStaticWebsiteLab",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::[YOUR-BUCKET-NAME]/*"
    }
  ]
}
```

6. **Tested.** Opened the website endpoint (not the object URL) and checked all pages, navigation, CSS, images, and project links. Also tested in an Incognito window to confirm anonymous visitor access.
7. **Made a visible improvement.** `[Describe the change, e.g. updated headline colour / added a section]`. Re-uploaded the file and refreshed the site to confirm the update was live.
8. **Explored versioning.** Enabled Bucket Versioning, re-uploaded `index.html` with the same key, and used **List versions** to identify the current and previous versions.
9. **Reviewed storage classes and lifecycle rules.** Reviewed only; no lifecycle rule was created because that needs instructor authorization.

## 4. Security Notes

- The bucket holds **only non-sensitive demo files** (portfolio pages and images).
- The bucket policy grants **`s3:GetObject` only**. It does not allow public write, delete, list, or administrative actions.
- The policy `Resource` is scoped to this single bucket's objects (`arn:aws:s3:::[BUCKET]/*`).
- Only **bucket-level** Block Public Access was adjusted, and only with instructor approval. Account-wide safeguards were left unchanged.
- Signed in with an **IAM identity**, not the root user.
- No access keys, secret keys, passwords, private keys, or payment details appear in screenshots or in this repository.
- Public-read must stay narrowly scoped because anything readable by `*` is readable by anyone on the internet. A broader policy could expose private files or allow tampering.

## 5. Cost Notes

- S3 charges for **storage, requests, data transfer, versioned copies, and enabled features**. A Free Tier label does not mean every action is free.
- Cost-control actions taken:
  1. Checked regional pricing and used the region specified by the instructor.
  2. Kept the bucket small (only needed files) and plan to remove unnecessary objects and old versions after assessment, as the instructor directs.
- Versioning keeps every overwritten copy, so old versions add storage cost until they are deleted or expired.
- **Storage class:** S3 Standard suits website files because they are accessed frequently and need immediate, low-latency retrieval. Archive classes (Glacier) are unsuitable for served content because retrieval takes minutes to hours.
- **Proposed lifecycle rule (not applied):** for old non-website files under an `archive/` prefix, transition to a Glacier class after 90 days and expire noncurrent versions after 180 days. This lowers storage cost for rarely accessed data.

## 6. Troubleshooting

| Symptom | What to check |
|---|---|
| `403 Forbidden / AccessDenied` | Bucket name and ARN in the policy (must end in `/*`), Block Public Access settings, and that the object exists |
| `404 Not Found` | Exact object key and capitalization (S3 keys are case-sensitive) |
| CSS or images not loading | Uploaded paths must match the HTML links (e.g. `images/profile.png`) |
| Changes not visible | Re-upload with the same key and hard refresh (Ctrl+F5) |

### Problem I faced

- **Problem:** `[e.g. The website endpoint returned 403 Forbidden after enabling static website hosting.]`
- **Diagnosis:** `[e.g. Checked the Permissions tab and found Block Public Access was still on and no bucket policy was attached.]`
- **Fix:** `[e.g. With instructor approval, turned off bucket-level Block Public Access and added the s3:GetObject policy. The site then loaded.]`

## 7. EC2 + Apache (Lab 04) vs S3 Static Website (Lab 05)

| Dimension | EC2 + Apache (Lab 04) | S3 Static Website (Lab 05) |
|---|---|---|
| What you manage | OS, Apache, patches, instance and server configuration | Bucket, objects, hosting and access settings |
| Content | Static files and server-side apps with suitable configuration | Static HTML/CSS/JS/assets; no server-side execution |
| Operations | Manage instance capacity and web-server operation | No EC2 server to administer |
| Security | Security Groups, OS/server config, web permissions | Bucket policy, Block Public Access, IAM and object access |
| Cost factors | Instance runtime, EBS, data transfer, public IPv4 and related resources | Storage, requests, data transfer, versions and enabled features |
| HTTPS / custom domain | Can be configured on the server or through additional services | S3 website endpoint is HTTP-only; production HTTPS commonly uses CloudFront and a domain |

## 8. Post-Lab Questions

**60. Differentiate a bucket, object, object key, and prefix.**
A bucket is the container that stores objects. An object is a file plus its metadata. An object key is the object's unique name within the bucket (e.g. `images/profile.png`). A prefix is a shared beginning of keys that the console displays like a folder; it is not a real directory.

**61. Why does an S3 static website need object-read access for visitors?**
Visitors browse anonymously without AWS credentials, so the objects must allow `s3:GetObject` for the public to be fetched and displayed.

**62. How does an IAM identity policy differ from a bucket policy?**
An IAM policy is attached to an identity (user, group, or role) and defines what that authenticated identity can do. A bucket policy is a resource-based policy attached to the bucket that defines who can access its objects, including anonymous users.

**63. What does Block Public Access protect against?**
It protects against accidental or unintended public exposure by overriding public ACLs or bucket policies that would otherwise make data publicly accessible.

**64. Why can S3 static website hosting not replace EC2 for every application?**
S3 only serves static files. It cannot run server-side code (PHP, Node.js, Python backends), connect to databases, or handle server logic, and its website endpoint is HTTP-only.

**65. How does versioning help recovery, and what cost consideration does it introduce?**
Versioning keeps previous versions of an object, so overwritten or deleted files can be restored (a delete only adds a delete marker). Each stored version consumes storage and incurs charges.

**66. Which approach would you choose for a simple static portfolio, and why?**
S3 static hosting. There is no server to patch or manage, it is inexpensive, it scales automatically, and a static portfolio needs no server-side execution.

**67. Name two steps you took to improve security and control cost.**
Security: scoped the bucket policy to `s3:GetObject` only on this bucket and uploaded only non-sensitive files. Cost: checked regional pricing and removed unnecessary objects and versions after assessment.

## 9. Individual Reflection

`In this lab I learned how Amazon S3 stores files as objects inside buckets, and how object keys and prefixes work instead of real folders. Hosting my portfolio as a static website showed me that public access takes more than turning on website hosting. The bucket policy, the s3:GetObject permission, and Block Public Access all have to line up, and a mismatch gives a 403 error. Comparing S3 with the EC2 + Apache setup from Lab 04, I found S3 much simpler for a static site because there is no server to patch or manage, and it costs less. Its limits are that it can’t run server-side code and the website endpoint is HTTP-only. Versioning and lifecycle rules also taught me that storage costs can grow if old versions are left behind. In the future I would add CloudFront and a custom domain to get HTTPS and better performance.`

