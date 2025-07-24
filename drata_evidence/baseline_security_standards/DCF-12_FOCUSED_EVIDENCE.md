# DCF-12: Baseline Configuration and Hardening Standards - Evidence

**Control**: Baseline Configuration and Hardening Standards  
**Organization**: CadreIQ  
**Upload Date**: July 24, 2025  
**Framework**: SOC 2 Type II  

## Control Requirement
CadreIQ has identified and documented baseline security configuration standards for all system components in accordance with industry-accepted hardening standards or vendor recommendations.

## Evidence Provided

### 1. Industry-Accepted Standards Documentation
**File**: AWS_BASELINE_SECURITY_STANDARDS.md (included in this upload)

**Standards Implemented**:
- **CIS Amazon Web Services Benchmark v1.5.0**
- **NIST 800-53 Security Controls**  
- **AWS Security Best Practices**

**Key Controls Documented**:
- IMDSv2 enforcement (CIS 2.1.1)
- RDS encryption at rest (CIS 2.2.1) 
- VPC Flow Logs (CIS 2.1.3)
- Security group least privilege (CIS 4.1)
- IAM password policies (CIS 1.4)
- TLS encryption standards (NIST SC-8)

### 2. Implementation Method
**Infrastructure as Code**: AWS Copilot with CloudFormation  
**Version Control**: Replit-based Git repository with local change management  
**Deployment**: Direct deployment from development environment to AWS ECS  

All security configurations are defined in AWS Copilot manifests (infra/copilot/) and deployed consistently across environments, ensuring baseline standards are maintained through Infrastructure as Code.

### 3. Periodic Review Process
- **Quarterly**: Security standards documentation review
- **Monthly**: Infrastructure drift detection  
- **Daily**: Automated compliance validation via Drata

## Compliance Statement
This evidence demonstrates CadreIQ's comprehensive baseline security configuration standards based on industry-accepted frameworks (CIS Benchmarks, NIST 800-53) with consistent implementation through Infrastructure as Code.

**Contact**: security@cadreiq.com