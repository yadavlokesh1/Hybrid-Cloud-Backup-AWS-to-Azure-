# Hybrid Cloud Backup — AWS to Azure 🔁

## What is this?
So basically I built a backup system where a Windows server on AWS automatically backs up its data to Azure Blob Storage. Think of it as — if AWS goes down, your data is still safe sitting on Azure. Classic disaster recovery stuff.

Wanted to get hands-on with multi-cloud setups since everyone talks about it in interviews but very few people have actually done it.

---

## The Problem
Most companies don't put all their eggs in one cloud basket. But setting up cross-cloud backup isn't exactly plug and play. This project simulates a real-world scenario where your on-prem/cloud server on AWS needs an offsite backup on Azure — with automation, security, and cost optimization baked in.

---

## Architecture (how it all connects)

```
+----------------------------+        +----------------------------+
|        AWS (EC2)           |        |         Microsoft Azure     |
|                            |        |                            |
|  Windows Server 2022       |        |  Azure Blob Storage        |
|                            |        |  (Cool Tier)               |
|  Veeam Agent               |        |                            |
|  backs up to               |        |  veeam-backups/            |
|  C:\VeeamBackup            |------->|  windows-backup/           |
|                            | rclone |  AzureBackupJob/           |
|  Task Scheduler            |        |                            |
|  runs sync at 3AM          |        |  Azure Key Vault           |
|                            |        |  (stores credentials)      |
+----------------------------+        +----------------------------+
```

---

## Tools I Used

- **AWS EC2** — Windows Server 2022, this is the "on-prem" server
- **Veeam Agent for Windows** — handles the actual backup job
- **rclone** — the bridge that moves data from AWS to Azure
- **Azure Blob Storage (Cool Tier)** — where backups land, cool tier because it's cheaper for data you don't access daily
- **Azure Key Vault** — stored credentials here instead of hardcoding them anywhere
- **Windows Task Scheduler** — automates the rclone sync daily at 3AM

---

## What I Did (step by step)

**1. Set up the AWS side**
- Launched a Windows Server 2022 EC2 (t3.medium)
- Connected via RDP
- Installed Veeam Agent for Windows (free version)

**2. Set up Azure Storage**
- Created a storage account and blob container
- Set access tier to Cool (saves ~60% vs Hot tier)
- Grabbed the access key for rclone auth

**3. Azure Key Vault for security**
- Created a Key Vault
- Stored the storage account name and key as secrets
- No credentials hardcoded anywhere in the project

**4. Configured Veeam backup job**
- Created a backup job pointing to C:\VeeamBackup
- Set retention to 7 restore points
- Scheduled daily at 2AM

**5. Set up rclone for cross-cloud sync**
- Downloaded rclone on the Windows VM
- Configured it with Azure Blob credentials
- Ran first manual sync — files showed up in Azure Portal ✅

**6. Automated everything**
- Created a Windows Scheduled Task
- Runs rclone sync every day at 3AM (1hr after Veeam backup finishes)

---

## Screenshots
> check the `/screenshots` folder

| File | What it shows |
|------|--------------|
| 01-Veeam-backup-success.png | Veeam backup completed successfully |
| 02-Azure-storage-account-overview.png | Backup files (.vbm, .vbk) visible in Azure Blob — Cool tier |
| 03-Key-Vault-secrets.png |Azure Key Vault secrets page |
| 04-Task-Scheduler.png | Scheduled task set up |

---

Key Takeaways


Hands-on experience working across AWS and Azure in a single workflow
Understood how to think beyond just execution — factored in cost (Cool tier) and security (Key Vault) from the start
Built an end-to-end automated pipeline, not just a one-time manual setup
Worked with production-grade tools like Veeam, rclone, and Azure Key Vault

---

*Built by Lokesh Yadav | 2026*
