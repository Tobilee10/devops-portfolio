# EC2 Server Decommissioning Guide

## 1. Purpose

This guide explains how to safely decommission the Amazon EC2 instance used for the Enterprise Linux Server Deployment project.

Decommissioning is the controlled process of retiring infrastructure resources that are no longer required. It includes preserving important data, removing unused resources, verifying cleanup, and reviewing any remaining costs.

The objectives are to:

- Prevent unnecessary AWS charges.
- Safely remove unused cloud infrastructure.
- Preserve important project documentation, configurations, and evidence.
- Avoid accidentally deleting shared resources.
- Verify that no unnecessary billable resources remain.
- Maintain an accurate record of infrastructure changes.

> **Important:** Do not decommission the server until deployment, testing, documentation, and evidence collection are complete. Confirm that no other project or service depends on the resources being removed.

## 2. Scope

This guide covers the EC2 infrastructure and any additional AWS resources created exclusively for this project.

Potential resources include:

- EC2 instance.
- EBS root volume and additional EBS volumes.
- Elastic IP addresses, if allocated.
- EBS snapshots, if created.
- Project-specific security groups.
- Project-specific AWS key pair.
- CloudWatch alarms and log groups, if created.
- Other project-specific networking or supporting resources, if applicable.

This guide does not authorize the deletion of shared AWS resources, resources belonging to other projects, or the local monitoring environment.

## 3. Project Infrastructure Overview

The project uses the following infrastructure arrangement:

| Component | Location | Purpose |
| --- | --- | --- |
| EC2 Ubuntu server | AWS | Managed Linux server |
| Node Exporter | EC2 server | Exposes server metrics |
| Prometheus | Local workstation | Collects and evaluates metrics |
| Grafana | Local workstation | Visualizes metrics |
| Alertmanager | Local workstation | Routes monitoring alerts |

Prometheus, Grafana, and Alertmanager run locally through Docker Compose. Node Exporter runs as a systemd service on the EC2 server.

**Monitoring decommissioning consideration:** Removing the EC2 server does not mean the local monitoring stack must also be deleted. Preserve the monitoring configuration and dashboards as project evidence unless you separately intend to retire the local monitoring environment.

## 4. Pre-Decommissioning Checklist

Complete the following checks before removing any infrastructure:

- [ ] Deployment tasks are complete.
- [ ] All relevant tests and verification checks have passed.
- [ ] Project documentation is updated.
- [ ] Screenshots and command outputs have been saved locally.
- [ ] Important configuration files have been backed up.
- [ ] No other project depends on the EC2 instance.
- [ ] No important application data remains on the server.
- [ ] The EC2 instance ID has been recorded.
- [ ] The AWS region has been confirmed.
- [ ] Related EBS volume, security group, and key pair IDs or names have been recorded.
- [ ] Any required backup or recovery copy has been verified.
- [ ] The decision to terminate the instance has been approved by the resource owner.

Do not proceed if resource ownership or data-retention requirements are unclear.

## 5. Preserve Project Evidence

Before terminating the EC2 instance, preserve evidence that demonstrates the implementation and operational state of the project.

Recommended evidence includes:

**Server information**

- Operating system and kernel version.
- CPU, memory, and disk information.
- Network configuration.
- Running services and system uptime.

**Security configuration**

- SSH hardening configuration.
- Firewall status and rules.
- Fail2ban status.
- User and group configuration.
- File permissions and ownership.

**Automation and operations**

- Ansible execution results.
- Bash automation results.
- Backup verification results.
- Scheduled task configuration.
- Relevant system logs.

**Monitoring**

- Node Exporter service status.
- Prometheus target health.
- Prometheus configuration validation.
- Grafana dashboard screenshots.
- Alert rule validation results.
- Alertmanager health and notification test results.

Store the evidence in the project's `screenshots/` directory or another documented evidence directory.

Preserve relevant configuration files, including:

- Prometheus configuration.
- Prometheus alert rules.
- Grafana dashboard exports.
- Alertmanager configuration.
- Docker Compose configuration.
- Ansible playbooks and inventory examples.
- Bash automation scripts.

