# Kubernetes FinOps Analysis Skill
> Generate FinOps reports for Kubernetes infrastructure using Dynatrace telemetry and industry-standard frameworks

Analyze K8s cost optimization opportunities using the **FinOps Foundation framework** and industry benchmarks.

---

## When to Use This Skill

Use this skill when the user asks for:
- FinOps report for Kubernetes infrastructure
- Cost optimization analysis for K8s clusters
- Kubernetes resource utilization and waste analysis
- Container cost allocation and chargeback/showback
- Infrastructure rightsizing recommendations
- Cloud cost analysis for K8s workloads

**Triggers:**
- "FinOps report"
- "K8s cost analysis"
- "Kubernetes cost optimization"
- "container cost allocation"
- "K8s rightsizing"
- "cloud spend on Kubernetes"

---

## Output Directory

**IMPORTANT:** All generated FinOps reports MUST be saved to the `/report` directory in the workspace.

Filename format: `FinOps_K8s_Report_YYYY-MM-DD.md` or `FinOps_K8s_Report_[ClusterName]_YYYY-MM-DD.md`

---

## Framework Anchors

### 1. FinOps Foundation (finops.org)

**The authoritative framework for cloud financial management.**

**Key Resources:**

| Resource | URL | What It Provides |
|----------|-----|------------------|
| **Calculating Container Costs** | `finops.org/wg/calculating-container-costs/` | Why traditional cost allocation breaks down with K8s (shared resources, node-level billing); practical approaches for namespace, label-based, and workload-level allocation |
| **Inform → Optimize → Operate Framework** | `finops.org/framework/` | Structure for report recommendations: **Inform** = cost visibility to namespace/pod level; **Optimize** = rightsizing and waste elimination; **Operate** = ongoing governance |
| **Cost Allocation Capability** | `finops.org/wg/cloud-cost-allocation/` | Metadata strategy (required tags like Cost Center, Environment); KPIs for measuring allocation maturity; percentage of tag-compliant costs |

### 2. CNCF (Cloud Native Computing Foundation)

**Industry benchmarks and best practices.**

| Resource | URL | What It Provides |
|----------|-----|------------------|
| **FinOps Microsurvey** | `cncf.io` (surveys) | **70%** of orgs cite overprovisioning as top cost driver; **45%** lack ownership; **43%** have unused resources |
| **Resource Requests/Limits Best Practices** | `cncf.io/blog/2022/10/20/kubernetes-best-practice-how-to-correctly-set-resource-requests-and-limits/` | Technical grounding for rightsizing recommendations; covers HPA, cluster autoscaler, why incorrect limits cause waste and instability |

### 3. FinOps FOCUS Spec (v1.3+)

**The standard for split cost allocation.**

- **FOCUS 1.3** added split cost allocation columns specifically for shared K8s resources
- Enables practitioners to see the methodology behind cost splits, not just the output
- Critical for chargeback/showback at team level

**Reference:** `focus.finops.org`

---

## Report Structure

### Industry-Aligned Flow

Map Dynatrace telemetry to industry benchmarks:

#### 1. **Visibility Gap**
- What % of K8s spend is unattributed?
- **Benchmark:** Most orgs start below 60% allocation accuracy (FinOps Foundation)
- **Dynatrace Data:** Namespace/pod labels vs. missing cost owner tags

#### 2. **Rightsizing Findings**
- CPU/memory requested vs. actual utilization
- **Benchmark:** 70% of orgs have overprovisioning issues (CNCF)
- **Dynatrace Data:** `dt.kubernetes.container.cpu_usage` vs. `requests_cpu`, `limits_cpu`

#### 3. **Ownership Gaps**
- Namespaces or workloads with no cost owner
- **Benchmark:** 45% of orgs lack accountability (CNCF)
- **Dynatrace Data:** Missing `tags[cost-center]`, `tags[team]`, `tags[environment]`

