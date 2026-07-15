# IAM (Identity and Access Management)

## Executive Overview

**IAM (Identity and Access Management)** is one of the most foundational services in the entire AWS ecosystem. It is the service responsible for controlling **who** can access your AWS account and **what** they are allowed to do once they are in. Every single interaction with AWS — whether through the web-based Management Console, the Command Line Interface (CLI), the Software Development Kit (SDK), or an AWS service acting on your behalf — passes through IAM's authentication and authorization layer.

IAM is critically important for the AWS Certified Cloud Practitioner (CLF-C02) exam because:

- It is almost always the **first service** taught in any AWS course, since you cannot securely use any other AWS service without understanding identities and permissions.
- It is deeply tied to the **Shared Responsibility Model**, one of the most heavily tested concepts on the exam.
- It introduces core security best practices (least privilege, MFA, credential rotation) that show up repeatedly throughout the exam in the context of other services.
- It is a **global service** — a unique characteristic that distinguishes it from most other AWS services, which are region-scoped.

In short, IAM is the security gatekeeper of your AWS account. Understanding it thoroughly is not optional — it underpins your ability to reason about security, access control, and governance across the entire AWS platform.

---

## 1. IAM Fundamentals

### 1.1 What is IAM?

**IAM stands for Identity and Access Management.** It is the AWS service used to:

- Manage **users** and **groups** of users within your AWS account.
- Manage **roles**, which grant permissions to AWS services or applications rather than to human beings.
- Define and attach **policies** (JSON documents) that describe exactly what actions an identity is permitted (or forbidden) to perform.

### 1.2 IAM is a Global Service

Unlike EC2, S3 (partially), or RDS, **IAM is not tied to any specific AWS Region**. When you create a user, group, role, or policy in IAM, it is created once and is immediately available and consistent across **all AWS Regions**. This is why, when you open the IAM console, the region selector in the top-right corner of the Management Console is greyed out / inactive — there is no region to choose because IAM operates at the account level globally.

