# [Operation Dead Deploy]

## Scenario

Junior intern created test environment for important experiments that were non-compliant to company's naming standards. Intern was able to give any name to resources, groups and deployments even though a naming policy was in place to stop it from happening.
Through investigation, it is concluded that the naming policy's effect was in "audit" rather than "deny", which allowed deployment of non-compliant resources.

## Environment
Live multi-user Azure training tenant, Reader access, Azure Portal.


## Investigation
1. Login
2. Check all resources, our resource groups follow the proper naming syntax (rg-), as mentioned our intern has cut a lots of corner, I don't think he will be bothering with naming our resources properly too.
3. For initial check we can filter out all resources that doesn't follow our naming conventions.
4. We find 2 resource groups that does not follow our naming conventions. See Fig.1
<p align="center">
  <img src="https://github.com/user-attachments/assets/de798937-5adc-4d77-9edd-ad6c3faf4bb3" alt="Image description">
  <br>
  <em>Fig.1 The two resource groups</em>
</p>

5. Upon research, we find the group NetworkWatcherRG is automatically created by Azure, we made a mental note of it, and it has been keep aside from active investigation.
6. The other group however does look like a human work. The naming looks human. Investigating further.
7. Looking into the Resource group we have only one resource deployed. It is a storage account.
<p align="center">
  <img src="https://github.com/user-attachments/assets/aa9f5f23-e98d-40a8-88bd-18f338414694" alt="Image description">
  <br>
  <em>Fig.2 Single Storage Account deployed.</em>
</p>

8. Looking through different blades of the Resource group. Nothing in Activity logs, No Role Assignment or Deny. No events
9. Looking into Deployments, we do have a have deployment, the deploymentName does angle us more towards confirmation that is the resource group the Intern has created.
<p align="center">
<img width="1504" height="801" alt="Screenshot 2026-10-30 at 3 17 22 PM" src="https://github.com/user-attachments/assets/99273d4a-b4b3-4453-aba7-eee0470b0da8" />
<em> Fig.3 Deployment </em>
</p>

10. Looking into the deployment overview and input we find the flag that the intern has been instructed to use whenever created any resources. See Fig. 4
<p align="center">
<img width="1504" height="801" alt="Screenshot 2026-10-30 at 3 26 02 PM" src="https://github.com/user-attachments/assets/1d55a05e-efb1-42b2-8498-bdac7918827a" />
<em> Fig.4 InternFlag</em>
</p>

11. Moving forward, checking the resource that was deployed. One of Tags is an owner tag, which confirms again the user of this who resource group is the intern. There is the internFlag present as well. See Fig.5
<p align="center">
<img width="1504" height="801" alt="Screenshot 2026-10-30 at 3 32 59 PM" src="https://github.com/user-attachments/assets/56db3dfa-faaf-4f88-abae-39967297ef6d" />
<em>Fig.5 The Resource</em>
</p>


12. Now that we have confirmed that this resource group belongs to the intern. We need to investigate the policy that allowed this non-compliant naming convention deployment. Moving to Resource group and selecting policy from the left blade
13. We can see that in resource-group policy compliance tab the naming convention policy is showing non-compliant. See Fig.6

<p align="center">
<img width="1504" height="801" alt="Screenshot 2026-10-30 at 3 40 50 PM" src="https://github.com/user-attachments/assets/7e07bc8c-53d0-4003-a23d-3dcb6763250b" />
<em>Fig.6 Naming convention policy</em>
</p>

14. Clicking and looking into the policy, we find the reason, the naming convention policy "Effect Type" is set to Audit. Which means it will only raise alert on non-compliance, rather than denying deployment. See Fig.7

<p align="center">
<img width="1504" height="801" alt="Screenshot 2026-10-30 at 3 45 15 PM" src="https://github.com/user-attachments/assets/2cf2bbdc-23d0-452a-9798-58bbb2a10508" />
<em>Fig.7 Effect Type: Audit</em>
</p>  

## What broke / what surprised me
The thing that surprised me was how small misses in policy creation can lead to disastrous outcome, in our case a naming policy was set to audit rather than deny, which allowed for the naming convention disaster.

## Findings and recommendations

1. Intern was unaware of the naming policy, they might also be unaware of other governing policy.
2. Policy was set to Audit, rather than deny. Unless there is a particular reason to keep it in audit, deny should be a preferred mode.

### Recommendation
1. Train employees on the governing policy.
2. DevOps team should document policy in details, explaining why a policy works the way it works.

## What I learned
1. Using filter to find resources.
2. Importance of tags.
3. Attention to details in policy creation and asking why it works the way it works.
4. How to investigate an identity attack in Entra Id.
