---
layout: post
---

This is a rough checklist of the things to look out when securing an infrastructure. It is by no means a definitive guide and is meant to serve as a checklist for interview prep. Some would call this a 1000 ft view of the security of an infratructure. Additions to this checklist are welcome (please use the comments section).

_Disclaimer: I wrote this back when I was just starting out in the industry. Please point out stuff that you feel isn't relevant given today's security landscape._


### Network
1. Segment the network based on specific use cases/teams/applications/permissions.
2. Establish L4 and L7 layer firewalls.
3. Enable L4 DDoS protection.
4. Add appropriate rules to filter the traffic.
5. Add HTTP load balancing.
6. Utilize Shared VPC or VPC peering for better segmentation.
7. Protect Public DNS. Explore DNSSEC.
8. Avoid creating massive subnets.
9. Set up VPN access to cloud / private networks.
10. Set up and monitor internet requests through a NAT gateeway.
11. Restrict and monitor cross-environment communication.
12. Make sure TLS >= 1.2 is deployed and correctly used across HTTP services.


### Identity
1. Bind roles to groups.
2. Follow the principle of least privilege.
3. Avoid using wildcards to assign permissions.
4. Set up a managed identities system like AD or AWS IAM. Do not manually manage identities.

### Data Protection
1. Protect data using encryption. Avoid broken or deprecated ciphers and libraries.
2. Control the access and usage of data via ACLs/Permissions/etc.
3. Prevent and track data exfiltration. Deploy tools or VPC controls to track this.
4. Deploy tools and measure that indetify and redact PII.
5. Enforce the use of tools such as secrets manager (or others) to protect secrets.
6. Make sure your code/secrets isn't vulnerable to Google dorking.
7. Create a secure data lifecyle policy (Storage -> Retention -> Versioning -> Deletion).

### Security Operations / Vulnerability Management
1. Deploy software scans for vulnerabilities.
2. Setup appropriate logging, monitoring, and alerting mechanisms.
3. Promote secure by default practices such a secure terraform modules, etc.
4. Allow scanning or other security packages in CI/CD pipelines.
5. Enable static analysis of code and IaC.
6. Promote secure package and container image deployments via managed artifactories.
7. Set up a standard suite of security integration tests.
8. Enforce hardened OS images and secure boot.
9. Create a mechanism to review audit logs.

### Compliance
1. Verify authN and authZ practices and access control.
2. Review key management practices.
3. Monitoring of compliance violations.
4. Monitoring of network, identity, permissions, etc violations.
