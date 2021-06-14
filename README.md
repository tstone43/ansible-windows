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
```[windows]
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

### Cloning this Repo to my Linux Host
- I want to clone this repo now to my Linux host, so I can start building some Ansible playbooks for Windows
- 




