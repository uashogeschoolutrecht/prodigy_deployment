<div align="center">

  <h3 align="center">Prodigy deployment on a VM</h3>

  <p align="center">
    Ansible playbooks to setup a cloud enviroment for prodigy
  </p>
</div>

##  Table of Contents

- [ About ](#-about-)
  - [Key features](#Key-features)
  - [Playbooks overview](#Playbooks-Overview)
- [ Getting Started ](#-getting-started-)
  - [Prerequisites](#prerequisites)
  - [Installation](#installing)
- [ Usage ](#-usage-)
  - [Accessing the Web Application & Managing Users](#Accessing-the-Web-Application-&-Managing-Users)
  - [Interacting with the Docker Container](#Interacting-with-the-Docker-Container)
  - [Persistent Data Storage](#Persistent-Data-Storage)
  - [Backups and file tranfers](#Backups-and-file-tranfers)

##  About <a name = "about"></a>

Prodigy is an interactive and modern annotation tool designed to help professionals and researchers efficiently collect data. It provides an intuitive interface that can be customized for various annotation tasks, making the data collection process streamlined and more accessible.

This project simplifies the deployment of Prodigy in a cloud environment. By leveraging Ansible playbooks, it ensures a standardized and reproducible setup process.

### Key Features

- Customizability: Tailor the setup to your needs by modifying the [ requirements.txt ](requirements.txt) or the [ run.sh ](run.sh) script, executed during the Docker container's startup.
- Environment Configuration: Easily regulate Prodigy's operations by setting environmental variables within the [ .env ](.env) file.
- Security: Ansible secrets are utilized to ensure sensitive data, such as the Prodigy license key, is kept secure. Furthermore, user access is controlled, ensuring only authorized individuals can use the tool.
- Backup & Storage: With integrated backup solutions, your data is persisted. Scheduled backups can be automatically stored in your chosen location, ensuring your annotations are protected.

### Playbooks Overview

This project contains three ansible playbooks.

1. [ setup_nginx ](ansible/setup_nginx.yml) sets up Nginx for Prodigy with basic authentication, and redirects authenticated users accessing the base URL to a session-specific path based on their username.
2. [ setup_docker ](ansible/setup_docker.yml) installs Prodigy in a Docker container on the host, utilizing Docker Compose for orchestration and exposing the service on port 8080.
3. [ setup_backup ](ansible/setup_backup.yml) sets up rclone on the host to automate daily compressed backups of a Prodigy database from a Docker container and transfers them to remote storage.

<p align="right"><a href="#readme-top">back to top</a></p>


## Getting Started <a name = "functions"></a>

### Prerequisites

This guide is designed to help you configure and deploy Prodigy. Before diving into the installation, it's essential to understand and prepare the necessary prerequisites for both your control machine and the target host.

__Control Machine__:

  - __Operating System__: Preferably Linux or MacOS. You can use Windows if leveraging the Windows Subsystem for Linux (WSL).
  - __Python__: Required to run Ansible.
        For Ansible 2.7 and earlier: Python 2 (version 2.6 or later).
        For Ansible 2.8 and later: Python 3 (version 3.5 or later) is recommended.
  - __Ansible__: Ensure you've installed Ansible.
  - __SSH__: Necessary for communicating with the remote prodigy host.

__Target Host:__

  - A Debian-based Linux distribution, as we're using the apt module.
  - SSH access with key-based authentication for the user running the playbook.
  - Sudo privileges for the user running the playbook.

### Installation

__1. Navigate to the Ansible Directory__

Navigate to the directory containing the Ansible configuration:
``` bash
cd ansible
```

__2. Configure the Inventory__

Set up your inventory file to specify the target host and user. Replace your.target.host.ip and username with the appropriate values:
``` ini
[prodigy]
your.target.host.ip ansible_user=username
```

__3. Create the Secrets File__

Initiate a new Ansible Vault file to securely store your sensitive data:
``` bash
ansible-vault create secret.yml
```

__4. Define Required Secrets__

In the secret.yml file, set the necessary variables for your setup. Ensure you replace placeholders with your actual values:
``` yml
http_username: username_for_basic_authentication
http_password: password_for basic_authentcation
prodigy_key: prodigy_license_key
rclone_url: webdav_link 
rclone_user: webdav_username
rclone_pass: webdav_password
```
For more information about the webdav variables: 
https://research-it.hu.nl/docs/Research%20Cloud/Hoe%20gebruik%20ik%20Research%20Cloud/HU_Research_Drive_koppelen_map/  

__5. Execute the Playbooks__

Run the Ansible playbooks. You'll be prompted to enter the Ansible Vault password:
``` bash
ansible-playbook -i inventory.ini prodigy_playbook.yml --ask-vault-pass
```

<p align="right"><a href="#readme-top">back to top</a></p>

## Usage <a name="time_line"></a>

### Accessing the Web Application & Managing Users

Once you've SSHed into the VM, you can manage users who are authorized to access the Prodigy web application and perform annotations.

__Adding Additional Users:__
To create a new user, run the following command, replacing username with the desired username. You'll be prompted to set a password for this user.

  ``` bash
  sudo htpasswd /etc/nginx/.htpasswd username
  ```

### Interacting with the Docker Container

Prodigy runs within a Docker container. To interact with it or manage its contents, follow these steps:

__1. Listing All Containers:__
To see a list of all containers and their statuses:
``` bash
sudo docker ps -a
```

__2. Accessing a Running Container:__
If the desired container is running, you can directly access its shell using the following command. Replace `container_id_or_name` with the actual container's ID or name.
```
sudo docker exec -it container_id_or_name /bin/bash
```

### Persistent Data Storage

When using this Docker setup, it's crucial to understand the data persistence strategy:

__1. Application Data:__
- The directory /app/data within the Docker container is mounted to /app/data  on the host machine.
- Files saved to /app/data inside the container are directly stored in /app/data on your host system.

__2. Database:__
- The app/.prodigy directory in the container, which houses the database, is mounted to the prodigy_db volume on the host.
- This ensures that your Prodigy database remains persistent, safeguarding it from data loss during container lifecycle events like restarts or removals.

Both these mechanisms ensure that vital data—whether related to the application or the database—remains intact and accessible even if the container is stopped, restarted, or deleted. Always refer to the mentioned directories and volumes when you aim to save or access persistent data.

### Backups and file tranfers

For data you want to send to your research drive:

__1. Database Backups:__
As rclone is configured with WebDAV credentials specific to your research directory, creating backups or copying files is straightforward. To create a backup of the Prodigy database:

``` bash
sudo /usr/local/bin/backup.sh
```
__2. Transferring Files to Research Vault:__
To securely transfer any file or directory to your research vault, use the following rclone command. Make sure to replace the placeholders with the actual paths of the source and destination directories.

```
sudo rclone copy "/path/to/localdir" "prodigy:/path/to/hostdir"
```

<p align="right"><a href="#readme-top">back to top</a></p>