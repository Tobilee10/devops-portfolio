# Enterprise Linux Server Deployment – Incident Response Plan

## 1. Introduction

This document defines the procedures for identifying, investigating, containing, eradicating, recovering from, and documenting security and operational incidents affecting the Enterprise Linux Server Deployment environment.

The objective is to minimize service disruption, protect system resources, preserve relevant evidence, restore normal operations safely, and reduce the likelihood of recurring incidents.

- **Project:** Enterprise Linux Server Deployment
- **Document:** Incident Response Plan
- **Status:** Prepared – Implementation and operational validation pending

> **Important:** This document defines the intended incident response procedures. The presence of a procedure in this document does not automatically mean that the corresponding response capability has been implemented or tested.

## 2. Objectives

The incident response process aims to:

- Detect security and operational incidents.
- Identify affected systems, services, and resources.
- Assess the severity and potential impact of incidents.
- Limit the impact and prevent further damage.
- Preserve relevant logs and investigation evidence.
- Restore normal server operations safely.
- Identify root causes.
- Prevent similar incidents from recurring.
- Maintain an auditable record of incident activities.

## 3. Scope

This plan covers incidents affecting the project's Linux server and its supporting monitoring environment.

The scope includes:

- Ubuntu Linux operating system.
- SSH access and authentication.
- UFW firewall.
- Fail2ban intrusion prevention.
- Nginx web server, where installed.
- Bash automation scripts.
- Ansible-managed configuration.
- Backup operations.
- Node Exporter.
- Prometheus, Grafana, and Alertmanager monitoring components.
- Relevant AWS EC2 infrastructure and security groups.

The plan does not replace a formal enterprise security incident response policy or an organization's legal, regulatory, and compliance obligations.

## 4. Incident Types and Severity

The following incidents are within the project scope.

| Incident | Description | Initial severity |
| --- | --- | --- |
| SSH brute-force attempts | Repeated unauthorized authentication attempts | High |
| Unauthorized access | Confirmed or suspected unauthorized account access | Critical |
| High CPU utilization | Unexpected or sustained resource consumption | Medium |
| Disk exhaustion | Storage approaches capacity or becomes unavailable | High |
| Nginx outage | Web service becomes unavailable | High |
| Fail2ban failure | Intrusion prevention service stops working | High |
| Malware suspicion | Suspicious processes, files, or system activity | Critical |
| Backup failure | Expected backup is missing or incomplete | High |
| Monitoring failure | Monitoring data or alert delivery becomes unavailable | Medium |
| Prometheus target failure | Managed server metrics are no longer collected | Medium |

Severity should be reassessed according to actual impact, exposure, duration, affected users, and the possibility of data compromise.

### 4.1 Severity definitions

**Critical**

An incident involving suspected system compromise, unauthorized privileged access, malware, or a serious security breach.

**High**

An incident causing significant service disruption, loss of important functionality, or a serious reduction in security controls.

**Medium**

An incident affecting system performance, monitoring, or a non-critical service without evidence of major compromise.

**Low**

A minor issue with limited impact that does not significantly affect service availability or security.

## 5. Incident Response Lifecycle

The project follows six incident response stages:

1. Preparation.
2. Identification.
3. Containment.
4. Eradication.
5. Recovery.
6. Lessons learned.

Each stage should be documented, and the response should be adjusted according to the incident's severity and impact.

## 6. Preparation

Preparation establishes the administrative, technical, and operational capabilities needed to respond to incidents.

Recommended preparation activities include:

- Maintain approved administrative access methods.
- Keep the operating system and installed packages updated.
- Apply SSH hardening.
- Configure UFW firewall rules.
- Install and configure Fail2ban.
- Monitor system and authentication logs.
- Maintain verified backups.
- Document important services and configurations.
- Maintain an inventory of AWS resources.
- Preserve copies of important configuration files.
- Ensure monitoring and alerting configurations are documented.
- Know how to access the server through AWS Systems Manager when it has been configured and tested.

**Useful commands:**

```bash
sudo systemctl --failed
sudo ufw status verbose
sudo fail2ban-client status
df -h
free -h
uptime
```

These commands provide an initial view of service failures, firewall status, intrusion prevention, storage, memory, and system load.

## 7. Incident Identification

When suspicious activity or service disruption is detected, collect information before making changes.

### 7.1 Review system logs

```bash
sudo journalctl -p warning -n 100 --no-pager
```

Review recent warning and higher-priority system events.

### 7.2 Review SSH events

On Ubuntu systems, the SSH service is commonly named `ssh`:

```bash
sudo journalctl -u ssh -n 100 --no-pager
```

Depending on the logging configuration, authentication events may also be available in:

