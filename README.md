# SIEM: Honeypot Azure Sentinel Monitoring and Geolocation Mapping
**Eric Pham**

**Personal Project**

**November 1, 2023 - November 8, 2023**

<h2>Introduction</h2>
The SIEM Honeypot project is a cybersecurity initiative designed to provide real-time monitoring and geolocation mapping of malicious login attempts on a virtual machine (VM) hosted in Microsoft Azure. By deploying a strategically configured VM acting as a honeypot, this project aims to attract and observe attacks from potential threat actors worldwide. The Azure Sentinel (SIEM) platform is utilized to collect and analyze security event data, providing valuable insights into the geographical origins of these attack attempts.
<h2>Project Walkthrough</h2>

**Creating Virtual Machine and Setting Up Firewall**
- Navigated to Microsoft Azure and created a Virtual Machine (VM) and created the username and password (remember these credentials because they will be important later in the project) 
- Created a resource group named ‘Honeypotlab’ where everything will be stored that’s related to this project.
- I will be using Windows 10 Pro as my operating system (OS)
- One setting that’s mandatory to change when creating a VM on Microsoft Azure is ‘NIC network security group’ which can be looked at as a firewall. Swapped from ‘Basic’ to ‘Advanced’ which allows me to configure the inbound firewall settings. In the settings, I set the destination port range to star (*), which means anything, and priority to 100 which is low. 
<img src="https://i.imgur.com/zsg0XXW.png" height="40%" width="40%"/>

The idea of this is to make the VM public so the VM doesn’t drop any sort of traffic and can be discovered online quickly so it can be attacked for the purpose of this project.

**Create Log Analytics Workspace**
- Creating a custom log that contains geographic information of where the attacker is coming from. 
<img src="https://i.imgur.com/swu2oLe.png" height="50%" width="50%"/>

- Searched and navigated to ‘Microsoft Defender for the Cloud’ > Environment Settings and enabled ‘Foundational CSPM’ and ‘Servers’
<img src="https://i.imgur.com/JskokD4.png" height="50%" width="50%"/>

- Navigated to ‘Data Collection’ and set ‘Store additional raw data - Windows security events’ from ‘None’ to ‘All Events’. Remember to save the changes. 
- This enables the ability to gather logs from the VM into the log analytics workspace. 
- Connected it to the VM by navigating to ‘Log Analytics workspace’ and then clicking on the workspace, in my case ‘law-honeypot’. On the left tabs, I navigated to ‘Virtual Machine’ and selected the VM I created and connected it. 
<img src="https://i.imgur.com/KbzvNC2.png" height="50%" width="50%"/>

- Searched and navigated to ‘Microsoft Sentinel’ and then clicked on the log analytics that was created, and pressed ‘Add’. 
- Connecting and Setting Up the VM
- Navigated back to the VM and found the public IP address. On the local computer, search for ‘Remote Desktop Connection’ and type the VM’s public IP address then click ‘Connect’.
<img src="https://i.imgur.com/9a7jxp1.png" height="50%" width="50%"/>

- Window Security popped up and clicked on ‘More choices’ and ‘Use a different account’. Typed the username and password of the VM.
- Accepted the certificate warning by pressing ‘Yes’ after clicking ‘Connect’. 
<img src="https://i.imgur.com/ZluVy8O.png" height="50%" width="50%"/>

- Set the privacy settings on the VM to no or off. 
<img src="https://i.imgur.com/k3EIANZ.png" height="50%" width="50%"/>

- Searched for ‘Event Viewer’ and navigated to Security, it will take a while to load for the first time. 
<img src="https://i.imgur.com/vXKcNe6.png" height="50%" width="50%"/>

**Fail Login Attempt to the VM & Observing Event Viewer Logs**
- Went back to my local device and navigated to ‘Remote Desktop Connection’ while leaving the VM on. Entered the VM public IP address, clicked ‘Connect’, ‘More choices’, and ‘Use a different account’ and entered the wrong username and password. A popup appeared saying ‘Your credentials did not work’.
<img src="https://i.imgur.com/MDr2tHy.png" height="50%" width="50%"/>

