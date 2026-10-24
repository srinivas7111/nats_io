
flowchart TB

    subgraph CLIENTS["Application Clients - Separate Networks / Subnets"]
        direction LR
        C1["Client 1: GKE Network"]
        C2["Client 2: OCP Network"]
        C3["Client 3: Enterprise Network<br/>Enterprise NATS SDK"]
        CN["Other Clients<br/>Private Subnets"]
        C1 ~~~ C2 ~~~ C3 ~~~ CN
    end

    subgraph PRIMARY["Primary NATS Platform - Development (GKE / OCP)"]
        direction TB

        ENDPOINT["PRIMARY NATS URL<br/>nats.hdfc.dev:4222<br/>Private DNS + Internal L4 Load Balancer<br/>TCP 4222 + mTLS"]

        subgraph CLUSTER["NATS Cluster - 3 Nodes"]
            direction LR
            N1["NATS-0<br/>JetStream Enabled"]
            N2["NATS-1<br/>JetStream Enabled"]
            N3["NATS-2<br/>JetStream Enabled"]

            N1 <-->|"Cluster Routes<br/>TCP 6222 + TLS"| N2
            N2 <-->|"Cluster Routes<br/>TCP 6222 + TLS"| N3
            N3 <-->|"Cluster Routes<br/>TCP 6222 + TLS"| N1
        end

        subgraph PERSISTENCE["Persistent Storage"]
            direction LR
            PV0[("Persistent Volume 0")]
            PV1[("Persistent Volume 1")]
            PV2[("Persistent Volume 2")]
            PV0 ~~~ PV1 ~~~ PV2
        end

        subgraph SERVICES["Platform Services"]
            direction LR
            GOV["GOVERNANCE<br/>Accounts and Subjects<br/>Stream / Consumer Policies"]
            AUTH["AUTHENTICATION<br/>Account JWT / TLS Configuration"]
            GOV ~~~ AUTH
        end

        subgraph OPERATIONS["Operations and Observability"]
            direction LR
            BACKUP["BACKUP / RESTORE<br/>JetStream Operations"]
            OBS["OBSERVABILITY<br/>Metrics, Logs, Traces<br/>Alerts"]
            BACKUP ~~~ OBS
        end

        NETPOL["NETWORK POLICIES<br/>Ingress, Egress and Inter-Node Rules"]
        EGRESS["EGRESS CONTROLS<br/>Approved Outbound Destinations"]

        ENDPOINT -->|"Client Connections: TCP 4222"| CLUSTER

        N1 --- PV0
        N2 --- PV1
        N3 --- PV2

        GOV -.->|"Policies and Standards"| CLUSTER
        AUTH -.->|"Authentication / TLS"| CLUSTER

        NETPOL -.->|"Ingress Rules"| ENDPOINT
        NETPOL -.->|"Cluster Traffic Rules"| CLUSTER
        NETPOL -.->|"Egress Rules"| EGRESS

        CLUSTER <-->|"Backup / Restore"| BACKUP
        CLUSTER -.->|"Telemetry Collection"| OBS

        CLUSTER -->|"Approved Outbound Connections"| EGRESS
        BACKUP -->|"Backup Export"| EGRESS
        OBS -->|"Telemetry Export"| EGRESS
    end

    subgraph EXTERNAL["External / Shared Services"]
        direction LR
        VAULT["Vault / PKI<br/>Secrets and Certificates"]
        DEST["Approved External<br/>Destinations"]
        MON["Enterprise Observability<br/>Platform"]
        STORAGE[("Protected Backup<br/>Storage")]
        VAULT ~~~ DEST ~~~ MON ~~~ STORAGE
    end

    C1 --> ENDPOINT
    C2 --> ENDPOINT
    C3 --> ENDPOINT
    CN --> ENDPOINT

    EGRESS -->|"Permitted Access"| VAULT
    EGRESS --> DEST
    EGRESS -->|"Telemetry Export"| MON
    EGRESS -->|"Backup Artifacts"| STORAGE

    subgraph DR_SCOPE["Future Scope - UAT / Production"]
        direction LR
        DR_ENDPOINT["DR NATS URL<br/>nats-dr.hdfc.dev:4222<br/>Private DNS + Internal L4 Load Balancer<br/>TCP 4222 + mTLS"]
        DR["Separate DR Cluster<br/>JetStream Enabled"]
        DR_ENDPOINT --> DR
    end

    CLIENTS -.->|"Future Client Failover"| DR_ENDPOINT
    CLUSTER -.->|"Future Async JetStream Replication"| DR

    classDef client fill:#DCEBFF,stroke:#2563EB,color:#172554,stroke-width:2px
    classDef endpoint fill:#CCFBF1,stroke:#0F766E,color:#134E4A,stroke-width:2px
    classDef nats fill:#DBEAFE,stroke:#1D4ED8,color:#1E3A8A,stroke-width:2px
    classDef storage fill:#FEF3C7,stroke:#D97706,color:#78350F,stroke-width:2px
    classDef governance fill:#F3E8FF,stroke:#9333EA,color:#581C87,stroke-width:2px
    classDef security fill:#FAE8FF,stroke:#A21CAF,color:#701A75,stroke-width:2px
    classDef network fill:#CCFBF1,stroke:#0F766E,color:#134E4A,stroke-width:2px
    classDef operations fill:#F3F4F6,stroke:#6B7280,color:#1F2937,stroke-width:2px
    classDef observability fill:#DCFCE7,stroke:#16A34A,color:#14532D,stroke-width:2px
    classDef dr fill:#FCE7F3,stroke:#DB2777,color:#831843,stroke-width:2px

    class C1,C2,C3,CN client
    class ENDPOINT,DR_ENDPOINT endpoint
    class N1,N2,N3 nats
    class PV0,PV1,PV2,STORAGE storage
    class GOV governance
    class AUTH,VAULT security
    class NETPOL,EGRESS network
    class BACKUP operations
    class OBS,MON observability
    class DR dr

    style CLIENTS fill:#EFF6FF,stroke:#2563EB,stroke-width:2px
    style PRIMARY fill:#F8FAFC,stroke:#64748B,stroke-width:2px
    style CLUSTER fill:#EFF6FF,stroke:#1D4ED8,stroke-width:2px
    style PERSISTENCE fill:#FFFBEB,stroke:#D97706,stroke-width:1px
    style SERVICES fill:#FAF5FF,stroke:#9333EA,stroke-width:1px
    style OPERATIONS fill:#FFFBEB,stroke:#D97706,stroke-width:1px
    style EXTERNAL fill:#F5F3FF,stroke:#7C3AED,stroke-width:2px
    style DR_SCOPE fill:#FDF2F8,stroke:#DB2777,stroke-width:2px,stroke-dasharray:5 5
