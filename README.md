# ansible-windows

### Setup Win 2019 Datacenter server in Azure
- open https://portal.azure.com
- Create a Resource
- Choose Windows Server 2019 Datacenter
- Suggestions during VM creation
  - Create a new resource group, calling mine ansible-lab
  - Create a small data disk on the VM
  - Configure network security group (NSG) with incoming RDP rule after VM creation to allow just your Public IP as source address
  - Configure NSG to allow incoming WinRM HTTPS communication port 5986 and allow just your Public IP as source address

### Installing Ansible and PyWinRM on Debian Linux (Ansible controller)
- pip install ansible
- PyWinRM is required to allow Ansible to use communicate with Windows over WinRM
- pip install "pywinrm>=0.3.0"

### Using NTLM Authentication with Ansible
- Instead of using Basic Authentication I'm opting to use NTLM since Basic is clear-text authentication
- I found the following site to help understand how to setup WinRM to do NTLM auth: https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html
- On the site above, I'm on the section titled "WinRM Setup"
- Open PowerShell ISE as Administrator on the Windows VM we have running in Azure.  In ISE I'm inputting the script shown below from site above.
```  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file 
```
- Next we should enumerate our WinRM listener on our Windows server, so we can see that our HTTPS listener is setup

```winrm enumerate winrm/config/Listener```

### Checking if Ansible Controller can Connect to Windows Host
- On Linux box, open Terminal
- Create directory to store our Ansible inventory file

```mkdir /etc/ansible```
- Create hosts file to store inventory

```sudo nano /etc/ansible/hosts```

- In the above file you will input the following:
```
[windows]
{public IP address of Windows host goes here}

[windows:vars]
ansible_user={local admin user of Windows host goes here }
ansible_password={local admin password of Windows host goes here}
ansible_connection=winrm
ansible_port=5986
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
```
- Run win_ping now to make sure Ansible can connect to your Windows host.  Note you should receive "SUCCESS" in output.
- Note that inputting the password in the inventory file like I did above is only for testing.  Use Ansible Vault to encrypt secret values in config files.

```ansible windows -m win_ping```

### Cloning Repo, Running Windows Updates, and Renaming Windows Host
- I want to clone this repo now to my Linux host, so I can start building some Ansible playbooks for Windows
- In Linux terminal run the following.  I would of course recommend using your own Git repository, so you can make changes easily.

```git clone https://github.com/tstone43/ansible-windows.git```

- After cloning the repo, I create a new folder called "win_updates" in my repo
- Under this new folder, I create a playbook.yml file that contains a play to run Windows Updates on my Windows host.
- I run the following command to execute the Windows Update playbook against my Windows host:

```ansible-playbook playbook.yml```
- I remote into the Windows server and see that the server installed Windows Updates.  I'm noticing the Cumulative Update for Windows didn't install.  Might have to review further.
- Next I create a new directory in this repo called "win_hostname"
- I switch into this directory and I create a playbook.yml
- This playbook.yml file will rename the host since I'm planning on making it a Domain Controler

```ansible-playbook playbook.yml```
- If you want to verify hostname change you can run this command from Ansible host:

```ansible windows -m ansible.windows.win_command -a "hostname"```

### Configuring Network Interface with Static IP
- Next I'm going into https://portal.azure.com and going to the Network Interface settings for my Windows VM
- I'm configuring the Network Interface to use a Static address instead of Dynamic in the Azure portal since this is a Domain Controller
- We still need to configure the IP address to be Static on the Windows Server
- There doesn't appear to be a module to set a static IP for a NIC in Ansible, so I'm going to try this using a PowerShell script I found: https://www.pdq.com/blog/using-powershell-to-set-static-and-dhcp-ip-addresses-part-1/ 
- A multi-line PowerShell script is being run to set the IP Address and Default Gateway
- Note there is a bug in Ansible if you have apostrophe in any comments in your multi-line script you will get an odd error
- A great way to debug your multi-line script is to run the playbook like this:

```ansible-playbook playbook.yml -vvv```
- Notice this multi-line script has "async: 100" configured otherwise timeout error occurs when running playbook
- To determine facts about Windows host I'm running this command:

```ansible windows -m setup```
- I am able to retrieve the name of the connection for the first interface by using this fact: {{ansible_interfaces[0].connection_name}}
- I am using the fact directly above to set DNS IP Address for the connection name, e.g., my connection name on Windows is "Ethernet 2"
- To verify that this playbook worked you can run this on Ansible host:

```ansible windows -m ansible.windows.win_command -a "ipconfig /all"```

### Local Administrator on Windows Host Needs Password
- Found that when using ansible.windows.win_domain that I can't create domain
- The reason being because the local admin becomes domain admin and needs strong password
- Require playbook to enable local admin and configure password
- Created folder called "win_user"
- Switched into this directory then created a playbook with ansible.windows.win_user module
- Straightforward to set the password for existing local admin user with this module

### Configuring Windows Server as Domain Controller
- The above collection needs to be installed on the Linux host with this command:

```ansible-galaxy collection install ansible.windows```
- Going to try the Ansible module called "ansible.windows.win_domain_controller"
- Next creating a folder called win_domain_controller and switching there to start building playbook
- Built out the playbook in win_domain_controller, but found that this module requires an existence of a Windows domain to work
- Discovered another module called ansible.windows.win_domain
- 







