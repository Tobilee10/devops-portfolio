# Enterprise Linux Server Deployment – Troubleshooting Guide

## 1. Introduction

This document provides troubleshooting procedures for common problems encountered while deploying, securing, configuring, automating, and monitoring the Ubuntu Linux server.

It serves as an operational reference for identifying issues, investigating their causes, applying corrective actions, and verifying successful recovery.

* **Project:** Enterprise Linux Server Deployment

* **Document:** Troubleshooting Guide

* **Status:** Prepared

> **Important:** Troubleshooting procedures should be performed carefully. Collect relevant evidence before changing configurations, and avoid destructive actions unless their consequences are understood.

## 2. Troubleshooting Methodology

Use the following structured approach when investigating an issue:

1. Identify and clearly describe the problem.

2. Determine the affected server, service, or component.

3. Collect relevant logs and command outputs.

4. Identify possible causes.

5. Determine the safest corrective action.

6. Apply one controlled change at a time.

7. Verify that the issue is resolved.

8. Document the root cause and resolution.

Avoid making multiple unrelated changes at once. This makes it easier to identify which change resolved the problem.

## 3. SSH Connection Problems

**Symptoms**

* Connection timed out.

* Permission denied.

* Connection refused.

* Unable to connect to the EC2 instance.

* SSH connection terminates unexpectedly.

**Diagnostic commands**

From the administrative workstation:

```
ssh -v -i ~/.ssh/enterprise-linux-server-key.pem ubuntu@<PUBLIC_IP>

```

On the EC2 server, where access is available:

```
sudo systemctl status ssh --no-pager

```

Validate the SSH daemon configuration:

```
sudo sshd -t

```

Check the listening port:

```
sudo ss -tlnp | grep ':22'

```

**Possible causes**

* Incorrect public IP address.

* Incorrect private key path.

* Incorrect private key permissions.

* Incorrect SSH username.

* AWS Security Group blocking TCP port 22.

* UFW blocking SSH.

* SSH service stopped.

* Incorrect SSH daemon configuration.

* Network connectivity problems.

**Resolution**

1. Verify the EC2 public IP address.

2. Confirm that the correct private key is being used.

3. Confirm the correct remote username, commonly "ubuntu" for Ubuntu EC2 images.

4. Verify AWS Security Group inbound rules.

5. Check UFW rules if the server is reachable through another administrative method.

6. Confirm that the SSH service is running.

Check private key permissions:

```
chmod 400 ~/.ssh/enterprise-linux-server-key.pem

```

Ensure SSH is running:

```
sudo systemctl enable --now ssh

```

Important: Do not change SSH or firewall rules without preserving an authorized administrative access path.

## 4. Permission Denied Errors

**Symptoms**

* Unable to access files.

* Permission denied when executing scripts.

* Unable to perform administrative operations.

* Ansible cannot write to a remote directory.

**Diagnostic commands**

```
whoami
id
ls -l <FILE>
namei -l <FILE_PATH>

```

Check sudo privileges:

```
sudo -l

```

**Possible causes**

* Incorrect file ownership.

* Insufficient file or directory permissions.

* User not authorized for sudo.

* Script does not have execute permission.

* Parent directory does not allow traversal.

* Incorrect group membership.

**Resolution**

Review file ownership and permissions before modifying them.

Example:

```
chmod +x scripts/system-health-check.sh

```

If ownership is incorrect, apply the appropriate owner and group:

```
sudo chown <USER>:<GROUP> <FILE_PATH>

```

For administrative commands:

```
sudo <COMMAND>

```

Grant additional privileges only when required. Avoid using unrestricted permissions such as "chmod 777" as a general solution.

## 5. UFW Firewall Problems

**Symptoms**

* SSH connection fails after firewall changes.

* Required application ports are inaccessible.

* Web services cannot be reached.

* Unexpected traffic is blocked.

**Diagnostic commands**

```
sudo ufw status verbose
sudo ufw status numbered
sudo ufw show raw

```

Check listening ports:

```
sudo ss -tulnp

```

**Possible causes**

* SSH allow rule missing.

* Incorrect administrator source IP.

* Required application port blocked.

* Firewall rules conflict with the intended network design.

* Application is not listening on the expected interface or port.

