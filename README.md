# End-to-End CI/CD Pipeline for PHP Application on Microsoft Azure

## Azure App Service + Azure DNS + GitHub Actions + Zero-Downtime Deployment

---

## Project Overview

This project demonstrates a production-ready CI/CD pipeline for deploying a PHP web application to Microsoft Azure using:

- Azure App Service (Linux)
- Azure Deployment Slots
- Azure DNS
- GitHub Actions
- Azure CLI
- Service Principal Authentication

The pipeline enables:

- Automated deployments
- Zero-downtime releases
- Custom domain integration
- HTTPS access
- Enterprise-grade CI/CD automation

---

## Architecture Diagram
![Diagram](docs/architecture-diagram.svg)

---

## CI/CD Workflow

```text
Developer Pushes Code
        ↓
GitHub Repository
        ↓
GitHub Actions Pipeline
        ↓
Build and Package Application
        ↓
Upload Build Artifact
        ↓
Deploy to Azure Staging Slot
        ↓
Swap Staging to Production
        ↓
Zero-Downtime Production Release
        ↓
Azure DNS + Custom Domain
        ↓
End Users Access Application
```

---

## Technologies Used

- PHP
- GitHub Actions
- Microsoft Azure
- Azure App Service
- Azure Deployment Slots
- Azure DNS
- Azure CLI
- Linux App Service Plan

---

## Project Structure

```text
elearning-php-ci-cd/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── docs/
│   ├── architecture-diagram.dot
│   └── architecture-diagram.svg
│
├── css/
├── img/
├── js/
├── lib/
├── scss/
│
├── index.php
├── about.php
├── contact.php
├── courses.php
├── team.php
├── testimonial.php
├── 404.php
│
├── README.md
├── LICENSE.txt
└── .gitignore
```

---

# Azure Infrastructure Setup

## 1. Create Resource Group

```bash
az group create \
  --name elearning-rg \
  --location eastus
```

---

## 2. Create App Service Plan

```bash
az appservice plan create \
  --name elearning-plan \
  --resource-group elearning-rg \
  --sku S1 \
  --is-linux
```

---

## 3. Create PHP Web App

```bash
az webapp create \
  --resource-group elearning-rg \
  --plan elearning-plan \
  --name elearning-app-12345 \
  --runtime "PHP|8.2"
```

---

## 4. Create Staging Slot

```bash
az webapp deployment slot create \
  --name elearning-app-12345 \
  --resource-group elearning-rg \
  --slot staging
```

---

# Azure DNS Configuration

## Create DNS Zone

```bash
az network dns zone create \
  --resource-group elearning-rg \
  --name auemeribetech.com.ng
```

---

## Create CNAME Record

```bash
az network dns record-set cname set-record \
  --resource-group elearning-rg \
  --zone-name auemeribetech.com.ng \
  --record-set-name www \
  --cname elearning-app-12345.azurewebsites.net
```

---

## Get Domain Verification ID

```bash
az webapp show \
  --name elearning-app-12345 \
  --resource-group elearning-rg \
  --query customDomainVerificationId \
  --output tsv
```

---

## Add TXT Verification Record

```bash
az network dns record-set txt add-record \
  --resource-group elearning-rg \
  --zone-name auemeribetech.com.ng \
  --record-set-name asuid.www \
  --value "<verification-id>"
```

---

## Bind Custom Domain

```bash
az webapp config hostname add \
  --webapp-name elearning-app-12345 \
  --resource-group elearning-rg \
  --hostname www.auemeribetech.com.ng
```

---

# GitHub Secrets

Add the following repository secrets:

```text
AZURE_WEBAPP_NAME
AZURE_CREDENTIALS
```

---

# Create Azure Service Principal

```bash
az ad sp create-for-rbac \
  --name "github-actions-sp" \
  --role contributor \
  --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/elearning-rg \
  --sdk-auth | gh secret set AZURE_CREDENTIALS
```

---

# GitHub Actions CI/CD Pipeline

File location:

```text
.github/workflows/deploy.yml
```

---

## Enterprise Deployment Pipeline

```yaml
name: Enterprise CI/CD - PHP Application

on:
  push:
    branches:
      - main

jobs:

  build:
    name: Build and Package Application
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Create Application Package
        run: zip -r app.zip .

      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: php-app-package
          path: app.zip

  deploy-staging:
    name: Deploy to Staging Slot
    runs-on: ubuntu-latest
    needs: build

    steps:

      - name: Download Build Artifact
        uses: actions/download-artifact@v4
        with:
          name: php-app-package

      - name: Authenticate to Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy Application to Azure Staging Slot
        run: |
          az webapp deployment source config-zip \
            --resource-group elearning-rg \
            --name ${{ secrets.AZURE_WEBAPP_NAME }} \
            --slot staging \
            --src app.zip

  swap-to-production:
    name: Swap Staging to Production
    runs-on: ubuntu-latest
    needs: deploy-staging

    steps:

      - name: Authenticate to Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Perform Zero-Downtime Slot Swap
        run: |
          az webapp deployment slot swap \
            --name ${{ secrets.AZURE_WEBAPP_NAME }} \
            --resource-group elearning-rg \
            --slot staging \
            --target-slot production
```

---

# Verify App Service Plan

```bash
az appservice plan show \
  --name elearning-plan \
  --resource-group elearning-rg \
  --query "{Name:name,SKU:sku.name,Tier:sku.tier}" \
  --output table
```

---

# Verify Deployment Slots

```bash
az webapp deployment slot list \
  --name elearning-app-12345 \
  --resource-group elearning-rg \
  --output table
```

---

# Live Application

```text
https://www.auemeribetech.com.ng
```

---

# DevOps Features Implemented

- Automated CI/CD Pipeline
- Azure App Service Deployment
- Deployment Slots
- Zero-Downtime Releases
- Custom Domain Integration
- Azure DNS Automation
- HTTPS Support
- Service Principal Authentication
- Infrastructure Automation with Azure CLI

---

# Common Issues and Fixes

| Issue | Solution |
|---|---|
| Slot deployment fails | Verify staging slot exists |
| Custom domain verification fails | Check TXT record |
| GitHub Actions authentication fails | Verify AZURE_CREDENTIALS secret |
| DNS not resolving | Wait for DNS propagation |
| HTTPS unavailable | Ensure custom domain is bound |

---

# Cleanup Resources

```bash
az group delete \
  --name elearning-rg \
  --yes \
  --no-wait
```

---

# Key DevOps Learnings

- CI/CD improves deployment reliability
- Deployment slots eliminate downtime
- Azure DNS simplifies domain management
- Service principals improve security
- GitHub Actions enables automation at scale

---

# Final Outcome

- Fully Automated CI/CD Pipeline
- Production-Ready Azure Architecture
- Zero-Downtime Deployment Strategy
- Enterprise Authentication Model
- Secure HTTPS Application Delivery

---

# Author

Anthony Uchenna Emeribe

Cloud / DevOps Engineer

---

# Support

If you found this project helpful, consider giving the repository a star.