**Security requirements:**

- Never upload private SSH keys.
- Never commit passwords, access tokens, SMTP credentials, or other secrets.
- Do not preserve sensitive production data in a public repository.
- Review configuration files for exposed IP addresses, credentials, and other sensitive information before publishing them.

## 6. Identify Project Resources

Review the AWS Management Console and identify every resource created exclusively for this project.

Record the relevant information:

| Resource | Details |
| --- | --- |
| EC2 instance name | `enterprise-linux-server` |
| EC2 instance ID | To be recorded |
| AWS region | Verify actual deployment region |
| EBS volume IDs | To be recorded |
| Security group ID | To be recorded |
| Key pair name | `enterprise-linux-server-key` |
| Elastic IP | If allocated |
| Snapshot IDs | If created |
| Other project resources | If applicable |

The AWS region must be verified against the actual deployment. Do not assume that the region listed in an old configuration file is still correct.

Use resource tags, names, IDs, and AWS resource relationships to establish ownership.

Never delete a resource based on its name alone.

## 7. Remove the EC2 Instance from Monitoring

Before terminating the server, update the monitoring configuration so that the local monitoring environment no longer expects the EC2 instance to be available.

### 7.1 Update Prometheus configuration

Locate the Prometheus configuration file:

`monitoring/prometheus/prometheus.yml`

Find the EC2 target under the `enterprise-linux-server` scrape job.

Remove the EC2 target or disable the scrape job if the server is no longer being monitored.

For example, remove the project-specific target:

```yaml
- job_name: "enterprise-linux-server"
  static_configs:
    - targets:
        - "YOUR_EC2_PUBLIC_IP:9100"
```

Preserve other scrape jobs, especially the Prometheus self-monitoring job.

### 7.2 Validate the configuration

Run:

```bash
docker compose -f monitoring/docker-compose.yml exec prometheus \
  promtool check config /etc/prometheus/prometheus.yml
```

The configuration should pass validation before reloading or restarting Prometheus.

Reload Prometheus using the method configured for the project. If a reload endpoint is enabled, use:

```bash
curl -X POST http://localhost:9090/-/reload
```

If reload is not enabled, restart the Prometheus service through Docker Compose:

```bash
docker compose -f monitoring/docker-compose.yml restart prometheus
```

### 7.3 Review project-specific alert rules

Review alert rules that depend on the EC2 server, including:

- `NodeExporterDown`
- `PrometheusTargetMissing`
- CPU, memory, disk, and network alerts for the EC2 target.

Remove or disable rules that are no longer relevant to the retired server. Preserve rules that monitor other targets or the local monitoring environment.

Validate the remaining rules:

```bash
docker compose -f monitoring/docker-compose.yml exec prometheus \
  promtool check rules /etc/prometheus/alerts/*.yml
```

### 7.4 Preserve the local monitoring stack

Do not delete the following simply because the EC2 instance is being retired:

- Prometheus container and configuration.
- Grafana container and dashboards.
- Alertmanager container and configuration.
- Docker Compose file.
- Persistent monitoring volumes.
- Local monitoring documentation.

These components are part of the project's implementation and evidence.

## 8. Terminate the EC2 Instance

After confirming that evidence has been preserved and monitoring has been updated, proceed with instance termination.

### 8.1 Using the AWS Management Console

1. Sign in to the AWS Management Console.
2. Open the EC2 service.
3. Select the verified AWS region.
4. Navigate to **Instances**.
5. Locate the instance named `enterprise-linux-server`.
6. Confirm its instance ID.
7. Confirm that it belongs exclusively to this project.
8. Select the instance.
9. Choose **Instance state -> Terminate instance**.
10. Review the termination confirmation.
11. Confirm termination only after verifying the instance identity.

> **Warning:** EC2 termination is an irreversible infrastructure action. Do not terminate an instance containing data or services that are still required.

### 8.2 Verify termination

Return to the EC2 Instances page and confirm that the instance transitions to the `terminated` state.

