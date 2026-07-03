# Ansible Interview Questions (Production Grade)

---

# 🔴1. What are the prerequisites for using Ansible?

Ansible is an **agentless automation tool**, so unlike Chef or Puppet, it does not require any agent to be installed on the target servers.

In production, the prerequisites are:

- A Linux machine (or WSL/macOS) to act as the **Ansible Control Node**.
- **Ansible** installed on the control node.
- **Python** installed on the managed Linux servers because Ansible modules execute using Python.
- **SSH connectivity** from the control node to the managed servers.
- **SSH key-based authentication** for secure passwordless access.
- An **inventory file** that contains the list of target servers.

For Windows servers, Ansible uses **WinRM** instead of SSH.

In our environment, GitHub Actions triggers Ansible playbooks after the infrastructure is provisioned using Terraform. The GitHub runner connects to Linux servers through SSH using private keys stored securely in GitHub Secrets. During server provisioning, we ensure Python is installed automatically so that Ansible modules can execute without any issues.

### Production Best Practices

- Use SSH key-based authentication instead of passwords.
- Store SSH keys in Vault or CI/CD Secrets.
- Ensure Python is installed during server bootstrap.
- Organize inventory using groups like Dev, QA, and Production.
- Never hardcode credentials inside playbooks.

---

# 🔴2. What language is Ansible written in?

Ansible is primarily written in **Python**, and almost all built-in modules are implemented using Python.

Although Ansible itself is written in Python, DevOps engineers generally don't write Python code while using Ansible. Instead, we write automation in **YAML playbooks**, and Ansible internally executes Python modules on the target servers.

Because of this architecture, Python must be available on Linux managed nodes. When a playbook runs, Ansible copies a temporary Python module to the remote server, executes it, returns the result, and removes the temporary files.

In production, this makes infrastructure automation very simple because teams only define the desired configuration in YAML while Ansible handles the execution logic.

### Real-Time Example

In our CI/CD pipeline, developers trigger deployments using GitHub Actions. The workflow executes an Ansible playbook written in YAML, but internally Ansible uses Python modules to install packages, configure services, update application files, and restart services automatically.

### Easy Way to Remember

- **Python** → Ansible Engine
- **YAML** → What DevOps Engineers Write

---

# 🔴3. Is Ansible idempotent? Why is that important?

Yes.

One of Ansible's biggest advantages is that it is **idempotent**, which means running the same playbook multiple times always produces the same final state. If the desired configuration already exists, Ansible skips that task instead of executing it again.

For example, if Nginx is already installed and the service is already running, Ansible detects that the desired state matches the current state and reports the task as **OK** instead of reinstalling the package or restarting the service unnecessarily.

In production, this is extremely important because deployment pipelines may execute the same playbook multiple times due to retries, rollbacks, or repeated deployments. Idempotency prevents duplicate configurations, minimizes service interruptions, reduces unnecessary changes, and ensures all servers remain in a consistent state.

### Real-Time Example

Suppose our deployment pipeline configures 50 application servers.

If the playbook runs again:

- Already installed packages are skipped.
- Existing users are not recreated.
- Existing directories remain unchanged.
- Running services are not restarted unless configuration changes.

This makes deployments predictable and safe.

### Easy Way to Remember

**Run once or run 100 times → Same desired state.**

---

# 🔴4. How does Ansible use SSH to connect to remote servers?

Ansible communicates with Linux servers using **SSH**, making it an agentless automation tool.

When a playbook starts, Ansible reads the inventory file, establishes an SSH connection to each target server, copies a temporary module to the remote machine, executes it using Python, collects the output, and removes the temporary files after execution.

Since everything happens over SSH, there is no permanent agent running on the managed server.

In production, we always use **SSH key-based authentication** instead of passwords. The private keys are stored securely in GitHub Secrets, Azure Key Vault, or HashiCorp Vault, and password authentication is disabled on production servers.

For large environments, Ansible also uses **SSH multiplexing**, which reuses existing SSH connections and significantly improves execution speed across hundreds of servers.

### Real-Time Example

In our production environment, after Terraform provisions EC2 instances, GitHub Actions automatically executes an Ansible playbook. The GitHub runner connects to each EC2 instance using SSH keys, installs application dependencies, configures Nginx, deploys the application, and restarts services without requiring any manual login.

### Easy Way to Remember

```text
Ansible Control Node
        │
        ▼
SSH Connection
        │
        ▼
Temporary Python Module
        │
        ▼
Execute Task
        │
        ▼
Return Output
        │
        ▼
Remove Temporary Files
```

---

# 🔴5. What are the types of modules in Ansible (Core vs Extra)?

Ansible modules are the building blocks that perform individual tasks such as installing packages, managing files, configuring services, or provisioning cloud resources.

Broadly, modules are categorized into **Core Modules** and **Extra (Community) Modules**.

### Core Modules

Core modules are officially maintained by the Ansible project.

They are:

- Stable
- Fully tested
- Included with Ansible
- Suitable for production workloads

Examples include:

- package
- yum
- apt
- service
- copy
- file
- user
- cron

We use these modules daily for Linux configuration management.

---

### Extra (Community) Modules

Extra modules are maintained by the Ansible community or cloud providers.

These modules provide integrations for technologies such as:

- AWS
- Azure
- VMware
- Kubernetes
- Docker
- GitHub
- Databases

Examples include:

