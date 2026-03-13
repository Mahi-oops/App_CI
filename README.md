# 🚀 Todo App – Docker + CircleCI + GitOps CI/CD Pipeline

Welcome to this repository 👋

This project demonstrates how a simple **Node.js application** travels through a modern **DevOps pipeline**.

Think of the workflow like a food delivery system 🍕

Developer writes code → Docker packages it → CircleCI delivers it → Docker Hub stores it → Kubernetes serves it to users.

So in short:

Code → Container → CI/CD → Image Registry → Deployment

---

# 📂 Project Structure

```
.
├── Dockerfile
├── .circleci/
│   └── config.yml
├── public/
├── src/
├── package.json
└── README.md
```

| File / Folder | What it does |
|---|---|
| Dockerfile | Builds the application container |
| .circleci/config.yml | CI/CD pipeline configuration |
| src | Application source code |
| public | Static files |
| package.json | Node project dependencies and scripts |

---

# 🐳 Dockerfile Explained

The Dockerfile uses **multi-stage builds** to keep the final image small and production ready.

### Step 1 – Builder Stage

```dockerfile
FROM node:18-alpine AS installer
```

We start with a lightweight Node.js image.

This stage is basically the **builder workspace** where the app is prepared.

---

```dockerfile
WORKDIR /app
```

Sets `/app` as the working directory inside the container.

Meaning all commands will run inside `/app`.

---

```dockerfile
COPY package*.json ./
```

Copies `package.json` and `package-lock.json`.

We copy these first because Docker caches dependencies if they don't change.

This makes builds **much faster** ⚡

---

```dockerfile
RUN npm install
```

Installs all required dependencies.

Think of this step as downloading all the tools required to build the app.

---

```dockerfile
COPY . .
```

Copies the entire project source code into the container.

---

```dockerfile
RUN npm run build
```

Builds the production version of the application.

This converts the raw source code into optimized production files.

---

### Step 2 – Deployment Stage

```dockerfile
FROM nginx:latest AS deployer
```

Now we switch to an **Nginx server**.

Nginx is excellent at serving static frontend applications.

---

```dockerfile
COPY --from=installer /app/build /usr/share/nginx/html
```

This copies the **built application files** from the previous stage.

Important point:

We only copy the build output and **not the entire Node environment**, which keeps the final Docker image small.

Result:

• Faster containers  
• Smaller images  
• Production ready deployment 🚀

---

# 🔄 CircleCI Pipeline

The CI/CD pipeline is defined inside:

```
.circleci/config.yml
```

Whenever code is pushed to GitHub, CircleCI automatically runs the pipeline.

Pipeline has **two jobs**:

1. Build and push Docker image
2. Update Kubernetes deployment manifest

---

# 🏗 Job 1 – Build and Push Docker Image

```
jobs:
  build_and_push:
    docker:
      - image: cimg/node:20.3.1
```

CircleCI runs this job using a Node.js container.

---

### Checkout repository

```
- checkout
```

This downloads the repository into the CircleCI environment.

Equivalent to running:

```
git clone <repo>
```

---

### Enable Docker

```
- setup_remote_docker
```

Allows Docker commands inside the pipeline.

Without this step, `docker build` would fail.

---

### Generate image version

```
version="build-$CIRCLE_BUILD_NUM"
```

Each build gets a unique version.

Example:

```
build-1
build-2
build-3
```

This prevents image overwriting.

---

### Build Docker Image

```
docker build -t avulamahesh/todo-app:$version .
```

Example result:

```
docker build -t avulamahesh/todo-app:build-23 .
```

Meaning we create a Docker image tagged `build-23`.

---

### Login to Docker Hub

```
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
```

Uses environment variables stored in CircleCI.

This securely logs into Docker Hub.

---

### Push image to Docker Hub

```
docker push avulamahesh/todo-app:$version
```

Now the image is available globally in Docker Hub.

---

# ⚙️ Job 2 – Update Kubernetes Manifest (GitOps)

After pushing the image, we update another repository containing Kubernetes manifests.

```
git clone https://github.com/Mahi-oops/App_manifest.git
```

This repository stores the deployment configuration.

---

### Set image tag

```
TAG=$CIRCLE_BUILD_NUM
((TAG--))
```

Example:

If build number = 24

```
TAG = 23
```

---

### Update deployment YAML

```
sed -i "s/build-.*/build-$TAG/g" manifest/deployment.yml
```

This replaces the Docker image version inside the Kubernetes deployment file.

Example:

Before

```
image: avulamahesh/todo-app:build-22
```

After

```
image: avulamahesh/todo-app:build-23
```

---

### Commit and push changes

```
git add .
git commit -m "new build with imgTag build-$TAG"
git push
```

The updated manifest is pushed back to GitHub.

This triggers the GitOps workflow and Kubernetes deploys the new version.

Deployment complete 🎉

---

# 📦 package.json

The `package.json` file contains:

• Project dependencies  
• Application scripts  
• Project metadata  

Common commands:

Install dependencies

```
npm install
```

Build production application

```
npm run build
```

Start application

```
npm start
```

---

# 🧪 Running the Project Locally

Install dependencies

```
npm install
```

Run development server

```
npm start
```

---

# 🐳 Running the App Using Docker

Build Docker image

```
docker build -t todo-app .
```

Run container

```
docker run -p 3000:80 todo-app
```

Open in browser

```
http://localhost:3000
```

---

# 🧠 DevOps Concepts Demonstrated

This repository demonstrates:

✔ Docker Multi-Stage Builds  
✔ CI/CD Pipeline using CircleCI  
✔ Docker Image Publishing  
✔ GitOps Workflow  
✔ Kubernetes Manifest Automation

---

# 👨‍💻 Author

Mahesh Avula  
Cloud / DevOps Engineer

Passionate about building scalable infrastructure, automation pipelines, and cloud-native systems ☁️⚙️