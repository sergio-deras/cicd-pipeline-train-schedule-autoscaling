Additional Information and Resources
Your team has built out a Jenkins Pipeline for deploying the train-schedule app to a Kubernetes cluster. Last week, a celebrity tweeted about how useful the app was. While this was great from a marketing perspective, the resulting large spike in usage caused performance degradation. Usage soon returned to normal levels, but the team wants to be prepared for the next time something like this happens.

You have been asked to implement autoscaling for the app. In the event of another usage spike, the Kubernates cluster will automatically create additional replicas of the app. Then when usage drops again, the cluster will remove the unnecessary replicas.

To do this, you will need to do the following tasks:

Install the Kubernetes metrics API in the cluster:
Clone the Kubernetes metrics repo: https://github.com/kubernetes-incubator/metrics-server.
Apply the standard configurations to install the Metrics API.
Configure a Horizontal Pod Autoscaler to autoscale the train schedule app:
Create a fork of the source code at https://github.com/linuxacademy/cicd-pipeline-train-schedule-autoscaling.
Add a CPU resource request in train-schedule-kube.yml for the pods created by the train-schedule deployment.
Define a HorizontalPodAutoscaler in train-schedule-kube.yml to autoscale in response to cpu load.
Generate some load on the app to see the autoscaler in action!
The train-schedule image from Docker Hub used for this learning activity includes a special endpoint that does some CPU-intensive calculations: /generate-cpu-load. By hitting this endpoint repeatedly, you can generate CPU load on the app containers to see autoscaling in action. A simple way to generate load is by spinning up a busybox container and running a loop inside it to call that endpoint.

Run a busybox container and open an intractive shell:

kubectl run -i --tty load-generator --image=busybox /bin/sh
Once you're inside the interactive shell, run a command to generate load:

while true; do wget -q -O- http://<kubernetes node public ip>:8080/generate-cpu-load; done
Learning Objectives
check_circle
Install the Kubernetes metrics API in the cluster.
To accomplish this, do the following:

Clone the Kubernetes metrics repo.
git clone https://github.com/kubernetes-incubator/metrics-server.git
Apply the standard configurations to install the Metrics API.
cd metrics-server/
git checkout ed0663b3b4ddbfab5afea166dfd68c677930d22e
kubectl create -f deploy/1.8+/
Wait a few seconds for the metrics server pods to start. You can see their status with kubectl get pods -n kube-system.
check_circle
Configure a Horizontal Pod Autoscaler to autoscale the train schedule app.
Check the example-solution branch of the source code repo for an example of the code changes needed in the train-schedule-kube.yml file: https://github.com/linuxacademy/cicd-pipeline-train-schedule-autoscaling/blob/example-solution/train-schedule-kube.yml.

To complete this task, you will need to do the following:

Create a fork of the source code at https://github.com/linuxacademy/cicd-pipeline-train-schedule-autoscaling.
Add a CPU resource request in train-schedule-kube.yml for the pods created by the train-schedule deployment. Check the example solution if you need to need to know where to add this.
resources:
  requests:
    cpu: 200m
Define a HorizontalPodAutoscaler in train-schedule-kube.yml to autoscale in response to CPU load.


apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: train-schedule
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: train-schedule-deployment
  minReplicas: 1
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
Generate some load on the app to see the autoscaler in action!