- amazon.aws.ec2_instance
- azure.azcollection.azure_rm_virtualmachine
- kubernetes.core.k8s
- community.docker.docker_container

These modules are installed through **Ansible Galaxy Collections**.

---

### Real-Time Example

In one of our projects:

- Terraform created the AWS infrastructure.
- Ansible Core Modules configured Linux servers by installing Nginx, Java, and application packages.
- Community AWS Modules managed EC2 tags and security groups.
- Kubernetes Modules deployed application manifests into the EKS cluster.

Using the right module reduced custom scripting and improved automation reliability.

### Best Practice

- Prefer **Core Modules** whenever possible.
- Use **Community Collections** only from trusted publishers.
- Keep collections version-controlled.
- Avoid using shell commands if an Ansible module already exists because modules are idempotent and provide better error handling.

### Easy Way to Remember

| Core Modules | Extra (Community) Modules |
|--------------|---------------------------|
| Built into Ansible | Installed separately |
| Officially maintained | Community/Cloud provider maintained |
| Stable and widely used | Used for cloud, Kubernetes, VMware, Docker, etc. |
| No additional installation | Installed using Ansible Galaxy Collections |
# Ansible Interview Questions (Production Grade)

---

# 🔴6. What is an Ansible Playbook? Write one to install Nginx.

An **Ansible Playbook** is a YAML file that defines a series of automation tasks to bring servers to the desired state. Instead of executing commands manually, we describe the desired configuration, and Ansible ensures every target server matches that state.

In production, playbooks are stored in Git repositories and executed through CI/CD pipelines such as GitHub Actions or Azure DevOps. We generally separate playbooks for application deployment, server configuration, security hardening, and patch management to keep automation modular and maintainable.

### Real-Time Example

In one of our projects, after Terraform provisioned EC2 instances, GitHub Actions automatically triggered an Ansible playbook to:

- Install Nginx
- Copy the application configuration
- Deploy the application
- Start and enable the Nginx service

This eliminated manual server configuration and ensured every server was configured consistently.

### Sample Playbook

```yaml
---
- name: Install and Configure Nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Install Nginx
      package:
        name: nginx
        state: present

    - name: Enable and Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

### Best Practices

- Keep playbooks small and reusable.
- Avoid hardcoding values.
- Store variables separately.
- Prefer modules over shell commands.
- Keep playbooks idempotent.

---

# 🔴7. How do you perform a dry run for an Ansible playbook?

Before executing any playbook in production, we usually perform a **Dry Run** using **Check Mode**.

Check Mode allows Ansible to simulate the execution without actually making any changes to the target servers. It shows what changes would be performed if the playbook were executed normally.

Command:

```bash
ansible-playbook site.yml --check
```

To see detailed output:

```bash
ansible-playbook site.yml --check --diff
```

The `--diff` option displays configuration differences before and after the proposed changes.

### Real-Time Example

Before deploying Nginx configuration changes to our production web servers, our GitHub Actions pipeline first executes:

```bash
ansible-playbook nginx.yml --check --diff
```

The DevOps team reviews the proposed changes. Once approved, the pipeline executes the actual deployment.

This reduces deployment risk and helps catch configuration mistakes before they affect production.

### Best Practices

- Run `--check` before production deployments.
- Combine with `--diff` for configuration reviews.
- Integrate dry runs into CI/CD approval workflows.

---

# 🔴8. What is an Ansible Role? Write a role to install and start Nginx.

An **Ansible Role** is a standardized way of organizing playbooks, variables, templates, handlers, and files into a reusable directory structure.

Instead of writing one large playbook, we divide automation into logical components called roles.

In production, almost every automation project is role-based because roles are easier to maintain, reuse, test, and share across multiple projects.

### Real-Time Example

In one of our environments, we had separate roles for:

- Common Linux Configuration
- Nginx
- Docker
- Application Deployment
- Monitoring Agent
- Security Hardening

Whenever a new server was provisioned, the required roles were simply assigned without rewriting automation.

### Sample Role Structure

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    ├── files/
    ├── vars/
    ├── defaults/
    ├── meta/
    └── README.md
```

### tasks/main.yml

```yaml
---
- name: Install Nginx
  package:
    name: nginx
    state: present

- name: Enable and Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

### Playbook Using the Role

```yaml
---
- hosts: webservers
  become: yes

  roles:
    - nginx
```

### Best Practices

- Keep each role focused on one responsibility.
- Store reusable variables in defaults/.
- Use handlers for service restarts.
- Version roles in Git.

---

# 🔴9. How do you structure an Ansible Role?

A well-structured Ansible Role follows a standard directory layout, making it reusable and easy to maintain.

Typical production structure:

```
roles/
└── nginx/
    ├── defaults/
    │   └── main.yml
    ├── vars/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    ├── meta/
    │   └── main.yml
    ├── tests/
    └── README.md
```

### Directory Purpose

| Directory | Purpose |
|------------|----------|
| tasks | Main automation tasks |
| handlers | Restart services when notified |
| templates | Jinja2 templates |
| files | Static files |
| vars | High-priority variables |
| defaults | Default variables |
| meta | Role dependencies |
| tests | Role testing |

### Real-Time Example

For our application deployment role:

- Tasks installed packages.
- Templates generated application configuration.
- Files stored SSL certificates.
- Handlers restarted services after configuration updates.
- Defaults stored environment-independent values.

This modular design made the same role reusable for Development, QA, UAT, and Production.

---

# 🔴10. What is Ansible Galaxy, and how does it help manage roles and dependencies?

**Ansible Galaxy** is the official repository for sharing and downloading Ansible Roles and Collections.

Instead of writing automation from scratch, we can install pre-built, tested roles created by the Ansible community or cloud providers.

In production, Galaxy significantly reduces development time because common automation such as installing Docker, Kubernetes, Nginx, Prometheus, or cloud integrations is already available.

### Install a Role

```bash
ansible-galaxy role install geerlingguy.nginx
```

### Install a Collection

```bash
ansible-galaxy collection install amazon.aws
```

### Requirements File

```yaml
roles:
  - name: geerlingguy.nginx

