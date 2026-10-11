# Assignment 3 — Automate React App Deployment Using Azure DevOps CI/CD

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will create a multi-stage Azure DevOps pipeline that builds, tests, publishes, and deploys a React application to an Ubuntu VM hosted on AWS or Azure. The pipeline will automatically run when changes are committed to `main`, transfer the production build as an artifact, and deploy it through Nginx.

---

# Task 0 — Verify the Starting Environment

## Goal

Confirm that Azure DevOps, the pipeline agent, Terraform, Ansible, and the selected cloud environment are ready.

No submission screenshot is required for this task.

---

# Task 1 — Import and Personalize the React Application

## Goal

Import the React application into Azure Repos and add your Full Name and the current date.

## Evidence

### Screenshot 1 — Imported React Project in Azure Repos

Add a screenshot of Azure Repos showing:

* Imported React project
* Repository name
* `main` branch
* Project files

![Screenshot 1](screenshots/a3-s1.png)

---

# Task 2 — Provision and Configure the Target VM

## Goal

Provision an Ubuntu VM using Terraform and configure Nginx, React SPA routing, SSH access, and deployment permissions using Ansible.

No separate submission screenshot is required for this task.

---

# Task 3 — Create or Update the SSH Service Connection

## Goal

Create or update an Azure DevOps SSH Service Connection that allows the pipeline to connect securely to the target VM.

No separate submission screenshot is required for this task.

> Do not include the VM password, SSH private key, token, or another secret in the submission.

---

# Task 4 — Author the Multi-Stage Azure Pipeline

## Goal

Create an Azure Pipeline containing Build, Test, Publish, and Deploy stages with an automatic trigger for commits to `main`.

## Evidence

### Screenshot 2 — Multi-Stage Pipeline YAML

Add a screenshot of the Azure Pipeline YAML open in the editor showing:

* Trigger
* Build stage
* Test stage
* Publish stage
* Deploy stage

![Screenshot 2a](screenshots/a3-s2a.png) ![Screenshot 2b](screenshots/a3-s2b.png) ![Screenshot 2c](screenshots/a3-s2c.png) ![Screenshot 2d](screenshots/a3-s2d.png)

> Do not expose passwords, private keys, tokens, or cloud credentials.

---

# Task 5 — Run the Pipeline and Resolve Configuration Issues

## Goal

Complete a successful end-to-end pipeline run containing all four stages.

## Evidence

### Screenshot 3 — Successful Multi-Stage Pipeline Run

Add a screenshot of one Azure DevOps pipeline run showing all four stages succeeded:

* Build
* Test
* Publish
* Deploy

1[Screenshot 3](screenshots/a3-s3.png)

---

# Task 6 — Verify the Deployment on the VM

## Goal

Confirm that the pipeline deployed the production-ready React files to the correct Nginx web root.

## Evidence

### Screenshot 4 — Post-Deployment Contents of /var/www/html

Add a screenshot of the pipeline SSH verification log or VM terminal showing the post-deployment contents of:

`/var/www/html`

![Screenshot 4](screenshots/a3-s4.png)

---

# Task 7 — Verify the Website and Automatic Trigger

## Goal

Confirm that the React application is accessible and that a commit to `main` automatically triggers the CI/CD pipeline.

## Evidence

### Screenshot 5 — Deployed React Application

Add a browser screenshot showing:

* Deployed React application
* VM public IP address in the browser address bar
* Your Full Name
* Deployment date

![Screenshot 5](screenshots/a3-s5.png)

## Final Application URL

`http://52.90.205.180`

Replace the placeholder and paste your final application URL below:

[\[Paste your final application URL here.\]](http://52.90.205.180)

---

# CI/CD Workflow Summary

Write a short explanation of the CI/CD workflow you created.

I built an automated pipeline in Azure DevOps that takes a React application from source code to a live website. Every time I commit a change to the main branch, the pipeline starts by itself and carries the change through four stages.

Infrastructure: I used Terraform to create an AWS EC2 virtual machine with a public IP, an SSH key, and a security group that allows ports 22 and 80. I then used Ansible to configure two machines. On the web VM, Ansible installed Nginx, cleared the old static site, and set up Nginx to serve a React single-page application. On the self-hosted agent VM, which runs on Azure, Ansible installed Node.js and npm so it can build the app.

Pipeline stages:

Build: The agent installs Node.js and the app’s dependencies, then runs npm run build to turn the React source code into static files. It checks that build/index.html exists and saves the build folder as an artifact.
Test: The agent installs the dependencies again and runs the unit tests. If any test fails, the pipeline stops and nothing reaches the server.
Publish: The agent downloads the build artifact, checks that index.html and the static folder are present, and publishes it as a verified deployment package.
Deploy: The pipeline uses an SSH service connection to copy the deployment package to /var/www/html on the web VM. It then confirms that index.html is there, that Nginx is running, and that the server answers with HTTP 200.

Why it works well: Each stage only runs if the previous one succeeds, so a broken build or failing test never reaches the live site. Artifacts pass the build between stages safely. The SSH service connection keeps the server password out of the YAML file and out of the logs. Nginx serves the finished static files, and the try_files rule makes React routes work even when a page is refreshed.

Result: After the pipeline finished, the site was live at http://52.90.205.180 with my name and date on the page. A small commit to main then started a new run automatically and updated the live site without any manual step.

---

# LinkedIn Requirement

## Evidence

### Screenshot 6 — LinkedIn Post

Add a screenshot of your LinkedIn post showing:

* Post text
* At least one image or link

![Screenshot 6](screenshots/a3-s6.png)

## LinkedIn Post URL


https://www.linkedin.com/posts/ayomikunphilip_devops-cicd-azuredevops-ugcPost-7515189074455007232-ybEL/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAF4cLMMBGj_ND3_b5bGU28ywvq8aZAW62fs
> Do not expose VM passwords, tokens, private keys, cloud credentials, or other sensitive information.

---

# Submission Instructions

* Complete all tasks in sequence.
* Include the short CI/CD workflow summary.
* Include Screenshots 1–6.
* Include the final application URL.
* Include the public LinkedIn post URL.
* Confirm that all screenshots are readable and show the required context.
* Do not expose passwords, PATs, private keys, cloud credentials, subscription IDs, account IDs, or other secrets.
* Follow the Assignment Submission Guidelines.

---

# Completion Checklist

* [ ] All tasks were completed in sequence
* [ ] The correct React repository was imported into Azure Repos
* [ ] Your Full Name and date were added to the application
* [ ] The pipeline YAML was authored and committed to the repository
* [ ] Commits to `main` trigger the pipeline automatically
* [ ] The pipeline contains Build, Test, Publish, and Deploy stages
* [ ] All four stages succeeded in the same pipeline run
* [ ] The production build moved between stages as a pipeline artifact
* [ ] The Deploy stage used the SSH Service Connection
* [ ] No password or secret is stored in the YAML
* [ ] `index.html` is directly inside `/var/www/html`
* [ ] Raw React source code was not deployed to the Nginx web root
* [ ] `node_modules/` was not deployed to the Nginx web root
* [ ] Nginx is active
* [ ] The application opens through the VM public IP address
* [ ] Your Full Name and date are visible in the browser screenshot
* [ ] Screenshots 1–6 are included and readable
* [ ] No password, token, private key, account ID, or other secret is visible
* [ ] The final application URL is included
* [ ] The LinkedIn post is published
* [ ] The LinkedIn post URL is included

---

*This submission is part of the DevOps Micro Internship (DMI) — Agentic AI Track.*
