# weather-app-k8s
Multi-tier app deployed on k8s with jenkins cicd

# 🌍 Deploying Real-World WeatherApp Using Kubernetes ☁️🚀

- A microservices application deployed on **Kubeadm Cluster**

---

## 📦 Project Architecture

This project contains **three core microservices**, fully containerized and deployed on a Kubernetes cluster:

```
                ┌──────────────┐
                │  UI (Python) │
                └──────┬───────┘
                       │
         ┌─────────────┼─────────────┐
         │                             │
┌────────▼────────┐         ┌─────────▼─────────┐
│ Auth Service    │         │ Weather Service   │
│ (Go + MySQL)    │         │ (Node.js + API)   │
└─────────────────┘         └───────────────────┘
```

---

## 🔧 Services Breakdown

### 1️⃣ 🛡️ Authentication Service (GoLang + MySQL)

This service manages user authentication.

#### 📌 Components:
- **MySQL StatefulSet** with:
  - A **Headless Service** for stable DNS
  - A **Secret** for DB credentials + JWT secret key
  - An **Init Job** to bootstrap the `weatherapp` DB and user
- **GoLang Auth API**:
  - Deployment with 1 replica
  - ClusterIP service on port `8080`

---

### 2️⃣ 🌤️ Weather API Service (Node.js)

A lightweight Node.js service that connects to Rapid API site to fetch real-time weather data.

#### 📌 Components:
- Deployment with **2 replicas** + Secret (for API key)
- ClusterIP service on port `5000`

---

### 3️⃣ 🖥️ UI Web Application (Python Flask)

Frontend interface that interacts with both Auth and Weather services.

#### 📌 Components:
- Deployment with **2 replicas** (for availability)
- ClusterIP service on port `3000`
- Ingress resource to expose HTTP/HTTPS
- TLS secured using self-signed certificate via OpenSSL

---

## 🧪 Step-by-Step Deployment Guide

### 🔁 Cluster Setup

- Two-node Cluster was provisioned with Kubeadm with containerd container runtime.
---

### 📂 MySQL Deployment

#### ✅ Headless Service

```yaml
clusterIP: None
```

#### ✅ Secret Creation

```bash
kubectl create secret generic mysql-secret --from-literal=root-password="rootpass" \
        --from-literal=auth-password="userpass" --from-literal=secret-key="xco0sr0fh4e52x03g9mv" 
  
```

#### ✅ Init Job (DB Bootstrap)

Creates the `weatherapp` database and user with permissions.

#### ✅ StatefulSet

```yaml
replicas: 1
```

#### ✅ Apply Resources

```bash
kubectl apply -f mysql-pv.yaml
kubectl apply -f headless-service.yaml
kubectl apply -f statefulset.yaml
kubectl apply -f init-job.yaml
```
---

### 🔐 Authentication Service

#### ✅ Deployment + Service

Exposes port `8080` inside the cluster.

#### ✅ Test the Auth API

```bash
kubectl run alpine --rm -it --image=alpine -- sh
apk add curl
curl -X POST http://weatherapp-auth:8080/users \
-H "Content-Type: application/json" \
-d '{"username": "testuser", "password": "testpassword"}'
# Response: {"success":"User added successfully"}
```

---

### 🌤️ Weather Service

#### ✅ Deployment + Secret

Store your RapidAPI key securely in a Kubernetes Secret.

#### ✅ Service

Exposed on port `5000` (ClusterIP).

#### ✅ Test Weather API

```bash
curl weatherapp-weather:5000
# Output: "The service is running"

curl weatherapp-weather:5000/cairo
# Output: JSON with current weather for Cairo
```

---

### 🖥️ UI Web App

#### ✅ Deployment + Service

Expose internally on port `3000`, using 2 replicas for availability.

#### ✅ Create TLS Certificate (HTTPS)

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout tls.key -out tls.crt -subj "/CN=weatherapp.local/O=weatherapp"
```

#### ✅ Create Secret for TLS

```bash
kubectl create secret tls weatherapp-ui-tls \
--cert tls.crt --key tls.key
```

#### ✅ Verifying kubernetes resources

```bash
kubectl get pods

```
![pods ](./Screenshot-0.png)


#### ✅ Ingress Object

Configure Ingress for both HTTP and HTTPS routing:

> 📌 Make sure to edit your `/etc/hosts` in your local machine:
```bash
127.0.0.1 weatherapp.local
```

```bash
kubectl apply -f ingress.yaml
```

---

## 🔍 Verifying Everything Works

✅ UI accessible at: https://weatherapp.local:<tls-ingress-nodeport > "i'm using ingress kubeadm "
✅ User Registration & Login works via Auth Service  
✅ Weather info loads dynamically from Node.js API  
✅ All services discover each other via internal DNS

---

## Screenshots

![pods ](./Screenshot-1.png)
![pods ](./Screenshot-2.png)
