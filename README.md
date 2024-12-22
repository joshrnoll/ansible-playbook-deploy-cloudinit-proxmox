# Deploy Cloud Init Templates to Proxmox

### Prerequisites
You will need SSH access to your proxmox server(s).

Ensure you have Ansible installed. Install the joshrnoll.homelab collection with the following command:

```
ansible-galaxy collection install joshrnoll.homelab
```

Clone this repository. Ensure all work is done from within the root of the repo.

### Steps:

#### 1. Create a Proxmox API token 
Go to ```Datacenter``` --> ```API Tokens```. For testing purposes, use **root@pam** as the user and **uncheck Privilege Separation**. Make note of your Token ID and secret, you will need them in the next step.

#### 2. Generate a secrets.yml file 
With this command, providing a password (key) when prompted:

```
ansible-vault create secrets.yml
```
Enter your contents, following the example in the ```example_secrets.yml``` file. Press ```ctrl + x``` and then ```Y``` to save and close the window.

If you need to edit the file later simply run:

```
ansible-vault edit secrets.yml
```

#### 3. Edit your variables in ```main.yml```

```YAML
# Optional customizations for ubuntu
proxmox_template_vm_ubuntu_storage: "zfs-pool-01" # name of your storage pool where the VM disks will be.
proxmox_template_vm_ubuntu_name: ubuntu-2204-template # name as it will appear in the GUI
proxmox_template_vm_ubuntu_memory: 4096 # Memory in MB
proxmox_template_vm_ubuntu_cores: 1 # CPU Cores
proxmox_template_vm_ubuntu_ciuser: serveradmin # Username for admin user
proxmox_template_vm_ubuntu_cipassword: "{{ cipassword }}" # From Ansible vault
proxmox_template_vm_ubuntu_sshkeys: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_rsa.pub') }}" # gets your ssh key from /home/user/.ssh/id_rsa.pub -- customize this to your needs
proxmox_template_vm_ubuntu_vlan: 50 # Optional. Remove this line if you do not use VLAN tagging.

# Optional customizations for fedora
proxmox_template_vm_fedora_storage: "zfs-pool-01" # name of your storage pool where the VM disks will be.        
proxmox_template_vm_fedora_name: fedora-40-template # name as it will appear in the GUI
proxmox_template_vm_fedora_memory: 4096 # Memory in MB
proxmox_template_vm_fedora_cores: 1 # CPU Cores
proxmox_template_vm_fedora_ciuser: serveradmin # Username for admin user
proxmox_template_vm_fedora_cipassword: "{{ cipassword }}" # From Ansible vault
proxmox_template_vm_fedora_sshkeys: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_rsa.pub') }}" # gets your ssh key from /home/user/.ssh/id_rsa.pub -- customize this to your needs
proxmox_template_vm_fedora_vlan: 50 # Optional. Remove this line if you do not use VLAN tagging.

# Set to true if you have slow storage to avoid file locks. Remove this line if you do not have slow storage.
proxmox_template_vm_slow_storage: true
```
#### 4. Edit ```hosts.yml``` 
Change ```server-name-here:``` to the name of your Proxmox server. This can be any descriptive name. If you have a cluster with multiple servers add additional lines. **Don't forget the colon:**


#### 5. Rename the ```server-name-here.yml``` file 
To whatever you called your server in ```hosts.yml```. If you have multiple servers, add more files. Ensure the filenames match what is in ```hosts.yml```.

#### 6. Edit each of your ```server-name-here.yml``` files
Change the ```ansible_user``` entry to the user that has SSH access to the server. Change the ```ansible_host``` entry to be the **IP address of the server**.

#### 7. Run the playbook with the following command:

```
ansible-playbook main.yml -i hosts.yml -J
```