#### 4. **Unused Resources**
- Idle workloads, terminated pods still consuming resources
- **Benchmark:** 43% of orgs have unused resources (CNCF)
- **Dynatrace Data:** Pods with 0 requests, low CPU/memory utilization over 7d

#### 5. **Recommendations**
Bucket by FinOps maturity stage:
- **Inform:** Improve cost visibility (add tags, enable allocation)
- **Optimize:** Rightsizing, HPA tuning, spot instance usage
- **Operate:** Policy enforcement, automated governance, ongoing reviews

---

## Required User Inputs

**ALWAYS prompt the user for these before generating a report:**

1. **Node Pricing:**
   - Ask: "Do you have node pricing data, or should I find indicative pricing online?"
   - If online: "Where is the K8s cluster deployed? (AWS/Azure/GCP region, or on-prem provider)"

2. **Cluster Context:**
   - Cluster name(s) to analyze
   - Time range (default: last 7 days)
   - Specific namespaces or workloads to focus on (optional)

3. **Cost Attribution:**
   - Are there existing cost allocation tags? (e.g., `cost-center`, `team`, `environment`)
   - What metadata fields should be used for cost attribution?

---

## Dynatrace DQL Queries for FinOps

### 1. Namespace Resource Requests vs. Actuals

```dql
// CPU overprovisioning by namespace
timeseries {
  cpu_requested = avg(dt.kubernetes.container.requests_cpu),
  cpu_used = avg(dt.kubernetes.container.cpu_usage)
}, by:{k8s.namespace.name}, from:now()-7d
| fieldsAdd cpu_waste_percent = (cpu_requested - cpu_used) / cpu_requested * 100
| sort cpu_waste_percent desc
```

```dql
// Memory overprovisioning by namespace
timeseries {
  mem_requested = avg(dt.kubernetes.container.requests_memory),
  mem_used = avg(dt.kubernetes.container.memory_working_set)
}, by:{k8s.namespace.name}, from:now()-7d
| fieldsAdd mem_waste_percent = (mem_requested - mem_used) / mem_requested * 100
| sort mem_waste_percent desc
```

### 2. Cost Owner Attribution

```dql
// Namespaces missing cost allocation tags
fetch dt.entity.cloud_application_namespace
| fields k8s.namespace.name, tags
| filter isNull(tags[cost-center]) or isNull(tags[team])
| summarize untagged_namespaces = count()
```

### 3. Idle Workloads

```dql
// Pods with near-zero CPU usage over 7d
timeseries avg_cpu = avg(dt.kubernetes.container.cpu_usage), 
by:{k8s.namespace.name, k8s.pod.name}, from:now()-7d
| filter avg_cpu < 0.01  // Less than 1% CPU
| sort avg_cpu asc
```

### 4. Node Capacity vs. Utilization

```dql
// Cluster-wide capacity analysis
timeseries {
  pods_allocatable = avg(dt.kubernetes.node.pods_allocatable),
  pods_actual = avg(dt.kubernetes.pods),
  cpu_allocatable = avg(dt.kubernetes.node.cpu_allocatable),
  cpu_used = avg(dt.kubernetes.node.cpu_usage),
  memory_allocatable = avg(dt.kubernetes.node.memory_allocatable),
  memory_used = avg(dt.kubernetes.node.memory_working_set)
}, by:{k8s.cluster.name}, from:now()-7d
```

### 5. Workload-Level Cost Attribution

```dql
// Cost attribution by workload type
fetch dt.entity.cloud_application
| fields k8s.workload.name, k8s.workload.kind, k8s.namespace.name, tags
| summarize count(), by:{k8s.workload.kind, tags[cost-center]}
```

---

## Node Pricing Resources

When the user asks for indicative pricing (no custom data provided):

### AWS EKS
- **On-Demand:** `aws.amazon.com/ec2/pricing/on-demand/`
- **Spot Instances:** `aws.amazon.com/ec2/spot/pricing/`
- **Savings Plans:** `aws.amazon.com/savingsplans/pricing/`

