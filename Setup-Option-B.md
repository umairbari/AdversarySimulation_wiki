## <a name="option_b">Option B: Step 1: Setting up Splunk:</a>

1. Launch Splunk Enterprise AMI on AWS (or on-prem version) (tested with version 7.2.5)
2. Commands from Splunk instance CLI
```
sudo su
yum install git -y
su splunk
cd ~
git clone https://github.com/timfrazier1/AdversarySimulation.git
cd /opt/splunk/etc/apps
git clone https://github.com/daveherrald/SA-attck_nav.git
tar -xzf ~/AdversarySimulation/resources/splunk_apps/phantom-app-for-splunk_275.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/phantom-remote-search_109.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/splunk-app-for-phantom-reporting_100.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/base64_11.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/add-on-for-microsoft-sysmon_810.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/lookup-file-editor_332.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/splunk-common-information-model-cim_4130.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/splunk-security-essentials_252.tgz
tar -xzf ~/AdversarySimulation/resources/splunk_apps/threathunting_141.tgz
/opt/splunk/bin/splunk restart
```

3. From the UI, navigate to "Settings" --> "Access Controls"
4. Click "Roles", then "admin"
5. Under the "inheritance" section, add the "phantom" role.  
6. Scroll down and click "save" at the bottom.
7. Go to the "Overview" tab of the ATT&CKsim app and Reset the layer by checking the "Reset" radio button and clicking Submit
8. Unless you have a valid certificate for Phantom, you will need to disable certificate validation by running:
```
curl -ku 'username:password' https://splunk_server:8089/servicesNS/nobody/phantom/configs/conf-phantom/verify_certs\?output_mode\=json -d value=0
```
 - with the appropriate substitutions, of course
 - CAVEAT EMPTOR: Disabling certificate checking is not allowed in Splunk Cloud and does make the setup less secure. See optional section below for help on creating a certificate if you need it.

9. Create the "security" index if using the inputs.conf below
  - Go to the "Settings" menu, then "Data" --> "Indexes"
  - Click the "New Index" button in the top right
  - Give it the name "security", leave the rest default and click "Save" at the bottom
10. You will need to make sure the lookup for attck_assets is correct, either using "Lookup Editor" Splunk app or manually editing.
 - Go to App context menu in upper left and go to "Lookup Editor" app
 - Look for the "attck_assets.csv" file and click on it
 - change the hostname, os and ip_address to match your test box(es)
 - If you haven't set up your test boxes (windows instructions below) then you may not have this yet.  I will remind you to revisit later in these instructions.


### Step 2: Setting up Phantom:
1. Launch Splunk Phantom AMI on AWS (or on-prem)
2. Login with admin/password  (You should change your password)
3. Go to Administration --> User Management --> Users
4. Click on "automation" User
5. Change "Allowed IPs" to "any" (or appropriate subnet, if you prefer to be more secure)
6. Copy everything in the "Authorization Configuration for REST API" section
7. Click "Save"

Go back to Splunk:
1. Navigate to Apps --> Phantom --> Phantom Server Configuration
2. Click "Create Server"
3. Paste clipboard into "Authorization Configuration"
4. Give it a name (such as Phantom AWS)
5. Click "Save"
 - If something is wrong, you will get an error here
6. You can also click the "Manage" menu under "Actions" on the right hand side and select "Test Connectivity" to explicitly verify that everything is working
7. After successful testing, click "Manage" again and "Set Default", as well as "Sync Playbooks"