Record the termination date and the instance ID in the project decommissioning record.

## 9. Verify EBS Volume Cleanup

EC2 termination does not guarantee that every attached EBS volume has been deleted.

The outcome depends on the volume's `DeleteOnTermination` setting.

### 9.1 Review volume status

1. Navigate to **EC2 -> Volumes**.
2. Identify the volumes previously attached to the project instance.
3. Review their volume IDs and attachment information.
4. Check whether they were automatically deleted.
5. Identify any remaining volumes.

### 9.2 Review retained volumes

If a project-specific volume remains:

1. Confirm that it belongs exclusively to this project.
2. Verify that no required data remains.
3. Confirm that no backup or retention requirement exists.
4. Delete it only after receiving the necessary approval.

Retained EBS volumes can continue to incur storage charges even after the EC2 instance is terminated.

Do not delete volumes used by other instances or applications.

## 10. Release Elastic IP Addresses

If an Elastic IP address was allocated specifically for this project, verify whether it is still required.

Steps:

1. Navigate to **EC2 -> Elastic IP addresses**.
2. Identify the project-specific address.
3. Confirm its allocation and association details.
4. Verify that it is no longer required by another resource.
5. Disassociate it if necessary.
6. Release it only when ownership and non-use have been confirmed.

Do not release an Elastic IP address used by another application or environment.

An allocated Elastic IP can incur charges depending on its allocation and usage status. Review current AWS pricing when assessing remaining costs.

## 11. Delete Unnecessary EBS Snapshots

If EBS snapshots were created for this project:

1. Navigate to **EC2 -> Snapshots**.
2. Identify snapshots associated with the project.
3. Confirm snapshot ownership and creation purpose.
4. Check whether they are required for recovery or retention.
5. Review whether AWS Backup or a lifecycle policy manages them.
6. Delete only snapshots that are no longer required and are safe to remove.

Snapshots can continue to incur storage charges after an EC2 instance has been terminated.

Do not delete snapshots that are subject to retention requirements or required by another recovery process.

## 12. Remove Project-Specific Security Groups

After terminating the instance and identifying dependent resources:

1. Navigate to **EC2 -> Security Groups**.
2. Identify security groups created exclusively for this project.
3. Confirm that they are not associated with active network interfaces or other resources.
4. Check whether other security groups reference them.
5. Delete only unused, project-specific security groups.

AWS may prevent deletion when a security group is still associated with a network interface or referenced by another security group.

Do not delete default or shared security groups.

## 13. Remove the Project Key Pair

If `enterprise-linux-server-key` was created exclusively for this project and is no longer needed:

1. Navigate to **EC2 -> Key Pairs**.
2. Locate the project key pair.
3. Confirm that it is not needed by another instance or deployment.
4. Delete the key pair from AWS.

**Important security distinction:**

Deleting an EC2 key pair from AWS does not automatically delete private key files already downloaded to a local computer.

If the private key is no longer required, securely remove the local copy after confirming that it is not needed for another environment.

If a private key has been exposed or compromised, deleting the AWS key pair alone is not sufficient. Replace the affected access credentials and update authorized access.

Never upload private SSH keys to GitHub.

## 14. Review Other Project-Specific AWS Resources

Review any additional AWS resources created specifically for the project.

Depending on the actual implementation, these may include:

- CloudWatch alarms.
- CloudWatch log groups.
- CloudWatch dashboards.
- Network interfaces.
- Load balancers and target groups.
- NAT gateways.
- Additional EBS resources.
- Automated backup resources.
- Other supporting infrastructure.

Only remove resources that were created for this project, are no longer required, and have no dependencies.

Do not delete resources that were not created by this project or are shared with another environment.

## 15. Final AWS Resource Audit

After cleanup, review the AWS Management Console to verify the final infrastructure state.

Check the following:

| Resource | Expected outcome |
| --- | --- |
| EC2 instance | Terminated |
| EBS root volume | Deleted if configured for deletion and no longer required |
| Additional EBS volumes | Deleted only if project-specific and unnecessary |
| Elastic IP | Released if allocated exclusively for this project |
| EBS snapshots | Deleted only if no longer required |
| Security groups | Unused project-specific groups removed |
| Key pair | Removed from AWS if no longer needed |
| Other resources | Reviewed and cleaned up where applicable |

