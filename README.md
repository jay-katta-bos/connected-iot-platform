# Connected Vehicle IoT Platform - Architecture & Design

<img src="./arch.drawio.svg">

## 1. Introduction

The Connected Vehicle IoT Platform is designed to support a variety of smart vehicles with real time monitoring, predictive maintenance and secure remote control functions. The platform will serve upto **5 million** active vehicles globally with requirements for low latency, high availability, scalability, and compliance (GDPR, regional data sovereignty).

### Objectives

- Real time vehicle telemetry ingestion and monitoring.
- Secure remote vehicle control with RBAC and MFA.
- Predictive analytics for proactive maintenance.
- Multi region deployment with regulatory compliance.
- Scalable infrastructure to handle millions of concurrent connections.
- Observability, logging, and upfront alerting.

## 2. System Overview

The platform is built around event driven microservices model on Azure Cloud. Vehicles will publish telemetry via **MQTT** into **Azure IoT Hub**, processed in real time through **Event Hubs & Azure Functions**, with long term storage in **MongoDB Atlas Clusters** and **Azure Data Lake**.

Frontend apps created with **(NextJS & React Native)** consume APIs exposed via **Node.js / Express services**, secured by **Azure AD B2C**. Global traffic is routed through **Azure API Management**, ensuring regional data residency and low latency access.

### Key Design Principles

- **Cloud Native**: Leverages Azure IoT Hub & Event Hubs for ingestion and streaming.
- **Security**: E2E encryption, RBAC, MFA & zero trust APIs.
- **Availability**: Multi region deployments with data sovereignty.
- **Scalability**: Horizontally scalable services and databases.
- **Telemetry**: Centralized monitoring, notifications and logging.

## 3. Component Interactions

1. **Vehicle ↔ IoT Hub**

   - Vehicles connect via MQTT to Azure IoT Hub.
   - Mutual TLS v1.3 authentication ensures secure communication.

2. **Ingestion & Streaming**

   - IoT Hub → Event Hubs → Azure Functions.
   - Validate payloads, filter anomalies and apply geofencing rules.

3. **Data Layers**

   - **Hot Data**: Redis for instant lookups (vehicle status, last known location).
   - **Warm Data**: MongoDB Atlas Clusters (trip history, driver details).
   - **Cold Data**: Data Lake for compliance, audit, analytics and ML workloads.

4. **Remote Control**

   - App → API Gateway → Express Services → IoT Hub → Vehicle.
   - MFA enforced for sensitive actions (unlock, remote start).

5. **User Interfaces**
   - NextJS (web) & React Native (Mobile).
   - Real time dashboards (websockets / rest polling).
   - Push Notifications (Azure Notification Hubs).

## 4. Security Measures

Security is embedded into every layer, namely:

- **Identity**: Azure AD B2C, MFA, RBAC by roles (owner, fleet manager, service teams).
- **Trust**: Services authenticate per request, no implicit trust.
- **Encryption**: TLS v1.3 in transit, AES256 for REST.
- **Protection**: Rate limiting, DDoS mitigation via Azure API Management.
- **Secrets**: Azure Key Vault for certifications / keys / secrets.
- **Audits**: Total access and event logs in analytics.

Vehicle to cloud comms are a primary attack vector and is a good reason for enforcing mutual TLS and short-lived certificate rotation.

## 5. Tech Stack

| Layer          | Tech                         | Notes                                    |
| -------------- | ---------------------------- | ---------------------------------------- |
| Frontend       | React (NextJS), React Native | Large ecosystem, fast prototyping        |
| Backend        | Express on App Services      | Async friendly, lightweight              |
| Ingestion      | Azure IoT Hub & Event Hubs   | Handles millions of connections reliably |
| Processing     | Azure Functions              | Serverless scaling, lower ops overhead   |
| Database       | MongoDB Atlas Clusters       | Supports regional compliances            |
| Cache          | Azure Cache for Redis        | Low latency queries                      |
| Storage        | Azure Data Lake              | Cost effective, analytics ready          |
| Auth & IAM     | Azure AD B2C                 | SSO, MFA, GDPR alignment                 |
| Monitoring     | Azure Monitor, App Insights  | Centralized monitoring                   |
| CI / CD        | Vercel, GitHub Actions       | GitHub integration, incremental builds   |
| Infrastructure | Terraform                    | Widely adopted, reproducible deployments |

