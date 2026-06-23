# Azure_SIEM
Documentation of my project on Azure SIEM 

![Uploading Gemini_Generated_Image_2igyf52igyf52igy.png…]()

Phase 1: Setting up the Azure Resource Group and Log Analytics Workspace

Step 1 — Creating a Resource Group
A Resource Group is just a folder that holds all our project's Azure resources together.
1.	Go to portal.azure.com and sign in
2.	In the top search bar, type "Resource groups" and click it
3.	Click + Create
4.	Fill in:
Subscription: Our Azure for Students sub
Resource group name: honeypot-lab (or whatever you like)
Region: East US 2 (close, low cost, good for students)
6.	Click Review + create → Create
 <img width="1096" height="391" alt="image" src="https://github.com/user-attachments/assets/9aec6b3a-5489-4d0e-a509-d60ee74bb1f2" />

________________________________________
Step 2 — Create a Log Analytics Workspace
This is the centralized database where all our logs will be stored. Sentinel plugs into this.
1.	Search "Log Analytics workspaces" in the top bar → click it
2.	Click + Create
3.	Fill in: 
o	Subscription: same as above
o	Resource group: select honeypot-lab
o	Name: honeypot-logs
o	Region: East US 2 (must match your resource group region)
4.	Click Review + create → Create
5.	Wait ~1 minute for deployment to finish, then click Go to resource
 <img width="1108" height="392" alt="image" src="https://github.com/user-attachments/assets/d3247249-baf5-4a2e-92f7-1ed78acd2f8a" />