collections:
  - amazon.aws
  - kubernetes.core
```

Install all dependencies:

```bash
ansible-galaxy install -r requirements.yml
```

### Real-Time Example

In one of our AWS projects, we used:

- `amazon.aws` collection for AWS resource management.
- `kubernetes.core` collection for EKS deployments.
- Community Docker collection for Docker automation.

Instead of developing custom modules, we reused trusted collections maintained by cloud providers.

### Best Practices

- Pin collection versions.
- Use only trusted publishers.
- Store `requirements.yml` in Git.
- Install dependencies automatically during CI/CD.

### Easy Way to Remember

| Roles | Collections |
|--------|-------------|
| Reusable automation | Group of modules, plugins, and roles |
| Task-focused | Technology-focused |
| Example: Nginx Role | Example: amazon.aws |
# Ansible Interview Questions (Production Grade)

---

# 🔴11. What is an Ansible ad-hoc command?

An **Ansible ad-hoc command** is a one-line command used to perform a quick task on one or more managed servers without writing a playbook.

In production, we use ad-hoc commands for operational tasks such as checking connectivity, restarting a service, creating a user, installing a package, collecting system information, or verifying disk usage. They are ideal for quick administration tasks but are **not recommended for repeatable deployments**, where playbooks should always be used.

### Common Examples

**Check connectivity**

```bash
ansible all -m ping
```

**Install Nginx**

```bash
ansible webservers -b -m package -a "name=nginx state=present"
```

**Restart Nginx**

```bash
ansible webservers -b -m service -a "name=nginx state=restarted"
```

**Check disk usage**

```bash
ansible all -m shell -a "df -h"
```

### Real-Time Example

Suppose our monitoring alerts indicate that the Nginx service has stopped on multiple web servers.

Instead of logging into each server manually, we execute:

```bash
ansible webservers -b -m service -a "name=nginx state=restarted"
```

Within seconds, the service is restarted across all servers.

### Best Practices

- Use ad-hoc commands only for quick operational tasks.
- Use playbooks for repeatable automation.
- Prefer Ansible modules over shell commands.
- Avoid making production configuration changes through ad-hoc commands.

---

# 🔴12. What is an Ansible loop, and when would you use it?

An **Ansible loop** allows the same task to be executed multiple times using different values.

Instead of writing repetitive tasks, we write a single task and iterate through a list of items. This keeps playbooks clean, readable, and easier to maintain.

In production, loops are commonly used for:

- Installing multiple packages.
- Creating multiple users.
- Creating directories.
- Copying multiple files.
- Managing firewall rules.
- Creating multiple Kubernetes namespaces.

### Example

```yaml
- name: Install Packages
  package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - unzip
```

### Real-Time Example

When provisioning new Linux application servers, we install all required software using a single loop instead of writing separate tasks.

Packages include:

- nginx
- git
- unzip
- java
- python3

If another package is needed later, we simply add it to the list without modifying the task itself.

### Best Practices

- Use loops instead of duplicate tasks.
- Store loop values in variables when possible.
- Keep loops simple and readable.

### Easy Way to Remember

**One Task → Multiple Items**

---

# 🔴13. What is Dynamic Inventory? Give a practical use case.

A **Dynamic Inventory** automatically discovers servers from cloud providers or external systems instead of maintaining a static inventory file.

Unlike a static inventory where servers are manually added or removed, dynamic inventory queries cloud platforms such as AWS, Azure, or GCP in real time and builds the inventory automatically.

This is especially useful in environments where servers are frequently created or terminated.

### Real-Time Example

In one of our AWS environments, Auto Scaling Groups dynamically launched and terminated EC2 instances based on application traffic.

Instead of manually updating the inventory every time a server was added or removed, we used the **amazon.aws.aws_ec2** inventory plugin.

When the playbook executed, Ansible automatically discovered all running EC2 instances based on tags such as:

- Environment = Production
- Application = Web

This ensured new servers were configured automatically without any manual intervention.

### Benefits

- No manual inventory updates.
- Works well with Auto Scaling.
- Automatically discovers new servers.
- Reduces configuration errors.
- Ideal for cloud environments.

### Easy Way to Remember

| Static Inventory | Dynamic Inventory |
|-----------------|-------------------|
| Manual | Automatic |
| Best for small environments | Best for cloud environments |
| Servers manually updated | Servers discovered automatically |

---

# 🔴14. How do you install Nginx on a group of servers but skip a specific server?

In production, there are multiple ways to exclude a server from a playbook execution.

The simplest approach is using the **--limit** option while excluding a specific host.

Example:

```bash
ansible-playbook nginx.yml --limit "webservers:!web03"
```

Here:

- `webservers` is the inventory group.
- `!web03` excludes the server named **web03**.

Another approach is to use a conditional statement inside the playbook.

Example:

```yaml
- name: Install Nginx
  package:
    name: nginx
    state: present
  when: inventory_hostname != "web03"
