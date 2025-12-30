---
title: How Xiaohongshu (RedNote) Handled TikTok Refugee Surges with Karmada
---

## Company Overview

Xiaohongshu (RedNote) is a leading lifestyle and social e-commerce platform in China with over 300 million monthly active users. The platform's core business centers on search, advertising, and recommendations, complemented by social networking and e-commerce capabilities. These services demand massive computational resources and handle enormous data volumes, with individual index tables reaching terabyte scale.

To meet these resource demands while maintaining business agility, Xiaohongshu operates a hybrid cloud infrastructure combining self-built data centers with multiple public cloud providers. This architecture enables elastic scaling and ensures the platform can respond quickly to traffic surges.

## The Challenge: Resource Fragmentation

Xiaohongshu's infrastructure evolved into a complex landscape of dozens of Kubernetes clusters spread across multiple cloud providers and self-built data centers. Both self-built data centers and managed Kubernetes services (TKE/ACK) from cloud vendors have cluster size limits for stability reasons. As Xiaohongshu's business grew rapidly, these limits necessitated a multi-cluster architecture to support the expanding infrastructure.

This cluster fragmentation created multiple operational problems:

- **Resource Silos and Poor User Experience**: Each cluster operated as an isolated resource silo. Business teams had to be aware of individual clusters when deploying applications, leading to a fragmented platform experience and significant communication overhead.

- **Frequent Application Migrations**: With rapid business growth, clusters would fill to capacity within months. Teams then had to migrate applications to new clusters—a frequent, labor-intensive process that consumed significant operational effort and introduced migration risks.

- **Operational Inflexibility**: Platform engineers conducting cluster maintenance inevitably impacted business teams, requiring them to pause releases or coordinate migrations. The lack of flexibility created friction between platform and application teams.

- **Cross-Cluster Elasticity Gap**: The platform lacked the ability to intelligently distribute workloads. Ideally, it should prioritize self-built data center resources while automatically using cloud resources when capacity became insufficient, but this required coordinated scheduling across isolated clusters.

### The Ideal vs. Reality Gap

The ideal deployment model was straightforward: business teams select a region, and the platform provides a unified resource pool. The reality was very different—a single region might contain dozens of isolated clusters with different types of machines, making efficient resource scheduling nearly impossible.

![Xiaohongshu Architecture](./static/xiaohongshu_01.webp)

### The TikTok Refugee Crisis: A Wake-Up Call

In January 2025, when TikTok faced potential restrictions, millions of users suddenly migrated to Xiaohongshu, causing massive traffic spikes. The self-built data centers' resources were immediately exhausted. The team faced three difficult options:

1. **Emergency server procurement**: But with Chinese New Year approaching, delivery was uncertain and costs would be astronomical
2. **Traffic redistribution**: Shift more traffic to cloud regions, but this would require scaling entire service chains, not just individual services, dramatically increasing costs and complexity
3. **Leverage federation**: Use the federated cluster system to elastically burst to cloud resources only for services under pressure

The third option, enabled by Karmada-based federation, proved to be the solution.

## The Solution: Karmada-Based Multi-Cluster Federation

Xiaohongshu chose to build a federated cluster system with Karmada at its core, focusing on two key principles:

1. **Hide cluster complexity from business teams**: Enable teams to think in terms of regions, not individual clusters
2. **Unify resource scheduling**: Create a global resource pool from fragmented clusters

### Architecture Design Principles

The solution included three core components:

#### 1. Unified API Gateway

Xiaohongshu maintained Kubernetes API compatibility to ensure zero migration cost for existing platforms and applications. They built a custom access layer that intelligently routes requests:
- Workload resources (Deployments, StatefulSets) route to Karmada's control plane
- Pod and PVC queries route to ClusterGateway, a custom component that aggregates data from member clusters

This design allows standard Kubernetes clients (kubectl, client-go) to work seamlessly without modification.

![Unified API Architecture](./static/xiaohongshu_04.webp)

#### 2. Multi-Tier Scheduling System

The scheduling architecture uses a "self-built first, cloud backup" policy:

- **Regional scheduling**: Prioritizes self-built data center resources, automatically bursting to cloud when capacity is insufficient
- **Pre-scheduling capability**: Captures resource snapshots from member clusters and simulates complete scheduling logic to determine optimal replica distribution
- **Smart scaledown**: Automatically removes cloud resources first during scaledown operations

#### 3. Enhanced Workload Orchestration

Xiaohongshu extended Karmada with custom features to ensure production-grade reliability:

**Fleet-Root mechanism**
Provides fine-grained rollout orchestration at the federation level, ensuring parameters like `MaxUnavailable` and `MaxSurge` work correctly across distributed clusters

**Federated HPA**
Moves autoscaling logic to the federation layer, enabling more efficient resource utilization across clusters

**Custom StatefulSet**
Developed a stateful workload controller with customizable index orchestration for search and recommendation services. This enables:
- Customizable pod indexing across clusters (e.g., cluster 1: pods 0-2, cluster 2: pods 3-5)
- Detailed status reporting for each pod index
- Precise rollout control with fine-grained parameters like `MaxUnavailable` and `MaxSurge`
- Incremental data updates without full reloads

