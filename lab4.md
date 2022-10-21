You are going to deploy the Datadog Agent using the Datadog Operator

# lab4

## Deploying the Datadog Operator
You are going to deploy the Datadog Agent using the Datadog Operator.

By using the Operator, you can use a single Custom Resource Definition (CRD) to deploy the node-based Agent, Cluster Agent, and Cluster Checks Runner. The Operator reports deployment status, health, and errors in the Operator’s CRD status.

Install the Datadog Operator with Helm by running the following commands (click Enter after copying the commands on the terminal):
```
helm repo add datadog https://helm.datadoghq.com
helm repo update
helm install my-datadog-operator datadog/datadog-operator --version=0.8.2
```
You can check the operator pod running executing the following command:
```
kubectl get pods
```
You should get an output similar to:
```
NAME                                     READY   STATUS    RESTARTS     AGE
my-datadog-operator-858b665ff5-lfxd9     1/1     Running   0            58s
```
Wait until the pod is running before continuing to the next section.

## Deploy the Datadog Agent
Now that the Datadog Operator is running in our cluster, you can deploy the Agent using a DatadogAgent definition.

First, create a Kubernetes secret with your Datadog API and app keys by executing the following command:
```
kubectl create secret generic datadog-secret --from-literal api-key=$DD_API_KEY --from-literal app-key=$DD_APP_KEY
```
Open the Editor tab and select the datadog-basic.yaml file and explore the contents.

You can see the part where the Datadog credentials are being set up referencing the secret that was just created:
```
credentials:
  apiSecret:
    secretName: datadog-secret
    keyName: api-key
  appSecret:
    secretName: datadog-secret
    keyName: app-key
```
The definition also contains three options for the Datadog Agent:
```
apm:
  enabled: true
log:
  enabled: true
  logsConfigContainerCollectAll: true
process:
  enabled: true
  processCollectionEnabled: true
```
Those options enable APM, the process, and the log agent.

Deploy the Datadog Agent by applying that DatadogAgent configuration:
```
kubectl apply -f /root/lab/manifest-files/datadog/datadog-basic.yaml
```
Check the workloads that have been deployed. Run the following command:
```
kubectl get deployments -l app.kubernetes.io/name=datadog-agent-deployment
```
You should get an output similar to the following one:
```
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
datadog-cluster-agent        1/1     1            1           5s
```
You should get an output similar to the following one:
```
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
datadog-cluster-agent        1/1     1            1           5s
```
The Datadog Cluster Agent might take a while to start running, so if you are getting an output similar to the following:
```
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
datadog-cluster-agent   0/1     1            0           3s
```
then rerun the command until the Cluster Agent pod shows as READY.

Once the Cluster Agent is running, check the Node Agent Daemonset. Run the following command:
```
kubectl get daemonset
```
You should get an output similar to the following one:
```
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
datadog   1         1         1       1            1           kubernetes.io/os=linux   10s
```
If you don't get a Daemonset yet, it is because the Cluster Agent is still getting started, and the Operator waits for the Cluster Agent to be running before creating the Daemonset. Wait some seconds and rerun the command above.

The operator, by default, deploys 2 workloads: the Datadog Node Agent and the Datadog Cluster Agent.

The Node Agent is a DaemonSet, so it should have been deployed to every node in the cluster, but it only got deployed to the worker node. This is because the control plane node has a default taint:
```
kubectl get nodes kubernetes -o jsonpath="{.spec.taints}"
```
Modify the DatadogAgent definition to include the corresponding toleration. Open the Editor tab and select the file called datadog-tolerations.yaml. Check the differences between this file and the previous one.

You can also check the differences between these two files by executing the following command:
```
diff -U1 --color /root/lab/manifest-files/datadog/datadog-basic.yaml /root/lab/manifest-files/datadog/datadog-tolerations.yaml
```
```
--- datadog-basic.yaml	2022-07-11 15:19:46.000000000 +0200
+++ datadog-tolerations.yaml	2022-07-11 16:04:21.000000000 +0200
@@ -14,2 +14,6 @@
     config:
+      tolerations:
+        - effect: NoSchedule
+          key: node-role.kubernetes.io/master
+          operator: Exists
       kubelet:
```
As you can see, the corresponding toleration to the node agent configuration has been added. Apply this new YAML file:
```
kubectl apply -f /root/lab/manifest-files/datadog/datadog-tolerations.yaml
```
You can see now that there are two Datadog node agents running, one for for the control plane node, one for the worker node, ensuring full visibility for our cluster:
```
kubectl get pods -l agent.datadoghq.com/component=agent -o wide
```
```
NAME                  READY   STATUS    RESTARTS   AGE   IP               NODE         NOMINATED NODE   READINESS GATES
datadog-agent-cxtwq   3/3     Running   0          12m   192.168.171.69   worker       <none>           <none>
datadog-agent-g8xxp   3/3     Running   0          12m   192.168.192.73   kubernetes   <none>           <none>
```