**Resolution**

Review the firewall rules and restore only the required access.

Example: allow SSH from a trusted administrator IP:

```
sudo ufw allow from <ADMIN_IP>/32 to any port 22 proto tcp

```

If an application requires a particular port, add an appropriately restricted rule.

After modifying the firewall, verify the resulting configuration:

```
sudo ufw status verbose

```

Do not disable the firewall without understanding the security consequences and having an alternative access-control plan.

## 6. Fail2ban Problems

**Symptoms**

* Fail2ban service is inactive.

* SSH jail is not enabled.

* Legitimate administrator IP is banned.

* Authentication failures are not being detected.

**Diagnostic commands**

```
sudo systemctl status fail2ban --no-pager
sudo fail2ban-client status
sudo fail2ban-client status sshd

```

Inspect logs:

```
sudo journalctl -u fail2ban -n 100 --no-pager

```

Validate the configuration:

```
sudo fail2ban-client -t

```

**Possible causes**

* Incorrect jail configuration.

* SSH jail disabled.

* Unsupported logging backend.

* Incorrect authentication log configuration.

* Fail2ban service not running.

* Incorrect filter configuration.

**Resolution**

1. Review the jail configuration.

2. Confirm that the SSH jail is enabled.

3. Check the configured logging backend.

4. Correct configuration errors.

5. Validate the configuration.

6. Restart the service.

```
sudo systemctl restart fail2ban

```

If a legitimate IP address is banned, verify its identity before unbanning:

```
sudo fail2ban-client set sshd unbanip <IP_ADDRESS>

```

Verify the jail status again:

```
sudo fail2ban-client status sshd

```

## 7. Nginx Service Problems

**Symptoms**

* Website is inaccessible.

* Nginx fails to start.

* HTTP returns an error.

* Configuration test fails.

* Reverse proxy returns an upstream error.

**Diagnostic commands**

```
sudo systemctl status nginx --no-pager
sudo nginx -t
sudo journalctl -u nginx -n 100 --no-pager

```

Check listening ports:

```
sudo ss -tlnp | grep -E ':80|:443'

```

Test local HTTP access:

```
curl -I http://localhost

```

**Possible causes**

* Incorrect Nginx configuration.

* Port conflict.

* Missing configuration files.

* Incorrect file permissions.

* Firewall or AWS Security Group restrictions.

* Upstream application unavailable.

* Incorrect reverse proxy destination.

**Resolution**

Validate the Nginx configuration:

```
sudo nginx -t

```

Reload after a successful validation:

```
sudo systemctl reload nginx

```

Restart only when necessary:

```
sudo systemctl restart nginx

```

If Nginx is acting as a reverse proxy, verify that the upstream application is running and listening on its expected port.

## 8. High Disk Utilization

**Symptoms**

* Disk usage reaches critical levels.

* Applications cannot write files.

* Services fail because storage is exhausted.

* Backup operations fail.

* The system reports insufficient disk space.

**Diagnostic commands**

Check filesystem usage:

```
df -h

```

Check inode usage:

```
df -i

```

Inspect block devices:

```
lsblk

```

Identify large directories:

```
sudo du -xhd1 /var 2>/dev/null | sort -h

```

Check journal disk usage:

```
sudo journalctl --disk-usage

```

**Possible causes**

* Large log files.

* Unnecessary packages.

* Application-generated files.

* Unmanaged backups.

* Insufficient storage capacity.

* Excessive temporary files.

* Inode exhaustion.

**Resolution**

1. Identify the filesystem approaching capacity.

2. Locate directories consuming significant space.

3. Determine whether logs, backups, or application files are responsible.

4. Preserve important data.

5. Safely remove unnecessary files.

6. Review log rotation and backup retention.

7. Verify disk and inode availability.

Review package cache:

```
sudo apt clean

```

If journal retention needs adjustment, configure appropriate limits rather than deleting required logs indiscriminately.

Never delete system files or application data without confirming their purpose.

## 9. High Memory Utilization

**Symptoms**

* Server becomes slow.

* Applications are terminated unexpectedly.

* Services fail due to insufficient memory.

* The system experiences swapping or out-of-memory events.

**Diagnostic commands**

