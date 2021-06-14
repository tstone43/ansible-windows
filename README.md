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