```

### Real-Time Example

During one of our production deployments, a particular web server was undergoing maintenance.

Instead of modifying the inventory file, we excluded it during deployment using:

```bash
--limit "webservers:!web03"
```

This allowed the deployment to continue on all other production servers without affecting the maintenance activity.

### Best Practices

- Prefer `--limit` for temporary exclusions.
- Avoid modifying inventory for one-time maintenance.
- Use conditions only when exclusion is permanent or logic-based.

---

# 🔴15. How do you run a playbook for different machines with different users and ports?

In production, different servers may require different SSH users or ports depending on their operating system, security policies, or cloud environment.

Ansible supports this by defining connection variables in the inventory.

Example Inventory:

```ini
[webservers]

web01 ansible_host=10.0.1.10 ansible_user=ec2-user ansible_port=22

web02 ansible_host=10.0.1.11 ansible_user=ubuntu ansible_port=2222

web03 ansible_host=10.0.1.12 ansible_user=azureuser ansible_port=22
```

When the playbook runs:

- web01 connects using **ec2-user**
- web02 connects using **ubuntu** on port **2222**
- web03 connects using **azureuser**

The playbook itself remains unchanged.

### Real-Time Example

In one of our hybrid cloud environments:

- AWS EC2 instances used **ec2-user**.
- Ubuntu VMs used **ubuntu**.
- Azure Virtual Machines used **azureuser**.

Some hardened Linux servers exposed SSH only on port **2222**.

Instead of maintaining separate playbooks, we configured connection details in the inventory, allowing a single playbook to deploy applications across all environments.

### Best Practices

- Store connection variables in inventory or group_vars.
- Never hardcode usernames inside playbooks.
- Use SSH key authentication.
- Organize inventories by environment (Dev, QA, UAT, Production).
- Use dynamic inventory for cloud environments whenever possible.

### Easy Way to Remember

Inventory controls:

- Which server?
- Which user?
- Which SSH port?

The playbook remains exactly the same.
# Ansible Interview Questions (Production Grade)

---

# 🔴16. How do you handle sensitive information like API keys or passwords in Ansible?

In production, we never hardcode sensitive information such as passwords, API keys, SSH private keys, database credentials, or cloud access tokens inside playbooks or Git repositories.

For small or on-premises environments, Ansible provides **Ansible Vault**, which encrypts sensitive files or variables. Only users with the Vault password can decrypt and use them during playbook execution.

Example:

```bash
ansible-vault create secrets.yml
```

Edit an encrypted file:

```bash
ansible-vault edit secrets.yml
```

Run a playbook using Vault:

```bash
ansible-playbook site.yml --ask-vault-pass
```

However, in enterprise production environments, we generally prefer external secret management solutions such as:

- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault

During deployment, Ansible retrieves secrets dynamically instead of storing them in Git.

### Real-Time Example

In one of our Azure projects, database passwords and application secrets were stored in **Azure Key Vault**.

When GitHub Actions executed the Ansible playbook, it authenticated with Azure, retrieved the secrets from Key Vault, and passed them securely to the application configuration without exposing them in logs or source code.

This approach improved security, simplified secret rotation, and eliminated hardcoded credentials.

### Best Practices

- Never commit secrets to Git.
- Use Ansible Vault only for small environments.
- Prefer centralized secret managers for production.
- Rotate secrets regularly.
- Restrict secret access using least-privilege IAM policies.

---

# 🔴17. How do you handle failures in a playbook (rollback, continue on failure)?

In production, failure handling depends on the impact of the task. Some failures should stop the deployment immediately, while others can be safely ignored.

Ansible provides multiple mechanisms for handling failures.

### Continue on Failure

For non-critical tasks, we can use:

```yaml
ignore_errors: yes
```

This allows the playbook to continue even if the task fails.

---

### Stop Immediately

For critical tasks such as application deployment or database migration, we allow the playbook to fail immediately.

This prevents partial deployments.

---

### Rollback

Ansible doesn't provide automatic rollback.

Instead, we design rollback procedures.

Typical rollback steps include:

- Restore the previous application version.
- Restore configuration backups.
- Restart the previous service.
- Restore database backups if required.

---

### Block, Rescue, Always

Ansible also supports structured error handling.

Example:

```yaml
block:
  - Deploy Application

rescue:
  - Restore Previous Version

always:
  - Send Notification
