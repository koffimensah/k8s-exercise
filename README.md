# k8s-exercise

PetClinic Kubernetes Deployment
A comprehensive guide to deploying the Spring PetClinic application on Kubernetes using Docker Desktop and Minikube.
Table of Contents

Overview
Prerequisites
Architecture
Quick Start
Deployment Components
Accessing the Application
Managing the Deployment
Troubleshooting
Useful Commands

Overview
This project demonstrates deploying the Spring PetClinic application on Kubernetes with:

ConfigMap for non-sensitive configuration
Secret for sensitive data
Deployment with 3 replicas for high availability
Service for load balancing and external access

Application Details:

Image: koffimensah/petclinic:v1.0
Port: 8080 (container)
Framework: Spring Boot with Maven

Prerequisites
Required Software

Docker Desktop with Kubernetes enabled
Minikube (for local development)
kubectl CLI tool

Verify Installation
bash# Check Docker Desktop Kubernetes
kubectl config current-context

# Check Minikube
minikube status

# Verify kubectl
kubectl version --client
Architecture
┌─────────────────────────────────────────┐
│         Kubernetes Cluster              │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │  Service (NodePort)               │ │
│  │  petclinic-service:30085          │ │
│  └──────────────┬────────────────────┘ │
│                 │                       │
│  ┌──────────────┴────────────────────┐ │
│  │  Deployment: petclinic-deployment │ │
│  │  Replicas: 3                      │ │
│  │                                   │ │
│  │  ┌─────┐  ┌─────┐  ┌─────┐       │ │
│  │  │ Pod │  │ Pod │  │ Pod │       │ │
│  │  └─────┘  └─────┘  └─────┘       │ │
│  │     ↓         ↓         ↓         │ │
│  │  ┌──────────────────────────┐    │ │
│  │  │  ConfigMap & Secret      │    │ │
│  │  │  (Environment Variables) │    │ │
│  │  └──────────────────────────┘    │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
Quick Start
1. Clone or Download the Manifest
Save the deployment manifest as petclinic-k8s.yaml.
2. Customize Secrets (Optional)
Encode your secret values:
bash# Generate base64 encoded values
echo -n 'your-database-password' | base64
echo -n 'your-api-key' | base64
echo -n 'your-jwt-secret' | base64
Update the Secret section in petclinic-k8s.yaml with your encoded values.
3. Deploy to Kubernetes
bash# Apply all resources
kubectl apply -f petclinic-k8s.yaml

# Watch deployment progress
kubectl get pods -w
4. Wait for Pods to Start
The application takes 4-5 minutes to start because it builds from source using Maven. Watch for pods to show 1/1 READY:
bash# Check pod status
kubectl get pods

# Expected output after ~4 minutes:
# NAME                                    READY   STATUS    RESTARTS   AGE
# petclinic-deployment-xxxxx-xxxxx        1/1     Running   0          5m
Deployment Components
ConfigMap (app-config)
Stores non-sensitive configuration:

APP_ENV: Application environment (production)
LOG_LEVEL: Logging level (info)
PORT: Application port (8080)
DATABASE_HOST: Database hostname
DATABASE_PORT: Database port
REDIS_HOST: Redis hostname
REDIS_PORT: Redis port

Secret (app-secret)
Stores sensitive data (base64 encoded):

DATABASE_PASSWORD: Database password
API_KEY: API authentication key
JWT_SECRET: JWT signing secret

Deployment (petclinic-deployment)

Replicas: 3 pods for high availability
Image: koffimensah/petclinic:v1.0
Resources:

Requests: 512Mi memory, 250m CPU
Limits: 1Gi memory, 500m CPU


Health Checks:

Liveness Probe: Initial delay 300s (5 min)
Readiness Probe: Initial delay 240s (4 min)



Service (petclinic-service)

Type: NodePort
Port: 8085 (service port)
TargetPort: 8080 (container port)
NodePort: 30085 (external access)

Accessing the Application
Method 1: Minikube Service (Recommended)
bash# Automatically open in browser
minikube service petclinic-service
Method 2: Direct NodePort Access
bash# Get Minikube IP
minikube ip

# Access at http://<minikube-ip>:30085
# Example: http://192.168.49.2:30085
Method 3: Port Forwarding
bash# Forward to localhost
kubectl port-forward service/petclinic-service 8085:8085

# Access at http://localhost:8085
Managing the Deployment
Scaling the Application
bash# Scale to 5 replicas
kubectl scale deployment petclinic-deployment --replicas=5

# Scale back to 3
kubectl scale deployment petclinic-deployment --replicas=3

# Verify scaling
kubectl get pods
Updating the Application
bash# Update to a new version
kubectl set image deployment/petclinic-deployment petclinic=koffimensah/petclinic:v2.0

