## <a name="option_c">Option C: Step 1: Spin up Detection Lab locally</a> 
  1. Follow instructions here to spin up DetectionLab locally: https://github.com/clong/DetectionLab
  2. From the console for "logger" vm (or via ssh):
  ```
  su splunk
  cd /opt/splunk/etc/apps
  git clone https://github.com/daveherrald/SA-attck_nav.git
  ```
  3. From the UI, Navigate to "Administrator" --> "Account Settings"
  4. Change Administrator password to a new value
  5. From the UI, navigate to "Apps" --> "Find More Apps"
    - Search for "Phantom" and install "Phantom App for Splunk"
    - Search for "lookup" and install "Lookup File Editor"
    - Search for "CIM" and install "Splunk Common Information Model (CIM)"
    - Install Base64 app from Splunkbase: https://splunkbase.splunk.com/app/1922/
    - Then restart Splunk
  7. From the UI, navigate to "Settings" --> "Access Controls"
  8. Click "Roles", then "admin"
  9. Under the "inheritance" section, add the "phantom" role.  
  10. Scroll down and click "save" at the bottom.
  11. Unless you have a valid certificate for Phantom, you will need to disable certificate validation by running:
  ```
  curl -ku 'username:password' https://<splunk-address>:8089/servicesNS/nobody/phantom/configs/conf-phantom/verify_certs\?output_mode\=json -d value=0
  ```
   - with the appropriate substitutions, of course
   - CAVEAT EMPTOR: Disabling certificate checking is not allowed in Splunk Cloud and does make the setup less secure.

  10. Create the "security" index if using the inputs.conf below
  11. Will need to make sure the lookup for attck_assets is correct, either using "Lookup Editor" Splunk app or manually editing.

    - Go to Apps --> Lookup Editor
    - Under the "Lookups" title and to the right, click on the filter labeled "App: All" and select "Adversary Simulator"
    - Click the only lookup there, "attck_assets.csv"
    - Adjust these lines to match your environment

  14. Possibly still needed(?): Modify the file `/opt/splunk/etc/apps/phantom/bin/ta_addonphantom/modalert_phantom_forward_helper.py`
    - Comment out the "return results" line and uncomment the "return 0" line

### Setting up Phantom:
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

### End of Step 1: Option C. At this point, you have the basic Splunk/Phantom setup with DetectionLab


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