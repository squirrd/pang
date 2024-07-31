# Design Decisions
[The exercise seems quite large](assignment.pdf); As an initial thought, writing all this in Golang from a blank slate is going to take a while and is not going to be very feature-rich or stable.

So I drafted some options as listed below.  **TLDR** I chose Option 3.

## Option 1

What is already available? OpenShift installs and configures the [Observabilty Operator](https://docs.openshift.com/container-platform/4.14/observability/monitoring/monitoring-overview.html) when the cluster is created ([GitHub repo](https://github.com/rhobs/observability-operator)). While [ARO has Azure Monitor for ARO](https://learn.microsoft.com/en-us/azure/openshift/howto-monitor-alerts) that already monitors cluster health. Either appear of these monitoring tools would meet the requirements of the coding exercise.

For OpenShift—Observability Operator, this would require configuring the various components of the stack using the Custom Resources defined by the operators (CRDs) or patching the config maps for the operator. This could be deployed to the fleet using Hive.

This would have limited oportunity for go programming and does not meet the objective.

> `Create` a metrics collection and monitoring system in Go that captures system metrics, stores them, and provides a way to query and visualise the metrics.

I assist our customers in configuring operators and features in OpenShift daily in MCS, and it would not be considered a go coding challenge. But it would be the simplest way to meet the requirements.

## Option 2
If these freely available options are not suitable - Then the next option would be to build a go tool that interacts with the OpenShift Obserability Operator to modify the multiple CRs and config maps.  The tool would simplify the input from many options in multiple CRs and config maps down to a simple/single YAML file.  

This could be developed as an operator using the go operator SDK. The operator would define one CRD to manage all the CRs and config maps in the Observability Operator stack.

If building this using an operator, it would be more of a translation exercise, mapping the requirements onto the new operator CRD and then translating that to the multiple CRDs that make up the observability stack. This is a way to obfuscate technical details from, say, our customers or to restrict customers from making poor decisions that break SREs ability to manage the cluster.  There would have been some go coding, but again, it does to meet the objective of `Creating` a system.

## Option 3
Build an operator that creates a metrics system based on already available open-source components. Deploying these components via an operator would require building a controller or manager that ensures the proper operation of the monitoring stack. This seems to meet the objectives of "Creating a system and using Go".

The following components would meet the requirements.
-   Metrics Collection:
    - Prometheus/node_exporter - Collects a range of node metrics
- Data Storage:
    - Prometheus - Contains a time-series database designed for logging large amounts of metric data
- API Endpoint:
    - Prometheus - Contains an API endpoint for querying its datastore, which includes aggregation
- Alerting Mechanism:
  Prometheus - Can generate alerts via custom alerting rules. For bonus points, integrate Prometheus/Alertmanager for better control over alerting.
- Visualization:
    - Graphana - Provides powerful tools for visualising and analysing data from sources like Prometheus. There is a graphing utility in Prometheus, but its limited and does not 

## Option 4
This option is discussed at the top but repeated here to complement the other options. It would highlight my Go API and Javascript skills, but it would be extremely time-consuming, feature-limited, and prone to bugs. This option would only be suitable where there is no operator available, no components to architect an operator solution, or the operator's feature is simple, like the Must-Gather operator.