```bash
sudo tail -n 100 /var/log/auth.log
```

### 7.3 Review Fail2ban events

```bash
sudo journalctl -u fail2ban -n 100 --no-pager
```

Check the service and jail status:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### 7.4 Inspect active processes

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

These commands help identify processes consuming unusually high CPU or memory.

### 7.5 Check disk and memory

```bash
df -h
free -h
```

### 7.6 Record initial observations

Document the following:

- Incident identification number.
- Date and time of detection, including timezone.
- Affected server or service.
- Observed symptoms.
- Relevant logs and command outputs.
- Services affected.
- Potential business or operational impact.
- Detection method.
- Actions already taken.

Avoid changing system configurations before collecting relevant initial evidence, unless immediate action is necessary to prevent further harm.

## 8. Incident Containment

Containment aims to prevent an incident from spreading or causing further damage.

Possible containment actions include:

- Restricting suspicious network access through AWS Security Groups.
- Block confirmed malicious sources where appropriate.
- Disable compromised accounts.
- Revoke or replace compromised credentials.
- Isolate affected services.
- Restrict access to sensitive files.
- Preserve relevant logs and evidence.
- Limit access to affected infrastructure.

### 8.1 Suspected EC2 compromise

For a suspected compromised EC2 instance:

1. Assess the severity and potential impact.
2. Preserve available logs and relevant evidence.
3. Identify affected network access paths.
4. Consider isolating the instance using a controlled AWS Security Group change.
5. Preserve an authorized investigation or recovery path.
6. Notify the appropriate system owner or security contact.

Before changing security groups, understand their existing rules and dependencies.

Do not terminate the instance, delete volumes, or destroy evidence before the investigation and recovery requirements have been considered.

For serious suspected compromise, rebuilding from a trusted image may be safer than attempting to trust the existing operating system.

## 9. Incident Eradication

After containing the incident, identify and remove the cause.

Activities may include:

- Identify the root cause.
- Remove unauthorized accounts or access.
- Remove malicious files after preserving required evidence.
- Correct vulnerable configurations.
- Rotate compromised credentials and keys.
- Apply required security updates.
- Review SSH, UFW, and Fail2ban configurations.
- Review user privileges and authorized SSH keys.
- Verify that unauthorized access paths are closed.
- Investigate whether the incident affected other resources.

Do not assume that removing one suspicious file or blocking one IP address fully resolves a security compromise.

If system integrity cannot be established, consider rebuilding the instance from a trusted image and restoring verified data.

## 10. Incident Recovery

Recovery aims to restore normal operations safely.

Possible recovery activities include:

- Restore verified backups where necessary.
- Rebuild compromised systems where appropriate.
- Correct system configurations.
- Restart affected services.
- Verify user access and permissions.
- Test application availability.
- Confirm firewall and security controls.
- Verify monitoring and alert delivery.
- Monitor the system for recurring symptoms.

### 10.1 Service verification

Check relevant services:

```bash
sudo systemctl status nginx --no-pager
sudo systemctl status ssh --no-pager
sudo systemctl status fail2ban --no-pager
```

Run only the checks relevant to services installed on the server.

### 10.2 System health verification

```bash
uptime
free -h
df -h
sudo systemctl --failed
```

### 10.3 Application verification

If Nginx is installed and configured:

```bash
sudo nginx -t
curl -I http://localhost
```

Use the appropriate application endpoint if the service is not exposed through Nginx.

### 10.4 Monitoring verification

For this project:

- Confirm Node Exporter is active on the EC2 server.
- Confirm Prometheus can scrape the server target.
- Confirm Grafana can query Prometheus.
- Confirm Prometheus alert rules are loaded.
- Confirm Alertmanager is healthy.
- Verify notification delivery where applicable.

Recovery is complete only after the affected service and its relevant security controls have been verified.

## 11. Incident Scenarios and Response Procedures

### 11.1 SSH Brute-Force Attempts

**Symptoms**

- Repeated authentication failures.
- Suspicious login attempts.
- Unexpected IP addresses attempting SSH access.
- Increased Fail2ban activity.

**Investigation**

