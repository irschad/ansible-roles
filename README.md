# Structure Playbooks with Ansible Roles

## Project Overview
This project demonstrates how to organize large Ansible playbooks into smaller, manageable files using Ansible roles. The primary objective is to improve maintainability and reusability by structuring tasks into well-defined roles.

---

## Technologies Used
- **Ansible**: For configuration management and automation.
- **Docker**: To containerize applications and manage services.
- **AWS**: As the cloud infrastructure provider.
- **Linux**: The target environment for playbooks and container orchestration.

---

## Project Description
The project includes:
1. Creating a new Linux user.
2. Setting up Docker and Docker Compose.
3. Deploying Docker containers using a structured and modular approach with Ansible roles.

### Roles and Sub-Folders
#### 1. `create_user`
- **Purpose**: Create a new Linux user and assign groups.
- **Structure**:
  - `tasks/`
    - `main.yaml`:
      ```yaml
      - name: Create new linux user
        user:
          name: irschad
          groups: "{{ user_groups }}"
      ```

#### 2. `start_containers`
- **Purpose**: Deploy Docker containers using a `docker-compose` file.
- **Structure**:
  - `defaults/`
    - `main.yaml`:
      ```yaml
      user_groups: adm,docker
      ```
  - `files/`
    - `docker-compose-full.yaml`:
      ```yaml
      version: '3'
      services:
        java-app:
          image: irschad/java-app:4.0
          environment:
            - DB_USER=user
            - DB_PWD=pass
            - DB_SERVER=mysql
            - DB_NAME=my-app-db
          ports:
            - 8080:8080
          container_name: my-java-app
        mysql:
          image: mysql
          ports:
            - 3306:3306
          environment:
            - MYSQL_ROOT_PASSWORD=my-secret-pw
            - MYSQL_DATABASE=my-app-db
            - MYSQL_USER=user
            - MYSQL_PASSWORD=pass
          volumes:
            - mysql-data:/var/lib/mysql
          container_name: mysql
        phpmyadmin:
          image: phpmyadmin
          environment:
            - PMA_HOST=mysql
          ports:
            - 8083:80
          container_name: myadmin
      volumes:
        mysql-data:
          driver: local
      ```
  - `tasks/`
    - `main.yaml`:
      ```yaml
      - name: Copy docker compose 
        copy:
          src: docker-compose-full.yaml
          dest: /home/irschad/docker-compose.yaml

      - name: Docker login
        docker_login:
          registry_url: "{{ docker_registry }}"
          username: "{{ docker_username }}"
          password: "{{ docker_password }}"

      - name: Start containers from compose
        community.docker.docker_compose_v2:
          project_src: /home/irschad
      ```
  - `vars/`
    - `main.yaml`:
      ```yaml
      docker_registry: https://index.docker.io/v1/
      docker_username: irschad
      ```

---

## Additional Files
#### 1. `deploy-docker-with-roles.yaml`
An Ansible playbook that orchestrates:
- Installing Docker and Docker Compose.
- Creating a new Linux user.
- Deploying containers using roles.

#### 2. `inventory_aws_ec2.yaml`
Dynamic inventory plugin for AWS EC2 instances:
```yaml
---
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Platform: preprod
  instance-state-name: running
```

#### 3. `ansible.cfg`
Ansible configuration file:
```ini
[defaults]
host_key_checking = False
inventory = inventory_aws_ec2.yaml
enable_plugins = aws_ec2
remote_user = ec2-user
```

---

## Execution Steps
1. **Prepare AWS EC2 Inventory**:
   Ensure the target EC2 instances are tagged appropriately and are in the `us-east-1` region.

2. **Execute the Playbook**:
   Run the following command to deploy the setup:
   ```bash
   ansible-playbook deploy-docker-with-roles.yaml
   ```

3. **Verify Deployment**:
   - SSH into the EC2 instance.
   - Confirm the user `irschad` is created.
   - Verify Docker containers are running using `docker ps`.

---

## Key Benefits of Using Roles
1. **Modularity**: Break down tasks into reusable components.
2. **Readability**: Simplify complex playbooks.
3. **Maintainability**: Easier updates and debugging.
4. **Reusability**: Share roles across multiple projects.

---

## Conclusion
This project highlights the best practices for structuring Ansible playbooks with roles. By breaking down tasks into modular components, we ensure a scalable and maintainable approach to automation.

---



