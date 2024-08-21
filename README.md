# Deploying a Go Application on a Kubernetes Cluster

This guide walks you through creating a Go application, pushing the code to GitHub, building a Docker image, and deploying it to a Kubernetes cluster set up with kubeadm on two AWS EC2 instances.

## Step 1: Create a Go Application

1. **Set Up the Project Directory:**
   ```bash
   mkdir go-k8s-app
   cd go-k8s-app
   ```

2. **Initialize a Go Module:**
   ```bash
   go mod init github.com/yourusername/go-k8s-app
   ```

3. **Write the Application Code:**

   Create a `main.go` file with the following content:
   ```go
   package main

   import (
       "fmt"
       "log"
       "net/http"
   )

   func handler(w http.ResponseWriter, r *http.Request) {
       fmt.Fprintf(w, "Hello, Kubernetes!")
   }

   func main() {
       http.HandleFunc("/", handler)
       log.Println("Starting server on :8080")
       log.Fatal(http.ListenAndServe(":8080", nil))
   }
   ```

4. **Test Locally:**
   ```bash
   go run main.go
   ```
   Open your browser and navigate to `http://localhost:8080`. You should see "Hello, Kubernetes!".

## Step 2: Push the Code to GitHub

1. **Initialize a Git Repository:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

2. **Create and Configure a GitHub Repository:**

   - Go to GitHub and create a new repository.
   - Copy the remote URL (e.g., `https://github.com/yourusername/go-k8s-app.git`).

3. **Add the Remote Repository and Push:**
   ```bash
   git remote add origin https://github.com/yourusername/go-k8s-app.git
   git push -u origin main
   ```

## Step 3: Create a Docker Image

1. **Write the Dockerfile:**

   Create a `Dockerfile` with the following content:
   ```Dockerfile
   # Use an official Go runtime as a parent image
   FROM golang:1.20-alpine

   # Set the working directory inside the container
   WORKDIR /app

   # Copy the current directory contents into the container at /app
   COPY . .

   # Build the Go app
   RUN go build -o main .

   # Make port 8080 available to the world outside this container
   EXPOSE 8080

   # Run the binary program produced by `go build`
   CMD ["./main"]
   ```

2. **Build and Test the Docker Image:**
   ```bash
   docker build -t yourusername/go-k8s-app .
   docker run -p 8080:8080 yourusername/go-k8s-app
   ```
   Visit `http://localhost:8080` to verify the application is running.

3. **Push the Docker Image to Docker Hub:**

   - Log in to Docker Hub:
     ```bash
     docker login
     ```

   - Tag and push the image:
     ```bash
     docker tag yourusername/go-k8s-app yourusername/go-k8s-app:latest
     docker push yourusername/go-k8s-app:latest
     ```

## Step 4: Deploy to Kubernetes

If you don't know how to setup kubernetes cluster

**Check out the below link whick includes deatailed steps for creating a Kubernetes cluster using kubeadm with two AWS EC2 instances: a Master Node and a Worker Node.**

https://github.com/MahammadYakub/Setup-Kubernetes-Cluster.git

1. **Create the Kubernetes Deployment:**

   Create a `deployment.yaml` file with the following content:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: go-k8s-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: go-k8s-app
     template:
       metadata:
         labels:
           app: go-k8s-app
       spec:
         containers:
         - name: go-k8s-app
           image: yourusername/go-k8s-app:latest
           ports:
           - containerPort: 8080
   ```

2. **Create the Kubernetes Service:**

   Create a `service.yaml` file with the following content:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: go-k8s-app-service
   spec:
     selector:
       app: go-k8s-app
     ports:
       - protocol: TCP
         port: 8080
         targetPort: 8080
         nodePor : 30007  # A port within the range 30000-32767
     type: nodePort
   ```

3. **Deploy to Your Kubernetes Cluster:**

   - Connect to your Kubernetes master node:
     ```bash
     ssh ec2-user@your-master-node-public-ip
     ```

   - Apply the deployment and service:
     ```bash
     kubectl apply -f deployment.yaml
     kubectl apply -f service.yaml
     ```

4. **Verify the Deployment:**

   - Check the status of the pods:
     ```bash
     kubectl get pods
     ```

   - Check the service status to get the external IP:
     ```bash
     kubectl get services
     ```

- **Access your application:**

   Use the public IP of one of your EC2 instances (master or worker) and the `nodePort` you specified (e.g., `30007`). For example:

   ```
   http://<EC2-instance-public-IP>:30007
   ```