```bash
sudo journalctl -u ssh -n 100 --no-pager
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

**Response**

1. Identify suspicious source IP addresses.
2. Review authentication logs and Fail2ban activity.
3. Verify AWS Security Group restrictions.
4. Review SSH authentication settings.
5. Restrict confirmed malicious sources where appropriate.
6. Check for successful unauthorized logins.
7. Continue monitoring authentication events.

Repeated failed login attempts alone do not prove that an account has been compromised.

### 11.2 Suspected Unauthorized Access

**Symptoms**

- Unknown user accounts.
- Unexpected sudo activity.
- Unrecognized SSH sessions.
- Unusual file or configuration changes.
- Unexpected privileged processes.

**Investigation**

```bash
who
w
last
getent group sudo
sudo journalctl --since "1 hour ago" --no-pager
```

Review authorized SSH keys and account configuration where appropriate.

**Response**

1. Preserve relevant evidence.
2. Identify affected accounts and access paths.
3. Contain unauthorized access.
4. Review SSH keys and account permissions.
5. Rotate potentially compromised credentials.
6. Investigate the root cause.
7. Assess whether other services or resources were affected.
8. Restore the server from a trusted state if necessary.

### 11.3 Nginx Service Outage

**Symptoms**

- Website unavailable.
- HTTP errors.
- Nginx service stopped.
- Application requests failing.

**Investigation**

```bash
sudo systemctl status nginx --no-pager
sudo nginx -t
sudo journalctl -u nginx -n 100 --no-pager
```

**Response**

1. Identify the cause of the outage.
2. Check Nginx configuration and file permissions.
3. Verify network access and firewall rules.
4. Check upstream application availability.
5. Correct configuration errors.
6. Restart or reload Nginx when safe.
7. Test service availability.
8. Monitor for recurring errors.

### 11.4 Disk Space Exhaustion

**Symptoms**

- Files cannot be written.
- Services fail.
- Disk utilization reaches a critical threshold.
- Applications report insufficient storage.

**Investigation**

```bash
df -h
df -i
sudo du -xhd1 /var 2>/dev/null | sort -h
```

**Response**

1. Identify the directories consuming storage.
2. Determine whether logs, backups, or application files are responsible.
3. Preserve required data.
4. Safely remove unnecessary files.
5. Review log rotation and backup retention.
6. Investigate unusually large or rapidly growing files.
7. Verify available disk space and inode availability.
8. Confirm affected services are operational.

Do not blindly delete files from `/var`, application directories, or backup locations.

### 11.5 Fail2ban Service Failure

**Symptoms**

- Fail2ban becomes inactive.
- SSH jail is unavailable.
- Repeated login attempts are not being blocked as expected.

**Investigation**

```bash
sudo systemctl status fail2ban --no-pager
sudo fail2ban-client status
sudo journalctl -u fail2ban -n 100 --no-pager
```

**Response**

1. Identify the service failure.
2. Review jail configuration.
3. Validate the logging backend.
4. Correct configuration errors.
5. Restart Fail2ban when safe.
6. Confirm the service is active.
7. Confirm the SSH jail is active.
8. Verify that the expected protection is working.

### 11.6 Backup Failure

**Symptoms**

- Backup script exits unsuccessfully.
- Expected backup files are missing.
- Backup files are incomplete.
- Backup destination is unavailable.

**Investigation**

Review:

- Script output and exit codes.
- File and directory permissions.
- Available disk space.
- Backup destination availability.
- Backup scheduling and execution logs.
- Integrity of existing backup files.

**Response**

1. Identify the failure.
2. Verify the source data.
3. Correct permissions or storage issues.
4. Confirm the backup destination is available.
5. Run the backup again.
6. Verify the resulting backup.
7. Perform a recovery test where appropriate.
8. Document the result.

A successful backup command does not automatically prove that the backup can be restored.

### 11.7 Monitoring Failure

**Symptoms**

- Prometheus target becomes unavailable.
- Node Exporter stops responding.
- Grafana dashboards show missing data.
- Alertmanager stops processing notifications.
- Monitoring alerts are not delivered.

**Investigation**

On the EC2 server:

```bash
sudo systemctl status node_exporter --no-pager
sudo journalctl -u node_exporter -n 100 --no-pager
```

On the local monitoring workstation:

```bash
docker compose -f monitoring/docker-compose.yml ps
```

Check the component health endpoints:

```bash
curl -f http://localhost:9090/-/healthy
curl -f http://localhost:9093/-/healthy
curl -f http://localhost:3000/api/health
```

**Response**

1. Identify the affected monitoring component.
2. Check container and systemd service status.
3. Review application logs.
4. Verify network connectivity and port restrictions.
5. Validate Prometheus configuration and alert rules.
6. Restore the affected component.
7. Confirm target health and dashboard data.
8. Verify alert notification delivery.

Remember that the local monitoring workstation must be running and connected for this project's monitoring stack to collect and process metrics continuously.

## 12. Evidence Preservation

During an incident, preserve relevant evidence where practical.

Examples include:

- Authentication logs.
- System journal entries.
- Fail2ban events.
- Process information.
- Network configuration.
- Relevant configuration files.
- AWS instance and security group information.
- Backup execution records.
- Monitoring and alerting records.
- Screenshots and command outputs.

### 12.1 Evidence handling

- Record the date and time of collection.
- Record the affected system.
- Preserve original evidence where possible.
- Restrict access to sensitive evidence.
- Store copies in an approved secure location.
- Avoid modifying original logs unnecessarily.
- Maintain a record of evidence collection and transfer.

Do not publish private IP information, usernames, authentication details, sensitive logs, or credentials in the public repository.

## 13. Incident Documentation Template

Use the following template to record each incident.

| Field | Details |
| --- | --- |
| Incident ID | Unique incident reference |
| Date and time | When the incident occurred |
| Affected component | Server, service, or resource |
| Incident category | Security or operational |
| Severity | Critical, High, Medium, or Low |
| Description | What happened |
| Detection method | How it was discovered |
| Impact | Services, users, and resources affected |
| Root cause | Identified cause |
| Containment action | Action taken to limit impact |
| Eradication action | Action taken to remove the cause |
| Recovery action | Action taken to restore service |
| Verification | How recovery was confirmed |
| Evidence location | Secure evidence storage location |
| Lessons learned | Preventive improvements |
| Incident status | Open, Investigating, Resolved, or Closed |

## 14. Escalation and Communication

Incidents beyond the scope of the project should be escalated to an experienced administrator, system owner, or security professional.

During escalation:

- Preserve available evidence.
- Avoid unnecessary destructive changes.
- Communicate the incident's severity and impact.
- Explain the investigation and containment actions already taken.
- Identify unresolved risks.
- Maintain a record of decisions and recovery activities.

For a suspected serious security compromise, prioritize containment and appropriate escalation over attempting an immediate unverified repair.

## 15. Post-Incident Review

After resolving an incident:

1. Review the incident timeline.
2. Identify and document the root cause.
3. Evaluate the effectiveness of the response.
4. Identify missing or ineffective security controls.
5. Review whether evidence was preserved adequately.
6. Update relevant documentation.
7. Implement preventive improvements.
8. Record lessons learned.
9. Verify that corrective actions are completed.

The purpose is continuous improvement, not simply restoring service.

## 16. Incident Response Readiness Checklist

Use this checklist to assess preparedness.

**Access and security**

- [ ] Administrative access methods are documented.
- [ ] SSH hardening is configured and verified.
- [ ] UFW rules are documented.
- [ ] Fail2ban is installed and its SSH jail is verified.
- [ ] AWS Security Group rules are documented.

**Operations and recovery**

- [ ] Important services are documented.
- [ ] Backup procedures are documented.
- [ ] Backup restoration has been tested.
- [ ] System logs are accessible.
- [ ] Recovery procedures are documented.

**Monitoring**

- [ ] Node Exporter status can be checked.
- [ ] Prometheus target health can be checked.
- [ ] Grafana dashboards are available.
- [ ] Alertmanager health can be checked.
- [ ] Notification delivery has been tested.

**Documentation**

- [ ] Incident response procedures are available.
- [ ] Incident documentation template is available.
- [ ] Evidence storage location is documented.
- [ ] Escalation contacts or procedures are documented.

## 17. Lessons Learned

A structured incident response process improves the reliability and security of Linux infrastructure.

Key lessons include:

- Early detection reduces the potential impact of incidents.
- Logs and monitoring data are essential for investigation.
- Containment should prevent further damage without unnecessarily destroying evidence.
- Recovery should be based on verified configurations and trusted backups.
- Security controls must be checked after recovery.
- Incident documentation supports accountability and continuous improvement.
- Regular testing helps identify weaknesses in response procedures.

## 18. Future Improvements

Possible improvements include:

- Automating incident detection and response where appropriate.
- Integrating monitoring alerts with an incident management platform.
- Introducing centralized log management.
- Implementing automated backup integrity checks.
- Testing backup restoration regularly.
- Introducing formal incident severity and escalation policies.
- Performing periodic security reviews.
- Conducting incident response simulation exercises.
- Integrating AWS Systems Manager for controlled administrative access where configured.

Maintaining a formal incident register.

Automated response actions should be carefully tested to avoid accidental service disruption or destructive changes.

## 19. Conclusion

A structured incident response process helps administrators respond to security threats and operational disruptions in a controlled and repeatable manner.

This plan establishes a practical foundation for incident detection, containment, investigation, recovery, and continuous improvement within the Enterprise Linux Server Deployment project.

Effective incident response is not only about restoring services. It also involves protecting data, preserving evidence, understanding root causes, and strengthening the infrastructure against future incidents.

## 20. Author

- **Project:** Enterprise Linux Server Deployment
- **Role:** Linux System Administrator / DevOps Engineer
- **GitHub:** https://github.com/tobilee10