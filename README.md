# This is a exercise to answer differents point of a list of 5
1. Ansible for Managing Windows Services
2. Windows Server Monitoring
3. VMWare
4. Windows updates management
5. Antivirus Management

Notes: 
- I'm not using ansible galaxy on purpose because I want to show direct usage of the roles. 
- I'm not using neither ansible vault to dont over complicate this example

## Folder Structure for all points

```
├── LICENSE
├── README.md
└── ansible-windows-services
    ├── inventory
    │   ├── development
    │   ├── production
    │   └── testing
    ├── playbooks
    │   ├── deploy_app.yml
    │   ├── deploy_vm.yml
    │   ├── install_iis.yml
    │   ├── install_packages.yml
    │   ├── install_w32time.yml
    │   ├── install_windows_update_module.yml
    │   ├── install_wmi_exporter.yml
    │   ├── manage_service.yml
    │   └── run_windows_updates.yml
    ├── roles
    │   ├── deploy_app
    │   │   ├── tasks
    │   │   │   └── main.yml
    │   │   └── vars
    │   │       └── main.yml
    │   ├── iis
    │   │   └── tasks
    │   │       └── main.yml
    │   ├── install_windows_update_module
    │   │   ├── handlers
    │   │   │   └── main.yml
    │   │   └── tasks
    │   │       └── main.yml
    │   ├── manage_services
    │   │   └── tasks
    │   │       └── main.yml
    │   ├── run_windows_updates
    │   │   ├── handlers
    │   │   │   └── main.yml
    │   │   └── tasks
    │   │       └── main.yml
    │   ├── w32time
    │   │   └── tasks
    │   │       └── main.yml
    │   └── wmi_exporter
    │       └── tasks
    │           └── main.yml
    └── vars
        ├── basicVM.yml
        ├── development.yml
        ├── production.yml
        └── testing.yml


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
   ansible-playbook -i inventory/development playbooks/install_iis.yml -e @vars/development.yml
  ```

  ```sh
   ansible-playbook -i inventory/testing playbooks/install_iis.yml -e @vars/testing.yml
  ```

  ```sh
   ansible-playbook -i inventory/production playbooks/install_iis.yml -e @vars/production.yml
  ```


### Install Windows Time Service (without parameters)

**File**: `playbooks/install_w32time.yml`

Usage:
  ```sh
    ansible-playbook -i inventory/development playbooks/install_w32time.yml
  ```

  ```sh
    ansible-playbook -i inventory/testing playbooks/install_w32time.yml
  ```

  ```sh
    ansible-playbook -i inventory/production playbooks/install_w32time.yml
  ```


### Routine Service Management Tasks check status, start, stop and restart

**File**: `playbooks/manage_service.yml`

**Usage:**
    
     - Service Selection: The service_name variable allows you to choose which service to manage.
     
     - Tags: The tags (check, start, stop, restart) allow you to run specific tasks within the playbook.

     - Dynamic Targeting: By using the target_server variable, you can dynamically specify which server or group of servers to target even using word all for managing services.

     
  ```sh
    ansible-playbook -i inventory/hosts playbooks/manage_service.yml --tags "VARIABLE" -e "target_server=VARIABLE" -e "service_name=VARIABLE"
  ```

### Deploy a Sample Application from a git repository defined in varibles

**File**: `playbooks/deploy_app.yml`

