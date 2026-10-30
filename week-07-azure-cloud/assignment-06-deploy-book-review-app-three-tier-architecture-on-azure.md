# Assignment 6 — Capstone: Deploy Book Review App (Three-Tier Architecture) on Azure

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

Deploy the Book Review App in a production-style three-tier architecture on Azure, with private application and database tiers, public and internal Load Balancers, protected secrets, monitoring, and restart-recovery testing.

---

# Task 1 — Design the Azure Three-Tier Architecture

## Goal

Create an architecture diagram and implementation plan showing the selected Azure services, subnet ranges, application ports, administration path, and traffic flow.

## Evidence

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

Create a Resource Group and VNet using `10.0.0.0/16`, with web subnet `10.0.1.0/24`, application subnet `10.0.2.0/25`, and delegated database subnet `10.0.3.0/26`. Associate a NAT Gateway with the web and application subnets and reserve a private frontend IP for the Internal Load Balancer.

## Evidence

#### Screenshot 2 — Resource Group overview showing the network resources created so far

![Screenshot 3](screenshots/A6-S3.png)

---

#### Screenshot 3 — VNet subnet configuration showing all three subnet names, CIDR ranges, and database subnet delegation

![Screenshot 4](screenshots/A6-S4.png)

---

#### Screenshot 4 — NAT Gateway configuration showing its Public IP and associations with the web and application subnets

![Screenshot 5](screenshots/A6-S5.png)

---

# Task 3 — Configure Security and Secret Management

## Goal

Configure NSGs for application, management, and health-probe traffic. Protect database credentials and the JWT signing secret. Create the Web VM with restricted SSH access and use it as the jump host for private App VM administration.

## Evidence

#### Screenshot 5 — Web, application, and database NSG rules showing permitted sources, ports, health-probe access, and applicable deny rules

![Screenshot 6a](screenshots/A6-S6a.png)  ![Screenshot 6b](screenshots/A6-S6b.png)

---

#### Screenshot 6 — Key Vault or the selected secret-management configuration, without displaying secret values

![Screenshot 7](screenshots/A6-S7.png)

---

# Task 4 — Deploy the Managed Database Tier

## Goal

Create Azure Database for MySQL Flexible Server using private access in the delegated database subnet. Configure Private DNS, keep secure transport enabled, disable public access, and document backup and availability settings.

## Evidence

#### Screenshot 7 — MySQL Flexible Server networking configuration showing the private subnet and Private DNS configuration

![Screenshot 8](screenshots/A6-S8.png)

---

#### Screenshot 9 — Terminal or service output proving the presentation layer is running

![Screenshot 9](screenshots/A6-S9.png)

---

# Task 5 — Deploy the Business or Application Tier

## Goal

Deploy the backend on one private Ubuntu VM using port **3001**. Connect to Azure MySQL over TLS, create `book_review_db` and the application database user, verify startup schema and sample-data initialization, and configure PM2 for automatic recovery.

## Evidence

#### Screenshot 9 — App VM networking configuration showing placement in the application subnet and no Public IP

![Screenshot 10](screenshots/A6-S10.png)

---

#### Screenshot 11 — Backend process, service, or listening-port evidence

![Screenshot 11](screenshots/A6-S11.png)

---

#### Screenshot 12 — Internal health-check or API response (without exposing secrets)

![Screenshot 12](screenshots/A6-S12.png)

---

# Task 6 — Deploy the Presentation or Web Tier

## Goal

Deploy the Next.js frontend on the existing Web VM on port **3000**. Configure Nginx on port **80** to forward page requests to the frontend and `/api/` requests to the Internal Load Balancer on port **3001**. Configure PM2 and verify recovery after reboot.

## Evidence

#### Screenshot 13 — Web VM overview or networking configuration showing its subnet and VM details

![Screenshot 13](screenshots/A6-S13.png)

---

#### Screenshot 14 — Nginx site configuration showing frontend and API routing, together with successful `sudo nginx -t` output

![Screenshot 14](screenshots/A6-S14.png)

---

#### Screenshot 15 — PM2 showing `book-review-frontend` online after reboot, with Nginx active and a successful local HTTP response

![Screenshot 15](screenshots/A6-S15.png)

---

