# DevOps Engineer - Technical Test
---

## Requirements

### CI Build Pipeline

1. **Trigger Mechanism**:  
   - The CI job will run job automatically when there is new push on **main** branch
   - The first job to be running is Build and Push Docker Image to Dockerhub

2. **Deployment Workflow**:  
   - Build and package application to Docker Image
   - Push recently Docker Image to Dockerhub
   - Pull Docker Image to targeted servers
   - Run Container Image in targeted servers

---

### Target Environment

The environment have the following specifications:

1. **Load Balancer**:
   - Using Nginx as Load Balancer
   - Accessible via HTTP on port 80.
   - Configured to use a **round-robin** strategy to distribute traffic between application servers.

2. **Application Servers**:
   - Two instances running this web application.
   - Each accessible via HTTP on port 3000.
   - The application must respond with:  
     `Hi there! I'm being served from {hostname}!`

---

## Tools and Technologies

To implement this solution, the following technology use:

- **CI Services**: GitHub Actions
- **Provisioning Tools**: Terraform
- **Local Environment**: Docker, Vagrant with VMWare Workstation Provider
- **Cloud Providers**: AWS
- **Load Balancers**: NGINX
- **Version Control**: GitHub

---

## Execution Plan

### Provision Local Server with Vagrant
1. Go to infrastructure/local directory
2. Check **vm.provider**, adjust with virtualization provider and server spec needs to provision
3. Adjust IP Address in each machine with the segment use in virtualization provider segment on this line
   * **Nginx**
      ```bash
      nginx.vm.network :private_network, ip: "192.168.74.60"
      ```

   * **Application Server**
      ```bash
      application.vm.network :private_network, ip: "192.168.74.#{i + 61}"
      ```
   * Adjust provision_nginx script under **upstream backend_servers** section to
4. Provision the server with run this command
   ```bash
  vagrant provision
   ```
5. Check server status and make sure machine states are running
   ```bash
  vagrant status
   ```
   ![Vagrant Status](/images/vagrant-status.png)
6. Run **vagrant ssh** following by name the server created
   ![Vagrant SSH to Server](/images/vagrant-ssh.png)

### Setting Github Actions Runner
1. SSH to each application1 and application2 server
2. Go to **/home/vagrant/actions-runner** directory
3. Go to Github Repository > Settings
   - New Self-hosted Runner
   - copy from **Configure** steps, adjust token value with token value show on configuration. For example:
     ```bash
     ./config.sh --url https://github.com/imanyauma/ntx-devops-test --token AEE43AJ4VJYFCSFF4DIENXLHIQALA
     ```
4. Run with **./run.sh &** command and refresh the terminal
5. Make sure Github Runner status is **idle** in  Github Repository > Settings > Runners
   ![Github Runner](/images/github-actions-runners.png)

### Check NGINX Server configuration
1. SSH to nginx server
2. Check Nginx configuration with this command **cat /etc/nginx/nginx.conf | grep "include /etc/nginx/sites-"** to make sure the highlighted line is commented
3. If the line is not commented, edit manually from /etc/nginx/nginx.conf, and comment on this line **include /etc/nginx/sites-enabled/**
4. Check Load Balancer configuration, make sure **/etc/nginx/conf.d/load_balancer.conf** is exist
5. If not exist, copy **load-balancer.conf** script to **/etc/nginx/conf.d/load_balancer.conf**
6. Restart Nginx service
6. Open Nginx IP Server on browser. Currently will show 502 error because we still not deploy any service in app server

---

## Check Web Application and Troubleshooting

This is a Node.js application runs with Docker. To check application running, follow this steps after deployment status is successful:

1. SSH to application1 and application2 server
2. Run this command to check Docker Container status in both server
   ```bash
   docker ps -a | grep ntx-devops-test
   ```
3. If container status is running, verify the application in browser using Nginx IP, refresh several times to check Load Balancing is working by routing request to different hosts (container id).
Host Container 1
![Container Host 1](/images/host-1-container.png)
Host Container 2
![Container Host 2](/images/host-2-container.png)

---

## Notes
- Needs to configure Github Actions Runner if runs on local server to run deployment job.
- Github Actions Runner runs on local server should be reconfigure manually because needs to fill some configurations manual.
- Currently deployment already tested on local environment to reduce cloud cost
- To provision Nginx Load Balancer with provision tools, needs to use some linux scripting to reconfigure default configuration.
--- 
