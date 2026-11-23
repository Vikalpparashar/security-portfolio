# [THE FRIDAY DEPLOY]

## Scenario

Reviewing the new notification service intern shipped before production promotion. Review the resource's deployed location, authentication method, information storage, and security of the information against the production baseline.

## Environment

- Platform: Microsoft Azure
- Cloud environment: Live multi-user Azure training tenant
- Tools: Azure Portal
- Access level: Reader (read-only)
- Scope: Assigned training tenant.

## Investigation

### THE WHERE

1. First thing to review is the region where the service is created. We can find this in Resource group > rg-\<name of the group\> --> (See Fig.1.1)
<p align="center"><img width="1881" height="644" alt="Screenshot 2026-11-22 at 11 56 08 AM" src="https://github.com/user-attachments/assets/8d4d799c-c3ee-42cb-bddf-6adaf7ddea82" /><em>Fig. 1.1 Wrong Region, Intern</em> </p>

2. If we review against the production baseline, we find, our intern has created resources, in a different region.  (See Fig. 1.2)
<p align="center"><img width="1881" height="941" alt="Screenshot 2026-11-22 at 1 44 09 PM" src="https://github.com/user-attachments/assets/977b4376-908a-4ee5-b04e-fdf1ac2f3951" /><em>Fig. 1.1 Check production dummy</em></p>
3. So now every time this new resource talks to other storage accounts and information stored in the different region, it will incur billing charges as exiting region is considered as egress data.

### THE WHO
4. Next review is to check, how does the fucntion app in it authenticate to its dependencies. Func App > Settings > identity. The intern app doesn't have either system assigned or user assigned identities, which inherently means it works on stored secrets if it has any ability to access storage. See Fig. 2.1
<p align="center"><img width="1881" height="941" alt="Screenshot 2026-11-22 at 3 02 24 PM" src="https://github.com/user-attachments/assets/3ac9e44d-ebc9-4ad5-a735-9ec7ccb41ad7" /><em>Fig. 2.1 Learn Managed Identity, Intern</em>
</p>
5. Review this against the production baseline. We will find a user assigned identity, that handles the authentication. See Fig. 2.2.
<p align="center"><img width="1881" height="941" alt="Screenshot 2026-11-22 at 3 07 45 PM" src="https://github.com/user-attachments/assets/ba217901-ebb9-46ab-b98b-76369ed1a56b" />
<em>Fig. 2.2 Managed identity for authentication on prod</em></p>

### THE DATA 
6. Next step is to review the storage account, on the compute resource, and make sure the storage is configured properly.
7. In the Production deployment, every container's Anonymous access level is set to private. See Fig. 3.1.
<p align="center"> <img width="1881" height="941" alt="Screenshot 2026-11-22 at 3 16 36 PM" src="https://github.com/user-attachments/assets/ea7bb975-dad8-4d4f-aae3-d1607393e859" /><em>Fig. 3.1 Anonymous access level set to private.</em></p>

8. The intern's storage whereas have a container with Anonymous access level set as blob. See Fig. 3.2.
<p align="center"><img width="1881" height="941" alt="Screenshot 2026-11-22 at 3 21 27 PM" src="https://github.com/user-attachments/assets/3a1b3294-ca82-4c7c-9d2c-574a52e29973" /><em>Fig. 3.2 Who is training this intern?</em></p>
9. This storage container can be accessed by anyone on the internet if they scraped the web and found the url to the container.

10. Open the container, select any file to read, click on it, select the URL and open it in incognito or any place else, It is accessible. See Fig. 3.3 and Fig. 3.4

<p align="center"><img width="1881" height="941" alt="Screenshot 2026-11-22 at 3 25 32 PM" src="https://github.com/user-attachments/assets/c31e3d23-f149-4d76-80d7-4ef7b57f7f7d" /> <em>Fig. 3.3 Url to open blob </em></p>
<p align="center"><img width="1881" height="1043" alt="Screenshot 2026-11-22 at 3 27 08 PM" src="https://github.com/user-attachments/assets/5bae3c58-9d15-4e5a-a64b-cf7d65fb16d1" /><em>Fig. 3.4 Accessed using Incognito </em></p>

### THE NETWORK ACCESS

11. Next Review is to look at the inbound and outbound network setting, which denies or allows access to the resource depending on parameters like priority, IP and Port. Function App > Settings > Networking. See Fig. 4.1 and Fig. 4.2
<p align="center"><img width="1881" height="1043" alt="Screenshot 2026-11-22 at 3 57 21 PM" src="https://github.com/user-attachments/assets/5c1c8fc2-6594-4043-ac52-ba1e0ee027cd" /><em>Fig. 4.1 Access with no restriction</em></em></p>
<p align="center"><img width="1881" height="1043" alt="Screenshot 2026-11-22 at 3 58 55 PM" src="https://github.com/user-attachments/assets/6db27b0a-aac0-4f22-a726-8314bf9e7c6f" /><em>Fig. 4.2 Access allowed to any IP and any PORT</em></p>


12. Compare it to the production baseline. there we have a DENY on everything. If in future we want to give access to a specific group or IP, we can create a rule with higher priority than this rule. See Fig. 4.3 and Fig. 4.4
<p align="center"><img width="1881" height="950" alt="Screenshot 2026-11-22 at 4 03 28 PM" src="https://github.com/user-attachments/assets/03972bbd-78b3-4b87-921d-755698083e30" /><em>Fig. 4.3 Enabled access with restrictions</em></p>
<p algin="center"><img width="1881" height="950" alt="Screenshot 2026-11-22 at 4 03 37 PM" src="https://github.com/user-attachments/assets/9c33fa57-fa37-4ba8-8389-1fbe12210ae7" /><em>Fig. 4.4 DENY EVERYTHING, DEAR INTERN</em>
  
## What broke / what surprised me

1. Change of region is also considered as egress and not just something inside the Azure framework, that surprised me.
2. Network access control has nothing to do with the Anonymous level access, they both are different and should be taken care of separately.

## Findings and recommendations

- Intern didn't create resource in correct region.
- Intern didn't use managed identity, and has  used stored passwords, most probably stored somewhere in the code.
- Intern has left a container with anonymous access level as blob, anyone on the internet can read the stored values.
- Intern has allowed access from everywhere.

### Reccomendation
- Provide Intern with an operations playbook, which defines the structure of any operation and clearly explains the procedures.
- Repetative training and continuous guidance for the intern, make sure they don't create resources on a Friday. Even if they do, it is under someone's guidance.

## What I learned

- Data transfer between different region is also considered as egress of data, and it will be billed like that.
- Use managed identities to handle authentication business, rather than storing passwords somewhere
- Anonymous access on container will let anyone read information stored, even if they are not authenticated into azure.
- I would not deploy things on EOD Friday, because reviewer wont have time to check it and the earliest it could be reviewed is Moday.
