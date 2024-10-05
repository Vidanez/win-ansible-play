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
│   ├── development.ini
│   ├── testing.ini
│   └── production.ini
├── playbooks/
│   ├── install_iis.yml
│   ├── manage_w32time.yml
│   ├── deploy_app.yml
│   └── service_management.yml
├── roles/
│   ├── iis/
│   │   └── tasks/
│   │       └── main.yml
│   ├── w32time/
│   │   └── tasks/
│   │       └── main.yml
│   └── app/
│       ├── tasks/
│       │   └── main.yml
│       └── files/
│           └── app.exe
└── vars/
    ├── development.yml
    ├── testing.yml
    └── production.yml
```


# 1. Ansible Windows Services Management

## Overview

This project demonstrates how to use Ansible to install, configure, and start a Windows service across multiple servers.
In this example services with parameters IIS (Internet Information Services) and without parameters Windows Time Service (w32time).
It also covers deploying a sample application and registering it as a service with environment-specific configurations.

## Prerequisites

1. **Ansible**: Install Ansible on your control machine from where we will run the playbooks.
   ```sh
   sudo apt-get update
   sudo apt-get install ansible
   ```
2. **pywinrm**: Install pywinrm on the control machine. We need it in order to manage the WinRM.
   ```sh
   pip install pywinrm
   ```

2. **WinRM Configuration on Windows Servers**: Run the following PowerShell script on each Windows server:
   https://docs.ansible.com/ansible/latest/os_guide/windows_winrm.html
   ```powershell
   winrm quickconfig -q
   winrm set winrm/config/service '@{AllowUnencrypted="true"}'
   winrm set winrm/config/service/auth '@{Basic="true"}'
   winrm set winrm/config/listener?Address=*+Transport=HTTP
   winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
   ```


## Playbooks

### Install IIS

**File**: `playbooks/install_iis.yml`

```yaml
- name: Install IIS
  hosts: windows
  tasks:
    - name: Install IIS
      win_feature:
        name: Web-Server
        state: present
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_feature_module.html

    - name: Configure IIS
      win_shell: |
        Import-Module WebAdministration
        Set-ItemProperty "IIS:\Sites\Default Web Site" -Name physicalPath -Value "C:\inetpub\wwwroot"
      args:
        executable: powershell
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_shell_module.html

    - name: Start IIS Service
      win_service:
        name: W3SVC
        start_mode: auto
        state: started
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
```

### Manage Windows Time Service

**File**: `playbooks/manage_w32time.yml`

```yaml
- name: Manage Windows Time Service
  hosts: windows
  tasks:
    - name: Ensure Windows Time Service is running
      win_service:
        name: w32time
        start_mode: auto
        state: started
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
```

### Routine Service Management Tasks check status, start, stop and restart

**File**: `playbooks/service_management.yml`

```yaml
- name: Check IIS Service Status
  hosts: windows
  tasks:
    - name: Check IIS Service
      win_service:
        name: W3SVC
        state: started
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html

- name: Start IIS Service
  hosts: windows
  tasks:
    - name: Start IIS Service
      win_service:
        name: W3SVC
        state: started
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html

- name: Stop IIS Service
  hosts: windows
  tasks:
    - name: Stop IIS Service
      win_service:
        name: W3SVC
        state: stopped
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html

- name: Restart IIS Service
  hosts: windows
  tasks:
    - name: Restart IIS Service
      win_service:
        name: W3SVC
        state: restarted
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
```

### Deploy Sample Application

**File**: `playbooks/deploy_app.yml`

```yaml
- name: Deploy Sample Application
  hosts: windows
  vars_files:
    - vars/{{ ansible_environment }}.yml
  tasks:
    - name: Copy application files
      win_copy:
        src: /path/to/application
        dest: "{{ app_destination }}"
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_copy_module.html

    - name: Register Application as a Service
      win_service:
        name: MyAppService
        path: "{{ app_destination }}\app.exe"
        start_mode: auto
        state: started
      # Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
```

## Inventory Files per each environment we want to use we need to declare the host that live at that environment.
There is options not cover here to do update these files dinamically, getting this information from VMWare or DNS

### Development

**File**: `inventory/development.ini`

```ini
[development]
dev_server1 ansible_host=dev1.example.com
dev_server2 ansible_host=dev2.example.com
```

### Testing

**File**: `inventory/testing.ini`

```ini
[testing]
test_server1 ansible_host=test1.example.com
test_server2 ansible_host=test2.example.com
```

### Production

**File**: `inventory/production.ini`

```ini
[production]
prod_server1 ansible_host=prod1.example.com
prod_server2 ansible_host=prod2.example.com
```

## Environment Variables

### Development

**File**: `vars/development.yml`

```yaml
app_destination: C:\dev\app
```

### Testing

**File**: `vars/testing.yml`

```yaml
app_destination: C:\test\app
```

### Production

**File**: `vars/production.yml`

```yaml
app_destination: C:\prod\app
```

## How to Use to deploy the same app in different environment taking in consideration posiible differences by variables

1. **Clone the Repository**:
   ```sh
   git clone https://github.com/Vidanez/win-ansible-play.git
   cd win-ansible-play
   ```

2. **Set Up Inventory Files**:
   Update the `inventory/development.ini`, `inventory/testing.ini`, and `inventory/production.ini` files with your server details.

3. **Run Playbooks**:
   - **Install IIS**:
     ```sh
     ansible-playbook -i inventory/development.ini playbooks/install_iis.yml
     ```

   - **Manage Windows Time Service**:
     ```sh
     ansible-playbook -i inventory/development.ini playbooks/manage_w32time.yml
     ```

   - **Routine Service Management**:
     ```sh
     ansible-playbook -i inventory/development.ini playbooks/service_management.yml
     ```

   - **Deploy Sample Application**:
     ```sh
     ansible-playbook -i inventory/development.ini playbooks/deploy_app.yml -e "ansible_environment=development"
     ```

## Modules would you typically use to manage windows

- win_service https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_service_module.html
- win_feature https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_feature_module.html
- win_shell https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_shell_module.html
- win_copy https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_copy_module.html
- win_template https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_template_module.html
```
# 2. Windows Server Monitoring
