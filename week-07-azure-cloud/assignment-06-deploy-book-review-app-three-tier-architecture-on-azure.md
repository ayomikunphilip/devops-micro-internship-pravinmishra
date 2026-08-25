# Assignment 6 — Capstone: Deploy Book Review App (Three-Tier Architecture) on Azure

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

This is the most important assignment of the course. You will deploy the Book Review App in a production-ready, best-practice-compliant three-tier architecture on Azure: separated presentation, application, and database tiers, least-privilege network access, a controlled public entry point, protected secrets, and availability/monitoring evidence.

---

# Task 1 — Design the Azure Three-Tier Architecture

## Goal

Create an architecture diagram and implementation plan identifying the presentation, application, and database components, the chosen Azure services, the public entry point, and the internal traffic paths.

### Evidence

#### Screenshot 1 — Architecture diagram showing the public entry point, three tiers, network boundaries, and traffic flow

![Screenshot 1](screenshots/A6-S1.png)

---

#### Screenshot 2 — Written architecture assumptions and selected Azure services
![Screenshot 2](screenshots/A6-S2.png) Region and naming: All resources are deployed in a single Azure region for simplicity and to avoid cross-region latency and cost. Resource and subnet names follow a consistent <tier>-bookreview convention so the role of each component is clear from its name alone.

Public entry point: An Azure Standard Load Balancer (public) is the only resource in the architecture that accepts traffic from the internet. It forwards HTTP traffic to the web tier VM on port 80.

Web tier: A single Ubuntu VM runs Nginx in front of a Next.js frontend. Nginx serves the site directly and reverse-proxies any request under /api/ to the application tier over the private network. This is how the web tier sends application requests to the internal application-tier endpoint, since the app tier has no public IP and the browser cannot reach it directly.

Application tier: A single Ubuntu VM runs the Node.js/Express backend on port 3001. It has no public IP and only accepts inbound traffic from the web subnet, on the API port and SSH (for management via jump host).

Database tier: Azure Database for MySQL Flexible Server, configured with private access (VNet integration). Public network access is disabled. Only the application subnet can reach it.

Secrets: The database admin password and the backend's JWT secret are generated and stored in Azure Key Vault, then pulled onto the VMs via the Azure CLI when writing environment files. Neither value is typed by hand into a file that gets screenshotted, or committed to source control.

Monitoring: A single Log Analytics Workspace collects diagnostics from both VMs, the Load Balancer, the Key Vault, and the MySQL server. One alert rule monitors VM CPU usage.

Availability: Each tier runs a single VM (Standard_B1s) rather than a scale set, due to the cost constraints of a student/free-tier subscription. This is a deliberate, documented trade-off rather than an oversight, and the Task 8 availability test is scoped accordingly — it demonstrates that the Load Balancer correctly detects and reacts to a VM outage, not zero-downtime failover.

Backup and recovery: Azure Database for MySQL Flexible Server's automatic daily backups (7-day retention by default) are used as-is, with no additional configuration required.

---

# Task 2 — Create the Azure Network Foundation

## Goal

Create a dedicated Resource Group and VNet with separate subnets for the web, application, and database tiers, keeping the application and database tiers without direct public access.

### Evidence

#### Screenshot 3 — Resource Group overview showing the assignment resources

![Screenshot 3](screenshots/A6-S3.png)

---

#### Screenshot 4 — VNet overview showing the address space and all required subnets

![Screenshot 4](screenshots/A6-S4.png)

---

#### Screenshot 5 — Route-table or Private DNS evidence where applicable

![Screenshot 5](screenshots/A6-S5.png)

---

# Task 3 — Configure Security and Secret Management

## Goal

Apply least-privilege NSG rules so traffic flows Internet → public entry point → web tier → application tier → database tier, and store credentials in Azure Key Vault or another approved secure mechanism.

### Evidence

#### Screenshot 6 — NSG rules proving least-privilege access between the tiers

![Screenshot 6a](screenshots/A6-S6a.png)  ![Screenshot 6b](screenshots/A6-S6b.png)

---

#### Screenshot 7 — Key Vault or approved secret-management configuration (without displaying secret values)

![Screenshot 7](screenshots/A6-S7.png)

---

# Task 4 — Deploy the Presentation (Web) Tier

## Goal

Deploy the Book Review App presentation layer on the approved web-tier compute service, configured to route requests to the internal application-tier endpoint, and not directly exposed except through the public entry service.

### Evidence

#### Screenshot 8 — Web-tier compute overview showing subnet and availability configuration

![Screenshot 8](screenshots/A6-S8.png)

---

#### Screenshot 9 — Terminal or service output proving the presentation layer is running

![Screenshot 9](screenshots/A6-S9.png)

---

# Task 5 — Deploy the Business (Application) Tier

## Goal

Deploy the Book Review App backend privately in the application subnet, configured to use the private database endpoint and secured environment values, reachable only through its internal endpoint.

### Evidence

#### Screenshot 10 — Application-tier compute overview showing private subnet placement

![Screenshot 10](screenshots/A6-S10.png)

---

#### Screenshot 11 — Backend process, service, or listening-port evidence

![Screenshot 11](screenshots/A6-S11.png)

---

#### Screenshot 12 — Internal health-check or API response (without exposing secrets)

![Screenshot 12](screenshots/A6-S12.png)

---

# Task 6 — Deploy the Managed Database Tier

## Goal

Create a private Azure managed database (public access disabled), with availability/backup/retention settings, the Book Review App schema imported, and access restricted to the application tier only.