**Usage:**
  ```sh
    ansible-playbook -i inventory/development playbooks/deploy_app.yml -e @vars/development.yml
  ```
  ```sh
    ansible-playbook -i inventory/testing playbooks/deploy_app.yml -e @vars/testing.yml
  ```
  ```sh
    ansible-playbook -i inventory/production playbooks/deploy_app.yml -e @vars/production.yml
  ```


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
     ansible-playbook -i inventory/development playbooks/deploy_app.yml -e "env=development"
  ```

Example for Testing Environment
  ```sh
     ansible-playbook -i inventory/testing playbooks/deploy_app.yml -e "env=testing"
  ```

Example for Production Environment
  ```sh
     ansible-playbook -i inventory/production playbooks/deploy_app.yml -e "env=production"
  ```

**Explanation**
    
    - Role: The deploy_app role handles cloning the application repository from Git and registering the application as a service.
    
    - Playbook: The playbook deploy_app.yml includes the deploy_app role and loads the environment-specific variables.
    
    - Running the Playbook: The playbook is run with the environment variable file passed as an extra variable.
    
    - Environment Variables: The environment-specific variables are defined in separate files (development.yml, testing.yml, production.yml) under the vars directory. These include the Git repository URL and the application path.
    
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

- CPU Usage
- Memory Usage
- Disk space Usage and amount of IOPs
- Network bandwidth outcome / Income, Error rate, Throughput, packet loss and latency.
- Services that we want to keep running

We need to identify potential bottlenecks or failures, you should:
- Establish Baselines: Determine normal performance levels for your metrics.
- Monitor Trends: Look for deviations from the baseline.
- Analyze Correlations: Identify relationships between different metrics (e.g., high CPU usage and increased latency).
- Use Visualization Tools: Tools like Grafana can help visualize data and spot trends.
- Set Thresholds and Alerts: Configure alerts for metrics that exceed predefined thresholds

To do the actual work of monitoring and alerting we have several alternatives, native like MMA with a SCOM (or Azure AMA) or using vmi_exporter and reciever/monitoring like Prometheus and visualization option of Grafana. in this example I would use the last one because is OpenSource

For the purpose of this exercise I'm just installing the vmi_exporter and configuring it to a prometheus server that is not part of the scope of the exercise to create

**File**: `playbooks/install_wmi_exporter.yml`

**Usage:**:
  ```sh
   ansible-playbook -i inventory/development playbooks/install_wmi_exporter.yml -e @vars/development.yml
  ```

  ```sh
   ansible-playbook -i inventory/testing playbooks/install_wmi_exporter.yml -e @vars/testing.yml
  ```

  ```sh
   ansible-playbook -i inventory/production playbooks/install_wmi_exporter.yml -e @vars/production.yml
  ```


# 3. VMWare

We have several options to create a virtual machine at VMWare from directly clicking in the bottom of the GUI (vcenter and ESXi), by a OVF or OVA (export /import), by and script in powercli, by the vcenter API or as part of a vRealise Automation.  
Using templates (Vms can be transform in templates) we can deploy templates direclyt in the GUI, using content library, using powercli and using API.
In order to automate the best aproach is to do it using templates. The VM approach is available but has the tendency to complicate things and we need to keep KISS approach. 

ISSUE using Ansible to deploy VMs. One of the thing that happens is that if we use ansible to deploy from template it use the "clone" feature and not the "deploy from template" feature, so this apply a limit the new VMs to be create only in the same datastore cluster where the template is located.
I think this was not available in the API, RESTAPI or Python or any other (Ruby Java..)

10 Years ago I created this powershell code https://github.com/Vidanez/DeployVMs/tree/master that is still valid to deploy VMs in bulk not cloning but deploying from template, because that feature it was available in the powercli.

### Appropriately Allocated for Optimal Performance
To ensure optimal performance, you need to manage the allocation of CPU, memory, and storage resources effectively. Here are some key practices:

Resource Allocation Settings can be used in order to take care of different VMs not interfere each other seeking for resources:

- Shares: Determine the relative importance of a VM. Higher shares mean higher priority when resources are contested.

- Reservation: Guarantee a minimum amount of resources for a VM in case of maximun consumption by others

- Limit: Set an upper bound on the resources a VM can consume.

Also this this the right other to apply them in a site with limited resources available.

Right-Sizing VMs:

Analyze actual resource usage over a time trying to cover peaks, and adjust CPU, memory, and storage allocations accordingly

Avoid overcommitting resources, even if the vendor chase you for it, which can lead to performance degradation.
    
Monitoring and Adjusting:

Continuously monitor VM performance and adjust resource allocations as needed and if we have licnese apply the usage of DRS within the cluster.

Use tools like VMware vRealize Operations (VROPS) or similar to automate performance management and optimization.

### Deploy a VM by ansible
For the purpose of this example to deploy a VM, I have create a collection of vars at 
 **vars/basicVM.yml**

and the usage would be:

  ```sh
  ansible-playbook -i localhost playbooks/deploy_vm_without_galaxy.yml -e @vars/basicVM.yml
  ```


# 4. Windows Updates Management

If we carry on with the same exercise we are doing, we will carry these operations using Ansible.
At roles/install_windows_updates/tasks/main.yml 
- We use Chocolatey to install the PSWindowsUpdate module, which provides cmdlets for managing Windows updates.
- We use the win_shell module to run PowerShell commands that check for available updates.
- We use the win_shell module to run PowerShell commands that install the updates.
- After installing updates, we capture the update history to determine which updates were successful and which failed
- Note: We have some registers declared in the code, we dont use them after just for debugging or logging purpose

At the playbooks/install_windows_updates.yml
- We do the installation
- We ensure the Windows Update service (wuauserv) is running

### Capturing Information for Successful and Unsuccessful Updates
At roles/run_windows_updates/tasks/main.yml:
- We capture the result of installing updates using register: update_result and process it in the handler Check update status.
- We capture the update history using the Get-WindowsUpdateLog cmdlet and save it using register: update_history.

At roles/run_windows_updates/handlers/main.yml:
- We log the update history to a file using the win_copy module.
- We retry failed updates using the Install-WindowsUpdate cmdlet with the specific KBArticleID.
- We notify about the update status, listing successful and failed updates.

At the playbooks/run_windows_updates.yml
- We run the installation of packages 
- We send a mail in case that the playbook fails

Monitoring, the playbook captures the update history and logs it to a file. If there are any failed updates, the playbook retries them and logs the results.
If there were any failed updates, the playbook would attempt to retry them and send an email notification.

### Usage
To Install the Windows Update Module
```sh
ansible-playbook -i inventories/development.yml playbooks/install_windows_update_module.yml
```
```sh
ansible-playbook -i inventories/test.yml playbooks/install_windows_update_module.yml
```
```sh
ansible-playbook -i inventories/production.yml playbooks/install_windows_update_module.yml
```
To Run Windows Updates
```sh
ansible-playbook -i inventories/development.yml playbooks/run_windows_updates.yml
```
```sh
ansible-playbook -i inventories/test.yml playbooks/run_windows_updates.yml
```
```sh
ansible-playbook -i inventories/production.yml playbooks/run_windows_updates.yml
```


# 5. Antivirus Management 

To do this part I'm taking in consideration Windows defender and ClamAV for linux because the are widely use and are free products.
We can also use Ansible to manage the installation, and schedule the run of the antivirus. Using scheduler at each OS, windows to use schedule and linux to use cronjob.

We have 2 roles fo the installation per kind of OS. Playbook will install based on the OS using variable related to os_family fact. 
Roles:
roles/install_windows_defender
roles/install_clamav
Playbook:
playbooks/install_antivirus.yml

### Usage
```sh
ansible-playbook -i inventories/development.yml playbooks/install_antivirus.yml
```
```sh
ansible-playbook -i inventories/test.yml playbooks/install_antivirus.yml
```
```sh
ansible-playbook -i inventories/production.yml playbooks/install_antivirus.yml
```

The rest of the roles will consider we are running against the appropiate group by OS.

Now let's schedule the run including update of the database per product in the following roles
Roles:
roles/update_windows_defender
roles/update_clamav

And playbooks to be called:
Playbooks:
playbooks/update_clamav.yml
playbooks/update_windows_defender.yml 

Also in this case we have declare variables to do the schedules inside the role to accomplish to all nodes. Whciih to be honest is not the best of the solution in terms of CPU usage but we are doing here a gues exercise.
Vars:
roles/update_windows_defender/vars/main.yml
roles/update_clamav/vars/main.yml

### Usage
Windows:
```sh
ansible-playbook -i inventories/development.yml playbooks/update_windows_defender.yml --limit 'all:!linux'
```
```sh
ansible-playbook -i inventories/test.yml playbooks/update_windows_defender.yml --limit 'all:!linux'
```
```sh
ansible-playbook -i inventories/production.yml playbooks/update_windows_defender.yml --limit 'all:!linux'
```
Linux:
```sh
ansible-playbook -i inventories/development.yml playbooks/update_clamav.yml --limit 'all:!windows'
```
```sh
ansible-playbook -i inventories/test.yml playbooks/update_clamav.yml --limit 'all:!windows'
```
```sh
ansible-playbook -i inventories/production.yml playbooks/update_clamav.yml --limit 'all:!windows'
```