```
free -h
top
ps aux --sort=-%mem | head

```

Check kernel out-of-memory events:

```
sudo journalctl -k | grep -i -E 'out of memory|oom|killed process'

```

**Possible causes**

* Applications consuming excessive memory.

* Too many running services.

* Memory leaks.

* Insufficient instance memory.

* Excessive caching or workload growth.

**Resolution**

1. Identify the process responsible.

2. Review its resource requirements and behavior.

3. Check application logs.

4. Determine whether the process is behaving abnormally.

5. Optimize or restart the application when appropriate.

6. Consider resizing the EC2 instance if justified by sustained workload requirements.

Do not terminate processes solely because they consume a large amount of memory without understanding their purpose.

## 10. High CPU Utilization

**Symptoms**

* Slow application response.

* High system load.

* Increased CPU utilization.

* Services become unresponsive.

**Diagnostic commands**

```
top
uptime
ps aux --sort=-%cpu | head

```

Check CPU information:

```
lscpu

```

**Possible causes**

* CPU-intensive processes.

* Uncontrolled application workloads.

* Background jobs.

* Unexpected processes.

* Resource limitations.

* Application defects or inefficient operations.

**Resolution**

1. Identify the process consuming excessive CPU.

2. Determine whether the workload is expected.

3. Review application and system logs.

4. Investigate scheduled jobs and background processes.

5. Optimize or restart the affected application when appropriate.

6. Consider resizing the EC2 instance if necessary.

Verify that CPU utilization returns to an acceptable level after corrective action.

## 11. Ansible Connectivity Problems

**Symptoms**

* Ansible ping fails.

* SSH authentication errors.

* Host unreachable.

* Playbook execution fails.

* Permission errors occur on the managed server.

**Diagnostic commands**

Run the Ansible connectivity test:

```
ansible linux_servers \
  -i ansible/inventory/hosts.ini \
  -m ping -vvv

```

Test SSH directly:

```
ssh -i ~/.ssh/enterprise-linux-server-key.pem ubuntu@<PUBLIC_IP>

```

Inspect the inventory:

```
cat ansible/inventory/hosts.ini

```

Check the Ansible version:

```
ansible --version

```

**Possible causes**

* Incorrect inventory configuration.

* Incorrect private key path.

* SSH connectivity problems.

* Incorrect remote username.

* Python interpreter unavailable on the managed server.

* Insufficient remote privileges.

* Incorrect Ansible configuration.

**Resolution**

1. Verify that SSH works independently of Ansible.

2. Review the inventory host address.

3. Confirm the remote username.

4. Confirm the private key path.

5. Verify Python availability on the managed server.

6. Review Ansible privilege escalation settings.

7. Rerun the connectivity test.

If privilege escalation is required, review the playbook's "become" configuration and the remote user's sudo permissions.

## 12. Package Installation Problems

**Symptoms**

* Unable to install packages.

* Package repository errors.

* Dependency conflicts.

* Package manager reports insufficient disk space.

**Diagnostic commands**

Update package indexes:

```
sudo apt update

```

Check available disk space:

```
df -h

```

Review package configuration:

```
apt-cache policy <PACKAGE_NAME>

```

**Possible causes**

* Repository connectivity issues.

* Package index problems.

* Incorrect repository configuration.

* Dependency conflicts.

* Insufficient disk space.

* Repository signing or authentication errors.

**Resolution**

1. Review the package manager error message.

2. Verify network and DNS connectivity.

3. Confirm repository configuration.

4. Check available disk space.

5. Correct the underlying issue.

6. Retry the installation.

If package dependencies are broken, inspect the package manager output before attempting repairs.

A possible recovery command is:

```
sudo apt --fix-broken install

```

Do not remove packages or modify repository configuration without understanding the impact.

## 13. Monitoring Problems

Project monitoring architecture

The monitoring architecture uses:

* Node Exporter running as a systemd service on the EC2 server.

* Prometheus running as a Docker container on the local workstation.

* Grafana running as a Docker container on the local workstation.

* Alertmanager running as a Docker container on the local workstation.

Prometheus collects metrics from Node Exporter. Grafana queries Prometheus, while Prometheus sends alerts to Alertmanager.

**Symptoms**

