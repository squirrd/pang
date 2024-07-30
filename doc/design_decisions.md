# Design Decisions

[The exercise seems quite large](assignment.pdf); it's more than a couple of hours' work. Writing all this in Golang from a blank slate is not going to be very feature-rich or stable.

Thinking outside the blank slate coding challenge…. How can I implement something like this in a matter of days?  Here are a few options


## Option 1
What is already available? OpenShift installs and configures the [Observabilty Operator](https://docs.openshift.com/container-platform/4.14/observability/monitoring/monitoring-overview.html) when the cluster is created ([GitHub repo](https://github.com/rhobs/observability-operator)).  While [ARO has Azure Monitor for ARO](https://learn.microsoft.com/en-us/azure/openshift/howto-monitor-alerts)  that already monitors cluster health.  Either appear of these monitoring tools would meet the requirements of the coding exercise.

For OpenShift - Observability Operator, this would require configuring the various components of the stack by using the Custom Resources defined by the operators (CRD’s) or by patching the config maps for the operator.  Configuring Operators and Features in OpenShift is what I do daily in MCS, and it would not be considered a coding challenge. But it would be the simplest way to meet the requirements.

## Option 2
If these freely available options are not suitable - Then the next option would be to build a go tool that interacts with the OpenShift Obserability Operator to modify the multiple CRs and config maps.  The tool would simplify the input from many options in multiple CRs, and config maps down to a simple set of questions or a single YAML file.  The answers or YAML file would then deploy these changes to the Observability Operator CRs.

This go tool could then be extended or developed as an operator using the go operator SDK. The operator would define that defines one CRD to manage all the CRs in the Observability Operator.  This is almost what the Observability Operator does by combining a number of existing operators into a single operator.

Option 2 would not require much coding apart from defining a CR and the ability to map the CR to template CRs for the Observability Operator and then deploy the generated CRs.  This option probably does not meet the Objective of the exercise “Create a metrics collection and monitoring system in Go” as this operator would only “leverage the existing Observibilty Operator” and not create a new system.

### Option 3
Create an operator that “Creates a … system” based on already available open source components.  The following components would meet the requirements

-   Metrics Collection:
    - Prometheus/node_exporter - Collects a range of node statistics
- Data Storage:
    - Prometheus - Contains a timeseries database designed for logging a large columns of metric data
- API Endpoint:
    - Prometheus - Contains an API endpoint for querying its datastore, which includes aggregation
- Alerting Mechanism:
  - Prometheus - Can generate alerting via custom alerting rules. For bonus points integrate Prometheus/Alertmanager for better control over alerting.
- Visualization:
    - Graphana - Provides powerful tools for visualizing and analyzing data from various sources like Prometheus.

## Option 4
Discussed above but repeated as an option to complement the options list. Start with a blank slate.  This would highlight my go API and javascript skills, but this is an option but it would be extremely time consuming, feature limited and prone to bugs - not that my code is buggy :-)  This option would only be suitable where there is no operator available, or no components to architect an operator solution or the the feature of the operator is simple, like the Must-Gather operator.
