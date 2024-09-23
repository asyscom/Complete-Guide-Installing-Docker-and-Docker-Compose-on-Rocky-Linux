---

# Complete Guide: Installing Docker and Docker Compose on Rocky Linux

Docker is a platform that automates the deployment of applications in containers, while Docker Compose allows you to define and manage multi-container applications. In this guide, we will see how to install Docker and Docker Compose on Rocky Linux, both manually and through an Ansible playbook.

## Requirements
- Root access or a user with `sudo` privileges
- Internet connection to download packages
- Server running Rocky Linux (version 8 or 9)

---

## Section 1: Manual Installation of Docker and Docker Compose on Rocky Linux

### 1.1 Update the system
Before starting the installation, update the system to ensure all packages are up to date:

```bash
sudo dnf update -y
```

### 1.2 Add the Docker repository

To install Docker, add the official Docker repository:

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 1.3 Install Docker

Now install Docker:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### 1.4 Start and enable Docker

Start Docker and ensure it starts automatically at boot:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 1.5 Verify Docker installation

To check if Docker was installed correctly:

```bash
sudo docker --version
```

Additionally, you can run a test container:

```bash
sudo docker run hello-world
```

### 1.6 Install Docker Compose

Docker Compose is included in the `docker-compose-plugin`, but if you want to install a specific version manually, follow these steps:

1. Download the latest version of Docker Compose:

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. Apply executable permissions:

   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. Verify Docker Compose installation:

   ```bash
   docker-compose --version
   ```

Now Docker and Docker Compose are installed and ready to use!

---

## Section 2: Installing Docker and Docker Compose with Ansible

If you prefer automating the process, you can use an Ansible playbook. This approach is particularly useful if you are managing multiple servers or environments.

### 2.1 Installing Ansible

To install Ansible on your control machine, run the following commands:

```bash
sudo dnf install epel-release -y
sudo dnf install ansible -y
```

### 2.2 Writing the Ansible playbook

Create a file called `install_docker.yml` with the following content:

```yaml
---
- name: Install Docker and Docker Compose on remote machine
  hosts: all
  become: yes

  tasks:
    - name: Ensure python3-dnf is installed (for RHEL/CentOS)
      package:
        name: python3-dnf
        state: present

    - name: Add Docker repository
      yum_repository:
        name: docker
        description: Docker Repository
        baseurl: https://download.docker.com/linux/centos/7/x86_64/stable/
        gpgcheck: yes
        enabled: yes
        gpgkey: https://download.docker.com/linux/centos/gpg

    - name: Install Docker
      yum:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: latest

    - name: Start and enable Docker
      systemd:
        name: docker
        enabled: yes
        state: started

    - name: Check Docker installation
      command: docker version

    - name: Show Docker version
      command: docker --version

    - name: Get Docker Compose binary name
      command: "uname -s"
      register: uname_s

    - name: Get architecture
      command: "uname -m"
      register: uname_m

    - name: Install Docker Compose
      get_url:
        url: "https://github.com/docker/compose/releases/latest/download/docker-compose-{{ uname_s.stdout }}-{{ uname_m.stdout }}"
        dest: /usr/local/bin/docker-compose
        mode: '0755'

    - name: Check Docker Compose installation
      command: docker compose version


### 2.3 Running the Ansible playbook

Save the file and ensure your Ansible inventory file (e.g., `hosts.ini`) contains the hosts where you want to install Docker.

Run the playbook:

```bash
ansible-playbook -i hosts.ini install_docker.yml
```

This playbook will install Docker and Docker Compose, start Docker, and verify their versions.

---

## Section 3: Final Considerations

In this guide, we explored two methods to install Docker and Docker Compose on Rocky Linux: manually and using Ansible. Manual installation is ideal for simpler environments or testing, while Ansible automates the process, making it easier to manage multiple servers. Depending on your needs, you can choose the method that suits you best.

### Maintenance Tips
- Remember to regularly update Docker to benefit from new features and security patches.
- You can use Ansible to automate the update of Docker and Docker Compose across multiple servers.

---
## Support the Project

If you found this guide helpful and would like to support the project, consider making a donation. Your contributions help maintain and improve this resource.

### Donate via Bitcoin
You can send Bitcoin directly to the following address:

**`bc1qy0l39zl7spspzhsuv96c8axnvksypfh8ehvx3e`**

### Donate via Lightning Network
For faster and lower-fee donations, you can use the Lightning Network:

**asyscom@sats.mobi**

Thank you for your support!

---

With this detailed guide, you will be able to efficiently install and configure Docker and Docker Compose on Rocky Linux, whether manually or by automating the process with Ansible.
