---
title: "Automating Security Scans on Amazon EKS"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---


### AUTOMATING SECURITY SCANS ON AMAZON EKS WITH KUBESCAPE, AWS CODEBUILD, AND AWS CODEPIPELINE

![Security Scan Architecture](/images/3-BlogsPosted/3.2-Blog2/blog2.png)

As organizations adopt Amazon EKS to run containerized applications, ensuring security and compliance becomes a critical responsibility. However, performing security checks manually is time-consuming, error-prone, and difficult to scale as Kubernetes environments continue to grow.

By integrating **Kubescape**, **AWS CodeBuild**, and **AWS CodePipeline**, security scanning can be automated as part of the CI/CD workflow, allowing vulnerabilities and configuration issues to be detected before applications are deployed to production.

---

### Security Challenges in Amazon EKS

Some common security challenges when operating Kubernetes clusters on Amazon EKS include:

* Kubernetes clusters may contain insecure or misconfigured settings.
* Workloads may run with excessive permissions, violating the principle of least privilege.
* Manual compliance validation against standards such as CIS Benchmark, NSA, and MITRE is time-consuming.
* Security scanning is often not integrated into the CI/CD pipeline.
* Tracking, managing, and remediating security findings can become difficult as environments grow.

---

### Solution Overview

This solution combines Kubescape with AWS CodeBuild and AWS CodePipeline to automate security scanning for Amazon EKS clusters.

The workflow consists of the following steps:

* A developer or end user triggers the pipeline.
* AWS CodePipeline starts the CI/CD workflow.
* AWS CodeBuild launches the build job.
* CodeBuild installs and executes Kubescape.
* Kubescape scans the Amazon EKS cluster using security frameworks such as CIS Benchmark, NSA, and MITRE.
* Scan reports are stored in Amazon S3.
* DevOps or Security teams review the results and remediate identified vulnerabilities.

---

### AWS Services Used

* **Amazon EKS:** Hosts Kubernetes clusters and containerized applications.
* **Kubescape:** An open-source Kubernetes security and compliance scanning tool.
* **AWS CodePipeline:** Automates the CI/CD workflow.
* **AWS CodeBuild:** Executes build jobs and runs Kubescape scans.
* **Amazon S3:** Stores security scan reports.
* **AWS Identity and Access Management (IAM):** Grants secure permissions between AWS services.
* **Amazon CloudWatch Logs:** Collects logs generated during the build and scanning processes.

---

### Kubescape Scanning Capabilities

Kubescape supports multiple security frameworks for evaluating Kubernetes environments:

* **CIS Benchmark Scan:** Verifies Kubernetes configurations against CIS security best practices.
* **NSA Framework Scan:** Evaluates cluster hardening and security recommendations published by the NSA.
* **MITRE Framework Scan:** Identifies weaknesses related to common attack techniques based on the MITRE ATT&CK framework.
* **Namespace Scan:** Scans a specific Kubernetes namespace instead of the entire cluster.

---

### Benefits of the Solution

Integrating Kubescape into the CI/CD pipeline provides several advantages:

* Automates security scanning for Amazon EKS clusters.
* Reduces manual effort and minimizes configuration oversights.
* Detects vulnerabilities early before production deployment.
* Simplifies compliance validation against CIS Benchmark, NSA, and MITRE standards.
* Centralizes scan reports in Amazon S3 for auditing and analysis.
* Enables deployment to be blocked when critical security issues are detected.
* Supports integration with ticketing systems such as Jira for vulnerability tracking and remediation.

---

### Security Best Practices

To further strengthen Amazon EKS security, consider implementing the following best practices:

* Apply Role-Based Access Control (RBAC) to restrict user permissions.
* Use IAM Roles for Service Accounts (IRSA) following the principle of least privilege.
* Store sensitive credentials securely using AWS Secrets Manager.
* Enable Amazon GuardDuty for continuous threat detection.
* Enforce Pod Security Standards to secure Kubernetes workloads.
* Use Kyverno policies to validate and enforce Kubernetes security policies.

---

### Conclusion

Security should not be treated as a final step after applications reach production. Instead, security scanning should be integrated directly into the CI/CD pipeline to identify risks as early as possible.

The combination of **Kubescape**, **AWS CodeBuild**, and **AWS CodePipeline** provides an effective DevSecOps solution for continuously assessing the security and compliance of Amazon EKS clusters. This approach helps organizations improve their cloud-native security posture, reduce operational risks, and build scalable, automated security workflows.

---

### References

* AWS Blog - Automating Security Scans for Amazon EKS using Kubescape
* Kubescape Documentation - https://kubescape.io/
* AWS CodePipeline Documentation - https://docs.aws.amazon.com/codepipeline/
* AWS CodeBuild Documentation - https://docs.aws.amazon.com/codebuild/
* Amazon EKS Documentation - https://docs.aws.amazon.com/eks/
* [Automating Security Scans on Amazon EKS - Facebook Discussion](https://www.facebook.com/groups/awsstudygroupfcj/posts/2205463036885323)
