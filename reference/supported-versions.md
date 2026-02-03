# Supported Versions

## Database and Platform Version Compatibility

Reference for supported database versions and platform requirements.

---

### Database Versions

#### PostgreSQL

| Version | Support Level | Notes                 |
| ------- | ------------- | --------------------- |
| 16.x    | Full          | Recommended           |
| 15.x    | Full          |                       |
| 14.x    | Full          |                       |
| 13.x    | Full          |                       |
| 12.x    | Full          |                       |
| 11.x    | Limited       | Security updates only |
| 10.x    | EOL           | Upgrade recommended   |

**Cloud variants:** Aurora PostgreSQL, Cloud SQL, Azure Database for PostgreSQL

#### MySQL

| Version | Support Level | Notes               |
| ------- | ------------- | ------------------- |
| 8.0.x   | Full          | Recommended         |
| 5.7.x   | Full          |                     |
| 5.6.x   | Limited       | Upgrade recommended |

**Cloud variants:** Aurora MySQL, Cloud SQL, Azure Database for MySQL

#### Oracle

| Version | Support Level | Notes                     |
| ------- | ------------- | ------------------------- |
| 23c     | Full          |                           |
| 21c     | Full          |                           |
| 19c     | Full          | Recommended               |
| 18c     | Full          |                           |
| 12c R2  | Full          |                           |
| 12c R1  | Limited       |                           |
| 11g R2  | Limited       | Some features unavailable |
| 11g R1  | Limited       |                           |
| 10g     | EOL           | Contact support           |

#### SQL Server

| Version | Support Level | Notes           |
| ------- | ------------- | --------------- |
| 2022    | Full          |                 |
| 2019    | Full          | Recommended     |
| 2017    | Full          |                 |
| 2016    | Full          |                 |
| 2014    | Limited       |                 |
| 2012    | Limited       |                 |
| 2008 R2 | EOL           | Contact support |

**Cloud variants:** Azure SQL Database, Azure SQL Managed Instance, RDS SQL Server

#### MongoDB

| Version | Support Level | Notes       |
| ------- | ------------- | ----------- |
| 7.0     | Full          |             |
| 6.0     | Full          | Recommended |
| 5.0     | Full          |             |
| 4.4     | Full          |             |
| 4.2     | Limited       |             |

**Cloud variants:** MongoDB Atlas

#### Snowflake

| Version | Support Level | Notes         |
| ------- | ------------- | ------------- |
| Current | Full          | Always latest |

#### BigQuery

| Version | Support Level | Notes         |
| ------- | ------------- | ------------- |
| Current | Full          | Always latest |

#### Teradata

| Version | Support Level | Notes |
| ------- | ------------- | ----- |
| 17.x    | Full          |       |
| 16.x    | Full          |       |
| 15.x    | Limited       |       |

#### DB2

| Version | Support Level | Notes |
| ------- | ------------- | ----- |
| 11.5    | Full          |       |
| 11.1    | Full          |       |
| 10.5    | Limited       |       |

---

### Cloud Platforms

| Platform        | Support Level |
| --------------- | ------------- |
| AWS             | Full          |
| Google Cloud    | Full          |
| Microsoft Azure | Full          |
| Oracle Cloud    | Full          |
| IBM Cloud       | Limited       |
| Alibaba Cloud   | Limited       |

---

### Kubernetes Versions

| Version | Support Level | Notes       |
| ------- | ------------- | ----------- |
| 1.29    | Full          |             |
| 1.28    | Full          | Recommended |
| 1.27    | Full          |             |
| 1.26    | Full          |             |
| 1.25    | Limited       |             |

**Managed Kubernetes:** EKS, GKE, AKS all supported

---

### Operating Systems (Self-Hosted)

| OS             | Version      | Support Level |
| -------------- | ------------ | ------------- |
| Ubuntu         | 22.04, 24.04 | Full          |
| Debian         | 11, 12       | Full          |
| RHEL           | 8, 9         | Full          |
| Amazon Linux   | 2, 2023      | Full          |
| Windows Server | 2019, 2022   | Limited       |

---

### Browser Support (Dashboard)

| Browser | Minimum Version |
| ------- | --------------- |
| Chrome  | 100+            |
| Firefox | 100+            |
| Safari  | 15+             |
| Edge    | 100+            |

---

### SDK/Client Requirements

#### Python SDK

| Python Version | Support    |
| -------------- | ---------- |
| 3.12           | Supported  |
| 3.11           | Supported  |
| 3.10           | Supported  |
| 3.9            | Supported  |
| 3.8            | Deprecated |

#### Node.js SDK

| Node Version | Support         |
| ------------ | --------------- |
| 22.x         | Supported       |
| 20.x         | Supported (LTS) |
| 18.x         | Supported (LTS) |
| 16.x         | Deprecated      |

#### Go SDK

| Go Version | Support   |
| ---------- | --------- |
| 1.22       | Supported |
| 1.21       | Supported |
| 1.20       | Limited   |

---

### Support Levels

| Level   | Definition                                      |
| ------- | ----------------------------------------------- |
| Full    | Fully tested, all features supported            |
| Limited | Basic functionality, some features may not work |
| EOL     | End of life, not supported                      |

---

### Version Deprecation Policy

- **6 months notice** before dropping support
- **Security patches** continue for 3 months after deprecation
- **Migration assistance** available for deprecated versions

→ [Connectors](../integration/connectors.md)
→ [Self-Hosted Deployment](../../operations/deployment/self-hosted.md)
