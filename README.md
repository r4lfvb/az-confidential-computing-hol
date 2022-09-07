> This lab builds on [the great work](https://techcommunity.microsoft.com/t5/azure-confidential-computing/secure-a-web-app-architecture-with-azure-confidential-computing/ba-p/2598108) from my colleague Raki Rahman 🏆

# Azure Confidential Computing Hands-on Lab

TLDR: [Start the lab!](./Lab%20Instructions/GetStarted.md) 🚀

## Desired outcomes:

After completing this lab you will:

1. Deploy the current Azure Confidential Computing capabilities
2. Learn about some common Confidential Computing use cases
3. Learn how to deploy and configure Azure Confidential Computing resources
4. Learn about the 2 Confidential Computing models (AMD SEV-SNP v Intel SGX)

## Scenario:

Contoso's Electric Scooter 🛴 business has been growing fast and they need to to keep track of the payroll and benefits for all the new people joining the company. After a few days, the development team have an initial architecture for the new Contoso HR (Human Resources)application:

![Architecture diagram showing a typical web app and common security concerns](Lab%20Instructions/Images/1-problem-intro.png)

Following an architecture review, the Contoso _Security and Compliance_ team have identified ways malicious actors could compromise sensitive PII (Personally Identifiable Information) data for example:

- A **Curious SQL DBA** with _db_owner_ can access sensitive tables, as well as leverage [SQL Server Extended Events Sessions](https://docs.microsoft.com/en-us/sql/relational-databases/extended-events/sql-server-extended-events-sessions?view=sql-server-ver16) to intercept query inputs.
- A **Curious VM admin** with access to application logs can read / manipulate sensitive application logs (e.g. erase a subset of the history).
- A **Curious host / provider admin** with access to the underlying Hypervisor can access the VM including application binaries and in-memory data.

Because Contoso already use Azure, the team have updated the app architecture to use Azure Confidential Computing capabilities to secure their sensitive information:

![Architecture diagram showing the confidnetial web app we will set up in this lab](Lab%20Instructions/Images/2-confidential-app.png)

The app architecture now implements these Azure capabilities:

| Service | Feature | Benefit |
| - | - | - |
| [Azure SQL DB](https://docs.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview?view=azuresql) | [Always Encrypted with secure enclaves](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves?view=sql-server-ver16) | Host a confidential database with sensitive columns encrypted using a CMK (Column Master Key) |
| [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) | [Managed HSM (Hardware Security Module)(Public preview)](https://docs.microsoft.com/en-us/azure/key-vault/managed-hsm/overview) | Store the _Always Encrypted_ CMK for the sensitive Azure SQL database |
| [Azure Confidential VM](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-vm-overview) | [AMD SEV-SNP (Secure Encrypted Virtualisation - Secure Nested Paging)](https://www.amd.com/system/files/TechDocs/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more.pdf) | Host a web app that queries the sensitive database, and persists sensitive logs (database query history) |
| [Azure Confidential Ledger](https://docs.microsoft.com/en-us/azure/confidential-ledger/overview) | N/A | Append-only, immutable ledger to store sensitive application logs |

Applying these capabilities the HR application can securely process and store sensitive data by reducing the level of trust required:

![Diagram showing the 3 boundries of trust between trusted VMs, confidential VMs and Enclaves](Lab%20Instructions/Images/3-trust-boundry.png)

---
Now that we have a plan let's [start the lab!](./Lab%20Instructions/GetStarted.md) 🚀