### Evidence

#### Screenshot 13 — Database overview showing private connectivity and public access disabled

![Screenshot 13](screenshots/A6-S13.png)

---

#### Screenshot 14 — Availability, backup, and retention configuration

![Screenshot 14](screenshots/A6-S14.png)

---

#### Screenshot 15 — Successful schema or connectivity verification (without exposing credentials)

![Screenshot 15](screenshots/A6-S15.png)

---

# Task 7 — Configure Traffic Management, Availability, and Monitoring

## Goal

Configure the approved public entry service with health probes and backend pools, internal routing for the application tier where required, and enable Azure Monitor/diagnostics/logs/alerts for the key resources.

### Evidence

#### Screenshot 16 — Public entry service showing listener, frontend endpoint, and healthy web targets

![Screenshot 16a](screenshots/A6-S16a.png)  ![Screenshot 16a](screenshots/A6-S16b.png)  ![Screenshot 16c](screenshots/A6-S16c.png)  ![Screenshot 16d](screenshots/A6-S16d.png)

---

#### Screenshot 17 — Internal application-tier load-balancing or routing configuration where applicable

![Screenshot 17](screenshots/A6-S17.png)

---

#### Screenshot 18 — Azure Monitor, diagnostic settings, logs, metrics, or alert evidence

![Screenshot 18a](screenshots/A6-S18a.png)   ![Screenshot 18b](screenshots/A6-S18b.png)

---

# Task 8 — Validate the Production-Style Deployment

## Goal

Confirm the Book Review App works end to end through the public endpoint, with at least one database read and one write, confirm private tiers are not internet-reachable, and complete a safe availability test.

### Evidence

#### Screenshot 19 — Browser showing the Book Review App through the public endpoint

![Screenshot 19](screenshots/A6-S19.png)

---

#### Screenshot 20 — Proof of successful database-backed read and write operations

![Screenshot 20](screenshots/A6-S20.png)

---

#### Screenshot 21 — Evidence that private tiers are not publicly accessible

![Screenshot 21](screenshots/A6-S21.png)

---

#### Screenshot 22 — Availability-test and healthy-target evidence

![Screenshot 22a](screenshots/A6-S22a.png)  ![Screenshot 22b](screenshots/A6-S22b.png)  ![Screenshot 22c](screenshots/A6-S22c.png) 

---

#### Public Endpoint

Paste your public endpoint URL here:

`http://20.87.241.53`

---

### Notes

Summarize what worked, issues encountered and how they were fixed, and the availability/security/secrets/monitoring/backup choices made.

What worked: Full three-tier deployment completed and verified end to end. Public Load Balancer routes to Nginx (web tier), which proxies /api/ requests privately to the app tier, which connects to a private MySQL Flexible Server. Registration, login, book browsing, and reviews all work through the public IP. Secrets live in Key Vault, and private tier isolation was confirmed from an external machine.

Issues encountered and fixes:

MySQL auth failure (ER_ACCESS_DENIED_ERROR): Azure's provisioned password hash didn't match what the driver expected. Fixed with ALTER USER to force a clean hash regeneration.
Load Balancer public IP unreachable despite healthy backend: The NSG only allowed the AzureLoadBalancer source tag, which covers health probes but not real client traffic, since a Standard LB preserves the original client IP. Added a rule allowing port 80 from Internet/Any.
Registration failing (connection refused to localhost:3001): The frontend had a hardcoded fallback URL that activated because an empty environment variable is falsy in JavaScript. Removed the fallback and rebuilt.
Double API prefix (404 on homepage): Two files handled the /api prefix inconsistently. Standardized the environment variable to /api and fixed the redundant hardcode.

Availability: Single VM per tier (Standard_B1s), a documented cost tradeoff. The availability test confirmed the Load Balancer correctly detects and recovers from a VM reboot, not zero downtime.

Security: NSGs follow least privilege. SSH to the web tier is IP restricted. App and database tiers have no public IPs. Default DenyAllInbound blocks anything not explicitly allowed.

Secrets: Database password and JWT secret generated and stored in Key Vault, pulled via Azure CLI when writing env files, never typed or committed in plaintext.

Monitoring: Log Analytics Workspace (law-bookreview) created. Diagnostic settings successfully configured for the Load Balancer, Key Vault, and MySQL server. VM guest diagnostics and the CPU alert rule were skipped due to the deprecated Azure Diagnostics extension and inconsistent Azure Monitor Agent enrollment in the portal. Platform level VM metrics remain visible via each VM's Monitor tab.

Backup: MySQL Flexible Server takes automatic daily backups with 7 day retention by default, confirmed in the portal.

---

# Submission Instructions

- Add all required screenshots and links in your submission
- Do not expose passwords, keys, connection strings, or subscription IDs

---

# Completion Checklist

- [ ] Task 1: Architecture diagram and assumptions documented (Screenshots 1–2)
- [ ] Task 2: Network foundation created with isolated tiers (Screenshots 3–5)
- [ ] Task 3: Least-privilege security and secret management configured (Screenshots 6–7)
- [ ] Task 4: Presentation tier deployed (Screenshots 8–9)
- [ ] Task 5: Application tier deployed privately (Screenshots 10–12)
- [ ] Task 6: Managed database tier deployed privately (Screenshots 13–15)
- [ ] Task 7: Public entry, internal routing, and monitoring configured (Screenshots 16–18)
- [ ] Task 8: End-to-end validation and availability test completed (Screenshots 19–22, Public Endpoint, Notes)
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