- Went back to the VM and refreshed ‘Event Viewer’. An ‘Audit Failure’ appeared in the list. Clicked on it and it displayed a lot of information, notably, the computer name and IP of the login attempt and the username entered. 
<img src="https://i.imgur.com/01pCBaw.png" height="50%" width="50%"/>

The project will mainly use the IP address to help locate where the attacker is from when they attempt to log in to the VM. I can’t show the IP but if you’re following along, it would be under ‘Network Information’. After obtaining the IP address, I will be using ipgeolocation.io which is an API to return the location of the attacker. I will take the information and put it onto my custom log and send it to ‘Log Analythics Workspace’ in Azure where I will then use Sentinel, the SIEM, to plot the attacker onto the map.

**Configuring the VM’s Firewall**
- Pinged the VM’s IP address on my main computer to see if I could get a response but it timed out.
<img src="https://i.imgur.com/ZEJ39Ni.png" height="50%" width="50%"/>

- Configured the Firewall so it can respond to ICMP echo requests so people can discover the VM on the internet. Set ‘Firewall state’ to off for ‘Domain Profile’, ‘Private Profile’, and ‘Public Profile’.
<img src="https://i.imgur.com/TupGNKP.png" height="50%" width="50%"/>

- Pinged the VM IP address again with the configured Firewall and was able to get a response.
<img src="https://i.imgur.com/8x1MHZ3.png" height="50%" width="50%"/>

**Implementing and Running PowerShell Script to Obtain Geographic Data**
- Opened ‘Windows PowerShell ISE’ on the VM. Clicked on ‘new’ and pasted a PowerShell Script that exports security logs. 
- Link to the script: https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1
<img src="https://i.imgur.com/s3YnrWb.png" height="50%" width="50%"/>

Note: Make sure to change the API Key to yours. You can find your API by creating an account at ipgeolocation.io. It’s also important to note that the free subscription will only allow 1000 requests per day. I bought the subscription for the purpose of this project which allows 150,000 requests per month. This will give me more data to use for this project. 

- Ran the script and navigated to ProgramData by using the application ‘Run’ on the VM and entered ‘C:\ProgramData\’. A log file appeared with the name ‘failed_rdp’. 
<img src="https://i.imgur.com/MNQmSZr.png" height="50%" width="50%"/>

- On Powershell ISE, you can see attackers trying to infiltrate the VM in the purple text.
<img src="https://i.imgur.com/IrUzDPi.png" height="60%" width="60%"/>

**Creating Custom Log and Extracting Fields**
- Opened up the log document ‘failed_rdp’ and copied the whole content then pasted it onto a notepad on the local computer, naming it ‘failed_rpd.log’. Saved it to a place where I can navigate to it easily.
Navigated to ‘Log Analytics workspaces’ on Azure, then selected the project’s workspace, navigated to ‘Tables’ under settings, and then created a new custom log (MMA-based’), selected the ‘failed_rpd.log’ that was created. 
- Entered, the information for ‘Collection paths’, the path is ‘C:\programdata\failed_rdp.log’.
<img src="https://i.imgur.com/mcl25Tg.png" height="60%" width="60%"/>

- Entered ‘FAILED_RDP_WITH_GIO’ as the custom log name. 
Navigated to ‘Logs’ and typed a script and clicked ‘Run’. It takes a few minutes for the logs to appear. 
<img src="https://i.imgur.com/VgJNs16.png" height="60%" width="60%"/>

Note: The script name is RDP_Failure_Log_Analysis and can be found under this project Git Hub.

Link to Script: https://github.com/RiceYuki/SIEM-Honeypot-Azure-Sentinel-Monitoring-and-Geolocation-Mapping/blob/main/RDP_Failure_Log_Analysis

