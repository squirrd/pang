# Plan
The plan is to 
- Combine all the components into a working system on OpenShift
	- Use the OpenShift object to deploy the objects
	-  Store these objects in a single object definition (YAML) file
	- Testing this can then be done by running
    ~~~
	oc create -f `pang_deploy.yml`
	~~~
- Once the pang (Prometheus, AlertManager, NodeExporter, Grafana can be deployed)
	- Use the OpenShift Operator SDK to create an operator.
		- Create a Custom Resource Definition for the operator that enables the user of a Customer Resource to modify the operator and allow it to meet the exercise's requirements.
		- This CRD is then used to define the arguments of the various components.
			- This is where `pang_deploy.yml` above is written into the operator as a template that allows the attributes from a Custom Resource to configure the operator's behaviour.
		- Modify the controller to ensure the objects behave as expected, reboot after CR change has occurred, self-heal, etc...

# Reality

 ### Combine all the components into a working system 
 
 - Build a lab cluster
 - Start by building the OCP objects for Prometheus
	 - Initially, a deployment object for the image
	 - It wouldn't start because the RBAC was the default secured role in the default Service Account
	 - Prometheus needs to be able to ask questions of the cluster, like how many and where are all your nodes so it can find and identify all the node_exporters to scrape metrics.  It needs to do the same for all the other exporters that are installed on the system
	 - I chose to steal the Role definition from the Prometheus image running in the Observability Operator
	 - I then needed to create a new Service Account and a Role Binding
	 - I then needed to add persistent storage so the stored data would survive a pod restart.
		 - This was by adding a PVC
		 - Configuring the deployment to add volumes and mounts is also needed. 
 - I then moved to the node_exporter, which proved to be challenging and very time-consuming.
	 - First, it had a similar role issue to Prometheus.  This was relatively easy to identify and remediate
	 - Next, the node exporter requires access to node-level metrics. This can't be done with a standard pod, as OpenShift tries to protect itself and other workloads by restricting access to these node-level resources.
		 - The existing node_exporter used a custom SCC.  I could get the new exporter to run using the other operators’ SCC, but this would be an anti-pattern because if the Observability Operator were to be removed in the future, my new operator would fail as it would be looking for an SCC that no longer exists.
		 - Trying to replicate the SCC failed. So, due to time constraints and this operator will never be deployed in anger, I decided to substitute the privileged SCC. This worked as expected
	 - Finally, the new exporter competes to access port 9100 with the existing exporter. This occurs because both the new and the old exporters run with a privileged host network (hostNetwork: true) so they are not in segmented pod networks where they can have the same port. This is so they can access the network metrics of the node.
		 - Moving the new exporter to 19100 was relatively easy once the issue was discovered
	 - This had a knock-on effect that Prometheus was now scraping the wrong ports  and this probably is the reason why Prometheus was not scraping metrics 
		 - Changing the port number should be simpler than this, but based on checks this did not work
~~~
 scrape_configs:
- job_name: 'node_exporter'
kubernetes_sd_configs:
  - role: node
relabel_configs:
  - source_labels: [__meta_kubernetes_node_label_kubernetes_io_hostname]
    target_label: instance
  - source_labels: [__address__]
    target_label: __address__
    replacement: '${1}:19100'
  ~~~

At this point, I felt like I was running out of time to complete the exercise.  And so, I chose to move on the next step as I had two pods that were running, and I might as well make an appropriator out of what I had.  

### Use the OpenShift Operator SDK to create an operator
I didn't get ver far with this.  Even though my research indicated that this would be possible on an MBP running on silicone, [this does not appear to be the case](https://docs.openshift.com/container-platform/4.14/operators/operator_sdk/osdk-installing-cli.html#osdk-installing-cli:~:text=For%20the%20amd64%20and%20arm64%20architectures%2C%20navigate%20to%20the%20OpenShift%20mirror%20site%20for%20the%20amd64%20architecture%20and%20OpenShift%20mirror%20site%20for%20the%20arm64%20architecture%20respectively.).

> 1.  For the  `amd64`  and  `arm64`  architectures, navigate to the  [OpenShift mirror site for the  `amd64`  architecture](https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/operator-sdk/)  and  [OpenShift mirror site for the  `arm64`  architecture](https://mirror.openshift.com/pub/openshift-v4/arm64/clients/operator-sdk/)  respectively.

And when running it on my MBP I received the following warning and panic's )-:

   

~~~
% operator-sdk init --domain=squirrell.com --repo=github.com/squirrd/pang
        .
WARN[0000] the platform of this environment (darwin/arm64) is not suppported by kustomize v3 (v3.8.7) which is used in this scaffold. You will be unable to download a binary for the kustomize version supported and used by this plugin. The currently supported platforms are: ["linux/amd64" "linux/arm64" "darwin/amd64"]
        
AND

mkdir -p /Users/dsquirre/Repos/pang/op/bin
    test -s /Users/dsquirre/Repos/pang/op/bin/controller-gen && /Users/dsquirre/Repos/pang/op/bin/controller-gen --version | grep -q v0.11.1 || \
    	GOBIN=/Users/dsquirre/Repos/pang/op/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.11.1
    /Users/dsquirre/Repos/pang/op/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
    panic: runtime error: invalid memory address or nil pointer dereference [recovered]
    	panic: runtime error: invalid memory address or nil pointer dereference
    [signal SIGSEGV: segmentation violation code=0x2 addr=0x0 pc=0x1011a0a64]
~~~    


# References
## Componets
**Prometheus**
GitHub Repository - https://github.com/prometheus/prometheus
Image Repository - https://quay.io/repository/prometheus/prometheus
Docs - https://prometheus.io/docs/prometheus/latest/getting_started/

**Node Exporter**
GitHub Repository - https://github.com/prometheus/node_exporter
Image Repository - https://quay.io/repository/prometheus/node-exporter
Docs - https://prometheus.io/docs/prometheus/latest/getting_started/

**Alert Manager**
GitHub Repository - https://github.com/prometheus/alertmanager
Image Repository - https://quay.io/repository/prometheus/alertmanager
Docs - https://prometheus.io/docs/prometheus/latest/getting_started/

**Grafana**
GitHub Repository - https://github.com/grafana/grafana
Image Repository - https://hub.docker.com/r/grafana/grafana-enterprise
Docs - https://grafana.com/docs/grafana/latest/

## Operator SDK
OpenShift Docs - https://docs.openshift.com/container-platform/4.14/operators/operator_sdk/osdk-about.html