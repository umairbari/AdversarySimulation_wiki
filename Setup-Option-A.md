This method is still in "beta", but when working, is the easiest method to get all components needed for ATT&CK Sim up and running.

1. Clone the DetectionLab fork here: https://github.com/timfrazier1/DetectionLab
2. Go to the Phantom AMI Page here and click "Continue to Subscribe" in the upper right: https://aws.amazon.com/marketplace/pp/Splunk-Inc-Splunk-Phantom/B07K2HPNJG ![Screenshot](https://github.com/timfrazier1/AdversarySimulation/blob/master/phantom_eula_1.png)
3. Then click "Accept Terms" button to accept the EULA:
![Screenshot](https://github.com/timfrazier1/AdversarySimulation/blob/master/phantom_eula_2.png)
4. Next, follow the instructions here to set up your Terraform profile and variables: https://github.com/timfrazier1/DetectionLab/blob/master/AWS/Terraform/Pre-Built_AMIs.md
5. After running "terraform apply" and typing "yes", the build process should begin.  WARNING: You are about to start spending money on AWS. 
 - Please note that this fork of DetectionLab only installs Splunk Universal Forwarder does not have some components enabled (such as OSquery and Bro) for speed of install reasons.  This can be easily changed by uncommenting lines 446-450 in the file https://github.com/timfrazier1/DetectionLab/blob/master/Vagrant/bootstrap.sh
6. Look for the green output text when the build completes (~20-30 minutes) and you should have your URLs to access Splunk and Phantom. Default username/password for Splunk is admin/changeme.  