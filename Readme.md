<h1 align="center" id="title">Spring Boot + Docker + Kubernetes</h1>

<p id="description">We'll create a Spring Boot Hello World API that returns a greeting message then:
</br>1-Containerize it using Docker
</br>2-Deploy it to Kubernetes  
</br>3-Expose it as a service</p>

<h2>Installation:</h2>

```
docker pull max3no/spring-boot-k8s
```

<h2>🛠️ Creating from Scratch:</h2>
<p>1. You can use Spring Initializr (start.spring.io) with the following dependencies:</p>

```
Spring Web (for REST API) 
Spring Boot DevTools (optional)
```

<p>2. Create a simple Rest Controller which returns some value</p>

```
Hello Kubernetes
```
<p>3. Add a Dockerfile in the root directory</p>

```
# Use OpenJDK 17 as the base image 
FROM openjdk:17-jdk-alpine  

# Set working directory 
WORKDIR /app
  
# Copy the JAR file into the container 
COPY target/docker-kubernetes-0.0.1-SNAPSHOT.jar app.jar  

# Command to run the application 
ENTRYPOINT ["java" "-jar" "app.jar"]
```
<li>Using jdk alpine because is a lightweight linux distro making container smaller.</li>
<li>WORKDIR creates working directory and all commands will run in this dir</li>
<li>Copying the jar into the container app directory, inside it is named as app</li>
<li>command to start the application when container runs</li>

<p>4. Build and Run the Docker Container</p>
<h6>Package the Spring Boot app:</h6>

```
mvn clean package -DskipTests
```

<h6>Build the Docker image(. at the end if in the same working dir)</h6>

```
docker build -t spring-boot-k8s .
```

<h6>Run the container to test locally</h6>

```
docker run -p 8080:8080 spring-boot-k8s
```

<h6>Check if the API is working</h6>

```
curl http://localhost:8080/api/hello
```

<h3>Push image to docker hub</h3>
We will be using docker hub for image storing, instead of locally fetch for kubernetes,
(which can be done with help of imagePullPolicy: IfNotPresent)

<p><u>1. Login to Docker Hub</u></p>

```
docker login
```
<p><u>2. Tag the image (replace your-username with your Docker Hub username)</u></p>

```
docker tag spring-boot-k8s your-username/spring-boot-k8s
```
<p><u>3. Push it to Docker Hub</u></p>

```
docker push your-username/spring-boot-k8s
```
<p><u>4.Update your deployment.yaml to use</u></p>

```
image: your-username/spring-boot-k8s
```


<h3>Deploy to Kubernetes</h3>
<p><u>1. Create a Kubernetes Deployment YAML(deployment.yaml)</u></p>

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-boot
  template:
    metadata:
      labels:
        app: spring-boot
    spec:
      containers:
        - name: spring-boot-container
          image: {your-username}/spring-boot-k8s
          ports:
            - containerPort: 8080

```
Change {your-username} to your docker username, we will be pushing image to docker hub
and then kubernetes engine will pull it from there.

<p><u>2. Create a Kubernetes Service YAML(service.yaml)</u></p>

```
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-service
spec:
  selector:
    app: spring-boot
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

<p><u>3. Start Minikube (Local Kubernetes Cluster)</u></p>
<h6>Install Minikube, if not there.</h6>

```
minikube start
```

```
minikube image load spring-boot-k8s
```

<p><u>4 Deploy the App to Kubernetes</u></p>

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Check if pods are running

```
kubectl get pods
```

Check the service URL

```
minikube service spring-boot-service --url
```

Test the API
```
curl <YOUR_MINIKUBE_URL>/api/hello

```


<h2>Testing Kubernetes Features</h2>

1-<u>To verify if your application is running</u>  
You should see multiple instances of your Spring Boot application if replicas are set.

```
kubectl get pods
```

2-<u>Increase or Decrease Replicas (Scaling Test)</u>  
Either update the deployement.yaml file and change the replicas then apply the changes   
<b>OR</b>  
Scale using **CLI**

````
kubectl scale deployment spring-boot-app --replicas=5
````
Check the pods again and should see 5 pods running now.

3-<u>Test Fault Tolerance (Pod Failure)</u>  
Manually delete a pod to test if Kubernetes automatically restarts it
````
kubectl delete pod <POD-NAME>
````

Check if a new pod is created:
````
kubectl get pods
````
4-Simulate High Load
Use External tools like apache benchmark to mimic multiple user requests.

<h2>🧐 Useful Commands</h2>
*   Check docker images

````
docker images
````

*  Reapplying kubernetes deployment

````
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
````


*  Find the service details

````
kubectl get svc
````