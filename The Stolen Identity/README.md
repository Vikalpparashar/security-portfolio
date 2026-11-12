
# [THE STOLEN IDENTITY]

## Scenario

Reconstructed a five-stage OAuth consent-phishing in a live Azure tenant through forensic analysis of two linked app registration.
A phished account had owner privileges to a privileged legacy app. Attacker created multiple paths of persistence, password of privileged legacy app's service principal, a rogue app with its service principal as the owner of the legacy app, exposed an API to redirect request to rogue app for credential harvesting.



## Environment

- Live training tenant with reader access
- Entra-Id
- App registration
- Service principals
- Exposed APIs



## Investigation

### ENTRY
1. Session Token was stolen of a phished user who had completed MFA.
2. After getting the access to the victim's identity, attacker have looked around and found that the victim is owner for a legacy system. See Fig. 1
<p align="center">
<img width="1498" height="864" alt="Screenshot 2026-11-05 at 6 52 51 PM" src="https://github.com/user-attachments/assets/7cd61c95-e85d-4da9-9f1c-0fc7630bbb36" />
<em>Fig1. Compromised App</em>
</p>

3. Looking through the different blades, gathering information. Incident team has used the Internal notes in branding and properties to track context during the investigation. See Fig. 2
<p align="center">
<img width="1510" height="868" alt="Screenshot 2026-11-05 at 7 20 41 PM" src="https://github.com/user-attachments/assets/d573b188-b7fc-451d-a8f3-30a3536136de" />
<em>Fig.2 Internal note by Incident Team</em>
</p>

### ESCALATE
4. Moving forward with the investigation, in the Certificates & secrets we find there is a single secret. It is expiring in 2099. That is not a normal cadence for password life.
<p align="center">
<img width="1510" height="868" alt="Screenshot 2026-11-05 at 7 28 08 PM" src="https://github.com/user-attachments/assets/e4ba3115-e0bc-451d-8c52-8e24a53b8d9e" />
<em>Fig.3 Sketcy credential</em>
</p>

5. The attacker must have added this credential as a form of persistence. They can now log in as the service principal without ever needing to login as the human again. But is that all?
6. Locate the service principal using the client Id present in the Overview. See Fig. 4
<p align="center"><img width="1600" height="868" alt="Screenshot 2026-11-05 at 7 59 50 PM" src="https://github.com/user-attachments/assets/ba62645b-8b59-410c-8bfb-167699a66086" /><em>Fig. 4 Service Principal</em></p>


7. Looking at the permissions blade in the service principal. Two permissions were granted to it, through **Admin Consent** and via **An Administrator**. See Fig. 5
- Directory.Read.All
- User.Read.All
<p align="center"><img width="1600" height="811" alt="Screenshot 2026-11-05 at 8 12 22 PM" src="https://github.com/user-attachments/assets/16994e19-9bf9-4e38-933e-33fcf129264f" /><em>Fig. 5 Admin granted permission</em>

8. The password was not just for persistence, the legacy app have more privilege than the compromised user, they (attacker) have elevated their privileges through that credential.

### PIVOT
9.  Looking at the Owners blade in the App Registration > Compromised App. There is a single owner visible.
<p align="center"><img width="1600" height="811" alt="Screenshot 2026-11-05 at 8 48 41 PM" src="https://github.com/user-attachments/assets/2e2fdc48-b0cb-463a-b028-63156fca423d" /><em>Fig. 6 Sketchy Owner</em></p>

10. Azure allows standard users to create app registration. The attacker created a new app registration and as the victim account was owner to the compromised app. Attacker added the service principal of the new app to the compromised app.
11. Attacker is the owner of this newly created app, so even if the credential is rotated for the victim and the compromised app, attacker can create new credentials for the compromised app.

### PERSIST
12. Investigating other blades, Expose an API stands out. There is a custom scope, an exposed API, this turns the legacy app into a callable backend resource, which means attacker's own app can request delegated access to it.

 <p align="center"><img width="1441" height="811" alt="Screenshot 2026-11-06 at 3 19 11 PM" src="https://github.com/user-attachments/assets/032b243d-cd31-490b-b25f-b274034c2c33" /><em>Fig. 7 Exposed Api</em>

### LOOT
13. In the Authentication blade of Rogue app, we find a redirect API (every app lets user declare redirect URI). The attacker had used the delegated access scope, the rogue app's client Id, the redirect URI, to create a working credential harvester/ phishing URL. A victim who is signed in on a corporate device clicks accept and the authorization code lands on the attacker's server

<p align="center"><img width="1441" height="811" alt="Screenshot 2026-11-06 at 11 40 27 AM" src="https://github.com/user-attachments/assets/bae7f802-ee7e-4307-bfbf-59f4631f564b" /><em>Fig. 8 Authorization code dump</em></p>



## What broke / what surprised me

The thing that took me hours to understand was the role of the rogue app in persistence, my thinking at that point was, if the phished user's credentials, and the legacy application's credentials were rotated, why would the people doing it not delete a rogue app. Only to realize, I am the one investigating and finding this rogue app, if the investigation was not performed and only the credentials were rotated, the attacker would have a backdoor, to login change the credentials of the legacy app, given the rogue application was an owner. Hence, the persistence. 



## Findings and recommendations


Following are the findings of the investigation
1. The phished user's MFA was not phishing resistant one, after the compromise of password the user willing completed the MFA leading to stolen session token.
2. Due to improper access reviews, the victim was accidentally left owner to a privileged legacy app.
3. The followed abuse of privilege is called a confused deputy attack, where a higher privileged program or service is tricked by lower level user or program.
4. The credentials on the legacy app had expiration year of 2099
5. 

### Recommendations
1. Perform quarterly access review for Administrative users and semi annual review for non-administrative users,
2. Perform monthly phishing simulation and training for employees especially with higher privileges.
3. Make sure the privileged apps validate the true origin and intent of every request before using its elevated permissions.
4. Setup policy to stop users to set the cadence of credentials to a long period.

## What I learned

1. Identity is a boundary, if set properly can save a tenant from cataclysmic outcomes.
2. Importance of phishing resistant MFAs.
3. Importance of password rotation.
4. Simple ways to pivot and persist within a tenant.
5. During investigation, it is important to understand why an event has happened as it had, like in my case I couldn't understand why did the attacker created new app, as for me if i am deleting the legacy app's credential, why wont i delete the rogue app? but that was the point of investigating, to find these backdoors, which if missed, allows hackers to use the entra as BAU.
