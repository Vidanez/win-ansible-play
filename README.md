#This is a exercise to answer differents point of a list of 5
1. Ansible for Managing Windows Services
2. Windows Server Monitoring
3. VMWare
4. Windows updates management
5. Antivirus Management

## Folder Structure for all points

```
ansible-windows-services/
├── ansible.cfg
├── inventory/
├── playbooks/
│   ├── install_iis.yml
│   ├── install_w32time.yml
│   ├── deploy_app.yml
│   └── manage_service.yml
├── roles/
│   ├── iis/
│   │   └── tasks/      
│   ├── w32time/
│   │   └── tasks/
│   └── app/
│       ├── tasks/
│       └── files/
└── vars/

    
```


# 1. Ansible Windows Services Management

## Overview

This project demonstrates how to use Ansible to install, configure, and start a Windows service across multiple servers.
In this example we will use service with inputs parameters IIS (Internet Information Services) and without inputs parameters Windows Time Service (w32time). 
I'm keeping it easy and not using ansible-galaxy and focusing into those 2 service as example. We could change it to do it from a list in a loop easily to install or remove a list of files and use bigger jinja templates to control variables and complicate installations
It also covers deploying a sample application and registering it as a service with environment-specific configurations, pulling it from GIT.

## About WinRM in the latest version of Ansible:
WinRM is a management protocol used by Windows to remotely communicate with another server. It is a SOAP-based protocol that communicates over HTTP/HTTPS and is included in all recent Windows operating systems. Since Windows Server 2012, WinRM has been enabled by default, but in most cases, extra configuration is required to use WinRM with Ansible.

Historically Ansible used Windows Remote Management (WinRM) as the connection protocol to manage Windows nodes. The psrp and winrm connection plugins both operate over WinRM and can be used as the connection plugin for Windows nodes. The psrp connection plugin is a newer connection plugin that offers a few benefits over the winrm connection plugin, for example:
  -Can be slightly faster
  -Less susceptible to timeout issues when the Windows node is under load
  -Better support for proxy servers

   https://docs.ansible.com/ansible/devel//os_guide/intro_windows.html
   https://docs.ansible.com/ansible/latest/os_guide/windows_winrm.html
   
   Note: SSH is also available   

## Prerequisites in this easy example

1. **Ansible**: Install Ansible on your control machine from where we will run the playbooks.
   ```sh
   sudo apt-get update
   sudo apt-get install ansible
   ```
2. **pywinrm**: Install pywinrm on the control machine. We need it in order to manage the WinRM.
   ```sh
   pip install pywinrm
   ```

3. **WinRM Configuration on Windows Servers**: Run the following PowerShell script on each Windows server to enable WinRM basic by HTTP and easy configuration username/password.
  
   ```powershell
   winrm quickconfig -q
   winrm set winrm/config/service '@{AllowUnencrypted="true"}'
   winrm set winrm/config/service/auth '@{Basic="true"}'
   winrm set winrm/config/listener?Address=*+Transport=HTTP
   winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
   ```


## Playbooks

### Install IIS (With parameters)

**File**: `playbooks/install_iis.yml`

**Usage:**:
  ```sh
   ansible-playbook -i inventory/development playbooks/install_iis.yml -e @vars/development.yml```
  ```sh
   ansible-playbook -i inventory/testing playbooks/install_iis.yml -e @vars/testing.yml```
  ```sh
   ansible-playbook -i inventory/production playbooks/install_iis.yml -e @vars/production.yml```

### Install Windows Time Service (without parameters)

**File**: `playbooks/install_w32time.yml`

Usage:
  ```sh
    ansible-playbook -i inventory/development playbooks/install_w32time.yml```
  ```sh
    ansible-playbook -i inventory/testing playbooks/install_w32time.yml```
  ```sh
    ansible-playbook -i inventory/production playbooks/install_w32time.yml```

### Routine Service Management Tasks check status, start, stop and restart

**File**: `playbooks/manage_service.yml`

