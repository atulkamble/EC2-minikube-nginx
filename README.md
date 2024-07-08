# EC2-minikube-nginx

To host an application in Minikube running on an AWS EC2 instance and expose it for access from a web browser, you can follow these steps:

### Clone this repository
```
git clone https://github.com/atulkamble/EC2-minikube-nginx.git
cd EC2-minikube-nginx
```
### Step 1: Set Up EC2 Instance
1. **Launch an EC2 Instance**:
    - Choose an AMI (Amazon Machine Image) such as Ubuntu Server.
    - Select an instance type (e.g., t2.medium or larger).
    - Configure security groups to allow necessary ports (SSH for access, HTTP/HTTPS for web traffic, and any custom NodePort ranges you plan to use).
    - Launch the instance.

2. **Connect to Your EC2 Instance**:
    Use SSH to connect to your EC2 instance:
    ```sh
    ssh -i /path/to/your-key.pem ubuntu@<your-ec2-public-ip>
    ```

### Step 2: Install Minikube and Dependencies on EC2
1. **Install Docker**:
    ```sh
    sudo apt-get update
    sudo apt-get install -y docker.io
    sudo usermod -aG docker $USER
    newgrp docker
    ```

2. **Install Kubectl**:
    ```sh
    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```

3. **Install Minikube**:
    ```sh
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```

4. **Start Minikube**:
    ```sh
    minikube start --driver=docker
    ```

### Step 3: Deploy Your Application
1. **Create a Deployment**:
    ```sh
    kubectl create deployment webapp --image=nginx
    ```

2. **Expose the Deployment**:
    ```sh
    kubectl expose deployment webapp --type=NodePort --port=80
    ```

### Step 4: Configure Security Group for NodePort
1. **Determine the NodePort**:
    ```sh
    kubectl get services webapp
    ```

2. **Edit Security Group**:
    - Go to the EC2 console.
    - Find the security group associated with your instance.
    - Add an inbound rule to allow traffic to the NodePort (e.g., TCP 30000-32767).

### Step 5: Access the Application
1. **Get Minikube IP**:
    ```sh
    minikube ip
    ```

2. **Access the Application**:
   Open your web browser and navigate to:
    ```
    http://<your-ec2-public-ip>:<node-port>
    ```
   For example, if the NodePort is `32000`, you would go to:
    ```
    http://<your-ec2-public-ip>:32000
    ```

### Step 6: (Optional) Use Ingress for Friendlier URLs
1. **Enable the Ingress Addon**:
    ```sh
    minikube addons enable ingress
    ```

2. **Create an Ingress Resource**:
   Create a file named `ingress.yaml`:
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: webapp-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - host: <your-domain>
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: webapp
                port:
                  number: 80
    ```

3. **Apply the Ingress Resource**:
    ```sh
    kubectl apply -f ingress.yaml
    ```

4. **Edit your DNS Records or Hosts File**:
   Point your domain to the EC2 public IP address. If testing locally, you can add an entry to your `/etc/hosts` file:
    ```sh
    sudo nano /etc/hosts
    ```

   Add the following line:
    ```
    <your-ec2-public-ip> <your-domain>
    ```

   Save and close the file.

5. **Access the Application**:
   Open your web browser and navigate to:
    ```
    http://<your-domain>
    ```

By following these steps, you'll be able to host your application in Minikube on an EC2 instance and expose it for access from a web browser, either using NodePort or Ingress for friendlier URLs.