![Federation Architecture](./static/xiaohongshu_03.webp)

## Results and Benefits

- **Operational efficiency**: Eliminated manual multi-cluster deployments and migrations
- **Resource utilization**: Significantly improved through global resource pooling and federation-level HPA
- **Cost optimization**: Service-level scaling instead of chain-wide scaling during traffic surges
- **Developer experience**: Zero-cost migration for existing applications due to full Kubernetes API compatibility
- **GPU efficiency**: Reduced minimum replica counts from 10x per model to 1x per model, freeing hundreds of GPU cards
- **Resilience**: Successfully handled unexpected 10x+ traffic spike during TikTok migration event

### Search and Recommendation: Precise Management at Scale

The search and recommendation services are among the most critical systems at Xiaohongshu, processing massive index tables reaching terabyte scale. With the custom StatefulSet controller, these large-scale stateful workloads can now be precisely managed and orchestrated across the federation:

- **Large-Scale Management**: Successfully manages massive distributed applications with thousands of replicas across multiple clusters while maintaining data consistency
- **Precise Release Control**: Enables fine-grained rollout orchestration with parameters like `MaxUnavailable` and `MaxSurge`, ensuring safe and controlled deployments without service disruption
- **Elastic Cross-Cluster Deployment**: Applications can elastically expand across clusters based on demand, providing the flexibility needed for critical business workloads

![Stateful Workload Architecture](./static/xiaohongshu_07.webp)

### GPU Resource Pool Unification for LLM Inference

Large language model inference had unique challenges:

- **GPU scarcity**: Tens of thousands of GPU cards scattered across dozens of clusters
- **HPA inefficiency**: Without federation, HPA required at least one replica per cluster. For 100 models across 10 clusters, this meant 1,000 minimum replicas even during low traffic
- **Deployment complexity**: Teams had to manually deploy models to multiple clusters

With federation:
- **Single deployment**: Business teams deploy once; federation handles distribution
- **Optimized HPA**: Federation-level HPA can scale to a single replica globally during low traffic
- **Dramatic efficiency gains**: GPU utilization improved significantly by eliminating mandatory per-cluster replicas

![GPU Resource Pool](./static/xiaohongshu_09.webp)

### Surviving the Traffic Surge

During the TikTok user migration crisis, the federation system proved its value:

- **Rapid response**: Cloud clusters (ACK) were dynamically added to the federation
- **Targeted scaling**: Only services experiencing pressure scaled to the cloud, not entire service chains
- **Automatic scaledown**: After traffic normalized, cloud resources were released automatically
- **Cost optimization**: Service-level precise scaling minimized costs compared to chain-wide scaling

This approach delivered the lowest risk and cost profile during the crisis.

![Traffic Surge Response](./static/xiaohongshu_08.webp)

## Why Karmada?

Xiaohongshu evaluated multiple federation solutions and selected Karmada for several reasons:

1. **Kubernetes-native API**: No application refactoring required; standard Kubernetes clients work without modification
2. **Resource-centric design**: Focus on resource scheduling aligns with Xiaohongshu's efficiency goals
3. **Independent architecture components**: Separate etcd and scheduler enable horizontal scaling
4. **Flexible cluster access**: Both push and pull modes support diverse network topologies
5. **Active community**: Open governance and responsive community support ongoing improvements
6. **Extensibility**: CRD-based policy system enabled custom enhancements like Fleet-Root

## Lessons Learned

1. **Start with resource efficiency**: Federation's value goes beyond disaster recovery to fundamental resource optimization
2. **Maintain API compatibility**: Zero migration cost is critical for adoption by existing platforms and teams
3. **Extend thoughtfully**: Native Karmada needed production-grade enhancements like federation-level rollout control
4. **Plan for heterogeneity**: Network and infrastructure differences between cloud and on-premises require careful architecture
5. **Community engagement**: Contributing back to Karmada and collaborating on features like Volcano integration benefits everyone

## Future Plans

Xiaohongshu continues to expand their Karmada-based federation:

1. **Broader workload coverage**: Extending federation to AI training workloads and Spark big data jobs
2. **Enhanced scheduling**: Collaborating with the Karmada community on Volcano scheduler integration for more sophisticated scheduling decisions
3. **Advanced policies**: Developing more intelligent scheduling policies based on cost, latency, and resource characteristics
4. **Deeper integration**: Further automation of cluster lifecycle management and resource optimization

![Future Architecture](./static/xiaohongshu_10.webp)

## Conclusion

Xiaohongshu's journey with Karmada demonstrates how multi-cluster federation can transform hybrid cloud operations from a management burden into a competitive advantage. By focusing on resource efficiency, maintaining Kubernetes compatibility, and extending Karmada thoughtfully, the company built a system that not only solved immediate operational pain points but also provided the foundation for handling unexpected challenges—like a sudden 10x traffic spike from millions of new users.

The success during the TikTok migration event proved the architecture's core idea: unified resource scheduling across fragmented clusters enables both operational efficiency and business resilience. As Xiaohongshu continues to scale, Karmada remains central to their strategy for managing complexity while maintaining agility in a dynamic multi-cloud environment.
