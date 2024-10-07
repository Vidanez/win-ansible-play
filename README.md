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
    │   ├── install_and_start_services.yml
    │   ├── install_iis.yml
    │   ├── install_packages.yml
    │   ├── install_w32time.yml
    │   └── manage_service.yml
    ├── roles
    │   ├── deploy_app
    │   │   ├── tasks
    │   │   │   └── main.yml
    │   │   └── vars
    │   │       └── main.yml
    │   ├── iis
    │   │   └── tasks
    │   │       └── main.yml
    │   ├── install_packages
    │   │   ├── tasks
    │   │   │   └── main.yml
    │   │   └── vars
    │   │       └── main.yml
    │   ├── manage_services
    │   │   └── tasks
    │   │       └── main.yml
    │   └── w32time
    │       └── tasks
    │           └── main.yml
    └── vars
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
        Shares: Determine the relative importance of a VM. Higher shares mean higher priority when resources are contested.
        Reservation: Guarantee a minimum amount of resources for a VM in case of maximun consumption by others
        Limit: Set an upper bound on the resources a VM can consume.
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

# 5. Antivirus Management 
