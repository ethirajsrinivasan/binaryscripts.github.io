---
layout: post
title: Automating Docker Container Management with Terraform and Ansible
subtitle: Use Terraform and Ansible together to provision and manage Docker containers with infrastructure as code
categories: Docker
tags: [Docker, Terraform, Ansible, DevOps, Infrastructure as Code, Automation, Container Management]
excerpt: Discover how to combine Terraform and Ansible to automate Docker container provisioning, deployment, and configuration. Learn best practices for infrastructure as code and container orchestration at scale.
---
Managing Docker containers at scale requires a robust, repeatable approach that minimizes manual intervention and ensures consistency across environments. By combining **Terraform** and **Ansible**, you can achieve complete automation — from provisioning Docker hosts to deploying and configuring containers.

In this post, we’ll show how to use **Terraform for infrastructure provisioning** and **Ansible for Docker configuration management**, creating a powerful and flexible automation pipeline for container-based deployments.

---

#### Why Use Terraform and Ansible Together?

| Tool        | Purpose                         | Role in Docker Automation                  |
|-------------|----------------------------------|---------------------------------------------|
| **Terraform** | Infrastructure Provisioning     | Create Docker hosts (VMs, cloud resources)  |
| **Ansible**   | Configuration Management        | Install Docker, deploy and manage containers |

Using both enables **infrastructure as code (IaC)** with Terraform and **idempotent configuration** using Ansible playbooks — ideal for DevOps workflows and CI/CD pipelines.

---

#### Architecture Overview

```
[Terraform]
↓
Provision VM or cloud instance (EC2, GCP, etc.)
↓
[Ansible]
↓
Install Docker, deploy containers, manage volumes/networks
```

You can also use Terraform to dynamically generate Ansible inventory for seamless integration.

---

#### Step 1: Provision Infrastructure with Terraform

Example Terraform script to launch a Docker host on AWS EC2:

```hcl
provider "aws" {
region = "us-east-1"
}

resource "aws_instance" "docker_host" {
ami           = "ami-0c02fb55956c7d316"
instance_type = "t2.micro"

tags = {
Name = "docker-node"
}

provisioner "remote-exec" {
inline = [
"sudo apt-get update",
"sudo apt-get install -y python3"
]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
}
}
```

Once created, Terraform can output the instance IP to be used in your Ansible inventory.

---

#### Step 2: Create Ansible Playbook for Docker Setup

Sample playbook: `docker.yml`

```yaml
---
- hosts: docker_hosts
  become: true
  tasks:
  - name: Install Docker
    apt:
    name: docker.io
    state: present
    update_cache: true

  - name: Start Docker
    service:
    name: docker
    state: started
    enabled: true

  - name: Run NGINX container
    docker_container:
    name: web
    image: nginx:latest
    state: started
    ports:
    - "80:80"
    ```

---

#### Step 3: Dynamically Generate Ansible Inventory from Terraform

You can output instance IPs from Terraform:

```hcl
output "docker_host_ip" {
value = aws_instance.docker_host.public_ip
}
```

Save the IPs to a dynamic inventory file for Ansible:

```ini
[docker_hosts]
<terraform-output-ip> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

#### Step 4: Run the Automation Pipeline

1. Apply Terraform:

```bash
terraform init
terraform apply
```

2. Get the output IP:

```bash
terraform output docker_host_ip
```

3. Add IP to Ansible inventory and run playbook:

```bash
ansible-playbook -i inventory.ini docker.yml
```

This deploys Docker and runs your containers automatically.

---

#### Best Practices

- Store Terraform and Ansible in **separate directories** for clean separation
- Use **remote state** (Terraform Cloud, S3) for team collaboration
- Encrypt secrets using **Ansible Vault**
- Parameterize playbooks and Terraform with **variables**
- Use **Ansible roles** for reusable Docker setups
- Automate via **GitHub Actions, Jenkins, or GitLab CI**

---

#### Real-World Use Cases

- Auto-scaling container hosts across cloud environments
- Deploying microservices without Kubernetes
- Setting up test environments in CI pipelines
- Managing dev/QA Docker infrastructure as code

---

#### Conclusion

Combining **Terraform** and **Ansible** for Docker container automation offers the best of both worlds — scalable provisioning with infrastructure as code and powerful, declarative configuration management. This workflow enables teams to deploy reliable, repeatable, and production-grade Docker environments across cloud and on-premises infrastructure.

Use this approach to streamline your **DevOps pipelines**, reduce manual errors, and embrace modern infrastructure automation practices.
