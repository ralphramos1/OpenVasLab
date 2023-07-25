<h1>OpenVas Vulnerability Management Home Lab</h1>

<h2>Description</h2>
Hello, today we'll walk through the process of installing OpenVAS on Azure. We'll create a Windows virtual machine and install some old, deprecated software on it to simulate potential vulnerabilities. Our goal is to perform vulnerability scans on the virtual machine and identify any security issues it might have. Once we've detected vulnerabilities, we'll proceed to remediate them and run another scan to observe the results.
<br />
<br>
<h2>Why is Vulnarbility Management important?</h2>
Vulnerability management is important because it helps organizations proactively identify and address security weaknesses in their systems, applications, and networks. By regularly scanning for vulnerabilities, businesses can reduce the risk of cyberattacks, data breaches, and financial losses. It also ensures compliance with regulations and helps maintain the integrity of sensitive data. Through timely patching and continuous improvement, vulnerability management strengthens an organization's overall security posture, safeguarding its assets and reputation in an ever-changing threat landscape.

<h2>Create our free Azure account</h2>

<p align="left">
Let's begin by registering for a Azure account here https://azure.microsoft.com/en-us/free/, and once that's done, we can proceed with the login process.
<br />
<br />
<h2>Prepare Vulnerability Management Scanner (OpenVAS)</h2>
Go to the Azure portal: https://portal.azure.com<br>
Navigate to the Marketplace and search for "OpenVAS secured and supported by HOSSTED."<br>
Select the pre-set configuration with the weakest settings.<br>
Click "Continue to Create VM."<br>
Fill in the details:<br>
Resource Group: Vulnerability-Management<br>
VM Name: OpenVAS (Note the region and Vnet, I will be using West US 3)<br>
Authentication: Username → cyberdemo / Password → Cyberdemo702!<br>
Monitoring: Disable Boot Diagnostic<br>
Click "Create."<br>
Once the VM is created, use PowerShell (Windows) or Terminal (MacOS) to SSH into it using the credentials you provided.<br>
Wait for the deployment to finish ("Your OpenVAS is deploying, please wait" → "hossted stage done").<br>
Access the web app URL shown and log in with the provided username and password (or try admin/admin). After logging in, reset the admin password to: Cyberdemo702!<br/>
<p align="center"><img src="https://i.imgur.com/saNDEbr.png" height="80%" width="80%"><br>
<br>
<p align="center">
<h2>Create a vulnerable virtual machine in the Azure portal by following these steps:</h2>
Go to https://portal.azure.com/ and search for Virtual Machines.<br>
Create a new Virtual Machine with the Resource Group named "Vulnerability-Management."<br>
Name the VM as "Win10-Vulnerable" and set the region to be the same as the OpenVAS VM (West US 3).<br>
Assign the Virtual Network as the same as OpenVAS (important for connectivity).<br>
Choose the image as "Windows 10 Pro" and select any size with 2 vCPUs.<br>
Set the username as "cyberdemo" and the password as "Cyberdemo702!" for login.<br>
Under Networking, ensure the VM is on the same Vnet as OpenVAS.<br>
<p align="center"><img src="https://i.imgur.com/AnABq7P.png" height="80%" width="80%"><br>
<p align="left">Create the VM and after it's created, check if you can Remote Desktop Protocol (RDP) into it using the provided credentials.<br>
Once logged in, make the VM vulnerable by disabling the Windows Firewall and installing outdated software, such as an old version of Firefox, VLC Player, and Adobe Reader. Restart the VM after making these changes.<br/>
<p align="center"><img src="https://i.imgur.com/V6Pigtx.png" height="80%" width="80%">
<br />
<br />
<p align="center"><h2>Configure OpenVAS to perform the first unauthenticated scan against the Vulnerable VM</h2>
Log in to OpenVAS and navigate to Assets → Hosts → New Host.<br>
Add the private IP address of the Client VM as a new Host.<br>
Create a new Target from the Host and name it "Azure Vulnerable VMs."<br>
Take note of the credentials for later use, as SMB credentials will be added later.<br>
Create a new Task named "Scan - Azure Vulnerable VMs" with the "Azure Vulnerable VMs" as the Scan Target and save the task.<br>
Start the "Scan - Azure Vulnerable VMs" Task and take note of the scan status.<br>
Note that the scan did not find the outdated softwares we installed because this was a unauthenticated scan.<br/>
<img src="https://i.imgur.com/cTMW1bN.png" height="100%" width="100%">
<h2>After the unauthenticated scan is completed, configure the VM for credentialed scans</h2>
Disable Windows Firewall, User Account Control, and enable Remote Registry on the Client VM.<br>
Launch Registry Editor (regedit.exe) in "Run as administrator" mode, navigate to HKEY_LOCAL_MACHINE hive, and open SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System.<br>
Create a new DWORD (32-bit) value named "LocalAccountTokenFilterPolicy" with a value of 1 to enable credentialed scans. Restart the VM after making the changes.<br>
<p align="center"><img src="https://i.imgur.com/LcCwfev.png" height="60%" width="60%">

<p align="left"><h2>Configure credentials for credentialed scans in OpenVAS</h2>
In OpenVAS, go to Configuration → Credentials → New Credential.<br>
Name the new credential as "Azure VM Credentials," allow insecure use, and set the Username as "cyberdemo" and Password as "Cyberdemo702!".<br>
Go to Configuration → Targets and clone the previous Target, naming it "Azure Vulnerable VMs - Credentialed Scan."<br>
Ensure the Private IP is still accurate for the VM and select the previously created SMB credentials for the new target.<br>
<img src="https://i.imgur.com/DTB8rDc.png" height="100%" width="100%">
<h2>Execute the Credentialed Scan against the Vulnerable VM:</h2>
In OpenVAS, go to Scans → Tasks and clone the "Scan - Azure Vulnerable VMs" Task, then edit it.<br>
Name the new Task as "Scan - Azure Vulnerable VMs - Credentialed" and set the Targets as "Azure Vulnerable VMs - Credentialed Scan."<br>
Start the new Credentialed Scan and wait for it to finish (it will take longer than the unauthenticated scan).<br>
After the scan finishes, review the findings, including SMB Login status and individual vulnerabilities, especially Criticals from the outdated Adobe Reader and Firefox version.<br>
<img src="https://i.imgur.com/9VcNDN8.png" height="100%" width="100%">
<h2>Remediate Vulnerabilities on the Client VM</h2>
Log back into the Win10-Vulnerable VM and uninstall Adobe Reader, VLC Player, and Firefox.<br>
Restart the VM after making the remediations.<br>
<img src="https://i.imgur.com/A8MaBPP.png" height="100%" width="100%">
<h2>Verify Remediations:</h2>
Re-initiate the "Scan - Azure Vulnerable VMs - Credentialed" scan in OpenVAS and observe the results to verify that the vulnerabilities have been remediated. As shown in the image below, the scan results indicate that Adobe Reader, Firefox, and VLC Player have been removed, but there are still several vulnerabilities remaining on the workstation.<br>
<img src="https://i.imgur.com/KYHCxs6.png" height="100%" width="100%"><br>
<h2>Conclusion</h2>
Upon completing the update process for my virtual machine, its security has been significantly enhanced. This highlights the crucial significance of regularly keeping operating systems and applications up to date (in our case, uninstalling old applications). These updates often contain crucial security bug fixes that play a vital role in protecting your system from potential vulnerabilities and threats.
</p>