#### For Windows Server 2019:
  1. Stand up AWS AMI for Windows Server 2019 Base
  2. Download and modify swiftonsecurity sysmon-config:
    - Key exclusions of splunk processes under process creation:
      - ```<Image condition="is">C:\Program Files\Splunk\bin\btool.exe</Image>
        <Image condition="is">C:\Program Files\Splunk\bin\splunkd.exe</Image>
        <Image condition="is">C:\Program Files\SplunkUniversalForwarder\bin\splunk-winprintmon.exe</Image>
        <Image condition="is">C:\Program Files\SplunkUniversalForwarder\bin\splunk-powershell.exe</Image>
        <ParentImage condition="is">C:\Program Files\SplunkUniversalForwarder\bin\splunkd.exe</ParentImage>
        ```
    - ~~Key inclusion for Mimikatz:~~
      - ~~<TargetImage condition="is">C:\windows\system32\lsass.exe</TargetImage>~~

  3. Download and install sysmon:
    https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
    ```
    .\Sysmon.exe -accepteula -i .\sysmonconfig-export.xml
    ```

  4. Enable powershell logging to match settings at: https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html

  5. Download and install Splunk Universal Forwarder:
    https://www.splunk.com/en_us/download/universal-forwarder.html
    ```
    wget -OutFile splunkforwarder-7.3.0-657388c7a488-x64-release.msi 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=windows&version=7.3.0&product=universalforwarder&filename=splunkforwarder-7.3.0-657388c7a488-x64-release.msi&wget=true'
    ```
  6. Install Universal Forwarder as local system (accept defaults)
    - create local admin splunk_svc_acct
    - point to Splunk AWS Box IP as indexer (enter default port)
  7. Modify inputs.conf to include:
  ```
# Windows platform specific input processor.
[WinEventLog://Application]
disabled = 0
index = security
[WinEventLog://Security]
disabled = 0
index = security
[WinEventLog://System]
disabled = 0
index = security

  [WinEventLog://Microsoft-Windows-TaskScheduler/Operational]
  disabled = 0
  index = security

  [WinEventLog://Microsoft-Windows-WinRM/Operational]
  disabled = 0
  index = security

  [WinEventLog://Microsoft-Windows-Sysmon/Operational]
  disabled = 0
  renderXml = true
  index = security

  [WinEventLog://Microsoft-Windows-PowerShell/Operational]
  disabled = 0
  index = security
  blacklist1 = 4105,4106
  blacklist2 = EventCode="4103" Message="(?:SplunkUniversalForwarder\\bin\\splunk-powershell.ps1)"

  [monitor://C:\var\log\transcripts\]
  disabled = false
  index = security
  sourcetype = powershell_transcript
 ```
  8. WinRM should be turned on and working with http out of the box. You can check with the following command: `winrm get winrm/config`
    - Make sure that under "service", then "Auth" that Kerberos is set to true.  
    - In order to use HTTP, "AllowUnencrypted" under "service" will also need to be set to true.
  9. Optional: To use HTTPS, we will be following these instructions: https://www.visualstudiogeeks.com/devops/how-to-configure-winrm-for-https-manually
    - Basically, you will need to run the following in powershell:

    ```
    winrm quickconfig
    New-SelfSignedCertificate -DnsName "<YOUR_DNS_NAME>" -CertStoreLocation Cert:\LocalMachine\My
    winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="<YOUR_DNS_NAME>"; CertificateThumbprint="<COPIED_CERTIFICATE_THUMBPRINT>"}'
    # Add a new firewall rule
    port=5986
    netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=$port
    ```

    - Open port 5986 inbound on AWS for Server (sometimes this is already present)

    To review WinRM config:
    ```
    winrm get winrm/config -format:pretty
    ```

#### For Windows 10:
  1. Stand up AWS Workspaces windows 10 box
  2. Download and install Splunk Universal Forwarder:
    https://www.splunk.com/en_us/download/universal-forwarder.html
    - with wget:
    ```
    wget -OutFile splunkforwarder-7.3.0-657388c7a488-x64-release.msi 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=windows&version=7.3.0&product=universalforwarder&filename=splunkforwarder-7.3.0-657388c7a488-x64-release.msi&wget=true'
    ```
  3. Install Universal Forwarder as local system (accept defaults)
    - create local admin splunk_svc_acct
    - ~~point to Splunk AWS Box IP as deployment Server (enter default port)~~
    - point to Splunk AWS Box IP as indexer (enter default port)
    -
  4. Download and modify Olaf Hartong's sysmon-config.  I like this one here: https://github.com/olafhartong/sysmon-configs/blob/master/sysmonconfig-v10.xml
    - You may want to add more exclusions for splunk processes such as:
      - ```
      	 <ProcessCreate onmatch="exclude">
      <Image condition="is">C:\Program Files\Splunk\bin\btool.exe</Image> <!--Splunk Processes-->
      <Image condition="is">C:\Program Files\Splunk\bin\splunkd.exe</Image> <!--Splunk Processes-->
		<Image condition="is">C:\Program Files\SplunkUniversalForwarder\bin\splunk-winprintmon.exe</Image> <!--Splunk Processes-->
		<Image condition="is">C:\Program Files\SplunkUniversalForwarder\bin\splunk-powershell.exe</Image> <!--Splunk Processes-->
		<ParentImage condition="is">C:\Program Files\SplunkUniversalForwarder\bin\splunkd.exe</ParentImage> <!--Splunk Processes-->
        ```

  5. Download and install sysmon:
    https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
    ```
    sysmon.exe -accepteula -i sysmonconfig-v10.xml
    ```

