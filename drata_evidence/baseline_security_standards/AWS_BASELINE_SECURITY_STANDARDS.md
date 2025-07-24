# CadreIQ AWS Baseline Security Configuration and Hardening Standards

**Document Version**: 1.0  
**Last Updated**: July 24, 2025  
**Compliance Framework**: CIS Amazon Web Services Benchmark v1.5.0, NIST 800-53  
**Environment**: AWS Production Infrastructure  

## Executive Summary

This document defines CadreIQ's baseline security configuration and hardening standards for AWS infrastructure, implementing industry-accepted security controls based on CIS Benchmarks and NIST Security Configuration Guidelines. All configurations are enforced through Infrastructure as Code (AWS Copilot) and automated compliance monitoring.

## 1. Compute Security Hardening (EC2/ECS)

### 1.1 Instance Metadata Service (IMDS) Security
**CIS Benchmark Reference**: 2.1.1 - Ensure the Instance Metadata Service (IMDS) v2 is enabled and IMDS v1 is disabled

**Standard Configuration**:
```yaml
# AWS Copilot ECS Task Definition
MetadataOptions:
  HttpTokens: required          # Enforces IMDSv2
  HttpPutResponseHopLimit: 1    # Restricts metadata access
  HttpEndpoint: enabled         # Enables metadata service
```

**Implementation Evidence**:
- **Control**: AWS EC2 Instances IMDSv1 Disabled
- **Status**: PASSED (July 24, 2025)
- **Validation**: Drata automated verification via AWS API confirms `HttpTokens: required`
- **Coverage**: All active EC2/ECS instances enforce IMDSv2

### 1.2 Container Security Standards
**NIST 800-53 Reference**: SC-39 - Process Isolation

**Standard Configuration**:
- Container images use minimal base images (Alpine/Distroless)
- Non-root user execution enforced
- Read-only root filesystem where possible
- Resource limits enforced (CPU/Memory)

## 2. Database Security Hardening (RDS)

### 2.1 Encryption at Rest
**CIS Benchmark Reference**: 2.2.1 - Ensure RDS instances have encryption at rest enabled

**Standard Configuration**:
```yaml
# RDS Instance Security
StorageEncrypted: true
KmsKeyId: "alias/aws/rds"      # AWS managed key
BackupRetentionPeriod: 7       # Minimum 7 days
```

**Implementation Evidence**:
- **Database**: cadreiq-dev
- **Encryption**: ✅ Enabled with AWS managed KMS key
- **Backup Retention**: ✅ 7 days configured
- **Public Access**: ✅ Disabled (VPC-only access)

### 2.2 Database Access Controls
**CIS Benchmark Reference**: 2.2.3 - Ensure RDS instances do not have a public endpoint

**Standard Configuration**:
- All databases deployed in private subnets
- PubliclyAccessible: false
- VPC Security Groups restrict access to application tier only
- Auto minor version updates enabled for security patches

## 3. Network Security Hardening (VPC)

### 3.1 VPC Flow Logs
**CIS Benchmark Reference**: 2.1.3 - Ensure VPC Flow Logs are enabled

**Standard Configuration**:
```yaml
# VPC Flow Logs
DeliverLogsPermissionArn: <CloudWatch_Role_ARN>
LogDestination: CloudWatch Logs
LogFormat: AWS Default
TrafficType: ALL
```

**Implementation Evidence**:
- **VPC Flow Logs**: ✅ Enabled on all production VPCs
- **Destination**: CloudWatch Logs for centralized monitoring
- **Coverage**: ALL traffic (accepted, rejected, all)

### 3.2 Network Segmentation
**NIST 800-53 Reference**: SC-7 - Boundary Protection

**Standard Configuration**:
```yaml
# Network Architecture
VPC CIDR: 10.0.0.0/16
Public Subnets: 10.0.1.0/24, 10.0.2.0/24    # Load balancers only
Private Subnets: 10.0.10.0/24, 10.0.11.0/24  # Applications
Database Subnets: 10.0.20.0/24, 10.0.21.0/24 # Databases only
```

