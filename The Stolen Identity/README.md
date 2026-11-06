# [THE STOLEN IDENTITY]

## Scenario
2 to 3 sentences. What was the situation and what question did the investigation answer? Frame it like a work ticket, not homework.
Phished account led to legacy application tampering. Reconstructing the kill chain of an identity-based breach.


## Environment
One list: platform, services, tools, access level. Honest framing: "live multi-user Azure training tenant, Reader access."

## Investigation
The core. Numbered steps IN YOUR OWN WORDS: what you looked at, what you found, what you concluded at each step. 6 to 12 screenshots of meaningful moments (portal views, query results, before/after).
1. After getting the access to the victim's identity, attacker must have looked around and found that the victim is owner for a legacy system. 
2. Open Entra-Id and then App registration, the compromised is there. See Fig. 1
<p align="center">
<img width="1498" height="864" alt="Screenshot 2026-11-05 at 6 52 51 PM" src="https://github.com/user-attachments/assets/14691fca-dd4c-4144-b48d-62cbe44835ba" />
<em>Fig1. Mad-Hat-Legacy-Sync-Server</em>
</p>

2. Looking through the different blades, gathering information. Incident team has used the Internal notes in branding and properties to track context during the investigation. See Fig. 2
<p align="center">
<img width="1510" height="868" alt="Screenshot 2026-11-05 at 7 20 41 PM" src="https://github.com/user-attachments/assets/d573b188-b7fc-451d-a8f3-30a3536136de" />

<em>Fig.2 Internal note by Incident Team</em>
</p>

3. Moving forward with the investigation, in the Certificates & secrets we find there is a single secret. It is expiring in 2099. That is not a normal cadence for password life.
<p align="center">
<img width="1510" height="868" alt="Screenshot 2026-11-05 at 7 28 08 PM" src="https://github.com/user-attachments/assets/e4ba3115-e0bc-451d-8c52-8e24a53b8d9e" />

<em>Fig.3 Sketcy credential</em>
</p>

4. The attacker must have added this credential as a form of persistence. They can now log in as the service principal without ever needing to login as the human again. But is that all?
5. Locate the service principal using the client Id present in the Overview. See Fig. 4
<p align="center"><img width="1600" height="868" alt="Screenshot 2026-11-05 at 7 59 50 PM" src="https://github.com/user-attachments/assets/ba62645b-8b59-410c-8bfb-167699a66086" /><em>Fig. 4 Service Principal</em></p>


6. Looking at the permissions blade in the service principal. Two permissions were granted to it, through **Admin Consent** and via **An Administrator**. See Fig. 5
  a) Directory.Read.All
  b) User.Read.All
<p align="center"><img width="1600" height="811" alt="Screenshot 2026-11-05 at 8 12 22 PM" src="https://github.com/user-attachments/assets/16994e19-9bf9-4e38-933e-33fcf129264f" /><em>Fig. 5 Admin granted permission</em>

7. Looking at the Owners blade in the App Registration > Compromised App. There is a single owner visible. Not what was expected.
<p align="center"><img width="1600" height="811" alt="Screenshot 2026-11-05 at 8 48 41 PM" src="https://github.com/user-attachments/assets/2e2fdc48-b0cb-463a-b028-63156fca423d" /><em>Fig. 6 Sketchy Owner</em></p>

8. Azure allows standard users to create app registration. The attacker created a new app registration and as the victim account was owner to the compromised app. Attacker added the service principal of the new app to the compromised app.
9. Attacker is the owner of this newly created app, so even if the credential is rotated for the victim and the compromised app, attacker can create new credentials for the compromised app.
10. Investigating other blades, Expose an API stands out. There is a custom scope, 

## What broke / what surprised me
The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.

## Findings and recommendations
What you determined, plus 2 or 3 recommendations as if you were reporting to the resource owner.


https://login.microsoftonline.com/1b58a501-1042-4ec0-a183-8076bfe6cf52/oauth2/v2.0/authorize?client_id=b7a0bef0-0194-426e-8f29-8f031a56c94e&response_type=code&redirect_uri=https://madhat.io/pages/token-captured?flag=MadHat%7BteNant%5FwiDe%5FtOkEns%5FwAs%5FTHe%5FpRizE%7D&scope=api://e1003550-fdff-4bef-9458-37df6e5e1b31/Legacy.Sync
## What I learned
3 to 5 bullets. At least one technical, one "what I'd do differently."
Carl from accounting mistakingly left as an Owner of the Mad-Hat-Legacy-Sync-Service enterprise app
