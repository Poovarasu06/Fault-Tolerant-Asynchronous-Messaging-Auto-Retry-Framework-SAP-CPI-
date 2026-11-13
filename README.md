# Fault-Tolerant Asynchronous Messaging & Auto-Retry Framework (SAP CPI)

![SAP BTP](https://img.shields.io/badge/SAP_BTP-Cloud_Integration_(CPI)-008FD3?style=flat-square&logo=sap)
![Protocol](https://img.shields.io/badge/Adapters-HTTPS_%7C_JMS_%7C_HTTP_%7C_Mail-blue?style=flat-square)
![Scripting](https://img.shields.io/badge/Scripting-Groovy-4298B8?style=flat-square&logo=apachegroovy)
![Status](https://img.shields.io/badge/Runtime_Status-Started-brightgreen?style=flat-square)

A decoupled, two-stage asynchronous integration architecture built on **SAP Cloud Integration (SAP CPI)**[cite: 1, 2]. This project guarantees reliable message delivery[cite: 1, 2], separates message ingestion from downstream endpoint availability[cite: 1, 2], and implements an automated exception-handling framework[cite: 2] using **JMS Queues**[cite: 1, 2], **Groovy Scripting**[cite: 2], and **Content-Based Routing**[cite: 2].

---

## Architecture Overview

To prevent data loss during downstream target outages, the integration is split into two distinct Integration Flows (`iFlows`) connected via an internal SAP CPI JMS Queue[cite: 1, 2]:

```text
[Sender] ──(HTTPS)──▶ [iFlow 1: Ingestion & Enrich] ──(JMS Queue)──▶ [iFlow 2: Consumer & Retry Engine] ──(HTTP)──▶ [Receiver]
                                                                                 │
                                                                         (On Runtime Error)
                                                                                 ▼
                                                                     [Exception Subprocess]
                                                                                 │
                                                                                 ▼
                                                                      [LIP_Handle_Exception]
                                                                         ├── Permanent Error ──▶ [Mail Alert]
                                                                         ├── Retryable (< Max) ─▶ [Re-queue to JMS]
                                                                         └── Retry Limit Met ───▶ [Mail Alert]