**Implementation Evidence**:
- **Multi-tier Architecture**: Public, Private, Database subnet isolation
- **Internet Gateway**: Attached to public subnets only
- **NAT Gateway**: Provides outbound internet for private subnets
- **Route Tables**: Configured for proper traffic segmentation

## 4. Security Group Hardening

### 4.1 Principle of Least Privilege
**CIS Benchmark Reference**: 4.1 - Ensure security groups restrict access

**Standard Configuration**:
```yaml
# Application Load Balancer Security Group
Ingress:
  - Port: 443 (HTTPS)
    Source: 0.0.0.0/0        # Public HTTPS access
  - Port: 80 (HTTP)
    Source: 0.0.0.0/0        # Redirect to HTTPS only

# Application Security Group  
Ingress:
  - Port: 5000
    Source: ALB Security Group # Only ALB can reach application

# Database Security Group
Ingress:
  - Port: 5432
    Source: Application Security Group # Only applications can reach DB
```

**Implementation Evidence**:
- **Load Balancer**: Public HTTPS (443) and HTTP (80) for SSL redirect
- **Application Tier**: Access restricted to load balancer only
- **Database Tier**: Access restricted to application tier only
- **Outbound Rules**: Least privilege principle applied

## 5. Data Protection Standards

### 5.1 Encryption in Transit
**NIST 800-53 Reference**: SC-8 - Transmission Confidentiality

**Standard Configuration**:
- TLS 1.2+ enforced for all external connections
- AWS Certificate Manager (ACM) for SSL/TLS certificates
- HTTP to HTTPS automatic redirection
- Internal service-to-service encryption where applicable

**Implementation Evidence**:
- **SSL Certificate**: AWS ACM certificate for demo.cadreiq.com
- **TLS Version**: Minimum TLS 1.2 enforced on ALB
- **HTTPS Redirect**: HTTP traffic automatically redirected to HTTPS

### 5.2 Encryption at Rest
**NIST 800-53 Reference**: SC-13 - Cryptographic Protection

**Standard Configuration**:
- RDS: AWS managed KMS encryption
- EBS Volumes: Default encryption enabled
- S3 Buckets: Server-side encryption with AES-256
- CloudWatch Logs: Encrypted with service default keys

## 6. Identity and Access Management (IAM)

### 6.1 IAM Hardening Standards
**CIS Benchmark Reference**: 1.4 - IAM password policy

**Standard Configuration**:
```yaml
# IAM Password Policy (SOC 2 Compliant)
MinimumLength: 14
RequireUppercase: true
RequireLowercase: true  
RequireNumbers: true
RequireSymbols: true
PasswordReusePrevention: 24
MaxPasswordAge: 90
```

**Implementation Evidence**:
- **Password Policy**: 14+ characters, complexity requirements
- **MFA**: Enforced for all administrative accounts
- **Root Account**: Secured, not used for daily operations
- **Service Accounts**: Principle of least privilege with specific policies

### 6.2 Role-Based Access Control
**NIST 800-53 Reference**: AC-2 - Account Management

**Standard Implementation**:
- **CadreIQ-Developers**: Application deployment and development access
- **CadreIQ-DevOps**: Infrastructure management and monitoring
- **CadreIQ-Security**: Security review and compliance management
- **cadreiq-deployer**: Automated deployment service account

## 7. Monitoring and Logging

### 7.1 CloudWatch Monitoring
**CIS Benchmark Reference**: 3.1-3.15 - CloudWatch monitoring metrics

**Standard Configuration**:
```yaml
# Lambda Error Monitoring
MetricFilters:
  - FilterName: LambdaErrors
    FilterPattern: "ERROR"
    MetricTransformation:
      MetricName: LambdaErrorCount
      MetricNamespace: CadreIQ/Lambda
      
# VPC Flow Log Monitoring
LogGroups:
  - LogGroupName: /cadreiq/vpc/flowlogs
    RetentionInDays: 30
```