```

---

### Real-Time Example

During one of our application deployments:

- The new application package failed health checks.
- The deployment task failed.
- The rescue block restored the previous application package.
- Nginx was restarted.
- Slack and email notifications were sent automatically.

This minimized downtime and ensured users continued accessing the previous stable release.

### Best Practices

- Always validate deployments.
- Keep rollback artifacts available.
- Backup configurations before deployment.
- Use block/rescue for production deployments.
- Send deployment notifications.

---

# 🔴18. What is a Callback Plugin in Ansible?

A **Callback Plugin** is an Ansible plugin that controls how playbook execution results are displayed or processed.

Instead of changing the automation itself, callback plugins customize what happens after each task completes.

Common uses include:

- Better console output.
- Logging playbook execution.
- Sending Slack notifications.
- Email notifications.
- Generating JSON reports.
- Integrating with monitoring systems.

### Real-Time Example

In one of our CI/CD pipelines, after every Ansible deployment:

- Success notifications were sent to Slack.
- Failed deployments triggered Microsoft Teams alerts.
- Deployment logs were archived.
- Execution summaries were published in the pipeline.

This allowed developers and operations teams to monitor deployment status without manually checking the pipeline.

### Common Callback Plugins

- default
- yaml
- json
- profile_tasks
- timer

### Best Practices

- Enable execution logging.
- Integrate notifications with collaboration tools.
- Use profile_tasks to identify slow tasks.
- Archive deployment logs.

### Easy Way to Remember

**Playbook Finished → Callback Plugin Processes the Result**

---

# 🔴19. What is Ansible Tower / AWX, and how do you implement RBAC?

**AWX** is the open-source version of **Ansible Tower** (now Red Hat Ansible Automation Platform).

It provides a web-based interface and centralized management for Ansible automation.

Instead of running playbooks manually from the command line, teams can:

- Launch playbooks through a web UI.
- Schedule jobs.
- Manage inventories.
- Store credentials securely.
- Monitor job history.
- Control user access.

One of its biggest advantages is **Role-Based Access Control (RBAC).**

RBAC allows us to define exactly who can:

- View inventories.
- Execute playbooks.
- Manage credentials.
- Edit projects.
- Administer the platform.

### Real-Time Example

In one of our enterprise environments:

- Developers could execute application deployment jobs.
- Operations engineers managed inventories.
- DevOps administrators managed credentials and projects.
- Production deployments required approval before execution.

This prevented unauthorized changes and provided a complete audit trail for every deployment.

### Best Practices

- Integrate AWX with LDAP or Azure AD.
- Assign permissions using roles instead of individual users.
- Separate Development and Production inventories.
- Store credentials inside AWX Credential Manager.
- Enable audit logging.

### Easy Way to Remember

AWX = **Centralized Ansible Management + RBAC + Scheduling + Audit Logs**

---

# 🔴20. Can you create infrastructure using Ansible? How is it different from Terraform?

Yes.

Ansible can provision infrastructure using cloud modules for AWS, Azure, and GCP.

For example, Ansible can create:

- EC2 Instances
- Azure Virtual Machines
- Virtual Networks
- Security Groups
- Load Balancers
- Storage Accounts

However, in production we generally use **Terraform** for infrastructure provisioning and **Ansible** for server configuration.

This is because the two tools are designed for different purposes.

### Terraform

Terraform is an **Infrastructure as Code (IaC)** tool.

It:

- Creates infrastructure.
- Maintains infrastructure state.
- Detects configuration drift.
- Generates execution plans.
- Supports infrastructure lifecycle management.

Examples:

- VPC
- Subnets
- EKS
- AKS
- EC2
- RDS
- Load Balancers

---

### Ansible

Ansible is a **Configuration Management and Automation** tool.

It:

- Installs software.
- Configures operating systems.
- Deploys applications.
- Manages services.
- Performs patching.

Examples:

- Install Docker.
- Configure Nginx.
- Deploy Java applications.
- Update configuration files.
- Restart services.

---

### Real-Time Example

In one of our production projects, the deployment workflow was:

```
Developer
      │
      ▼
GitHub Actions
      │
      ▼
Terraform
      │
Creates AWS Infrastructure
      │
      ▼
EC2 Instances
      │
      ▼
Ansible
      │
Installs Java
Installs Nginx
Deploys Application
Configures Services
      │
      ▼
Application Ready
```

Terraform provisioned the AWS infrastructure, including VPCs, EC2 instances, Load Balancers, and Security Groups.

Once provisioning completed successfully, GitHub Actions automatically triggered Ansible to configure the operating system, install application dependencies, deploy the application, configure Nginx, and start the required services.

Using both tools together gave us complete infrastructure automation with consistent server configuration.

---

## Easy Way to Remember

| Terraform | Ansible |
|------------|----------|
| Infrastructure Provisioning | Configuration Management |
| Declarative | Declarative |
| Maintains State File | No State File |
| Creates Cloud Resources | Configures Existing Servers |
| Best for Infrastructure Lifecycle | Best for Software Deployment & Configuration |

---

## Interview Closing

> "Although Ansible can provision cloud resources, in production we use Terraform for infrastructure provisioning because of its strong state management and lifecycle capabilities. Once Terraform creates the infrastructure, Ansible takes over to configure the servers, install required software, deploy applications, and manage services. Using both together gives us a complete and scalable Infrastructure as Code solution."

# Ansible Interview Questions (Production Grade)

---

# 🔴21. How would you use Ansible to automate the deployment of a cloud infrastructure?

In production, we usually don't use Ansible as the primary Infrastructure as Code (IaC) tool. Instead, we use **Terraform** to provision cloud infrastructure and **Ansible** to configure the provisioned servers.

A typical deployment workflow looks like this:

```text
Developer
      │
      ▼
GitHub Actions
      │
      ▼
Terraform
      │
Creates Infrastructure
(VPC, EC2, ALB, RDS)
      │
      ▼
Dynamic Inventory
      │
      ▼
Ansible
      │
Configure Servers
Install Packages
Deploy Application
Start Services
      │
      ▼
