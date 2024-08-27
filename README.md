First we installed all the required files to run. i am using the dockerhub . first we need to create the flask app and then we go to the k8s where we create the yaml files .yaml stand for yet another markup language but now it change to yaml is not markup language.
 DNS Resolution in Kubernetes
DNS resolution allows services within the Kubernetes cluster to communicate via their service names. For example, the Flask application can access MongoDB by referring to mongo-service as the host name.
Kubernetes uses the kube-dns or CoreDNS service for DNS resolution, mapping service names to IP addresses within the cluster.
7. Resource Requests and Limits
Requests ensure that the Kubernetes scheduler will only place a pod on a node if it can guarantee the requested resources.
Limits cap the resources a container can use, preventing it from consuming too much.



Flask MongoDB Kubernetes Deployment

This project demonstrates deploying a Python Flask application that interacts with a MongoDB database on a Kubernetes cluster. The application allows users to insert and retrieve data from MongoDB via a RESTful API.

Prerequisites
Before starting, ensure the following are installed on your system:

Python 3.8 or later
Docker
Minikube or another local Kubernetes alternative
kubectl command-line tool
pip for managing Python packages
Local Setup (Part 1)
1. Create Project Directory

mkdir flask-mongodb-app
cd flask-mongodb-app

2. Set Up Virtual Environment

python3 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`.

3. Create Flask Application
Create a file named app.py and add the provided Flask and MongoDB code.

4. Create Requirements File
Create a requirements.txt file with the following content:
Flask==2.0.2
Werkzeug==2.0.3
pymongo==3.12.0

6. Install Python Dependencies

pip install -r requirements.txt
6. Set Up MongoDB Using Docker

docker pull mongo:latest
docker run -d -p 27017:27017 --name mongodb mongo:latest
7. Set Up Environment Variable
Create a .env file in the project directory:

echo "MONGODB_URI=mongodb://localhost:27017/" > .env
export $(cat .env | xargs)
8. Run the Flask Application

export FLASK_APP=app.py
export FLASK_ENV=development
flask run
On Windows:

set FLASK_APP=app.py
set FLASK_ENV=development
flask run
9. Accessing the Application
Open your browser and navigate to http://localhost:5000.

Kubernetes Setup (Part 2)
1. Set Up Minikube
Start Minikube:


minikube start
2. Build and Push Docker Image
Create a Dockerfile in your project directory:

dockerfile

FROM python:3.8-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
Build the Docker image:


docker build -t flask-mongodb-app .
Push it to a container registry:


docker tag flask-mongodb-app:latest yourusername/flask-mongodb-app:latest // code to push the image to registry. first we tag the image and then we push it on registry.
docker push yourusername/flask-mongodb-app:latest
3. Deploy on Kubernetes
Create the following Kubernetes YAML files:

Flask Deployment (flask-deployment.yaml)
MongoDB StatefulSet (mongo-statefulset.yaml)
Flask Service (flask-service.yaml)
MongoDB Service (mongo-service.yaml)
Horizontal Pod Autoscaler (flask-hpa.yaml)
Apply the YAML files:


kubectl apply -f flask-deployment.yaml
kubectl apply -f mongo-statefulset.yaml
kubectl apply -f flask-service.yaml
kubectl apply -f mongo-service.yaml
kubectl apply -f flask-hpa.yaml


Testing and Validation
Accessing the Flask Application
After deploying the application, access it via the NodePort exposed by Minikube. Find the Minikube IP:

minikube ip
Then, access the Flask application at http://<minikube-ip>:30001.

Testing Autoscaling
Simulate high traffic to test autoscaling:


kubectl run -i --tty load-generator --image=busybox /bin/sh
# Inside the shell:
while true; do wget -q -O- http://flask-service:5000/; done
Monitor the autoscaler:

kubectl get hpa flask-hpa
Clean Up
To stop and clean up the resources:

kubectl delete -f flask-hpa.yaml
kubectl delete -f flask-service.yaml
kubectl delete -f mongo-service.yaml
kubectl delete -f flask-deployment.yaml
kubectl delete -f mongo-statefulset.yaml
minikube stop
minikube delete
Design Choices
StatefulSet for MongoDB: Chosen to ensure persistent storage and maintain the state across pod restarts.
Horizontal Pod Autoscaler: Configured to ensure scalability under high load conditions.
Persistent Volume and Persistent Volume Claim: Used to ensure MongoDB data persists even if the pod is restarted.
