# devops-task5-Prometheus-grafana-kubernetes

Problem Statement
Integrate Prometheus and Grafana and perform in following way:

1. Deploy them as pods on top of Kubernetes by creating resources Deployment, ReplicaSet, Pods or Services

2. And make their data to be remain persistent

3. And both of them should be exposed to outside world.


**STEP 1: First we start minikube using following command on windows command prompt.**
           minikube start 
    
    
  **STEP 2: ConfigMap**

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Podscan consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume. ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable.


apiVersion: v1
kind: ConfigMap
metadata:
  name: prom-server-config
  labels:
    env: prom-metrics
data:
 prometheus.yml: |-
  global:
    scrape_interval: 15s
    evaluation_interval: 15s
  alerting:
    alertmanagers:
    - static_configs:
      - targets: []
      scheme: http
      timeout: 10s
      api_version: v1
  scrape_configs:
  - job_name: prometheus
    static_configs:
    - targets: ['localhost:9090']
      
  - job_name: node1
    static_configs:
    
    - targets: ['192.168.99.103:9100']
    
  **STEP 3: PersistentVolumeClaim**
   First we create PVC for both Prometheus and grafana to keep remain their data persistent. A PersistentVolumeClaim (PVC) is a request for storage by a user.
                                          
                                          
                                      apiVersion: v1
                                      kind: PersistentVolumeClaim
                                      metadata:
                                        name: grafana-pvc
                                        labels:
                                          env: grafana-pvc1

                                      spec:
                                        accessModes:
                                        - ReadWriteMany
                                        resources:
                                          requests:
                                            storage: 2Gi
                                      ------------------------
                                      apiVersion: v1
                                      kind: PersistentVolumeClaim
                                      metadata:
                                        name: prom-storage-pvc
                                        labels:
                                          env: prom-pvc
                                      spec:
                                        accessModes:
                                         - ReadWriteMany
                                        resources:  
                                          requests:

                                      storage: 2Gi


**STEP 4: Deployment**
Now we create deployment for both Prometheus and Grafana .

A Deployment provides declarative updates for Pods and Replica-Sets. The Replica-Set creates Pods in the background. We use Deployment to Roll-out a Replica-Set.

                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: prometheus-deploy
                      labels:
                        env: prom-metrics

                    spec:
                      selector:
                        matchLabels:
                          env: prom-metrics

                      replicas: 1
                      template:
                        metadata:
                          name: prometheus-pod
                          labels:
                            env: prom-metrics

                        spec:
                          containers:
                          - name: prometheus-container
                            image: vimal13/prometheus
                            volumeMounts:
                            - name: prom-config-volume
                              mountPath: /etc/prometheus/
                            - name: prom-storage-volume
                              mountPath: /prometheus/
                          volumes:
                          - name: prom-config-volume
                            configMap:
                              name: prom-server-config
                          - name: prom-storage-volume
                            persistentVolumeClaim:
                              claimName: prom-storage-pvc
                    ---
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: grafana-deploy
                      labels:
                        env: grafana

                    spec:
                      selector:
                        matchLabels:
                          env: grafana

                      replicas: 1
                      template:
                        metadata:
                          name: grafana-pod
                          labels:
                            env: grafana

                        spec:
                          containers:
                          - name: grafana-container
                            image: vimal13/grafana
                            volumeMounts:
                            - name: grafana-storage-volume
                              mountPath: /var/lib/grafana
                          volumes:
                          - name: grafana-storage-volume
                            persistentVolumeClaim:
                              claimName: grafana-pvc
 **STEP 5: Service**

Now we create service for both Prometheus and grafana to expose our application to outside world using yml file.

                       apiVersion: v1
                kind: Service
                metadata:
                  name: prometeus-service
                spec:
                  type: NodePort
                  selector:
                    env: prom-metrics
                  ports:
                  - port: 9090
                    targetPort: 9090
                    nodePort: 31003
                ---
                apiVersion: v1
                kind: Service
                metadata:
                  name: grafana-service
                  labels:
                    env: grafana

                spec:
                  selector:
                    env: grafana
                  type: NodePort
                  ports:
                  - port: 3000
                    targetPort: 3000 

                nodePort: 32000      
             
**Step 6: Kustomization**
Here we specify all the files in sequence so that we can run just 1 file to run all the files.
                
              apiVersion: kustomize.config.k8s.io/v1beta1
              apiVersion: kustomize.config.k8s.io/v1beta1
              kind: Kustomization
              resources:
                - prom_service.yml
                - prom_pvc.yml
                - prom_config.yml
                - prometheus.yml
              ---
              apiVersion: kustomize.config.k8s.io/v1beta1
              kind: Kustomization
              resources:
                - grafana_svc.yml
                - grafana_pvc.yml
                - grafana_deploy.yml
               
                
**Finally our setup is configured on top of kubernetes.**
**To run Prometheus we use following IP and Port.**
   http://localhost:31003
   
 **To run Grafana we use following IP and Port.**
  http://localhost:32000

           