Review AWS Billing and Cost Management to identify any remaining project-related charges.

Remember:

- Billing information may take time to reflect recent resource changes.
- Some resources may continue to incur charges after EC2 termination.
- Costs may be associated with resources outside the EC2 service.
- Review the applicable AWS region and billing period.

Do not assume that terminating an instance automatically stops all AWS charges.

## 16. Decommissioning Verification Checklist

Complete this checklist after the cleanup process.

**Infrastructure**

- [ ] EC2 instance is terminated.
- [ ] Unneeded EBS volumes are deleted.
- [ ] Unused Elastic IP addresses are released.
- [ ] Unneeded snapshots are deleted.
- [ ] Unused project-specific security groups are removed.
- [ ] Unneeded AWS key pairs are removed.
- [ ] Other project-specific resources are reviewed.
- [ ] No shared resources were accidentally deleted.

**Monitoring**

- [ ] EC2 target has been removed from the Prometheus scrape configuration.
- [ ] Project-specific alert rules have been reviewed.
- [ ] Prometheus configuration has passed validation.
- [ ] Local monitoring configuration has been preserved.
- [ ] Grafana dashboards and monitoring evidence have been preserved.

**Documentation and security**

- [ ] Project documentation is preserved.
- [ ] Screenshots and command outputs are saved locally.
- [ ] No secrets or private keys have been committed to GitHub.
- [ ] Local private key files have been reviewed and securely handled.
- [ ] Decommissioning date and resource identifiers are recorded.

**Cost verification**

- [ ] AWS Billing and Cost Management has been reviewed.
- [ ] Remaining resources have been investigated.
- [ ] No unnecessary project resources are knowingly left running.

## 17. Decommissioning Record

Maintain a record of the decommissioning activity.

| Field | Record |
| --- | --- |
| Project | Enterprise Linux Server Deployment |
| EC2 instance name | `enterprise-linux-server` |
| EC2 instance ID | To be recorded |
| AWS region | To be verified |
| Decommissioning date | To be recorded |
| Instance termination status | To be verified |
| Remaining resources | To be recorded |
| Billing review status | To be recorded |
| Evidence location | Project evidence directory |

This record provides an audit trail and helps demonstrate responsible infrastructure lifecycle management.

## 18. Lessons Learned

Decommissioning is an important part of infrastructure lifecycle management.

Key lessons include:

- Cloud resources should be removed when they are no longer required.
- Infrastructure ownership must be established before deleting resources.
- Data and evidence must be preserved before terminating infrastructure.
- EBS volumes and snapshots may continue to incur charges after instance termination.
- Elastic IP addresses and other supporting resources require independent review.
- Monitoring configurations should be updated when monitored infrastructure is retired.
- Resource cleanup must be verified rather than assumed.
- Cost management is part of responsible cloud administration.

## 19. Future Improvements

Possible improvements to the decommissioning process include:

- Automating infrastructure cleanup using Terraform.
- Applying consistent resource tags for ownership and cost tracking.
- Using AWS Config to monitor resource configuration and compliance.
- Creating automated cost alerts.
- Maintaining an infrastructure inventory.
- Introducing formal approval and change-management procedures.
- Using infrastructure-as-code plans to review proposed deletions before execution.

Automated cleanup should include safeguards against deleting shared or production resources.

## 20. Conclusion

A successful infrastructure deployment does not end when a server becomes operational.

Responsible infrastructure management also includes maintenance, monitoring, backup, incident response, and safe decommissioning.

By following this process, unnecessary cloud expenses can be reduced, important information can be preserved, and infrastructure resources can be retired in a controlled and verifiable manner.

## 21. Author

- **Project:** Enterprise Linux Server Deployment
- **Role:** Linux System Administrator / DevOps Engineer
- **GitHub:** https://github.com/tobilee10