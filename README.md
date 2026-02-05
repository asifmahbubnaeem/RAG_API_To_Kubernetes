

# Deploy a RAG API to Kubernetes

**Project Link:** [View Project](http://learn.nextwork.org/projects/ai-devops-kubernetes)

**Author:** asif naeem  
**Email:** asifnaim0123@gmail.com

---

---

## Introducing Today's Project!

In this project, we will deploy our RAG api using kubernetes.  We are doing this project to learn how kubernetes works and why people use them today. Kubernetes will help us with managing and scaling containsers - which is extremely useful at scale. i.e. when we have got a ton of containers to co-ordinate.

### Key services and concepts

Key concepts I learnt include Kubernetes, Minikubes, Clusters, Deployments, Serrvices, NodePorts, self-healing, 

### Challenges and wins

This project took me approximately two hours. The most challenging part was understanding the components that make the service works, along with setting up the service in linux/ubuntu platform which has some dissmilarities with mac eneveronment setup. It was most rewarding to test the cluster setup and run the service from it, also to check the self-healing actions in Kubernetes cluster when I manually deleted a Pod.

### Why I did this project

I did this project because I wanted to learn Kubernetes and have hands on experience. One thing we can apply from this is how to run and access containseriesed applications those are running in Kubernetes clusters.(i.e using Deployments and Services)

---

## Setting Up My Docker Image

In this step, we are setting up a Docker image, which is a package that contains our app's code, dependencies and settings etc. We need a Docker image because Kubernetes requires one so that it knows what kind of containers it is running, scaling, managing etc.

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_i9j0k1l2)

### What the Docker image contains

I ran docker images and saw that I have created a docker image named as rag-app. The image size is 1.34 GB The IMAGE ID is 46fdd6d273ea

### Docker image vs container

---

## Installing Kubernetes Tools

In this step, We are installing kubectl and Minikube. We need these tools because we are using Minikube to start and run the kubernetes clusters locally and kubectl is useful for running commands that lets us control our clusters e.g. view logs, show what services and pods are running.

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_u1v2w3x4)

### Verifying the tools are installed

We have installed Minikube using homebrew with simple command : 'brew install minikube'. We have installed kubectl by command 'brew install kubectl'. Both  were super easy and we could download them in minutes. We also have verfied the installation successfully by chcki g the version number for each

### Minikube vs kubectl

---

## Starting My Kubernetes Cluster

In this step, we are starting a Minikube cluster. We also need to load a docker image into Minikube so that Minikube has an image it can use to spin up containers that will run the RAG api.

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_g3h4i5j6)

### Loading the Docker image into Minikube

We have started the cluster by running the command 'minikube start' and saw lots of status updates in our terminal as Minikube prepared and started the services necessary for our kubernetes cluster to work. Then we ran 'kubectl get nodes' to load our image. kubectl get nodes showed the status as 'Ready' , which tells us that our cluster is ready for us to load our Docker image.

### Why load image into Minikube

The 'eval $(minikube docker-env)' command switches the environment that our docker commands point to - after running this command every docker command I ran talked to Minikube's docker daemon instead of our local machinine docker daemon Without loading the image into Minikube, Kubernetes would not have an image for creating and running the containers - the RAG API simply would not be running. This is different from regular Docker because kubernetes does not create images but docker does and kubernetes is used for managing and scaling containers once they are created.

---

## Deploying to Kubernetes

In this step, we are deploying our RAG api using kubernetes. We need a Deployment because that is responsible for telling how many copies of our app we are running, what version of app we are using etc. It defines the 'ideal state' of our app that Kubernetes will try to maintain. 

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_s5t6u7v8)

### How the Deployment keeps my app running

The deployment.yaml file tells Kubernetes what to run(our rag-app Docker image), how many copies (1 - copy for now), how to find it (the app=rag-api label), how to connect(the OLLAMA_HOST environment variable tells our RAG API where to find OLLAMA running on our computer.)

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_a3b4c5d6)

### What did you observe when checking your pods?

We ran kubectl get pods and saw that we have one pod with name rag-app-deployment-6558594578-7wmbl and status Running which means the pod is now running our RAG API. The READY column showed 1/1 which indicates that 1 conatiner is ready out of total 1 in this Pod.

---

## Creating a Service

In this step, we are creating a service!  We need a Service which provides a stable networking endpoint i.e. a stable IP address that lets us connect to our API. If a service did not exist then we would have to directly connect to the Pod that is running our containerized API(and this connection would break each time the Pod gets restarted by Kubernetes i.e. very fragile.)

### What does the service.yaml file do?

The service.yaml file tells Kubernetes how to access the RAG API. The selector finds Pods by matching the selector label 'rag-api'. The port configuration allows our port 8000 to be connected to port 8000 on our Pods. NodePort enables access from outside by assigning a port on each node that routes to the service.

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_m5n6o7p8)

### What kubectl commands did you run to create the service?

We have applied our Service file by running command kubectl apply -f service.yaml 
We then have verified that the Service was created by running kubectl get services and confirming that our service named as : rag-app-service, was running. 

---

## Accessing My API Through Kubernetes

In this step, we are testing our API through kubernetes - we will be accessing our API through NodePort and then running our POST request with  "What is cucumber?" question with our RAG API - now running via a Kubernetes cluster.

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_y7z8a9b0)

### How I accessed my API

I tested my API by running the command 
curl -X POST "http://192.168.49.2:31977/query" -G --data-urlencode "q=What is Kubernetes?"
The response showed as follows:
{"answer":"In this context, the question \"What is Kubernetes?\" refers to the Kubernetes container orchestration platform, which is a popular and widely used tool for managing containers at scale. With Kubernetes, organizations can easily deploy, manage, and scale their applications across multiple cloud or on-premise environments without having to manually manage their infrastructure."}

This confirms that our kubernetes setup is working as expected. The main difference between Docker and Kubernetes deployment is, doker setup = running on container while kubernetes setup = running in a cluster, where container mangement is handled for us.

### Request flow through Kubernetes

The request flow went from my computer to the NodePort URL(port 32390) then to the Kubernetes service (on Port 8000) then to the Pod with label 'rag-api'. The Service routed traffic by selecting the Pod with label 'rag-api'.

---

## Testing Self-Healing

In this project extension, I'm demonstrating Kubernetes' self-healing abilities. Self-healing is important because it ensures our app stays running (high availability) - even if a Pod/group of containers are unhealthy/stopped running. Kubernetes can replace it with a healthy Pod so that incoming requests are still processed and served. In short , users receive uninterrupted service!

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_w8x9y0z1)

### What did you observe when you deleted the pod?

When I deleted the pod, I saw the status change from Running to Terminating to Completed. A new pod was created because Kubernetes was comparing its desired state with its actual state and tried to keep the desired state by creating new Pods in case of any Pod deleted/crashed.

![Image](http://learn.nextwork.org/delighted_chartreuse_swift_buddha's_hand_citron/uploads/ai-devops-kubernetes_sm3j8k9l)

### How the Service routed traffic to the new pod

The Service automatically would have routed traffic to the new Pod because the new Pod would have been created with the label 'rag-api'  which is the selector service uses to connect to the new Pod. Without Kubernetes deleting this running containers would have caused this entire RAG-API service to stop serving. Self-healing is critical in production because it removes human efforts to up and running Pods in case of any Pod failure due to any hardware/software issues that causes the Pods to crash. New Pods/containsers gets auto created and run the app like Netflix, Spotify to always up and running and providing a smooth user experience in production

---

---
