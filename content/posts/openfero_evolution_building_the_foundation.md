---
title: 'OpenFero Evolution: Building the Foundation 
for community-driven Self-Healing Framework'
date: 2025-05-20T21:00:00+00:00
draft: false
Tags:
  - kubernetes
  - self-healing
  - cloud native
  - openfero
  - open source
---

| ![4 people in front of a sunset](https://cdn-images-1.medium.com/max/1000/0*oltUPFCNHjjq0yGH) |
|:--:|
| *Photo by Helena Lopes on Unsplash* |

Hi guys, in late 2022, I shared the story of my hobby project ["OpenFero: The journey to a self-healing platform"](/posts/the_journey_to_a_self_healing_platform/). What started as a personal project to extend Kubernetes' self-healing capabilities has evolved significantly over the past two and a half years. Today, I'm excited to provide an update on OpenFero's journey and highlight some of the exciting new features and developments.

> As a quick reminder: Open Fero is a play on words from the Latin "opem fero" (meaning "to help") and "OpenSource". OpenFero is a framework for self-healing in cloud-native environments, designed to execute remediation actions in response to alerts from monitoring systems.

## From Personal Project to Organization

One of the most significant developments is that OpenFero has found a new home! What began as a personal repository under my GitHub account has evolved into a dedicated GitHub organization.

The project is now housed at [github.com/OpenFero](https://github.com/OpenFero), making it easier for the community to contribute and collaborate. This organizational structure also provides a foundation for potential expansion with additional repositories for extensions and documentation.

## New User Interface: Visualizing Self-Healing in Action

Perhaps the most visible enhancement to OpenFero is the introduction of a comprehensive web-based user interface. The original version was primarily API-driven, requiring users to interact with the system through command-line tools or custom integrations.

The new UI provides:

- A dashboard view showing all received alerts and their current status
- Detailed information about triggered healing jobs (operarios)
- A configuration viewer for operarios definitions
- Dark mode support for those late-night troubleshooting sessions
- Collapsible alert views for better organization of large alert volumes
- Timestamps in your local timezone for improved usability

The UI makes the self-healing process more transparent and accessible, allowing operations teams to monitor the system's activities more effectively and understand exactly what remediation actions have been taken.

## Enhanced Operarios Traceability

Operarios—OpenFero's name for the remediation jobs—now provide enhanced traceability. The system collects comprehensive data about triggered jobs, creating an audit trail that shows:

- When a job was triggered
- What alert caused it
- Complete execution details
- Success or failure status

This improved traceability is invaluable for post-incident analysis and continuous improvement of your self-healing processes.

## Improved Reliability with Cluster Mode

For production environments where high availability is critical, OpenFero now offers a cluster mode. This mode:

- Enables multiple OpenFero instances to run in parallel
- Implements leader election to prevent duplicate job executions
- Includes Pod Disruption Budgets (PDBs) for stability during cluster maintenance
- Maintains consistent state across instances

These enhancements make OpenFero suitable for even the most demanding production environments.

## Simplified Installation with Helm

OpenFero's installation process has been streamlined with an official Helm chart. Getting started is now as simple as:

```bash
helm pull oci://ghcr.io/openfero/openfero/charts/openfero --version 0.2.1
helm install openfero oci://ghcr.io/openfero/openfero/charts/openfero --version 0.2.1
```

The chart intelligently detects your cluster configuration and applies appropriate settings, making deployment smooth across different Kubernetes environments.

## Testing and Interacting with OpenFero

Once installed, you can interact with OpenFero in several ways:

1. **Using the new UI**: Access the dashboard at `http://openfero-service:8080/`
2. **Using the Swagger UI**: Explore and test the API at `http://openfero-service:8080/swagger/`
3. **Using curl for direct API access**:
   ```bash
   curl -X POST http://openfero-service:8080/alert \
     -H 'Content-Type: application/json' \
     -d '{
       "status": "firing",
       "alerts": [{
         "status": "firing",
         "labels": {
           "alertname": "TestAlert",
           "severity": "warning"
         }
       }]
     }'
   ```

## Performance Improvements

Behind the scenes, OpenFero has undergone significant optimizations. Recent updates have reduced algorithm complexity in the member list store, resulting in better performance with large numbers of alerts and operarios. This ensures the system remains responsive even in busy environments with high alert volumes.

## The Road Ahead

While OpenFero has come a long way, the journey continues. Future development plans include:

- Further expansion of the UI capabilities
- Enhanced metrics and observability of the self-healing actions
- Introduction of own remediationJob CRD for improved configuration and validation experience
- Improved documentation and examples for common remediation scenarios

## Join the OpenFero Community

OpenFero's growth has been fueled by community interest and contributions. If the idea of automated, code-agnostic self-healing for cloud-native environments resonates with you, consider getting involved:

- Star the [OpenFero repository](https://github.com/OpenFero/openfero)
- Try it out in your environment and provide feedback
- Contribute code, documentation, or ideas
- Share your operarios examples for common remediation scenarios

I'm grateful for every contribution, be it code, discussion, feedback, or documentation. Together, we're building a more resilient future for cloud-native platforms.

Remember, in the world of OpenFero, we don't just respond to incidents—we anticipate and remediate them automatically, letting your platform heal itself while you focus on building rather than fixing.