### Azure AKS
- **VM Pricing:** `azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/`
- **Spot VMs:** `azure.microsoft.com/en-us/pricing/spot-vms/`

### Google GKE
- **Machine Types:** `cloud.google.com/compute/all-pricing`
- **Preemptible VMs:** `cloud.google.com/compute/docs/instances/preemptible`

### Generic Formula (when exact pricing unavailable)
```
Pod Cost = (Node Hourly Cost × Pod Resource Requests) / Node Total Capacity
```

**Example:**
- Node: 16 vCPU, 64 GB RAM, $0.50/hour
- Pod: 2 vCPU request, 8 GB request
- Pod Cost: ($0.50 × 2/16) + ($0.50 × 8/64) = $0.0625 + $0.0625 = $0.125/hour

---

## Report Best Practices

### Structure
1. **Executive Summary**
   - Total cluster spend (if available)
   - % wasted due to overprovisioning
   - Top 3 recommendations

2. **Visibility Analysis**
   - % of resources with cost owner tags
   - Gaps in cost attribution
   - Comparison to 60% industry baseline

3. **Efficiency Analysis**
   - CPU/memory overprovisioning by namespace
   - Idle workloads (candidates for termination)
   - Comparison to 70% overprovisioning benchmark

4. **Ownership & Accountability**
   - Namespaces/workloads missing owners
   - Comparison to 45% accountability gap benchmark

5. **Recommendations (Inform/Optimize/Operate)**
   - **Inform:** Tagging strategy, cost allocation setup
   - **Optimize:** Rightsizing actions, HPA tuning, spot instance adoption
   - **Operate:** Policy enforcement, automated reviews, budget alerts

### Key Metrics to Include
- **Utilization Rate:** `actual_usage / requested_resources`
- **Waste Percentage:** `(requested - actual) / requested * 100`
- **Allocation Coverage:** `% of pods with cost owner tags`
- **Cost per Namespace/Workload:** Calculated from node pricing

### Benchmarks to Reference
- **60%** allocation accuracy threshold (FinOps Foundation)
- **70%** overprovisioning rate (CNCF)
- **45%** lack ownership (CNCF)
- **43%** unused resources (CNCF)

---

## Skill Dependencies

This skill builds on:
- `dt-obs-kubernetes.md` — K8s entity types, metrics, and queries
- `dt-dql-essentials.md` — DQL syntax and best practices
- `dt-obs-hosts.md` — Host-level resource metrics (if needed)

---

## Example Usage

**User:** "Create a FinOps report for our Kubernetes cluster"

**Agent Actions:**
1. Read this skill file
2. Read `dt-obs-kubernetes.md` for K8s data model
3. Prompt user:
   - "Do you have node pricing data?"
   - "Where is the cluster deployed?"
   - "What is the cluster name?"
4. Execute DQL queries (namespace utilization, cost attribution, idle workloads)
5. Calculate waste percentages and compare to benchmarks
6. Generate report in `/report/FinOps_K8s_Report_YYYY-MM-DD.md`
7. Structure recommendations by Inform/Optimize/Operate framework

---

## Reference Links

**FinOps Foundation:**
- Framework: https://www.finops.org/framework/
- Container Costs: https://www.finops.org/wg/calculating-container-costs/
- Cost Allocation: https://www.finops.org/wg/cloud-cost-allocation/

**CNCF:**
- FinOps Surveys: https://www.cncf.io/reports/
- K8s Best Practices: https://www.cncf.io/blog/

**FOCUS Spec:**
- Specification: https://focus.finops.org/

**Cloud Provider Pricing:**
- AWS: https://aws.amazon.com/ec2/pricing/
- Azure: https://azure.microsoft.com/en-us/pricing/
- GCP: https://cloud.google.com/pricing/

---

## Update Protocol

When discovering new FinOps patterns or queries:
1. Document new DQL queries in this file
2. Add industry benchmarks as they are published
3. Update reference links if URLs change
4. Store cloud provider pricing snapshots in `reference/` if needed