________________________________________
Step 3 — Confirm it worked
Once we're on the workspace page, we should see:
•	A left sidebar with options like "Logs", "Agents", "Tables"
•	Our workspace ID and primary key under Agents (we'll need these later)
 <img width="322" height="625" alt="image" src="https://github.com/user-attachments/assets/57437fcc-602c-46b5-bc25-8fb782830b84" />

That's Phase 1 done. The "database" for our SIEM is now live.
Phase 2: Connecting Microsoft Sentinel to the Log Analytics Workspace
Step 1 — Enable Microsoft Sentinel
1.	In the Azure portal search bar, type "Microsoft Sentinel" and click it
2.	Click + Create
3.	we'll see a list of Log Analytics workspaces — select honeypot-logs
4.	Click Add Microsoft Sentinel
5.	Wait 1 minute; Sentinel will provision on top of your workspace
We should land on the Sentinel overview dashboard. This is our SIEM.
 <img width="1125" height="356" alt="image" src="https://github.com/user-attachments/assets/949dea5c-cb22-40c6-b2cc-ca761c7914f9" />

________________________________________
Step 2 — Add the Windows Security Events Connector
This tells Sentinel to collect login events (including failed ones) from our VM later.
1.	In the Sentinel left sidebar, click Content hub
2.	In the search box, type "Windows Security Events"
3.	Click the result → click Install (bottom right panel)
4.	Wait for it to say Installed
If this fail here's the workaround:
1.	Go back to portal.azure.com
2.	Open Microsoft Sentinel → select honeypot-logs
3.	In the left sidebar click Data connectors
4.	At the top we'll see a banner saying "More content at Content hub"; click that
5.	Search "Windows Security Events" and install from there
 <img width="1125" height="502" alt="image" src="https://github.com/user-attachments/assets/7138f1da-07c4-46ca-a333-1b1560986f4f" />

________________________________________
Step 3 — Configure the Data Connector
1.	In the Sentinel left sidebar, click Data connectors
2.	Search "Windows Security Events via AMA" and click it
3.	Click Open connector page
4.	We'll see a section called Configuration, leave this for now. we'll complete it in Phase 3 after the VM exists, because the connector needs a machine to point at.
 <img width="1125" height="446" alt="image" src="https://github.com/user-attachments/assets/b027cec7-c210-40c2-b8bf-559f90cf4b54" />

 <img width="1125" height="484" alt="image" src="https://github.com/user-attachments/assets/8e57f561-03fc-4477-b7f3-9a7a0aafbdb0" />

________________________________________
Step 4 — Confirm Sentinel is live
Back on the Sentinel overview page, we should see:
•	Data connectors tile (showing 0 connected for now, that's fine)
•	Incidents, Workbooks, Analytics in the left sidebar
•	Your workspace name honeypot-logs shown at the top
________________________________________
Phase 3: deploying the Windows honeypot VM and opening it to the internet
Step 1 — Create the Virtual Machine
1.	In the Azure portal search bar, type "Virtual machines" and click it
2.	Click + Create → Azure virtual machine
3.	Fill in the Basics tab:
Project details:
•	Subscription: Azure for Students
•	Resource group: honeypot-lab
Instance details:
•	VM name: honeypot-vm
•	Region: East US 2
•	Availability options: No infrastructure redundancy required
•	Image: Windows Server 2022 Datacenter (free with student credits)
•	Size: Standard_B1s (1 vcpu, 1GB RAM — cheapest, enough for this)
Administrator account:
•	Username: something other than Administrator (attackers try that first) — // PotLabUser
•	Password: make it strong and write it down — you'll need it to RDP in later // Potlabuser@602
4.	Click Next: Disks → leave defaults → click Next: Networking
________________________________________
Step 2 — Configure Networking (the critical part)
This is where you expose the VM to the internet.
1.	On the Networking tab: 
o	Virtual network: let it create a new one automatically
o	Public IP: make sure one is assigned (default)
o	NIC network security group: select Advanced
o	Click Create new under the network security group
2.	In the NSG creation panel: 
o	Delete the existing default inbound rule (click the ... → Remove)
o	Click + Add an inbound rule
o	Set: 
	Source: Any
	Source port ranges: *
	Destination: Any
	Destination port ranges: *
	Protocol: Any
	Action: Allow
	Priority: 100
	Name: DANGER-AllowAll
o	Click Add
o	Click OK to save the NSG
 <img width="1125" height="486" alt="image" src="https://github.com/user-attachments/assets/a2d91420-f1f1-487c-8d1d-fecf73c6bf5f" />

3.	Back on the Networking tab, leave everything else as default
________________________________________
Step 3 — Finish creating the VM
1.	Click Next through Management, Monitoring, Advanced — leave all defaults
2.	Click Review + create
3.	Review the summary — confirm it shows honeypot-lab, East US 2, Windows Server 2022, Standard_B1s
4.	Click Create
5.	Deployment takes 3–5 minutes — wait for "Your deployment is complete"
6.	Click Go to resource
 <img width="1125" height="496" alt="image" src="https://github.com/user-attachments/assets/173ff778-00c4-4465-9fb3-04caddbbd643" />

________________________________________
Step 4 — Disable the Windows Firewall on the VM
The NSG opens the door at the Azure level, but Windows has its own firewall too. You need to disable it so attackers can actually reach the machine.
1.	On your VM page, copy the Public IP address (shown on the overview) // 52.159.125.66
2.	On your local computer, open Remote Desktop Connection (search "RDP" in Windows start menu, or use Microsoft Remote Desktop on Mac)
3.	Connect to the public IP, log in with your labuser credentials
 <img width="641" height="616" alt="image" src="https://github.com/user-attachments/assets/5c026fca-b90b-4ac4-a7fc-88eebfbc058d" />

4.	Once inside the VM, open the Start menu and search "wf.msc" → open it
5.	Click Windows Defender Firewall Properties
6.	On the Domain Profile, Private Profile, and Public Profile tabs — set Firewall state to Off
 <img width="770" height="646" alt="image" src="https://github.com/user-attachments/assets/8f0da098-b738-4b10-bcf3-2b56525282fe" />

7.	Click Apply → OK
Difficulty I had that I was not able to connect RDP to the cloud VM. Then I checked by steps and found that inbound rule was not created. I might have not saved when I created it. This was stopping my host windows from connecting to this cloud VM.
________________________________________
Phase 4: Log ingestion and verification
Step 1 — Create a Data Collection Rule
1.	In the Azure portal search bar, type "Monitor" and click it
2.	In the left sidebar, click Data Collection Rules and  Click + Create
3.	Fill in the Basics tab:
Field	Value
Rule name	honeypot-dcr
Subscription	Azure for Students
Resource group	honeypot-lab
Region	East US 2
Platform type	Windows
5.	Click Next: Resources
 <img width="1025" height="453" alt="image" src="https://github.com/user-attachments/assets/180eb961-db9c-462f-af70-8c64382f21e5" />

________________________________________
Step 2 — Add your VM as a resource
1.	Click + Add resources
2.	Expand your subscription → expand honeypot-lab
3.	Check the box next to HoneyPot-VM
4.	Click Apply
5.	You'll see a prompt asking to install the Azure Monitor Agent on the VM — click Enable to confirm
6.	Click Next: Collect and deliver
 <img width="1067" height="332" alt="image" src="https://github.com/user-attachments/assets/0fab1ee6-6471-4b7d-968e-cf4c2b62e0aa" />

________________________________________
Step 3 — Configure what logs to collect
1.	Click + Add data source
2.	For Data source type, select Windows Event Logs
3.	Select Basic and check these boxes: 
<img width="661" height="749" alt="image" src="https://github.com/user-attachments/assets/852a591f-1ae4-4312-bfc0-aba91b8ab761" />

4.	Click Next: Destination
5.	For destination: 
o	Destination type: Log Analytics Workspace
o	Subscription: Azure for Students
o	Account or namespace: honeypot-logs
6.	Click Add data source
 <img width="897" height="355" alt="image" src="https://github.com/user-attachments/assets/17e32c85-ae0f-4e16-8931-491adc299dfe" />

7.	Click Next: Review + create → Create
 <img width="1069" height="273" alt="image" src="https://github.com/user-attachments/assets/49bc9c17-7a97-405c-b971-663400e3f129" />

________________________________________
Step 4 — Verify the Agent installed on your VM
1.	Go to Virtual machines → HoneyPot-VM
2.	In the left sidebar click Extensions + applications
3.	You should see AzureMonitorWindowsAgent listed with status Provisioning succeeded
4.	This may take 5–10 minutes to appear — refresh the page periodically.
 <img width="1031" height="300" alt="image" src="https://github.com/user-attachments/assets/0ac03d82-fc2f-42d1-b3d0-ed52d1596f77" />

________________________________________
Step 5 — Verify logs are flowing into Sentinel
1.	Go to Microsoft Sentinel → select honeypot-logs
2.	In the left sidebar click Logs
 <img width="1065" height="469" alt="image" src="https://github.com/user-attachments/assets/3fb0f361-9808-4326-b6c0-ae026f47204b" />

3.	In the query editor, type this KQL query and click Run:
SecurityEvent
| where EventID == 4625
| take 10
4.	If you get results back — even just one row — logs are flowing and Phase 4 is complete
5.	If you get no results, wait 15–20 minutes and try again. The agent needs time to initialize and attackers need a moment to find you
 <img width="1005" height="569" alt="image" src="https://github.com/user-attachments/assets/55f5b25d-f30f-41d8-bf45-0143f71c7c6d" />

________________________________________
Phase 5: KQL detection and alerting

Step 1 — Query Failed RDP Logons
In Sentinel → Logs, run this to pull all failed login attempts with attacker details:

Event
| where EventID == 4625
| where TimeGenerated > ago(24h)
| project TimeGenerated, Account, IpAddress, WorkstationName, LogonTypeName
| order by TimeGenerated desc
________________________________________
Step 2 — Enrich IPs with Geolocation
Azure doesn't give us geo data natively, so we use a watchlist. Download a free IP-to-geo CSV (like from db-ip.com) and upload it to Sentinel:
•	Go to Sentinel → Watchlists → + New
•	Upload the CSV, name it geoip
•	Set the search key to the IP column

Then join it in KQL:

let GeoIPDB = _GetWatchlist('geoip');
Event
| where EventID == 4625
| where TimeGenerated > ago(24h)
| evaluate ipv4_lookup(GeoIPDB, IpAddress, network)
| project TimeGenerated, IpAddress, Account, country, latitude, longitude
| summarize AttackCount = count() by country, latitude, longitude
| order by AttackCount desc
________________________________________
Step 3 — Visualize on a Map
•	In Sentinel → Workbooks → + New
•	Add a query element, paste the geo query above
•	Change the visualization type to Map
•	Set Latitude and Longitude fields accordingly
•	Size the bubbles by AttackCount
This gives us a live world map of brute-force sources
________________________________________
Step 4 — Create an Analytics Alert Rule
•	Go to Sentinel → Analytics → + Create → Scheduled query rule
•	Name: Brute Force RDP Detection
Event
| where EventID == 4625
| summarize FailedAttempts = count() by IpAddress, Account, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
•	Run every 5 minutes, lookup last 5 minutes
•	Alert when results > 0
________________________________________
Step 5 — Confirm Incidents Appear
•	Go to Sentinel → Incidents
•	We should see brute-force incidents auto-generated with attacker IP, targeted account, and timestamps

<img width="1124" height="486" alt="image" src="https://github.com/user-attachments/assets/bab6cd9e-a65b-4b65-abf4-0f0e66b21d5e" />