**Implementation Evidence**:
- **Lambda Monitoring**: Error detection and alerting configured
- **VPC Flow Logs**: Centralized network traffic monitoring
- **Database Monitoring**: RDS performance and security metrics
- **Application Monitoring**: ECS task and container metrics

## 8. Backup and Recovery

### 8.1 Database Backup Standards
**CIS Benchmark Reference**: 2.2.4 - RDS backup and recovery

**Standard Configuration**:
```yaml
# RDS Backup Configuration
BackupRetentionPeriod: 7
PreferredBackupWindow: "02:00-03:00"
DeleteAutomatedBackups: false
DeletionProtection: true
```

**Implementation Evidence**:
- **Automated Backups**: Daily snapshots with 7-day retention
- **Manual Snapshots**: Created for restoration testing
- **Recovery Testing**: Annual restoration tests documented
- **Cross-Region**: Available for disaster recovery if needed

## 9. Compliance Validation

### 9.1 Automated Compliance Monitoring
**Implementation Method**: Drata + AWS Config + CloudWatch

**Active Controls**:
- **IMDSv2 Enforcement**: Automated daily validation
- **Encryption Verification**: RDS and EBS encryption status
- **Security Group Analysis**: Overly permissive rule detection
- **VPC Flow Log Status**: Network monitoring compliance
- **Backup Validation**: Daily backup completion verification

### 9.2 Industry Standards Mapping

| Control | CIS Benchmark | NIST 800-53 | Implementation | Status |
|---------|---------------|-------------|----------------|--------|
| IMDSv2 Required | 2.1.1 | SC-39 | ECS Task Definition | ✅ PASS |
| RDS Encryption | 2.2.1 | SC-13 | KMS Encryption | ✅ PASS |
| VPC Flow Logs | 2.1.3 | AU-2 | CloudWatch Logs | ✅ PASS |
| Security Groups | 4.1 | AC-3 | Least Privilege | ✅ PASS |
| TLS Encryption | 4.4 | SC-8 | ALB TLS 1.2+ | ✅ PASS |
| Password Policy | 1.4 | IA-5 | 14+ chars, complexity | ✅ PASS |
| MFA Enforcement | 1.5 | IA-2 | Administrative accounts | ✅ PASS |
| Backup Testing | N/A | CP-4 | Annual restoration | ✅ PASS |

## 10. Infrastructure as Code

### 10.1 Configuration Management
**Tool**: AWS Copilot CLI with CloudFormation
**System**: Replit-based Git repository with AWS Copilot deployment

**Standard Implementation**:
```yaml
# copilot/cadreiq-web/manifest.yml
name: cadreiq-web
type: Backend Service

http:
  healthcheck: '/healthz'

image:
  build: 'Dockerfile'

secrets:
  - DATABASE_URL
  - OPENAI_API_KEY

network:
  vpc:
    enable_logs: true        # VPC Flow Logs
    placement: 'private'     # Private subnet deployment

count:
  min: 1
  max: 10
  auto_scaling:
    target_cpu: 70
    target_memory: 80
```

**Evidence Files**:
- **Infrastructure Definitions**: `/infra/copilot/` directory
- **Deployment History**: Git commit history with infrastructure changes
- **Configuration Validation**: Copilot CLI validates configurations before deployment

## 11. Continuous Compliance

### 11.1 Automated Validation
- **Daily**: Drata automated control validation
- **Weekly**: Security group and IAM policy review
- **Monthly**: Infrastructure drift detection
- **Quarterly**: Full compliance audit and documentation update

### 11.2 Documentation Maintenance
- **Ownership**: CadreIQ Security Team
- **Review Cycle**: Quarterly or upon infrastructure changes
- **Version Control**: Maintained in Git repository with change tracking
- **Approval Process**: Security team approval required for standard changes

---

**Document Prepared By**: CadreIQ Infrastructure Team  
**Next Review Date**: October 24, 2025  
**Compliance Validation**: Drata SOC 2 Type II  
**Contact**: security@cadreiq.com