* Prometheus target is down.

* Grafana dashboards display no data.

* Metrics are missing.

* Monitoring containers fail to start.

* Alertmanager does not process notifications.

* Email notifications are not delivered.

### 13.1 Node Exporter problems

On the EC2 server:

```
sudo systemctl status node_exporter --no-pager

```

Inspect service logs:

```
sudo journalctl -u node_exporter -n 100 --no-pager

```

Check whether port 9100 is listening:

```
sudo ss -tlnp | grep ':9100'

```

Test the metrics endpoint locally on the EC2 server:

```
curl -f http://localhost:9100/metrics

```

Possible causes:

* Node Exporter service stopped.

* Incorrect service configuration.

* Executable missing or incorrect permissions.

* Port 9100 blocked.

* Network connectivity problems.

Resolution:

1. Verify the service status.

2. Review service logs.

3. Confirm that the metrics endpoint responds.

4. Verify AWS Security Group rules.

5. Restart the service when appropriate.

```
sudo systemctl restart node_exporter

```

### 13.2 Prometheus problems

On the local monitoring workstation:

Check container status:

```
docker compose -f monitoring/docker-compose.yml ps

```

Inspect logs:

```
docker logs enterprise-prometheus --tail 100

```

Validate the Prometheus configuration:

```
docker exec enterprise-prometheus \
  promtool check config /etc/prometheus/prometheus.yml

```

Possible causes:

* Invalid Prometheus configuration.

* Incorrect EC2 target address.

* Node Exporter unavailable.

* Network connectivity problems.

* Incorrect scrape configuration.

Resolution:

1. Validate the configuration.

2. Review Prometheus logs.

3. Confirm that the target address is correct.

4. Verify connectivity to the EC2 server on TCP port 9100.

5. Check Prometheus target health.

6. Restart or reload Prometheus after correcting the configuration.

Do not expose the Node Exporter port publicly to "0.0.0.0/0". Restrict access to the trusted monitoring workstation address.

### 13.3 Grafana problems

Check container status:

```
docker logs enterprise-grafana --tail 100

```

Check the Grafana health endpoint:

```
curl -f http://localhost:3000/api/health

```

Possible causes:

* Grafana container unavailable.

* Incorrect Prometheus data source.

* Incorrect dashboard queries.

* Missing metrics.

* Incorrect dashboard configuration.

Resolution:

1. Confirm that the Grafana container is running.

2. Check the Grafana health endpoint.

3. Verify the Prometheus data source.

4. Confirm that Prometheus is returning metrics.

5. Review dashboard queries and time ranges.

The Prometheus data source should use the Docker Compose service address:

http://prometheus:9090

### 13.4 Alertmanager problems

Inspect container logs:

```
docker logs enterprise-alertmanager --tail 100

```

Check health:

```
curl -f http://localhost:9093/-/healthy

```

Possible causes:

* Invalid Alertmanager configuration.

* SMTP authentication failure.

* Incorrect receiver configuration.

* Prometheus cannot reach Alertmanager.

* Container unavailable.

Resolution:

1. Review Alertmanager logs.

2. Validate its configuration.

3. Verify Docker Compose service connectivity.

4. Review SMTP configuration without exposing credentials.

5. Confirm that Prometheus references "alertmanager:9093".

6. Test notification delivery.

Never place SMTP passwords or other credentials directly in the public configuration files.

## 14. Docker Compose Problems

**Symptoms**

* Monitoring containers fail to start.

* Containers repeatedly restart.

* Ports are not bound.

* Services cannot communicate.

* Persistent monitoring data is unavailable.

**Diagnostic commands**

Check container status:

```
docker compose -f monitoring/docker-compose.yml ps

```

Review Compose configuration:

```
docker compose -f monitoring/docker-compose.yml config -q

```

Inspect container logs:

```
docker logs enterprise-prometheus --tail 100
docker logs enterprise-alertmanager --tail 100
docker logs enterprise-grafana --tail 100

```

Check Docker service:

```
sudo systemctl status docker --no-pager

```

**Possible causes**

* Invalid Compose YAML.

* Incorrect environment variables.

* Missing configuration files.

* Port conflicts.

* Volume permission problems.

* Container networking issues.

