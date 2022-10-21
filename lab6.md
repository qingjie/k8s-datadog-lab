# Lab6

## The kubernetes view in Datadog

You will start browsing the data we are getting in our Datadog account, starting with the high level view of Kubernetes cluster.

Navigate to Infrastructure > Kubernetes and browse around.

~[](lab6-img1.png)

What type of information do you get?

You can see that you get an overview of the different clusters that you are monitoring with Datadog, as well as the number of other Kubernetes resources like Deployments, Pods or Namespaces.

Drill down to the list of Pods in our cluster by hovering on Pods, then clicking on "View in Explorer":

~[](lab6-img2.png)

~[](lab6-img3.png)

You can see the list of Pods running on your cluster. Having a view of all pods running in your cluster can be useful to be able to check at a glance that everything is running properly, but as the number of pods grows, you need to start grouping and filtering to have a more comprehensive understanding of the state of your cluster.

EXERCISE: Try to use this view to get all pods in the kube-system namespace grouped by node. You should get something similar to this:

~[](lab6-img4.png)

Continue browsing the Kubernetes Explorer page. What other resources are you able to see here?

## The Kubernetes Integration DashBoards

Once you start monitoring a Kubernetes cluster in Datadog, the Kubernetes integrations will be enabled and Kubernetes related out-of-the-box dashboards will appear in your list of Dashboards. Navigate to Dashboards > Dashboard List:

~[](lab6-img5.png)

If the list of Kubernetes dashboards is not available in your Datadog account, enable them by installing the Kubernetes integration.

Navigate to the Integrations > Integrations and search for Kubernetes. Click on the "Install Integration" button:

~[](lab6-img6.png)

NOTE: There are multiple integrations for Kubernetes. Install all of them to get all dashboards:

~[](lab6-img7.png)

These dashboards will help you understand your Kubernetes clusters better, and they include relevant metrics for several components and Kubernetes resources.

Navigate to Dashboards > Dashboard List to search for the Kubernetes Cluster Overview dashboard. Have a look to the different visualizations it offers.

One interesting widget is the one called Cluster Health Overview, in which you will be able to easily see how much capacity remains in your cluster (CPU, Memory and number of pods):

## The live contrainers view in Datadog

The high level view of your Kubernetes cluster is very useful to pinpoint potential problems in your cluster, but sometimes it is necessary to have a more detailed view of each of the containers running in your infrastructure.

Navigate to Infrastructure > Containers. The Live Containers view gives you an overview of all containers running in your infrastructure, and their live usage for CPU and memory.

Understanding your workloads' CPU and memory consumption can help you plan for capacity and growth in your Kubernetes cluster.

EXERCISE: Find the container running in the worker node with the highest memory usage.

## The Processes View in Datadog

To debug a potential issue it is sometimes useful to see exactly what process a container is running. A common method to do this is to SSH into the node they are running on and perform a ps command, but this is not very productive when you have dozens of hosts and hundreds of pods and containers.

The Datadog Process Agent is collecting process information that you can filter in the Live Process view, allowing you to find the process you are looking for. Navigate to Infrastructure > Processes.

EXERCISE: In the process view, filter by host to get only containers running in the control plane node, and get the different options that the Kubernetes API server is running with.