# Check rollout status
kubectl rollout status deployment/petclinic-deployment

# View rollout history
kubectl rollout history deployment/petclinic-deployment
Rolling Back
bash# Rollback to previous version
kubectl rollout undo deployment/petclinic-deployment

# Rollback to specific revision
kubectl rollout undo deployment/petclinic-deployment --to-revision=2
Updating Configuration
bash# Edit ConfigMap
kubectl edit configmap app-config

# Edit Secret
kubectl edit secret app-secret

# Restart pods to pick up changes
kubectl rollout restart deployment/petclinic-deployment
Troubleshooting
Pods Not Starting
Check pod status:
bashkubectl get pods
kubectl describe pod <pod-name>
Common issues:

CrashLoopBackOff: Check logs with kubectl logs <pod-name> --previous
ImagePullBackOff: Verify image name and availability
Pending: Check resource availability with kubectl describe node

Application Not Accessible
Check service:
bash# Verify service is running
kubectl get svc petclinic-service

# Check endpoints
kubectl get endpoints petclinic-service

# If endpoints are empty, pods aren't ready yet
Verify health checks:
bash# Check if readiness probe has passed
kubectl describe pod <pod-name> | grep -A 10 Conditions
Slow Startup
The application takes 4-5 minutes to start because it:

Downloads Maven dependencies
Compiles the source code
Runs validation checks
Starts the Spring Boot application

Monitor startup:
bash# Follow logs in real-time
kubectl logs -f <pod-name>

# Check last 50 lines
kubectl logs <pod-name> --tail=50
Port Already in Use
If port 8085 or 30085 is in use, edit petclinic-k8s.yaml:
yamlspec:
  ports:
  - protocol: TCP
    port: 8086         # Change this
    targetPort: 8080
    nodePort: 30086    # Change this (30000-32767)
Then reapply:
bashkubectl apply -f petclinic-k8s.yaml
Useful Commands
Viewing Resources
bash# Get all resources
kubectl get all

# Get pods with labels
kubectl get pods -l app=petclinic

# Get detailed pod info
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>

# View logs from all pods
kubectl logs -l app=petclinic --tail=50
Executing Commands in Pods
bash# Open shell in pod
kubectl exec -it <pod-name> -- /bin/bash

# Run single command
kubectl exec <pod-name> -- env
kubectl exec <pod-name> -- curl localhost:8080
Resource Management
bash# View resource usage
kubectl top pods
kubectl top nodes

# Get events
kubectl get events --sort-by=.metadata.creationTimestamp

# View deployment status
kubectl get deployment petclinic-deployment
Cleanup
bash# Delete all resources
kubectl delete -f petclinic-k8s.yaml

# Or delete individually
kubectl delete deployment petclinic-deployment
kubectl delete service petclinic-service
kubectl delete configmap app-config
kubectl delete secret app-secret
Environment Variables Reference
From ConfigMap
VariableValueDescriptionAPP_ENVproductionApplication environmentLOG_LEVELinfoLogging verbosityPORT8080Application portDATABASE_HOSTdb-serviceDatabase hostnameDATABASE_PORT5432Database portREDIS_HOSTredis-serviceRedis hostnameREDIS_PORT6379Redis port
From Secret
VariableDefault (base64)DescriptionDATABASE_PASSWORDcGFzc3dvcmQxMjM=Database passwordAPI_KEYbXktc2VjcmV0LWFwaS1rZXk=API keyJWT_SECRETand0LXNlY3JldC10b2tlbi0xMjM=JWT secret
Best Practices

Never commit secrets - Keep sensitive data out of version control
Use namespaces - Separate environments (dev, staging, prod)
Set resource limits - Prevent resource exhaustion
Implement health checks - Enable automatic recovery
Use labels - Organize and select resources efficiently
Monitor logs - Track application behavior
Regular backups - Backup ConfigMaps and Secrets

Production Considerations
For production deployments, consider:

Pre-built images - Build the application during image creation, not at runtime
External databases - Don't run databases in the same cluster
Persistent storage - Use PersistentVolumes for stateful data
Ingress controller - Use Ingress instead of NodePort for production traffic
TLS/SSL - Enable HTTPS with certificates
Monitoring - Integrate Prometheus and Grafana
Auto-scaling - Use Horizontal Pod Autoscaler (HPA)
Resource quotas - Limit resource consumption per namespace

Support and Resources

Spring PetClinic: https://github.com/spring-projects/spring-petclinic
Kubernetes Documentation: https://kubernetes.io/docs/
kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
Docker Desktop: https://docs.docker.com/desktop/
Minikube: https://minikube.sigs.k8s.io/docs/

License
This deployment configuration is provided as-is for educational purposes.