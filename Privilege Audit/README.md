# [PRIVILEGE AUDIT]

## Scenario
Perform Access Review to document and determine the validity of RBAC access given on the resources and groups in the Mad Hat subscription. Perform a 4 stage audit, each using a different audit method utilized in a real RBAC audit.
## Environment

- Platform: Microsoft Azure
- Cloud environment: Live multi-user Azure training tenant
- Tools: Azure Portal, Resource Graph explorer, Azure cli, Privilege Identity Management
- Access level: Reader (read-only), PIM (Reader) on a specific group
- Scope: Assigned training tenant.
## Audit
### First Stage: Audit via IAM CSV
1. Go to Subscriptions and select the subscription to audit. Click on Download Role assignment. See Fig 1.
<p align="center"><img width="1865" height="789" alt="Screenshot 2026-11-15 at 3 50 58 PM" src="https://github.com/user-attachments/assets/f9e54495-f996-4315-b2ba-aab381d190d2" /><em>Fig. 1. Downloading the Role assignements</em></p>

2. The downloaded Role Assignment list is then audited, we can identify users and their permissions on each of the resource groups.
3. In this audit we find, a user that has owner permission on many of the resource groups. See Fig. 2.
<p align="center"><img width="1865" height="835" alt="Screenshot 2026-11-15 at 4 01 13 PM" src="https://github.com/user-attachments/assets/119fa9ab-f359-42a6-8175-88bba3b420e9" /><em>Fig. 2. Why it is always you?</em></p>

### Second Stage: Audit using the Azure cli
4. Audit with CSV, has a major problem, it buries the unknown deleted account. We can use the CLI's Get-AzRoleAssignment or az role assignment list --resource-group \<Resource_name\>
5. Login to Azure Portal, click on the terminal icon and select subscription.
6. Now, we can either do permission review for all the subscription at once or select resource group using --resource-group, See Fig. 3. 
<p align="center"><img width="1663" height="908" alt="Screenshot 2026-11-15 at 4 51 38 PM" src="https://github.com/user-attachments/assets/63cb10bc-9ceb-4cfc-8918-71710e7f60ba" /><em>Fig. 3. Testing permissions on prod resource group</em></p>
7. An orphaned role permission is found.

### Third Stage: Search using Resource Graph Explorer (KQL)
8. The only "drawback" with cli is that we can search a single scope at a time. For a multi-faceted search. We can use Resource Graph explorer. See Fig. 4.
<p align="center"><img width="1663" height="992" alt="Screenshot 2026-11-15 at 5 34 19 PM" src="https://github.com/user-attachments/assets/fc3138b2-66aa-4ee8-9cf0-4333795468e1" /><em>Fig. 4. Getting all the permission list</em></p>

9. After looking through the whole scope we only had one orphaned role permission, note it's detail.

### Fourth Stage: Privilege Identity management
10. Azure Resource Graph, only shows active access and not eligible access, for access review on that we need to go to Privilege Identity Management.
11. On manage we can select management groups, subscriptions, resource groups, resources. Depending on the scope of the review, we select the desired scope, and manage resource. See Fig. 5.
<p align="center"><img width="1663" height="992" alt="Screenshot 2026-11-15 at 6 01 32 PM" src="https://github.com/user-attachments/assets/209b356c-8310-41ad-95f9-1b08a5f5442f" /><em>Fig. 5. Azure Privilege Identity Management</em>
</p>

12. Upon clicking assignments and subsequently the export button we have an audit report of when and whom was the privilege delegated to. See Fig. 6 and Fig. 7
<p align="center"><img width="1663" height="992" alt="Screenshot 2026-11-15 at 6 08 07 PM" src="https://github.com/user-attachments/assets/fbc8f0eb-14b9-4de3-94f7-1b868d78a9df" /><em>Fig. 6. Exporting Audit Log</em>
</p>
<p align="center"></p><img width="1369" height="524" alt="Screenshot 2026-11-15 at 6 18 53 PM" src="https://github.com/user-attachments/assets/505c18b7-2a1c-44aa-8fcd-894b1d9e8aa0" /><em>Fig. 7. Audit log</em>

13. We check if the access that were provided were correctly assigned or is there is a mis-assignment.

### HUNT
14. The security team flags an over provisioned account that is assigned owner of a private resource group.
15. To access this resource group, we need to get a minimum of read privilege on this group, thankfully the security team has setup a PIM escalation for us.
16. Our Eligible assignment is located at Privileged Identity Management > My Roles > Azure Resources. See Fig. 8 . Activate it, See Fig. 9.
<p align="center"><img width="1625" height="524" alt="Screenshot 2026-11-15 at 6 50 47 PM" src="https://github.com/user-attachments/assets/d8f0d219-6ad0-4f47-a5c4-a907d48e69a5" /><em>Fig.8 The eligible assignment</em>
</p>
<p align="center"><img width="1665" height="524" alt="Screenshot 2026-11-15 at 6 53 24 PM" src="https://github.com/user-attachments/assets/d416dda3-10b1-4cff-9659-71ecda2adedc" /><em>Fig. 9 Activating the privilege</em></p>
 
17. Now that the assignment is active, extracting the audit log, from PIM, using the Manage > Azure Resource > Assignment > Export > CSV. See Fig. 10.
<p align="center"><img width="1665" height="524" alt="Screenshot 2026-11-15 at 7 01 45 PM" src="https://github.com/user-attachments/assets/718d5bb1-ac2f-4aac-b75f-797aa41ffe4a" /><em>Fig. 10 Audit log of the private resource group</em></p>

18. We now know the user with the over-permissioned account.
19. Checking all the permissions on the group using the cli, we find it is that one overly-permissioned user. That's a relief. See Fig. 11.
<p align="center"><img width="1665" height="836" alt="Screenshot 2026-11-15 at 7 09 18 PM" src="https://github.com/user-attachments/assets/3ac3b6c0-cdc6-45fb-be4f-b75da3a8d2a7" /><em>Fig. 11 The usual suspect strikes again</em>
</p>

## What broke / what surprised me

1. Took me days to understand why the cli output was coming out different from what was expected. The permission in this training environment was not sufficient enough.
2. Understanding the KQL took me some time, how the query works, where to look at in the output.
3. Orphaned permissions and the risks of Shadow IT.

## Findings and recommendations

- During the audit we found,
1. A user with owner permission to multiple groups and resources.
2. A lingering permission of a deleted user.
3. An Over-privileged account that has access to a private group


## What I learned 
 - There is no single method or tool that provides a complete picture of an organization's identity landscape.
 - Null principal names are intentional data points representing orphan permissions and they dont appear in IAM blade / export.
 - Using multiple Azure tools gives perspective on the issue. We can find interesting data by drawing inferences from the outputs.
### ARTIFACTS: SEES AND MISSES
| Method               | Sees                                        | Misses                              |
|----------------------|---------------------------------------------|-------------------------------------|
| IAM blade / export   | Active assignments at scope, inherited      | Group members, orphaned principals  |
| Azure CLI            | Same, plus null principalName (orphans)     | One scope per run                   |
| Resource Graph (KQL) | Whole tenant in one query                   | Eligible assignments                |
| PIM export           | Eligible vs active, activation history      | Assignments outside PIM             |