# Task 7 — Configure Load Balancers, Health Checks, and Monitoring

## Goal

Configure a Standard Public Load Balancer using TCP **80 → 80** and a Standard Internal Load Balancer using TCP **3001 → 3001**. Verify backend health, update the allowed frontend origin, and configure monitoring and at least one alert rule.

## Evidence

#### Screenshot 16 — Public Load Balancer frontend IP, web backend membership, TCP 80 → 80 rule, and health-probe status

![Screenshot 16a](screenshots/A6-S16a.png)  ![Screenshot 16a](screenshots/A6-S16b.png)  ![Screenshot 16c](screenshots/A6-S16c.png)  ![Screenshot 16d](screenshots/A6-S16d.png)

---

#### Screenshot 17 — Internal Load Balancer private frontend IP, application backend membership, TCP 3001 → 3001 rule, and health-probe status

![Screenshot 17](screenshots/A6-S17.png)

---

#### Screenshot 18 — Monitoring evidence showing a relevant metric or collected log and the configured alert rule

![Screenshot 18a](screenshots/A6-S18a.png)   ![Screenshot 18b](screenshots/A6-S18b.png)

---

# Task 8 — Validate the Production-Style Deployment

## Goal

Verify registration, login, book viewing, and persistent review creation through the Public Load Balancer. Confirm private-tier restrictions and automatic service recovery after controlled App VM and Web VM restarts. Downtime is expected during these single-instance restart tests.

## Evidence

#### Screenshot 19 — Browser showing the Book Review App through the Public Load Balancer endpoint, with the address bar visible

![Screenshot 19](screenshots/A6-S19.png)

---

#### Screenshot 20 — Successful registration/login, book viewing, and review creation, with database evidence that the review persists

![Screenshot 20](screenshots/A6-S20.png)

---

#### Screenshot 21 — App VM networking showing no Public IP, MySQL private-access settings, and relevant NSG restrictions

![Screenshot 21](screenshots/A6-S21.png)

---

#### Screenshot 22 — Post-restart PM2/service status, healthy Load Balancer backends, and a successful application response

![Screenshot 22a](screenshots/A6-S22a.png)  ![Screenshot 22b](screenshots/A6-S22b.png)  ![Screenshot 22c](screenshots/A6-S22c.png) 

---

#### Public Endpoint

Paste your Public Load Balancer website URL here:

`http://20.87.241.53`

---

#### Deployment Notes

Summarize what worked, issues encountered, fixes applied, and sources used. Identify the backend route used for direct and Internal Load Balancer verification, including its expected response.

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

- Include your full name and Screenshots **1–23** under the corresponding evidence headings.
- Use multiple captures under an evidence heading where needed for readability.
- Include the working **public endpoint URL** and published **LinkedIn post URL**.
- Complete the architecture, database, recovery, and deployment notes.
- Ensure screenshots are readable and show your own environment.
- Do not expose passwords, private keys, tokens, connection strings containing credentials, subscription IDs, or other sensitive information.
- Submit your work using the method specified in the assignment guidelines.

---

# Completion Checklist

- [ ] Task 1: Architecture diagram and required notes completed (Screenshot 1)
- [ ] Task 2: VNet, subnets, database delegation, and NAT Gateway configured (Screenshots 2–4)
- [ ] Task 3: NSGs and secret management configured; Web VM created for administration and frontend hosting (Screenshots 5–6)
- [ ] Task 4: Private Azure MySQL server, Private DNS, and backup settings configured (Screenshots 7–8)
- [ ] Task 5: Private backend deployed, database initialized, and PM2 recovery verified (Screenshots 9–12)
- [ ] Task 6: Frontend and Nginx routing configured, with recovery verified (Screenshots 13–15)
- [ ] Task 7: Public and Internal Load Balancers healthy; monitoring and alert configured (Screenshots 16–18)
- [ ] Task 8: Application workflows, private-tier restrictions, and restart recovery verified (Screenshots 19–22)
- [ ] Architecture, database, recovery, and deployment notes completed
- [ ] LinkedIn post published with required hashtags, credit note, and tags (Screenshot 23)
- [ ] Full name, public endpoint URL, and LinkedIn post URL included
- [ ] All screenshots readable and no sensitive information exposed

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

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