**Usage:**
 Dynamic Targeting: By using the target_server variable, you can dynamically specify which server or group of servers to target even using word all 
 for managing services.
 Service Selection: The service_name variable allows you to choose which service to manage.
 Tags: The tags (check, start, stop, restart) allow you to run specific tasks within the playbook.
  ```sh
    ansible-playbook -i inventory/hosts playbooks/manage_service.yml --tags "VARIABLE" -e "target_server=VARIABLE" -e "service_name=VARIABLE"```




### Deploy Sample Application

**File**: `playbooks/deploy_app.yml`

**Usage:**
  ```sh
    ansible-playbook -i inventory/development playbooks/deploy_app.yml -e @vars/development.yml```
  ```sh
    ansible-playbook -i inventory/testing playbooks/deploy_app.yml -e @vars/testing.yml```
  ```sh
    ansible-playbook -i inventory/production playbooks/deploy_app.yml -e @vars/production.yml```


## Inventory Files per each environment we want to use we need to declare the host that live at that environment.
There is options not cover here to do update these files dinamically, getting this information from VMWare or DNS

### Development

**File**: `inventory/development`


### Testing

**File**: `inventory/testing`


### Production

**File**: `inventory/production`


## Environment Variables

### Development

**File**: `vars/development.yml`


### Testing

**File**: `vars/testing.yml`


### Production

**File**: `vars/production.yml`


## How to Use to deploy the same app in different environment taking in consideration posible differences by variables

1. **Clone the ansible repo:**

   ```sh
   git clone https://github.com/Vidanez/win-ansible-play.git
   cd win-ansible-play
   ```
2. **Set Up Inventory Files:**
   Update the `inventory/development`, `inventory/testing`, and `inventory/production` files with your server details.

3. **Example for install an app:**

Example for Development Environment
   ```sh
      ansible-playbook -i inventory/development playbooks/deploy_app.yml -e "env=development"```

Example for Testing Environment
   ```sh
      ansible-playbook -i inventory/testing playbooks/deploy_app.yml -e "env=testing"```

Example for Production Environment
   ```sh
      ansible-playbook -i inventory/production playbooks/deploy_app.yml -e "env=production"```

**Explanation**
    - Environment Variables: The environment-specific variables are defined in       separate files (development.yml, testing.yml, production.yml) under the vars directory. These include the Git repository URL and the application path.

    - Role: The deploy_app role handles cloning the application repository from Git and registering the application as a service.
Playbook: The playbook deploy_app.yml includes the deploy_app role and loads the environment-specific variables.
    - Running the Playbook: The playbook is run with the environment variable file passed as an extra variable.
This setup ensures that the application is pulled from the specified Git repository and deployed with environment-specific configurations. If you have any more questions or need further assistance, feel free to ask!


## Modules would you typically use to manage windows
https://docs.ansible.com/ansible/devel//os_guide/intro_windows.html#which-modules-are-available

- win_service https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
- win_feature https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_feature_module.html
- win_shell https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_shell_module.html
- win_copy https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_copy_module.html
- win_template https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_template_module.html

# 2. Windows Server Monitoring
The usual things to monitor in a machine not taking in consideration the virtualization layer where it is running are:
    - CPU
    - Memory
    - Disk space and amoutn of IOPs
    - Network bandwith outcome / Income. Based in the kind of netwrok itnereface we can easily detect exahustation of the device
    - Network latency ( we needd to establish lacenty raange acceptable from/to external from the VM device consuming the sevices)  
    - Services that we want to keep running

# 3. VMWare

We have several options to create a virtual machine at VMWare from directly clicking in the bottom of the GUI (vcenter and ESXi), by a OVF or OVA (export /import), by and script in powercli, by the vcenter API or as part of a vRealise Automation.  
Using templates (Vms can be transform in templates) we can deploy templates direclyt in the GUI, using content library, using powercli and using API.
In order to automate the best aproach is to do it using templates. The VM approach is available but has the tendency to complicate things and we need to keep KISS approach. 

ISSUE using Ansible to deploy VMs. One of the thing that happens is that if we use ansible to deploy from template it use the clone feature and not the deploy so this limit the new VM to be create only in the same datastore cluster where the template is located.

10 Years ago I created this code https://github.com/Vidanez/DeployVMs/tree/master that is still valid to deploy VMs in bulk not cloning but deploying from template.


# 4. Windows Updates Management

# 5. Antivirus Management 