### End of Step 1: At this point, you have the basic Splunk/Phantom/Windows set up in AWS


## Step 2: Further Phantom Setup with Apps & Playbooks:
  Back in Phantom: (Assumption is that the Phantom VM has internet connectivity to download Atomic Red Team)

- Setting up Win RM
  1. Go to Apps from the main menu
  2. Search for "Windows" and find the Windows Remote Management app under unconfigured Apps
  3. Click "Configure New Asset" on the right hand side and give the asset a name while in the "Asset Info" tab
  4. Under "Asset Settings", point to your Win Server hostname for test connectivity
  5. For AWS Windows box following above Windows Setup:
    - Use HTTP or HTTPS as default protocol depending on how you set it up above
    - If HTTPS, change port to 5986
    - Leave domain blank
    - Use NTLM transport and input your admin user and password
  6. For DetectionLab Windows box:
    - Use the Win10 box IP address for testing
    - Use HTTP as default protocol and leave port as 5985
    - Use WIN10 as the domain
    - Use NTLM transport and input "vagrant" user and "vagrant" password
  7. Save and "Test connectivity" to validate
- Setting up Splunk connectivity from Phantom
  1. From main menu, go to Apps
  2. Search for "Splunk" and click on "unconfigured apps" to find it
  3. Click "Configure new asset" on the right hand side
  4. Give it a name with no spaces under "asset info" then click "asset settings" tab
  5. Put in the IP/Hostname, username and password for your Splunk instance
  6. Change the timezone to UTC (unless you have this set differently in Splunk)
  7. Go to the Ingest Settings tab and select (or define) a label to assign to the inbound Splunk events
  8. Click "Save" go back to the "Asset Settings" tab and then click "Test Connectivity" at the bottom to validate that it is working
- Setting up Atomic Red Team app
  1. Download https://github.com/timfrazier1/AdversarySimulation/blob/master/phatomicredteam.tgz
  2. Go to Apps from the main menu
  3. Click the "Install App" button in the upper right
  4. Select the downloaded "phatomicredteam.tgz" file
  5. Once app is installed, find "Atomic Red Team" in unconfigured Apps
  6. Click "Configure new Asset" on the right hand side
  7. Give your asset a name with no spaces under "asset info" then click "asset settings" tab
  8. Leave the default URL as https://github.com/redcanaryco/atomic-red-team.git if you want to use the main ART repo, otherwise, use your own fork
  9. Hit "Save" and then "Test Connectivity" to build the list of tests
  10. You should see a "Repo Created Successfully" message
- Setting up the Playbook
  1. From the main menu, go to Administration -> Administration Settings -> Source Control
  2. Select "configure new repository", then paste https://github.com/timfrazier1/AdvSimPlaybooks into the URL field
  3. Use "AdvSim" as the name and "master" as the branch
  4. Check the "read-only" box and click "Save"
  5. From the main menu, go to "Playbooks"
  6. On the listing screen, click the "Repo" column and select "AdvSim" to see the playbooks associated with the repo
  7. In the "Status" column, click the dropdown next to "Inactive" and select "Active"
  8. If you want, you can click on "Modular Simulation" to view the playbook in the editor


### At this point, you should be all setup