Application Ready
```

Once Terraform provisions the infrastructure, Ansible automatically discovers the newly created instances using **Dynamic Inventory**.

It then performs configuration tasks such as:

- Updating Linux packages.
- Installing Java, Docker, or Nginx.
- Creating application users.
- Configuring firewall rules.
- Deploying application code.
- Starting and enabling services.
- Installing monitoring agents.

### Real-Time Example

In one of our AWS projects:

- Terraform provisioned the VPC, EC2 instances, Security Groups, Application Load Balancer, and RDS.
- GitHub Actions automatically triggered an Ansible playbook.
- Ansible installed Docker and Nginx, configured application files, deployed the latest Docker image from Amazon ECR, started the application, and verified health checks.

The entire infrastructure and application stack was deployed automatically without any manual intervention.

### Best Practices

- Use Terraform for provisioning.
- Use Dynamic Inventory.
- Store secrets in Vault or Cloud Secret Managers.
- Keep playbooks modular using Roles.
- Integrate Ansible into CI/CD pipelines.

---

# 🔴22. Write an Ansible playbook to deploy a Docker container.

In production, Ansible is commonly used to deploy Docker containers after the infrastructure has already been provisioned.

The playbook typically performs the following steps:

- Install Docker (if required).
- Start and enable Docker.
- Authenticate with the container registry.
- Pull the latest image.
- Deploy or update the container.
- Verify that the application is running.

### Sample Playbook

```yaml
---
- name: Deploy Docker Container
  hosts: appservers
  become: yes

  tasks:

    - name: Install Docker
      package:
        name: docker
        state: present

    - name: Start Docker Service
      service:
        name: docker
        state: started
        enabled: yes

    - name: Deploy Application Container
      community.docker.docker_container:
        name: myapp
        image: myrepo/myapp:latest
        state: started
        restart_policy: always
        ports:
          - "80:8080"
```

### Real-Time Example

In one of our projects:

- GitHub Actions built the Docker image.
- The image was pushed to **Amazon ECR**.
- After the push completed successfully, GitHub Actions executed an Ansible playbook.
- Ansible connected to the application servers, pulled the latest image from ECR, recreated the container if the image had changed, and verified that the application was healthy.

Because Ansible modules are idempotent, unchanged containers were not recreated unnecessarily.

### Best Practices

- Use Docker modules instead of shell commands.
- Store registry credentials securely.
- Use image tags instead of `latest` in production.
- Configure restart policies.
- Perform post-deployment health checks.

---

# 🔴23. Write an Ansible playbook to configure HAProxy as a load balancer.

In production, HAProxy is commonly used as a reverse proxy and load balancer to distribute traffic across multiple backend application servers.

Ansible automates:

- Installing HAProxy.
- Deploying the configuration.
- Validating the configuration.
- Restarting the service only if changes occur.

### Sample Playbook

```yaml
---
- name: Configure HAProxy
  hosts: loadbalancer
  become: yes

  tasks:

    - name: Install HAProxy
      package:
        name: haproxy
        state: present

    - name: Copy HAProxy Configuration
      template:
        src: haproxy.cfg.j2
        dest: /etc/haproxy/haproxy.cfg
      notify:
        - Restart HAProxy

  handlers:

    - name: Restart HAProxy
      service:
        name: haproxy
        state: restarted
```

### Real-Time Example

In one of our environments, we had four Java application servers behind HAProxy.

Whenever a new backend server was added, only the inventory was updated.

The Ansible playbook automatically regenerated the HAProxy configuration using a **Jinja2 template**, validated it, and restarted HAProxy only if the configuration changed.

This eliminated manual configuration and reduced deployment errors.

### Best Practices

- Generate configurations using Jinja2 templates.
- Validate configuration before restarting HAProxy.
- Restart services only when configuration changes.
- Store backend server information in inventory variables.

---

# 🔴24. How do you automate MySQL database backup and restore using Ansible?

In production, database backups are fully automated using Ansible and scheduled through Cron, AWX, or CI/CD pipelines.

The backup process generally includes:

- Creating the backup directory.
- Running `mysqldump`.
- Compressing the backup.
- Uploading it to cloud storage.
- Sending notifications.

### Sample Backup Playbook

```yaml
---
- name: Backup MySQL Database
  hosts: database
  become: yes

  tasks:

    - name: Create Backup Directory
      file:
        path: /backup/mysql
        state: directory

    - name: Backup Database
      shell: >
        mysqldump -u root -p{{ mysql_password }}
        mydb >
        /backup/mysql/mydb.sql
```

### Restore Example

```yaml
- name: Restore Database
  shell: >
    mysql -u root -p{{ mysql_password }}
    mydb <
    /backup/mysql/mydb.sql
```

### Real-Time Example

In one of our production environments:

- A nightly GitHub Actions workflow triggered an Ansible playbook.
- The playbook created a MySQL backup.
- The backup was compressed.
- Uploaded automatically to Amazon S3.
- Old backups older than 30 days were deleted.
- Slack notifications confirmed successful completion.

During disaster recovery testing, restoring the latest backup required only running the restore playbook, reducing recovery time significantly.

### Best Practices

- Encrypt backup files.
- Upload backups to remote storage.
- Automate backup verification.
- Test restoration regularly.
- Rotate old backups automatically.
- Never store database passwords inside playbooks.
# 🔴21. How would you use Ansible to automate the deployment of a cloud infrastructure?

In production, we usually don't use Ansible as the primary Infrastructure as Code (IaC) tool. Instead, we use **Terraform** to provision cloud infrastructure and **Ansible** to configure the provisioned servers.

A typical deployment workflow looks like this:

```text
Developer
      │
      ▼
GitHub Actions
      │
      ▼
Terraform
      │
Creates Infrastructure
(VPC, EC2, ALB, RDS)
      │
      ▼