## Fixing the etcd Check

Autodiscovery is a Datadog feature that identifies automatically the workloads running in a container and enables its corresponding integrations to gather relevant metrics for those services.

The Node Agent running on the control plane node discovers the Kubernetes components running on it and enables those integrations. You can check what integrations are discovered by running the following command:
```
kubectl exec -ti $(kubectl get pods -l agent.datadoghq.com/component=agent --field-selector spec.nodeName=kubernetes -o name) -- agent configcheck
```
On that output you will see integrations like kubelet, kube_scheduler, kube_controller_manager, kube_apiserver_metrics, etcd and others related to system metrics.

Let's now run the agent status command to check the status of those integrations:
```
kubectl exec -ti $(kubectl get pods -l agent.datadoghq.com/component=agent --field-selector spec.nodeName=kubernetes -o name) -- agent status
```
As you browse the output of that command, you will see that the etcd check is failing:
```
Check Initialization Errors
===========================


    etcd (3.2.0)
    ------------

    instance 0:

      could not invoke 'etcd' python check constructor. New constructor API returned:
Traceback (most recent call last):
File "/opt/datadog-agent/embedded/lib/python3.8/site-packages/datadog_checks/etcd/etcd.py", line 87, in __init__
  super(Etcd, self).__init__(
File "/opt/datadog-agent/embedded/lib/python3.8/site-packages/datadog_checks/base/checks/openmetrics/base_check.py", line 126, in __init__
  raise CheckException(
datadog_checks.base.errors.CheckException: The agent could not connect to any of the following URLs: ['https://10.132.1.220:2379/metrics', 'http://10.132.1.220:2379/metrics'].
```
The reason the Datadog Agent cannot communicate with the etcd pod is that it doesn't have access to the needed certificates. You will be following the official documentation where it is recommended mounting the host path where the etcd certificates are located, and modify the check configuration through a ConfigMap.

Open the Editor tab, select the datadog-etcd.yaml file, and explore the contents. Check the differences between this file and the previous one.

You can also check the differences between these two files by executing the following command:
```
diff -U1 --color /root/lab/manifest-files/datadog/datadog-tolerations.yaml /root/lab/manifest-files/datadog/datadog-etcd.yaml
```
Let's break down what this new configuration is trying to achieve.

By adding the confd key to our DatadogAgent definition you are telling the agent where to look for additional checks configuration (in this case, in a ConfigMap).
```
config:
  confd:
    configMapName: datadog-checks
```
By adding the following volume and volume mounts you are disabling the default etcd configuration:
```
volumes:
- hostPath:
- name: disable-etcd-autoconf
  emptyDir: {}
volumeMounts:
- name: disable-etcd-autoconf
  mountPath: /etc/datadog-agent/conf.d/etcd.d
```
By adding the following volume and volume mounts we are mounting the etcd certificates folder into the Datadog Agent pods:
```
volumes:
- hostPath:
    path: /etc/kubernetes/pki/etcd
  name: etcd-certs
volumeMounts:
- name: etcd-certs
  mountPath: /host/etc/kubernetes/pki/etcd
  readOnly: true
```
Finally, the ConfigMap with the new configuration for the etcd check is created, passing the certificates and keys path from the Agent volume mounts:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: datadog-checks
data:
  etcd.yaml: |-
    ad_identifiers:
      - etcd
    init_config:
    instances:
      - prometheus_url: https://%%host%%:2379/metrics
        tls_ca_cert: /host/etc/kubernetes/pki/etcd/ca.crt
        tls_cert: /host/etc/kubernetes/pki/etcd/server.crt
        tls_private_key: /host/etc/kubernetes/pki/etcd/server.key
```