**Setup Map in Sentinel**
- Navigated to ‘Microsoft Sentinel’ > ‘Workbooks’ > ‘Add Workbook’ > Edit.
- Removed the 2 basic analytics query that was there by default, clicked on ‘Add Query’, and pasted the same script that was used in ‘Logs’. Clicked ‘Run Query’ and made sure the script worked. 
- Once it worked, I changed and configured the settings so it could be displayed on a map. Selected ‘Map’ for Visualization, ‘Metric Label’ to ‘label’ and ‘Metric Value’ to ‘event_count’. Saved the workbook.
<img src="https://i.imgur.com/YrZxymh.png" height="60%" width="60%"/>

Note: For ‘Location Info using’ I’m using Latitude/Longitude to make the map’s visualization more detailed. You can switch it to ‘Country or region’ which can be beneficial because it will be easier to map out and be prone to fewer mistakes. 


<h2>Testing the SIEM Honeypot</h2>
This is what the map looks like on November 8, 2023 @ 8:01 P.M.
<img src="https://i.imgur.com/QmDrVUF.png" height="60%" width="60%"/>

This is what the map looks like on November 8, 2023 @ 8:31 P.M.
<img src="https://i.imgur.com/69qQvVK.png" height="60%" width="60%"/>

**Analysis**

In just a brief 30-minute testing period, a noticeable surge in attack attempts occurred, resulting in the addition of several previously unrepresented countries to the map. Notably, countries such as Brazil, Indonesia, Singapore, India, and the Netherlands were introduced to the threat landscape. This trend strongly suggests that prolonging the VM's operational duration will likely lead to the further inclusion of new countries on the geo-location map as a consequence of expanded attack activities.

<h2>Summary</h2>
In this project, we set up a virtual machine in Microsoft Azure, specifically configuring it as a honeypot to intentionally attract malicious traffic. This VM was made public to allow easy detection by potential attackers, and we initiated monitoring of security events.

To enhance the monitoring capabilities, we created a custom log that contains geographic information about where the attackers are coming from. This data is collected using Microsoft Defender for the Cloud and enhanced data collection settings. By connecting the VM to a Log Analytics workspace and Azure Sentinel, we can effectively log and analyze the incoming attack attempts.

The project also involved configuring the VM's firewall settings to enable it to respond to ICMP echo requests, allowing it to be discoverable on the internet. This step is crucial for attracting potential attackers to the honeypot.

To extract valuable information from the logs, we implemented a PowerShell script that exports security logs and identifies malicious login attempts. This script utilizes the ipgeolocation.io API to determine the geolocation of attackers. The obtained data is then integrated into a custom log in the Log Analytics workspace.

Subsequently, we created a custom log and extracted relevant fields, such as latitude, longitude, source host, and country, to facilitate analysis and visualization.

To visualize and analyze the geolocation data, we configured Microsoft Sentinel (SIEM) workbooks. These workbooks use the extracted data to create a map that visually represents the location of attack attempts on the VM. By identifying the source locations of these attacks, security professionals can gain a better understanding of potential threat actors' origins.

Throughout the project's testing phase, we observed a substantial increase in attack attempts within just 30 minutes, resulting in the mapping of new countries on the geolocation map. This observation underscores the importance of ongoing monitoring and the potential for discovering new threat sources.

In conclusion, the SIEM Honeypot project highlights the significance of proactive cybersecurity measures and the power of SIEM platforms like Azure Sentinel in analyzing and responding to potential threats. It demonstrates the value of geolocation data in understanding the global nature of cyberattacks and emphasizes the need for continuous monitoring to bolster security measures. The project provides a practical example of how organizations can use honeypots and SIEM solutions to enhance their cybersecurity posture and better protect their systems and data.

<h2> Alternative Project Link</h2>
If you find the pictures to be too blurry, I have linked a Google document that has better-quality pictures with the exact same project content.

https://docs.google.com/document/d/1QjU2JFniYg2MMAA_PY3043z-RHvJdlAyOjkohCV2wKQ/edit?usp=sharing
<h2>Sources</h2>

https://ipgeolocation.io/

https://azure.microsoft.com/

https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1
