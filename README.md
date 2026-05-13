# BizSafer Corporate Infrastructure Architecture

**Lead Architect:** [Md. Kamruzzaman](https://www.linkedin.com/in/4kamruzzaman/)  
**Engineering Lab:** [BizSafer](https://bizsafer.com)  
**Live Production:** https://www.bizsafer.com  
**Enterprise Inquiries:** kamruzzaman@bizsafer.com  

## System Overview

The cost of commodity infrastructure is unrecoverable downtime. Most engineering agencies sell high availability to clients while hosting their own corporate perimeters on fragile shared monoliths. We strictly reject this hypocrisy. 

The BizSafer Global Engineering Lab runs on the exact same zero-downtime, container-native baseline we deploy for our Tier-1 enterprise clients. This repository details the high-availability infrastructure and deployment architecture designed to support our own corporate perimeter.

### Infrastructure Topology

<pre>
graph TD
    Client([Global Enterprise Traffic]) --&gt;|HTTPS / WSS| CF[Cloudflare WAF &amp; Edge Routing]
    
    subgraph Zero-Downtime Cluster
        CF --&gt;|Validated Payloads| Proxy[Kamal Proxy]
        
        subgraph Edge Delivery Nodes
            Proxy --&gt;|Atomic Routing| NextJS[Next.js 16 Standalone Containers]
        end
        
        subgraph Isolated Application Core
            Proxy --&gt;|API Traffic| Laravel[Laravel 13 Core &amp; Queue Workers]
            NextJS --&gt;|Internal Fetches| Proxy
        end
        
        subgraph Stateful Persistence Layer
            Laravel --&gt;|Strict Isolation| PG[(PostgreSQL 18)]
            Laravel --&gt;|High-Concurrency| Redis[(Redis 8)]
            Laravel --&gt;|Object Persistence| S3[(External S3-Compatible Nodes)]
        end
    end
    
    classDef proxy fill:#2563eb,stroke:#fff,stroke-width:2px,color:#fff
    classDef edge fill:#f59e0b,stroke:#fff,stroke-width:2px,color:#fff
    classDef core fill:#dc2626,stroke:#fff,stroke-width:2px,color:#fff
    classDef data fill:#059669,stroke:#fff,stroke-width:2px,color:#fff
    
    class Proxy proxy
    class NextJS edge
    class Laravel core
    class PG,Redis,S3 data
</pre>

## Repository Architecture

We maintain strict decoupling within a monorepo structure to streamline our CI/CD pipelines while isolating runtime environments.
```text
├── .github/workflows/   # Strict GitHub Actions CI/CD pipelines  
├── bizsafer-api/        # Laravel 13 API core & queue workers  
├── bizsafer-web/        # Next.js 16 standalone edge nodes  
├── config/deploy.yml    # Kamal 2 orchestration configuration  
└── docker-compose.yml   # Local replication environment
```

## Locked SRE Metrics

* **Uptime Target:** >99.9% via a zero single point of failure container-native architecture.
* **Global Latency:** Sub-200ms latency enforced by edge computing and optimized database indexing.
* **MTTR:** <60s via automated self-healing rollouts and strict continuous delivery pipelines.

## The Architecture Stack

The system operates on a fully decoupled micro-architecture rather than a legacy monolith.

* **Edge Delivery & Security:** Cloudflare CDN, WAF, and global DNS routing.
* **Frontend Edge Node:** Next.js 16 deployed as a standalone Docker container.
* **API Core:** Laravel 13 API handling rigid data validation, rate limiting, and secure payload routing.
* **Stateful Persistence:** PostgreSQL 18 for core relational data.
* **High-Concurrency Queue:** Redis 8 managing background job processing and caching layers.

## Deployment & Orchestration

The cost of manual intervention is human error. The deployment pipeline is fully automated to guarantee bit-for-bit environment parity.

* **Containerization:** 100% Dockerized environments isolating the Next.js frontend, Laravel API, PostgreSQL, and Redis instances.
* **Continuous Integration:** Strict GitHub Actions pipelines handle automated testing, static analysis, and pushing immutable images to the GitHub Container Registry.
* **Continuous Delivery:** Kamal 2 orchestration utilizing kamal-proxy to execute atomic traffic swapping, manage zero-downtime rolling updates, and instantly roll back unhealthy deployments.

## Security Posture

* Multi-layer defense-in-depth utilizing **Cloudflare WAF** to block malicious payloads at the edge before they reach internal clusters.
* **Air-gapped database** pipelines enforcing strict resource isolation.
* Rigid API key validation and **rate-limiting middleware** to prevent payload flooding.
