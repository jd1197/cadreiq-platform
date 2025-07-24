# CadreIQ Platform

AI-powered sales management platform with role-based dashboards, strategic playbooks, and predictive analytics.

## Production Deployment

**Live Application:** https://demo.cadreiq.com  
**Authentication:** Password-protected (enterprise demo)

## Architecture

- **Frontend:** Streamlit web application
- **Backend:** Python with FastAPI endpoints
- **Infrastructure:** AWS ECS Fargate with Application Load Balancer
- **Database:** RDS Postgres with AWS Secrets Manager
- **Security:** HTTPS/SSL, private subnets, encrypted storage

## Features

### Role-Based Dashboards
- **CRO Dashboard:** Executive-level metrics and company-wide analytics
- **Regional Director:** Regional performance tracking and territory management  
- **Sales Manager:** Team management and individual rep coaching

### Core Modules
- **Execution Cockpit:** Daily/weekly task management and ritual tracking
- **Predictive Analytics:** Pipeline forecasting and deal risk assessment
- **Playbooks & Coach:** Strategic sales methodologies and AI coaching
- **Team Benchmarks:** Performance comparison and goal tracking

## Infrastructure

### AWS Resources
- **ECS Cluster:** `cadreiq-platform-production-Cluster-FA5FrfQUVSYf`
- **Load Balancer:** `cadrei-Publi-3W2ZH616bRin-1925434857.us-east-1.elb.amazonaws.com`
- **RDS Database:** `cadreiq-dev.c2zya2uw6ls1.us-east-1.rds.amazonaws.com`
- **SSL Certificate:** `arn:aws:acm:us-east-1:613249868486:certificate/a77f5ab3-54d3-44a7-9889-3d1d65e47c0d`

### Deployment
```bash
copilot svc deploy --name web --env production
```

## Development

### Local Development
```bash
streamlit run app.py --server.port 5000
```

### Testing
```bash
pytest test_main.py -v
coverage run -m pytest
coverage report
```

## Compliance

- SOC 2 Type II ready
- Version control with GitHub
- Infrastructure-as-code with AWS Copilot
- Comprehensive audit logging
- Encrypted data storage and transit

## Documentation

- **Infrastructure Diary:** `INFRA_DIARY.md`
- **Technical Charter:** `TECH_CHARTER_CadreIQ_ManagerOS.md`
- **Project Overview:** `replit.md`