## 6. Deployment

- **Terraform** for: IoT Hub, Event Hubs, MongoDB, Redis, App Service, Data Lake.
- **Deployment**:
  1. Run `terraform init && terraform apply` to provision infra.
  2. Deploy Express server to App Services via GitHub Actions.
  3. Deploy React frontend to Vercel via GitHub Actions.
  4. Build mobile apps via GitHub Actions and distribute via stores.
  5. Configure secrets in Key Vault, links with Functions / Services.

## 7. Why Cloud?

- **Cloud (Azure)**: Managed services, has global reach with autoscaling, integrated security and multi region availability.
- **On Premise**: Suitable only for regions with strict data residency laws.
- **Hybrid**: Edge devices buffer data locally, then sync with cloud when allowed.

Decision point: Start cloud first, layer hybrid edge processing where compliance requires it.

## 8. Multi Region & High Availability

- **Active deployments**: Multiple Azure regions (US, EU, APAC).
- **Global traffic**: Azure API Management directs requests based on latency / region.
- **Data sovereignty**: Regional MongoDB clusters to store user / vehicle details.
- **Disaster recovery**: Daily backups, cross region replication.
- **Uptime target**: 99.99% and will be covered under SLA.

Design insight: separating **hot data (Redis)** from **warm data (MongoDB)** prevents telemetry spikes from impacting realt time dashboards.

## 9. CI / CD

- GitHub Action pipelines:
  - Security scans per build (npm audit + snyk).
  - Linting, unit tests, integration tests and sonar integration.
  - Build backend on dev → deploy to staging → manual approval → production.
  - Build frontend on dev → push to staging → preview deployments → promote to production.

Deployment strategy: **Canary Releases** to validate platform stability before rollout.

## 10. Logging & Monitoring

- **Centralized**: Azure Log Analytics + Application Insights.
- **Metrics**: Vehicle telemetry, API latency and error rates.
- **Alerts**: Critical thresholds (latency >200ms, error >1%).
- **Tracing**: Application Insights for distributed tracing.

Op considerations: telemetry volume at this scale can draw down logs. should apply log sampling and retention policies.

## 11. Repository Structure

Private GitHub TurboRepo with the following structure:

- backend -> Express Server
- frontend -> NextJS App
- mobile -> React Native App
  - ios
  - android
- shared -> Shared components
- infra -> Terraform / Github Actions
- docs -> Architecture, diagrams, storybooks

## 12. Cost Estimates (Azure)

| Service            |    Estimated Monthly |
| ------------------ | -------------------: |
| IoT Hub            |               $7,500 |
| Event Hubs         |               $2,000 |
| Notification Hubs  |                 $500 |
| Redis Cache        |                 $200 |
| MongoDB Atlas      |               $3,500 |
| Data Lake Storage  |                 $300 |
| Monitor & Insights |               $1,000 |
| App Services       |                 $300 |
| Firewall           |                 $200 |
| **Total**          | **~$15,500 / month** |

The setup scales to support **5M+ vehicles** daily. Capacity increases linearly by scaling IoT Hub / Event Hub units.

## 13. Future Enhancements

- **Predictive analytics**: Use Azure ML for advanced predictive maintenance using Data Lake.
- **Edge computing**: Reduce latency by processing at vehicle / edge gateways.
- **Third party integration**: Maps, OEM services, AI assistants.
- **OTA updates**: Secure over the air software upgrades.

## 14. Conclusion

The architecture defined in this document leverages **Azure services & MERN stack** to provide a secure, scalable, and cost effective IoT platform. It balances simplicity with enterprise grade features (security and compliance) making it suitable for a global rollout to handle millions of vehicles.
