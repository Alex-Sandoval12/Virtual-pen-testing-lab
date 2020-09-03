## Summary:
Showing virtual pen testing environment using **VirtualBox** and giving a demo

## Description:
My victim was a Ubuntu VM with OWASP Juice Shop since it has many vulnerabilities. My attacker was a Kali VM because of the many pen testing tools it has pre-installed. From these tools I used nmap to do network recon, Burp helped me repeat HTTP requests and analyze the responses, and I used dirb to find hidden directories in the app. With this I did a SQLi, a dictionary attack and a XSS.

## Steps To Reproduce:

  1. Download and install the VirtualBox app for your OS
  2. Create the victim VM in VirtualBox **(You can skip this and download my victim OVA)**
     1. Create a new Ubuntu VM with the default settings, name it _OWASP Juice Shop v11.1.3_
     2. Download the Ubuntu image from the **Supporting Material**
     3. Enable a new internal network adapter and name it _CompletelySecureNetwork_
     4. Run the VM and select the downloaded Ubuntu image as the start-up disk and follow the installation process
     5. Once the installation is done access the netplan config file with: `sudo nano /etc/netplan/00-installer-config.yaml` and replace its contents with the **Netplan file** given in the **Supporting Material**, then apply changes with: `sudo netplan apply`
     6. Update the packages with: `sudo apt-get update && sudo apt-get upgrade -y`
     7. Install nodejs and npm with: `curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -`, then `sudo apt-get install -y nodejs`
     8. Download the Juice Shop .tgz file with: `wget https://github.com/bkimminich/juice-shop/releases/download/v11.1.3/juice-shop-11.1.3_node10_linux_x64.tgz`
     9. Unpack the file with: `tar -xvzf juice-shop-11.1.3_node10_linux_x64.tgz`
     10. Shut down the VM and disable the NAT adapter
     11. Run the VM and navigate to the Juice Shop directory with: `cd juice-shop_11.1.3/`
     12. Start the Juice Shop with: `npm start`
  3. Create the attack VM in VirtualBox
     1. Download the Kali OVA for VirtualBox
     2. Import the OVA to VirtualBox
     3. Run the VM and update the packages with: `sudo apt-get update && sudo apt-get upgrade -y`, then shut down the VM
     4. Change the network adapter of the VM from NAT to internal network and name the network _CompletelySecureNetwork_
     5. Run the machine, access the network interfaces file with: `sudo nano /etc/network/interfaces` and replace its contents with the **Network interfaces file** given in the **Supporting Material**, then restart the nerwork service with: `sudo systemctl restart networking.service`
  4. Configure Burp
     1. In Kali, open Burp Suite and click next until you get to the main window of Burp
     2. Open firefox and set Burp as the proxy for that browser
     3. Back in Burp turn off the intercept function in the proxy tab
  5. Scan the network with nmap
     1. Open a terminal and scan the network with: `nmap -sV 192.168.65.0/24` to find the victim's IP and open ports
  6. Perform a XSS attack
     1. Navigate to `http://192.168.65.128:3000` in firefox
     2. Click on the magnifying glass and search for `Apple`
     3. Notice that the query is reflected in the URL, replace `Apple` for `<IFRAME SRC="javascript:alert('XSS');"></IFRAME>` and let the magic happen
  7. Perform a SQL injection
     1. Reload the main page of the Juice Shop and then click on the Account button to try to login
     2. Enter `'` as the email and the password and try to login to check if it's vulnerable to a SQL injection
     3. Go to Burp, to the proxy tab, then to the HTTP history sub-tab and look for the POST request we just made, the URL section should say _/rest/user/login_, when you find it send it to Repeater by right clicking it
     4. Go to the Repeater tab and send again the request to confirm that the app is vulnerable to SQLi
     5. Change the `'` from the email field to `' OR 1=1--` and welcome to your VWA, mr. admin
  8. Find the confidential file with dirb
     1. Open a terminal and write the command: `dirb http://192.168.65.128:3000 /usr/share/wordlists/wfuzz/general/common.txt`
     2. Navigate to `http://192.168.65.128:3000/ftp` in firefox
     3. Open the _acquisitions.md_ file and you just ruined the surprise

## Supporting Material/References:
[VirtualBox app](https://www.virtualbox.org/wiki/Downloads)<br />
[My victim OVA](https://iteso01-my.sharepoint.com/:u:/g/personal/si721687_iteso_mx/EXqKRBEdaftKlIbd9RyPToIBZygu6LFxH9POvPupo8IPng?e=SpNGR8)<br />
[Ubuntu image](https://ubuntu.com/download/server/thank-you?version=20.04.1&architecture=amd64)<br />
[Netplan file](https://github.com/Alex-Sandoval12/Virtual-pen-testing-lab/blob/master/Victim%20VM/netplan%20config%20file)<br />
[Kali OVA from Offensive Security](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/#1572305786534-030ce714-cc3b)<br />
[Network interfaces file](https://github.com/Alex-Sandoval12/Virtual-pen-testing-lab/blob/master/Attack%20VM/network%20interfaces%20file)
