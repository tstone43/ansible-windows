# ansible-windows

## Ansible Controller on Mac to Configure Windows

### Setup Mac to be Ansible controller
- open Terminal on Mac
- brew install ansible

### Setup Win 2019 Datacenter server in Azure
- open https://portal.azure.com
- Create a Resource
- Choose Windows Server 2019 Datacenter
- Suggestions during VM creation
  - Create a new resource group, calling mine ansible-lab
  - Create a small data disk on the VM
  - Configure RDP network security group after VM creation to allow just your Public IP