Dynamic Inventory
      │
      ▼
Ansible
      │
Configure Servers
Install Packages
Deploy Application
Start Services
      │
      ▼
Application Ready
```

Once Terraform provisions the infrastructure, Ansible automatically discovers the newly created instances using **Dynamic Inventory**.

It then performs configuration tasks such as:

- Updating Linux packages.
- Installing Java, Docker, or Nginx.
- Creating application users.
- Configuring firewall rules.
- Deploying application code.
- Starting and enabling services.
- Installing monitoring agents.

### Real-Time Example

In one of our AWS projects:

- Terraform provisioned the VPC, EC2 instances, Security Groups, Application Load Balancer, and RDS.
- GitHub Actions automatically triggered an Ansible playbook.
- Ansible installed Docker and Nginx, configured application files, deployed the latest Docker image from Amazon ECR, started the application, and verified health checks.

The entire infrastructure and application stack was deployed automatically without any manual intervention.

### Best Practices

- Use Terraform for provisioning.
- Use Dynamic Inventory.
- Store secrets in Vault or Cloud Secret Managers.
- Keep playbooks modular using Roles.
- Integrate Ansible into CI/CD pipelines.

---

# 🔴22. Write an Ansible playbook to deploy a Docker container.

In production, Ansible is commonly used to deploy Docker containers after the infrastructure has already been provisioned.

The playbook typically performs the following steps:

- Install Docker (if required).
- Start and enable Docker.
- Authenticate with the container registry.
- Pull the latest image.
- Deploy or update the container.
- Verify that the application is running.

### Sample Playbook

```yaml
---
- name: Deploy Docker Container
  hosts: appservers
  become: yes

  tasks:

    - name: Install Docker
      package:
        name: docker
        state: present

    - name: Start Docker Service
      service:
        name: docker
        state: started
        enabled: yes

    - name: Deploy Application Container
      community.docker.docker_container:
        name: myapp
        image: myrepo/myapp:latest
        state: started
        restart_policy: always
        ports:
          - "80:8080"
```

### Real-Time Example

In one of our projects:

- GitHub Actions built the Docker image.
- The image was pushed to **Amazon ECR**.
- After the push completed successfully, GitHub Actions executed an Ansible playbook.
- Ansible connected to the application servers, pulled the latest image from ECR, recreated the container if the image had changed, and verified that the application was healthy.

Because Ansible modules are idempotent, unchanged containers were not recreated unnecessarily.

### Best Practices

- Use Docker modules instead of shell commands.
- Store registry credentials securely.
- Use image tags instead of `latest` in production.
- Configure restart policies.
- Perform post-deployment health checks.

---

# 🔴23. Write an Ansible playbook to configure HAProxy as a load balancer.

In production, HAProxy is commonly used as a reverse proxy and load balancer to distribute traffic across multiple backend application servers.

Ansible automates:

- Installing HAProxy.
- Deploying the configuration.
- Validating the configuration.
- Restarting the service only if changes occur.

### Sample Playbook

```yaml
---
- name: Configure HAProxy
  hosts: loadbalancer
  become: yes

  tasks:

    - name: Install HAProxy
      package:
        name: haproxy
        state: present

    - name: Copy HAProxy Configuration
      template:
        src: haproxy.cfg.j2
        dest: /etc/haproxy/haproxy.cfg
      notify:
        - Restart HAProxy

  handlers:

    - name: Restart HAProxy
      service:
        name: haproxy
        state: restarted
```

### Real-Time Example

In one of our environments, we had four Java application servers behind HAProxy.

Whenever a new backend server was added, only the inventory was updated.

The Ansible playbook automatically regenerated the HAProxy configuration using a **Jinja2 template**, validated it, and restarted HAProxy only if the configuration changed.

This eliminated manual configuration and reduced deployment errors.

### Best Practices

- Generate configurations using Jinja2 templates.
- Validate configuration before restarting HAProxy.
- Restart services only when configuration changes.
- Store backend server information in inventory variables.

---

# 🔴24. How do you automate MySQL database backup and restore using Ansible?

In production, database backups are fully automated using Ansible and scheduled through Cron, AWX, or CI/CD pipelines.

The backup process generally includes:

- Creating the backup directory.
- Running `mysqldump`.
- Compressing the backup.
- Uploading it to cloud storage.
- Sending notifications.

### Sample Backup Playbook

```yaml
---
- name: Backup MySQL Database
  hosts: database
  become: yes

  tasks:

    - name: Create Backup Directory
      file:
        path: /backup/mysql
        state: directory

    - name: Backup Database
      shell: >
        mysqldump -u root -p{{ mysql_password }}
        mydb >
        /backup/mysql/mydb.sql
```

### Restore Example

```yaml
- name: Restore Database
  shell: >
    mysql -u root -p{{ mysql_password }}
    mydb <
    /backup/mysql/mydb.sql
```

### Real-Time Example

In one of our production environments:

- A nightly GitHub Actions workflow triggered an Ansible playbook.
- The playbook created a MySQL backup.
- The backup was compressed.
- Uploaded automatically to Amazon S3.
- Old backups older than 30 days were deleted.
- Slack notifications confirmed successful completion.
# 🔴21. How would you use Ansible to automate the deployment of a cloud infrastructure?

In production, we usually don't use Ansible as the primary Infrastructure as Code (IaC) tool. Instead, we use **Terraform** to provision cloud infrastructure and **Ansible** to configure the provisioned servers.

A typical deployment workflow looks like this:

```text
Developer
      │
      ▼
