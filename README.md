# Blocking-a-Known-Malicious-Actor-In-Wazuh
This repo contains the step by step guide for how to block a known malicious actor using wazuh active-response modules.

## Blocking a known malicious actor
- This section provides illustrations on how to prevent unauthorized access to web resources on a web server by blocking malicious IP addresses. The process involves setting up an Apache web server on an Ubuntu virtual machine and conducting a test using a Kali virtual machine, which simulates an attacker.

In order to prevent access from a potentially harmful IP address, such as a Kali virtual machine which in this case is the attacker, it is necessary to add said IP address to the reputation database. Following this, the configuration of Wazuh will effectively restrict the Kali VM’s access to the Apache web server for a duration of 60 seconds.

In this use case, I will be using the Wazuh [CDB list](https://documentation.wazuh.com/current/user-manual/ruleset/cdb-list.html) and [active response](https://documentation.wazuh.com/current/getting-started/use-cases/active-response.html) capabilities.

### Configuring the Windows endpoint
1. Install the Apache web server
  - Install the latest [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe) package.
  - Download the Apache web server [ZIP installation file](https://www.apachelounge.com/download/VS17/binaries/httpd-2.4.63-250207-win64-VS17.zip).
  - Unzip the contents of the Apache web server zip file and copy the extracted Apache24 folder to the C: directory.
    ```
    Expand-Archive -Path .\httpd-2.4.63-250207-win64-VS17.zip -DestinationPath 'C:\'
    ```
  - Navigate to the C:\Apache24\bin folder and run the following command in a PowerShell terminal with administrator privileges:
    ```
    cd C:\Apache24\bin>httpd.exe
    ```
  - Click on Allow Access. This allows the Apache HTTP server to communicate on your private or public networks depending on your network setting. It creates an inbound rule in your firewall to allow incoming traffic on port 80.
  -  Click on Allow Access. This allows the Apache HTTP server to communicate on your private or public networks depending on your network setting. It creates an inbound rule in your firewall to allow incoming traffic on port 80.
    ![image](https://github.com/user-attachments/assets/1679d2ea-ea19-42ab-8c87-92b6390eca4f)


### Configure the Wazuh agent
1. Add the following to C:\Program Files (x86)\ossec-agent\ossec.conf to configure the Wazuh agent and monitor the Apache access logs:
   ```
   <localfile>
     <log_format>syslog</log_format>
     <location>C:\Apache24\logs\access.log</location>
   </localfile>
   ```
2. Restart the Wazuh agent in a PowerShell terminal with administrator privileges to apply the changes

### Configuring the Wazuh Server
- Here are the steps I followed to add the IP address of my Kali VM to a CDB list and configure rules and active response.
1. Install the wget utility to download the necessary artifacts using the command line interface:
   ```
   sudo apt update -y && sudo apt install wget -y
   ```
2. Download the Alienvault IP reputation database:
   ```
   sudo wget https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/alienvault_reputation.ipset -O /var/ossec/etc/lists/alienvault_reputation.ipset
   ```
3. Append the IP address of the attacker endpoint to the IP reputation database. Replace <ATTACKER_IP> with my Kali IP address in the command below:
   ```
   sudo echo "<ATTACKER_IP>" >> /var/ossec/etc/lists/alienvault_reputation.ipset
   ```
4. Download a script to convert from the .ipset format to the .cdb list format:
   ```
   sudo wget https://wazuh.com/resources/iplist-to-cdblist.py -O /tmp/iplist-to-cdblist.py
   ```
5. Convert the alienvault_reputation.ipset file to a .cdb format using the previously downloaded script:
   ```
   sudo /var/ossec/framework/python/bin/python3 /tmp/iplist-to-cdblist.py /var/ossec/etc/lists/alienvault_reputation.ipset /var/ossec/etc/lists/blacklist-alienvault
   ```
6. Optional: Remove the alienvault_reputation.ipset file and the iplist-to-cdblist.py script, as they are no longer needed:
   ```
   sudo rm -rf /var/ossec/etc/lists/alienvault_reputation.ipset 
   sudo rm -rf /tmp/iplist-to-cdblist.py
   ```
7. Assign the right permissions and ownership to the generated file:
   ```
   
   ```
8. 
