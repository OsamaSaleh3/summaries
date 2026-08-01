
## Executive Summary
This topic covers four services that extend identity management beyond basic IAM: **STS** (temporary credentials), **Cognito** (identities for your own app's users), **Directory Services** (integrating Microsoft Active Directory), and **IAM Identity Center** (one login across multiple AWS accounts). Think of them as answering: "How do I grant short-term access?", "How do I manage my app's customers?", "How do I bring in an existing corporate directory?", and "How do I log in once for everything?"

## Core Concepts Explained

### AWS STS (Security Token Service)
STS issues **temporary, short-term credentials** (access key + secret key + session token) instead of long-term ones. A user or service "assumes a role" via an STS API call, and STS hands back credentials that expire after a set time — like a visitor badge that stops working at the end of the day instead of a permanent employee keycard. Key use cases: **identity federation** (external identities getting temporary AWS access), **cross-account or same-account role access**, and quietly refreshing credentials for **EC2 instance roles** behind the scenes.

### Amazon Cognito
Cognito manages identities for **your web/mobile application's end users** — potentially millions of them — and explicitly is *not* for creating IAM users for outside customers. It keeps its own user database and also supports **social logins** (Google, Facebook, Twitter) via a redirect-based login flow. Rule of thumb: if you're building an app and need a sign-up/login system for customers, that's Cognito, not IAM.

### Microsoft Active Directory (AD) — Background
AD is a database of objects (users, computers, printers, file shares, security groups) traditionally run on a Windows Server, giving **centralized security management** on-premises. Classic example: a corporate laptop connects to a **Domain Controller**, letting one username/password work across every machine in the company network. AWS doesn't have AD natively, so AWS offers **Directory Services** to bring AD-style management into the cloud.

### AWS Directory Services (three flavors)
| Flavor | What it does |
|---|---|
| AWS Managed Microsoft AD | Real Microsoft AD hosted in AWS; supports MFA; can trust/join an existing on-prem AD |
| AD Connector | A proxy — redirects AWS auth requests to your existing on-prem AD; users still live on-prem; supports MFA |
| Simple AD | Standalone, AD-*compatible* directory in AWS; cannot join an on-prem AD |

> **Exam callout:** For the Cloud Practitioner exam specifically, you don't need the details above — just know that **"Directory Services" is the answer whenever a question mentions Active Directory or Microsoft Active Directory.**

### AWS IAM Identity Center (formerly AWS SSO)
This gives you **one login for multiple AWS accounts** in an Organization, plus single sign-on into business cloud apps, SAML 2.0 apps, and EC2 Windows instances. User identities can live in a **built-in identity store** or connect to a third-party provider (Active Directory, Okta, OneLogin). In practice: log in once at a single URL → land on a portal listing every account you have access to → click into any account's management console directly.

> **Exam callout:** Both the old name (**AWS Single Sign-On**) and new name (**IAM Identity Center**) may appear on the exam — treat them as the same feature. Trigger phrase: **"one login / access to multiple AWS accounts."**

## The Big Picture
These services solve different "who's logging in?" problems:
- **IAM** → your own company's employees, inside your AWS account(s).
- **STS** → temporary, expiring credentials for roles (federation, cross-account, EC2).
- **Cognito** → your product's external customers (web/mobile app users).
- **Directory Services** → bringing an existing corporate Active Directory into AWS.
- **IAM Identity Center** → one single login across many AWS accounts and external apps.

Together they cover the full spectrum: internal staff, temporary access, external app users, on-prem directories, and centralized multi-account login.

## Exam Focus
- **STS** = temporary/limited-privilege credentials → keywords: "temporary," "assume role," "federation," "expire."
- **Cognito** = identities for **your app's users**, not IAM users → keywords: "millions of users," "mobile/web app," "login with Google/Facebook."
- **Active Directory mentioned at all** → answer is **Directory Services** (don't overthink which of the 3 sub-types).
- **"One login for multiple AWS accounts"** → **IAM Identity Center** (aka AWS SSO) — memorize both names.
- IAM users are strictly for people **inside your company**; never create IAM users for app customers.

## Quick Reference Table
| Concept | What it is | Key thing to remember |
|---|---|---|
| STS | Temporary security credentials | Expiring keys; role assumption; powers EC2 instance roles |
| Cognito | Identity store for app users | For your customers, not employees; supports social login |
| Active Directory (on-prem) | Windows-based user/object database | Domain Controller = one login across company machines |
| AWS Directory Services | AWS's way to bring AD into the cloud | Answer whenever AD is mentioned; 3 types, don't stress details |
| IAM Identity Center (AWS SSO) | Single login for multiple AWS accounts | Same feature, two names; one URL, one portal, many accounts |