GitHub Actions
      │
      ▼
Terraform
      │
Creates Infrastructure
(VPC, EC2, ALB, RDS)
      │
      ▼
Dynamic Inventory
      │
      ▼
Ansible
      │
Configure Servers
Install Packages
Deploy Application
Start Services
      │
      ▼
Application Ready
```

Once Terraform provisions the infrastructure, Ansible automatically discovers the newly created instances using **Dynamic Inventory**.

It then performs configuration tasks such as:

- Updating Linux packages.
- Installing Java, Docker, or Nginx.
- Creating application users.
- Configuring firewall rules.
- Deploying application code.
- Starting and enabling services.
- Installing monitoring agents.

### Real-Time Example

In one of our AWS projects:

- Terraform provisioned the VPC, EC2 instances, Security Groups, Application Load Balancer, and RDS.
- GitHub Actions automatically triggered an Ansible playbook.
- Ansible installed Docker and Nginx, configured application files, deployed the latest Docker image from Amazon ECR, started the application, and verified health checks.

The entire infrastructure and application stack was deployed automatically without any manual intervention.

### Best Practices

- Use Terraform for provisioning.
- Use Dynamic Inventory.
- Store secrets in Vault or Cloud Secret Managers.
- Keep playbooks modular using Roles.
- Integrate Ansible into CI/CD pipelines.

---

# 🔴22. Write an Ansible playbook to deploy a Docker container.

In production, Ansible is commonly used to deploy Docker containers after the infrastructure has already been provisioned.

The playbook typically performs the following steps:

- Install Docker (if required).
- Start and enable Docker.
- Authenticate with the container registry.
- Pull the latest image.
- Deploy or update the container.
- Verify that the application is running.

### Sample Playbook

```yaml
---
- name: Deploy Docker Container
  hosts: appservers
  become: yes

  tasks:

    - name: Install Docker
      package:
        name: docker
        state: present

    - name: Start Docker Service
      service:
        name: docker
        state: started
        enabled: yes

    - name: Deploy Application Container
      community.docker.docker_container:
        name: myapp
        image: myrepo/myapp:latest
        state: started
        restart_policy: always
        ports:
          - "80:8080"
```

### Real-Time Example

In one of our projects:

- GitHub Actions built the Docker image.
- The image was pushed to **Amazon ECR**.
- After the push completed successfully, GitHub Actions executed an Ansible playbook.
- Ansible connected to the application servers, pulled the latest image from ECR, recreated the container if the image had changed, and verified that the application was healthy.

Because Ansible modules are idempotent, unchanged containers were not recreated unnecessarily.

### Best Practices

- Use Docker modules instead of shell commands.
- Store registry credentials securely.
- Use image tags instead of `latest` in production.
- Configure restart policies.
- Perform post-deployment health checks.

---

# 🔴23. Write an Ansible playbook to configure HAProxy as a load balancer.

In production, HAProxy is commonly used as a reverse proxy and load balancer to distribute traffic across multiple backend application servers.

Ansible automates:

- Installing HAProxy.
- Deploying the configuration.
- Validating the configuration.
- Restarting the service only if changes occur.

### Sample Playbook

```yaml
---
- name: Configure HAProxy
  hosts: loadbalancer
  become: yes

  tasks:

    - name: Install HAProxy
      package:
        name: haproxy
        state: present

    - name: Copy HAProxy Configuration
      template:
        src: haproxy.cfg.j2
        dest: /etc/haproxy/haproxy.cfg
      notify:
        - Restart HAProxy

  handlers:

    - name: Restart HAProxy
      service:
        name: haproxy
        state: restarted
```

### Real-Time Example

In one of our environments, we had four Java application servers behind HAProxy.

Whenever a new backend server was added, only the inventory was updated.

The Ansible playbook automatically regenerated the HAProxy configuration using a **Jinja2 template**, validated it, and restarted HAProxy only if the configuration changed.

This eliminated manual configuration and reduced deployment errors.

### Best Practices

- Generate configurations using Jinja2 templates.
- Validate configuration before restarting HAProxy.
- Restart services only when configuration changes.
- Store backend server information in inventory variables.

---

# 🔴24. How do you automate MySQL database backup and restore using Ansible?

In production, database backups are fully automated using Ansible and scheduled through Cron, AWX, or CI/CD pipelines.

The backup process generally includes:

- Creating the backup directory.
- Running `mysqldump`.
- Compressing the backup.
- Uploading it to cloud storage.
- Sending notifications.

### Sample Backup Playbook

```yaml
---
- name: Backup MySQL Database
  hosts: database
  become: yes

  tasks:

    - name: Create Backup Directory
      file:
        path: /backup/mysql
        state: directory

    - name: Backup Database
      shell: >
        mysqldump -u root -p{{ mysql_password }}
        mydb >
        /backup/mysql/mydb.sql
```

### Restore Example

```yaml
- name: Restore Database
  shell: >
    mysql -u root -p{{ mysql_password }}
    mydb <
    /backup/mysql/mydb.sql
```

### Real-Time Example

In one of our production environments:

- A nightly GitHub Actions workflow triggered an Ansible playbook.
- The playbook created a MySQL backup.
- The backup was compressed.
- Uploaded automatically to Amazon S3.
- Old backups older than 30 days were deleted.
- Slack notifications confirmed successful completion.