> **Exam Tip:** If a question asks "Which of the following services is global rather than regional?" — IAM is one of the classic correct answers (along with Route 53, CloudFront, and S3's bucket-namespace, though S3 itself stores data regionally).

### 1.3 The Root User

When you first create an AWS account, AWS automatically creates a **root user** for you. This root user:

- Has **complete, unrestricted access** to every resource and every service in the account.
- Is identified simply by the **email address and password** used to create the AWS account (it has no separate "username" the way IAM users do).
- Should **only** be used to perform the **initial setup** of your AWS account (e.g., creating your first IAM administrator user, setting up billing alerts, or performing certain account-level tasks that genuinely require root, such as closing the account).

> **Best Practice / Exam Tip:** After the initial account setup, the root user should **never be used again for day-to-day work**, and it should **never be shared** with anyone. Instead, you create individual IAM users for every person who needs access. This is one of the most frequently tested best practices on the CLF-C02 exam.

---

## 2. IAM Users

### 2.1 Definition

An **IAM user** represents **one physical person** (or, less commonly, one application) within your organization. The golden rule taught in the source material is:

> **One AWS (IAM) user = One physical person.**

If a colleague or friend needs access to your AWS account, you should **never** share your own credentials with them. Instead, you create a **separate IAM user** for them. This ensures:

- **Accountability** — actions can be traced back to the specific individual who performed them (important for auditing via services like AWS CloudTrail, which is covered elsewhere in the course).
- **Granular permission control** — each person only gets the access they specifically need.
- **Easy revocation** — if someone leaves the organization, you simply disable or delete their specific IAM user without affecting anyone else.

### 2.2 Creating IAM Users (Console Walkthrough)

When creating a user through the IAM console, you typically go through the following steps:

1. **Provide a username** (e.g., "Stephane").
2. **Decide whether the user needs Management Console access.** If so:
   - AWS offers the option to use **IAM Identity Center** (recommended by AWS as the modern, more centralized way to manage workforce access, especially across multiple accounts) **or** to create a traditional **IAM user** directly (simpler, and still very relevant/testable for the exam).
3. **Set a password**:
   - You can let AWS **auto-generate** a password, and force the user to change it upon first login (recommended default for users you are creating for someone else).
   - Or you can set a **custom password** and choose whether the user must change it at next sign-in.
4. **Assign permissions** — you can:
   - Add the user **directly** to permissions (attach a policy directly to the user).
   - **Add the user to a group** (recommended best practice).
   - Copy permissions from an existing user.
5. **Add tags (optional)** — Tags are **key-value metadata pairs** that can be attached to almost any AWS resource, including IAM users. For example, you might tag a user with `Department = Engineering`. Tags are optional but extremely useful for:
   - Organizing and categorizing resources.
   - Cost allocation and billing reports (tracking spend by department, project, or environment).
   - Automation and access control (some IAM policies can use tags as conditions).
6. **Review and create** the user. AWS will then let you download the sign-in credentials (or email sign-in instructions).

### 2.3 Signing In as an IAM User

- Every AWS account has a unique **Account ID** and a **sign-in URL** for IAM users, which by default looks like: `https://<account-id>.signin.aws.amazon.com/console`
- You can customize this URL by creating an **account alias** (a unique, human-readable name), which changes the sign-in URL to something like: `https://<your-alias>.signin.aws.amazon.com/console`. The alias must be globally unique across all of AWS.
- To sign in as an IAM user (as opposed to the root user), you must select "IAM user" on the sign-in page and provide the **account ID or alias**, plus the **IAM username and password**.
- A common practical trick for testing/demoing: open a **private/incognito browser window** to stay logged in simultaneously as both the root user (in a normal window) and an IAM user (in the private window), since AWS sign-in sessions are tied to the browser session/cookies.

> **Exam Tip:** Losing access to *both* your root account credentials *and* your admin IAM user credentials can leave you locked out of your AWS account, potentially requiring a support case with AWS to regain access. Always safeguard both.

---

## 3. IAM Groups

### 3.1 Definition and Purpose

A **group** in IAM is a collection of IAM users. Groups exist to make **permission management simpler and more scalable** — instead of attaching a policy to every individual user, you attach it once to the group, and every user in that group automatically inherits those permissions.

### 3.2 Key Rules About Groups

- **Groups can only contain users.** A group **cannot** contain another group. This nested-group structure is explicitly **not supported** in IAM, and this is a commonly tested nuance.
- **A user does not have to belong to any group.** It is technically possible for a user to exist with no group membership, but this is **not a best practice**, since permissions become harder to manage/audit at scale.
- **A single user can belong to multiple groups simultaneously.** This is a very common real-world pattern.

### 3.3 Worked Example

Consider an organization with six employees: Alice, Bob, Charles, David, Edward, and Fred.

- Alice, Bob, and Charles are developers → grouped into a **Developers** group.
- David and Edward work in operations → grouped into an **Operations** group.
- Charles and David are also both on the audit team → grouped into an **Audit Team** group.
- Fred does not belong to any group (not best practice, but technically valid).

This means:
- **Charles** belongs to **two** groups (Developers and Audit Team) and inherits the union of both groups' policies.
- **David** belongs to **two** groups (Operations and Audit Team) and inherits the union of both groups' policies.
- **Fred** has no group-based permissions unless he is given permissions directly (e.g., via an inline policy).

### 3.4 How Permissions Combine

When a user belongs to multiple groups, **all attached policies from every group apply simultaneously**, in addition to any policy attached directly to the user (an **inline policy**, discussed in Section 5.4). IAM evaluates all applicable policies together to determine the user's effective permissions.

---

## 4. IAM Roles

### 4.1 Why Roles Exist

**IAM Roles** are the third major IAM identity type, alongside users and groups — but with a fundamentally different purpose. Roles are **not** meant for physical people; they are meant to grant permissions to **AWS services** (or other trusted entities) so that those services can perform actions **on your behalf**.

For example: an **Amazon EC2 instance** (a virtual server, covered in a later section of the course) often needs to call other AWS services — for example, to read an object from S3 or write a log entry to CloudWatch. The EC2 instance itself is not a "user," so it cannot log in with a username and password. Instead, you attach an **IAM Role** to the EC2 instance. The instance then **assumes** that role, and any AWS API calls it makes are evaluated against the permissions defined in that role's attached policy.

### 4.2 How a Role and a Service Combine

A role and the service (e.g., an EC2 instance) that assumes it effectively become **one logical entity** for the purpose of making authorized API calls:

1. The EC2 instance is launched with an IAM Role attached (this is done via an **Instance Profile**, which is a container that passes the role to the EC2 instance — a detail worth knowing conceptually even if the transcript doesn't name it explicitly).
2. When the instance needs to call an AWS API (e.g., `s3:GetObject`), it uses temporary security credentials associated with the assumed role.
3. IAM checks whether the role's attached policy allows that specific action; if yes, the call succeeds, if no, it is denied.

### 4.3 Common Use Cases for Roles

- **EC2 Instance Roles** — the most common example, allowing virtual servers to interact with other AWS services securely, **without embedding long-term access keys inside the instance** (a major security win, since hardcoded credentials on a server are a significant risk if the server is compromised).
- **Lambda Function Roles** — AWS Lambda (a serverless compute service covered later in the course) needs a role to be able to access other resources like DynamoDB tables, S3 buckets, etc.
- **CloudFormation Roles** — CloudFormation (an Infrastructure-as-Code service) uses roles to create/modify/delete the actual AWS resources described in a CloudFormation template.

### 4.4 Creating a Role (Console Walkthrough)

The console offers several types of "trusted entities" you can create a role for, but the type relevant here (and heavily tested) is a **role for an AWS service**:

1. Choose **AWS service** as the trusted entity type.
2. Choose which service the role is for (e.g., **EC2**, **Lambda**, or virtually any AWS service).
3. **Attach a policy** — for example, attaching the managed policy `IAMReadOnlyAccess` would let an EC2 instance read (but not modify) IAM configuration.
4. **Name the role** (e.g., `DemoRoleForEC2`).
5. The **trust policy** is automatically configured to specify which service (in this case EC2) is allowed to "assume" this role — this is what distinguishes a service role from a user or group.
6. Review and create.

Once created, the role can later be **attached to an actual EC2 instance** when that instance is launched (this integration happens in the EC2 section of a broader course, since Roles by themselves don't "do" anything until something assumes them).

> **Exam Tip:** A very common exam scenario tests whether you know **not to store AWS Access Keys directly on an EC2 instance**. The correct, secure answer is always: **attach an IAM Role to the EC2 instance instead.**

---

## 5. IAM Policies

### 5.1 What is a Policy?

An **IAM Policy** is a **JSON document** that defines permissions. It specifies:
- What actions are allowed or denied,
- On which resources,
- For which principal (user, group, or role).

Despite being written in JSON, policies are conceptually simple — they describe, in a structured but essentially "plain English"-like way, what an identity **can** or **cannot** do. You do not need to be a programmer to read or write basic IAM policies.

### 5.2 The Principle of Least Privilege

A core security philosophy embedded throughout IAM is the **Principle of Least Privilege**:

> **Never grant more permissions than a user (or role) actually needs to perform their job.**

If a new user is given broad, unrestricted permissions by default, they could accidentally (or maliciously) launch expensive resources, delete critical infrastructure, or create serious security vulnerabilities. By granting only the specific permissions required — for example, access to exactly three services a user needs — you dramatically reduce the "blast radius" of a mistake or a compromised credential.

> **Exam Tip:** "Least Privilege" is one of the single most frequently tested security concepts on the CLF-C02 exam. Any exam answer choice that involves granting broad/administrator access "just to be safe" or "for convenience" is almost always the *wrong* answer.

### 5.3 Anatomy of an IAM Policy (JSON Structure)

A typical IAM policy document is structured as follows:

```json
{
  "Version": "2012-10-17",
  "Id": "OptionalPolicyId",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": [
        "ec2:Describe*",
        "elasticloadbalancing:Describe*",
        "cloudwatch:*"
      ],
      "Resource": "*"
    }
  ]
}
```

Breaking down each component:

| Element | Meaning |
|---|---|
| **Version** | The policy language version. It is almost always the fixed literal string `"2012-10-17"` — this is simply the current, stable version of the IAM policy grammar, and does not change. |
| **Id** | An optional identifier for the overall policy document. |
| **Statement** | An array containing one or more individual permission statements. Every policy has at least one statement. |
| **Sid** (Statement ID) | An optional identifier for an individual statement within the policy, useful for readability/organization. |
| **Effect** | Either `"Allow"` or `"Deny"` — determines whether the statement grants or explicitly blocks the specified actions. |
| **Principal** | Specifies **which** account, user, or role the policy applies to (mainly relevant in resource-based policies, such as an S3 bucket policy, rather than identity-based policies attached to a user). |
| **Action** | The list of specific API calls/operations being allowed or denied (e.g., `ec2:Describe*`, `s3:GetObject`). |
| **Resource** | The specific AWS resource(s) the actions apply to (e.g., a specific S3 bucket ARN, or `*` for "all resources"). |
| **Condition** (optional, not always present) | Additional logical conditions under which the statement is in effect (e.g., only allow access from a specific IP range, or only during certain times). |

> **Exam Tip:** For the CLF-C02 exam, you are expected to understand and recognize the **Effect, Principal, Action, and Resource** elements at a conceptual level — you will not be asked to write complex policies from scratch, but you should be able to read a simple policy and explain what it does.

### 5.4 Types of Policy Attachment

There are several ways permissions can reach a user:

1. **Attached to a Group** — the policy applies to every user who is a member of that group (the most scalable, recommended approach).
2. **Attached Directly to a User** — sometimes called attaching a **managed policy** directly.
3. **Inline Policy** — a policy created and embedded directly on a single specific user (or role/group), existing only as part of that one identity, rather than as a standalone, reusable object. Useful for one-off, unique permission requirements that shouldn't be reused elsewhere.
4. **Inherited via Multiple Groups** — as shown in the Charles/David example above, a user can simultaneously receive permissions from more than one group.

A user's **effective permissions** are the union of everything granted through all of these attachment paths combined — direct attachments, inline policies, and every group they belong to.

### 5.5 AWS Managed Policies vs. Custom Policies

- **AWS Managed Policies** are pre-built, ready-to-use policies maintained by AWS itself. Examples referenced include:
  - **`AdministratorAccess`** — grants `Action: "*"` on `Resource: "*"` with `Effect: Allow`, meaning it authorizes **every possible action on every possible resource** — full administrative control of the account. The asterisk (`*`) is a **wildcard** meaning "anything/everything."
  - **`IAMReadOnlyAccess`** — grants read-only / list-only visibility into IAM: e.g., `Get*` (any API call starting with "Get," such as `GetUser`) and `List*` (any API call starting with "List," such as `ListUsers` or `ListGroups`), all on `Resource: "*"`. Notably, this **does not** allow *creating* or *modifying* anything — a user with only this policy could view users and groups but could **not** create a new group, for example.
  - **`IAMFullAccess`** — grants full create/modify/delete permissions specifically for the IAM service (needed if a user should be able to create groups, users, etc., beyond just read-only visibility).
- **Custom (Customer-Managed) Policies** — created by you, either through:
  - The **Visual Editor**, a no-code, form-based UI for selecting a service (e.g., IAM), specific actions (e.g., `ListUsers`, `GetUser`), and target resources — ideal for people who are less comfortable writing raw JSON.
  - The **JSON Editor**, where you write or paste the policy document directly.

### 5.6 Practical Demonstration Insights

The transcript walks through several illustrative real-world scenarios that reveal how IAM permission evaluation actually behaves:

- **Removing a user from a group immediately revokes inherited permissions.** When the demo user "Stephane" was removed from the `admin` group, subsequent API calls (e.g., `aws iam list-users`) were immediately denied with an **Access Denied** error — proving that IAM permission checks happen live, on every request, not just at login time.
- **Console and CLI enforce the exact same permission set.** Whatever is denied in the Management Console UI is denied identically via the CLI, and vice versa, because both interfaces call the same underlying AWS APIs, which are gated by the same IAM policies.
- **Attaching a narrow policy (like `IAMReadOnlyAccess`) allows viewing but not modifying.** The demo user could view existing groups and users but received an error attempting to *create* a new group — a clear, concrete illustration of least privilege in action.
- **A user can simultaneously hold permissions from multiple sources** — e.g., `AdministratorAccess` (via the `admin` group), a separate managed policy like `AlexaForBusiness` (via a second group, e.g., `developers`), and `IAMReadOnlyAccess` (attached directly) — all three appearing simultaneously on that single user's "Permissions" tab, each labeled with **where it came from** (inherited from a specific group, vs. directly attached).

---

## 6. Multiple Ways to Access AWS

AWS explicitly supports **three distinct methods** of interacting with your account, and understanding each — along with what secures each — is directly testable.

### 6.1 The Management Console

- The **web-based graphical interface** used throughout most introductory hands-on demonstrations.
- Protected by a **username and password**, and optionally strengthened with **Multi-Factor Authentication (MFA)**.

### 6.2 The AWS Command Line Interface (CLI)

- **CLI** stands for **Command Line Interface** — a tool installed on your local computer (Windows, macOS, or Linux) that lets you interact with AWS services by typing text commands into a terminal/shell, all of which begin with the word `aws` (e.g., `aws s3 cp`, `aws iam list-users`).
- Protected by **Access Keys** (an Access Key ID + a Secret Access Key), which function like a username/password pair specifically for programmatic access.
- Key facts about the CLI:
  - It is **open-source** — its full source code is publicly available on GitHub.
  - It provides **direct access to the public APIs** of AWS services.
  - It enables you to **write scripts** to automate resource management tasks.
  - It is an **alternative to the Management Console** — some experienced practitioners exclusively use the CLI and never touch the console UI.
  - **AWS CLI v2** is the current/latest major version, offering improved performance, installation experience, and additional capabilities over v1, while using the exact same underlying APIs.
  - It is installed differently depending on OS:
    - **Linux**: download a zip installer, unzip it, and run the install script via `sudo`.
    - **Windows**: download and run an **MSI installer**, following a standard install wizard.
    - **macOS**: download and run a **.pkg graphical installer**.
  - After installation, running `aws --version` confirms a successful install (returning version info such as `aws-cli/2.x.x Python/x.x.x <OS>/... botocore/x.x.x`).
  - Before use, the CLI must be configured via `aws configure`, which prompts for:
    - **AWS Access Key ID**
    - **AWS Secret Access Key**
    - **Default region name** (choose a region geographically close to you for lower latency)
    - **Default output format**

### 6.3 The AWS Software Development Kit (SDK)

- **SDK** stands for **Software Development Kit** — a set of **language-specific libraries** that let developers call AWS APIs **programmatically from within application code**, rather than from a terminal.
- Also protected by **Access Keys**, just like the CLI.
- Supports a wide variety of programming languages, explicitly including: **JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, and C++.**
- There are also specialized SDKs for:
  - **Mobile** (Android and iOS applications).
  - **IoT (Internet of Things)** devices — e.g., connected sensors or smart locks.
- **Interesting technical detail:** the AWS CLI itself is actually **built on top of the AWS SDK for Python**, called **Boto** (specifically, `botocore`/`boto3`). This illustrates that the CLI is essentially a command-line wrapper around SDK functionality.

### 6.4 Access Keys — Critical Security Notes

- Access Keys consist of two parts: an **Access Key ID** (like a username) and a **Secret Access Key** (like a password).
- Generated through the Management Console (under your user's **Security Credentials** tab).
- **The Secret Access Key is shown to you only once**, at the moment of creation — if lost, you must deactivate/delete that key and generate a new one; AWS cannot show it to you again.
- **Never share Access Keys with anyone**, including colleagues — each individual should generate and use their own.
- When creating access keys via the console, AWS proactively suggests **more secure alternatives** depending on your intended use case, such as:
  - **AWS CloudShell** (a browser-based terminal, discussed below) for ad-hoc CLI usage without needing to manage long-lived keys.
  - **IAM Identity Center** for federated CLI/SDK authentication (a more advanced, centralized identity solution not deeply covered in introductory material).
- **Best Practice:** Rotate (regularly regenerate/replace) access keys periodically to limit the impact of any potentially leaked credentials.

---

## 7. AWS CloudShell

**CloudShell** is a **browser-based terminal environment**, accessible directly from an icon in the top navigation bar of the Management Console, that comes **pre-configured with the AWS CLI already installed** and automatically authenticated using your currently logged-in console credentials.

Key characteristics:

- **Free to use.**
- **Not available in every AWS Region** — availability should be checked before relying on it; if unavailable in your current region, you may need to switch to a supported region or simply use a locally installed CLI/terminal instead.
- API calls made from within CloudShell automatically use the **credentials of the console user currently logged in** — no need to separately configure access keys.
- The **default region** for CLI commands run in CloudShell is the region **currently selected in the console** (rather than requiring a `--region` flag on every command, or a locally configured default).
- Provides **persistent storage**: files you create in your CloudShell environment (e.g., via `echo "test" > demo.txt`) **persist across sessions**, even if you restart CloudShell.
- Supports **customization**: adjustable font size (small/medium/large) and color theme (light/dark).
- Supports **uploading and downloading files** directly through the CloudShell UI — a very convenient feature for moving files between your local machine and the AWS environment without extra tooling.
- Supports **multiple terminal tabs/splits** within a single CloudShell session, allowing you to run multiple terminals side-by-side.

> **Use Case:** CloudShell is ideal when you want quick, authenticated CLI access without installing or configuring anything locally — useful for quick checks, scripting, or troubleshooting directly from the browser.

---

## 8. Securing IAM Identities

### 8.1 IAM Password Policy

A **Password Policy** is an account-wide (or Identity Center-wide) configuration that enforces stronger password hygiene for all IAM users. Configurable options include:

- **Minimum password length.**
- **Requiring specific character types**, such as:
  - At least one uppercase letter
  - At least one lowercase letter
  - At least one number
  - At least one non-alphanumeric (special) character (e.g., `?`, `!`, `%`)
- **Allowing (or disallowing) IAM users to change their own password.**
- **Enabling password expiration**, forcing users to set a new password after a defined period (e.g., every 90 days).
- Optionally requiring **administrator intervention** to reset an expired password, rather than allowing self-service reset.
- **Preventing password reuse** — stopping users from cycling back to a previous password when forced to change it.

> **Why it matters:** A strong password policy is a primary defense against **brute-force attacks**, where an attacker systematically guesses passwords. Longer, more complex, regularly-rotated passwords make such attacks computationally infeasible in practice.

You configure this via **IAM Console → Account Settings → Password Policy**, where you can either accept the AWS default policy or fully customize each rule.

### 8.2 Multi-Factor Authentication (MFA)

**MFA (Multi-Factor Authentication)** adds a **second, independent layer of security** on top of the traditional username/password combination.

**Core concept:** MFA combines...
1. **Something you know** (your password), **with**
2. **Something you have** (a physical or virtual security device generating a time-based one-time code).

Even if an attacker steals or guesses a user's password, they **cannot log in** without also physically possessing the user's MFA device — dramatically reducing the risk of account compromise from stolen credentials alone.

> **Exam Tip:** MFA is described as an **absolute must** for protecting the AWS **root user**, and is strongly recommended for **all** IAM users, especially those with elevated (administrative) permissions, since a compromised admin account could allow an attacker to change configurations, delete resources, exfiltrate data, or rack up massive costs.

#### 8.2.1 MFA Device Options in AWS

| MFA Device Type | Details |
|---|---|
| **Virtual MFA Device** | A software-based authenticator app running on a smartphone. Examples: **Google Authenticator** (supports one account per device instance) and **Authy** (supports multiple accounts/tokens on a single device). This is the most common and accessible option, allowing you to protect multiple accounts/users (root, various IAM users) from a single phone. |
| **Universal 2nd Factor (U2F) Security Key** | A **physical hardware device** — e.g., a **YubiKey** (made by **Yubico**, a third-party vendor, not AWS itself). Small enough to attach to a keychain. A single U2F key can support authentication for multiple root and IAM users. |
| **Hardware Key Fob MFA Device** | A dedicated physical hardware token that generates rotating numeric codes, e.g., devices provided by **Gemalto** (again, a third-party vendor). |
| **AWS GovCloud (US) Hardware Token** | A specialized hardware key fob provided specifically for the **AWS GovCloud (US)** environment (used for U.S. government workloads with special compliance requirements), supplied by the third-party vendor **SurePassID**. |

> **Exam Tip:** Know that **virtual MFA (authenticator apps)** is the most common/tested option, and that **U2F/hardware key fobs** are physical alternatives provided by **third-party vendors**, not built/manufactured by AWS itself.

#### 8.2.2 Setting Up MFA (Console Walkthrough)

1. Navigate to your account name → **Security Credentials**.
2. Under MFA, choose to **assign an MFA device** and give it a memorable name.
3. Select the device type (e.g., **Authenticator app**).
4. AWS displays a **QR code**; scan it using a supported authenticator app on your phone (Google Authenticator, Authy, or similar compatible apps for Android/iOS).
5. The app begins generating a rotating **6-digit numeric code** that changes periodically (typically every 30 seconds).
6. AWS requires you to **enter two consecutive codes** (from two consecutive generation cycles) to confirm that:
   - The device is correctly synced.
   - The codes it generates are accurate and valid.
7. Once verified, the MFA device is activated. An AWS account can register **up to 8 MFA devices**.
8. **From then on**, logging in with that user/root account requires entering the current password **plus** the current MFA code shown in the authenticator app.

> **Important Caution:** If you lose access to your MFA device (e.g., you lose your phone) without a backup, you can be **locked out of your own account**, potentially requiring a support process with AWS to regain access. Always ensure you have a reliable way to keep or recover your MFA device before enabling this on critical accounts. You *can* remove/deactivate an MFA device later if needed (e.g., when replacing a phone).

---

## 9. IAM Security & Auditing Tools

IAM provides two key built-in tools to help you continuously monitor and tighten your security posture, directly supporting the principle of least privilege.

### 9.1 IAM Credentials Report

- **Scope: Account-level.**
- Generated on demand from the IAM console (**Credential Report → Download Credential Report**), producing a downloadable **CSV file**.
- Contains **one row per user in the account** (including the root user), showing details such as:
  - When the user was created.
  - Whether a password is enabled, and when it was last used/changed.
  - When the next password rotation is expected (if password expiration is enabled).
  - Whether **MFA is active** for that user.
  - Whether **access keys** exist for that user, and when they were last rotated/used.
  - Information about certificates associated with the user, where relevant.
- **Use Case:** Quickly identify accounts with **stale, unused, or non-rotated credentials** — for example, spotting a user who has never enabled MFA, or an access key that hasn't been rotated in a long time — allowing you to proactively tighten security.

### 9.2 IAM Access Advisor

- **Scope: Per-user level** (viewed from an individual user's detail page, under the **Access Advisor** tab).
- Shows **which AWS services a specific user has actually accessed permission to, and the last time they accessed each one**.
- **Use Case:** Directly supports the **principle of least privilege** — by reviewing which permissions/services a user is actually using (versus which they merely have access to but never use), an administrator can **identify and remove excessive/unused permissions**, tightening the user's access down to only what is genuinely needed.
- Can also show *how* a permission was granted (e.g., "granted via the AdministratorAccess policy"), giving administrators visibility into the source of a given access grant, which is helpful for troubleshooting and cleanup.

> **Exam Tip:** Remember the distinction — the **Credentials Report** is an **account-wide** audit of credential health/status, while **Access Advisor** is a **per-user** tool showing actual service usage patterns to help right-size permissions.

---

## 10. IAM Best Practices Summary

The source material distills the following concrete best practices, which are extremely likely to appear (directly or indirectly) on the CLF-C02 exam:

- **Do not use the root account** for anything beyond initial account setup.
- **One physical person = one IAM user.** Never share credentials; create a separate user for every individual.
- **Assign users to groups**, and manage permissions **at the group level** rather than per-user, for scalability and easier auditing.
- **Enforce a strong password policy** to protect against brute-force attacks.
- **Enable and enforce Multi-Factor Authentication (MFA)**, especially for the root account and for any administrative users.
- **Create and use IAM Roles** whenever granting permissions to AWS services (e.g., EC2 instances) — never hardcode long-term credentials inside a service/application.
- **Generate Access Keys only when needed** for CLI/SDK programmatic access, and treat them with the same secrecy as a password.
- **Regularly audit permissions** using the **IAM Credentials Report** and **IAM Access Advisor**.
- **Never, under any circumstances, share IAM user credentials or access keys** with anyone else.

---

## 11. IAM and the Shared Responsibility Model

The **Shared Responsibility Model** is a heavily emphasized, recurring exam topic that describes the division of security obligations between **AWS** and **the customer**. It applies across virtually every AWS service, and IAM offers one of the clearest illustrations of it.

### AWS is responsible for ("Security *of* the Cloud"):
- The **global infrastructure** and **network security** underlying all AWS services.
- The **configuration and vulnerability management** of the services AWS offers (e.g., patching the underlying IAM service software itself).
- Ensuring AWS's own **compliance certifications** are maintained (e.g., data center physical security, hardware lifecycle management).

### You (the customer) are responsible for ("Security *in* the Cloud"):
- **Creating and managing** your own IAM users, groups, roles, and policies.
- **Managing and reviewing** those policies to ensure they follow least privilege.
- **Enabling and enforcing MFA** on all relevant accounts — AWS makes it available, but does not force you to turn it on.
- **Rotating your own access keys** regularly.
- **Using IAM tools appropriately** to apply correct/appropriate permissions.
- **Analyzing access patterns and reviewing permissions** over time (e.g., via Credentials Reports and Access Advisor) — AWS provides the tools, but the ongoing analysis and governance is your job.

> **Exam Tip:** A simple mental model: **AWS secures the cloud infrastructure itself; you are responsible for how you configure and use the security tools (like IAM) within that infrastructure.** Expect exam questions phrased like "Who is responsible for enabling MFA on IAM user accounts?" — the answer is always **the customer**, never AWS.

---

## 12. Comprehensive IAM Summary

Pulling every concept together into one cohesive picture:

- **IAM Users** represent individual physical people and can log into the Management Console with a username and password.
- **IAM Groups** contain only users (never other groups) and are used to apply permissions collectively and consistently.
- **IAM Policies** are JSON documents that define permissions (Effect, Principal, Action, Resource, and optionally Condition) and can be attached to users directly (including as inline policies) or to groups (with inheritance to all group members).
- **IAM Roles** are identities intended for AWS services (like EC2 instances, Lambda functions, or CloudFormation) rather than for people, and are the secure, recommended mechanism for granting AWS services permission to act on your behalf.
- **Security hardening** is achieved through a strong **Password Policy** and **Multi-Factor Authentication (MFA)**.
- **Access** to AWS can happen through the **Management Console** (username/password + optional MFA), the **CLI** (access keys), or the **SDK** (access keys, embedded in application code) — with **CloudShell** offering a convenient, browser-based, pre-authenticated alternative to a locally installed CLI.
- **Auditing** is performed using the **IAM Credentials Report** (account-wide credential health) and the **IAM Access Advisor** (per-user service usage, supporting least-privilege refinement).

Mastering these building blocks is essential not just for the CLF-C02 exam's IAM-specific questions, but as a conceptual foundation for understanding security and access control across virtually every other AWS service covered later in your studies (EC2, S3, Lambda, RDS, and beyond).