* Incorrect healthcheck configuration.

**Resolution**

1. Validate the Compose configuration.

2. Review container logs.

3. Check required environment variables.

4. Verify file and directory permissions.

5. Check port availability.

6. Correct the underlying issue.

7. Start or recreate only the affected service when appropriate.

```
docker compose -f monitoring/docker-compose.yml up -d

```

Avoid deleting persistent Docker volumes during troubleshooting. Prometheus, Grafana, and Alertmanager may store important project data in those volumes.

## 15. System Service Failures

**Symptoms**

* A service fails to start.

* Application becomes unavailable.

* Service enters a failed state.

* Service repeatedly restarts.

**Diagnostic commands**

List failed services:

```
sudo systemctl --failed

```

Inspect a specific service:

```
sudo systemctl status <SERVICE_NAME> --no-pager

```

Review service logs:

```
sudo journalctl -u <SERVICE_NAME> -n 100 --no-pager

```

Check service configuration:

```
sudo systemctl cat <SERVICE_NAME>

```

**Possible causes**

* Incorrect service configuration.

* Missing executable or configuration files.

* Incorrect permissions.

* Port conflicts.

* Dependency failures.

* Application startup errors.

**Resolution**

1. Inspect the service status.

2. Review the journal logs.

3. Identify the root cause.

4. Correct the configuration or dependency.

5. Validate the configuration where applicable.

6. Restart the affected service when safe.

7. Verify that the service remains active.

## 16. Network Connectivity Problems

**Symptoms**

* Server cannot reach external services.

* DNS resolution fails.

* Ansible cannot connect to the EC2 instance.

* Monitoring workstation cannot scrape Node Exporter.

* Applications cannot communicate with required endpoints.

**Diagnostic commands**

Check IP addresses:

```
ip address

```

Review routes:

```
ip route

```

Test DNS resolution:

```
getent hosts example.com

```

Test network connectivity:

```
ping -c 4 <DESTINATION_IP>

```

Test a TCP endpoint:

```
nc -vz <HOST> <PORT>

```

**Possible causes**

* Incorrect routing.

* DNS resolution problems.

* AWS Security Group restrictions.

* UFW firewall rules.

* Incorrect destination address.

* Service not listening.

* Network interface problems.

**Resolution**

1. Verify the source and destination addresses.

2. Check routing and DNS resolution.

3. Confirm the destination service is listening.

4. Review AWS Security Group rules.

5. Review UFW rules.

6. Test connectivity again.

Remember that a failed ping does not necessarily mean a service is unavailable. ICMP traffic may be blocked while TCP connectivity still works.

## 17. Troubleshooting Record

Record significant issues using the following template:

| Field | Description | 
| ----- | ----- | 
| Date and time | When the issue occurred | 
| Affected component | Service or infrastructure component | 
| Symptoms | What happened | 
| Diagnostic commands | Commands used during investigation | 
| Root cause | Identified cause | 
| Corrective action | Action taken | 
| Verification | How resolution was confirmed | 
| Lessons learned | What can prevent recurrence | 

Keep records of important changes and recurring issues to improve future operations.

## 18. General Operational Recommendations

* Collect evidence before changing configurations.

* Check service logs before restarting services.

* Validate configurations before applying changes.

* Avoid unnecessary server reboots.

* Preserve existing administrative access during security changes.

* Monitor disk, memory, and CPU utilization.

* Verify backups before relying on them.

* Avoid deleting persistent Docker volumes during troubleshooting.

* Document all significant changes.

* Verify that the service works after every corrective action.

* Protect sensitive information in logs and configuration files.

* Escalate suspected security compromises rather than attempting unverified repairs.

## 19. Conclusion

Effective troubleshooting requires structured investigation, accurate diagnosis, safe corrective actions, and verification.

This guide provides a practical reference for diagnosing common Linux, AWS, networking, security, automation, Docker, and monitoring problems encountered during the Enterprise Linux Server Deployment project.

Consistent troubleshooting practices improve system reliability, reduce unnecessary downtime, and strengthen operational security.

## 20. Author

* **Project:** Enterprise Linux Server Deployment

* **Role:** Linux System Administrator / DevOps Engineer

* **GitHub:** https://github.com/tobilee10