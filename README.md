# 🚀 Automated CI/CD Pipeline for a Dockerized Website

A complete DevOps CI/CD project that automates the process of **testing, containerizing, publishing, and deploying a website** using GitHub Actions, Docker, Docker Hub, and AWS EC2.

The project separates **Continuous Integration (CI)** from **Continuous Deployment (CD)** into independent GitHub Actions workflows.

---

## 📌 Project Overview

This project demonstrates how a simple website can be automatically deployed to an AWS EC2 server whenever changes are pushed to the `main` branch.

The pipeline performs the following:

1. Developer pushes code to GitHub.
2. GitHub Actions starts the CI pipeline.
3. Application dependencies are installed.
4. Automated tests are executed.
5. The application is built.
6. A Docker image is created.
7. The Docker image is pushed to Docker Hub.
8. The CD workflow is triggered after successful CI.
9. GitHub Actions connects to the AWS EC2 server through SSH.
10. The deployment directory is created automatically.
11. A Docker Compose configuration is created automatically.
12. The latest Docker image is pulled from Docker Hub.
13. The previous container is stopped.
14. The new container is started.
15. The website becomes available through the EC2 public IP.

---

# 🏗️ Architecture

```text
                         ┌──────────────────┐
                         │     Developer    │
                         │                  │
                         │  git push main   │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │     GitHub       │
                         │   Repository     │
                         └────────┬─────────┘
                                  │
                                  ▼
                  ┌─────────────────────────────┐
                  │            CI               │
                  │                             │
                  │  Checkout Code              │
                  │       ↓                     │
                  │  Install Dependencies       │
                  │       ↓                     │
                  │  Run Tests                  │
                  │       ↓                     │
                  │  Build Application          │
                  │       ↓                     │
                  │  Build Docker Image         │
                  │       ↓                     │
                  │  Push to Docker Hub         │
                  └──────────────┬──────────────┘
                                 │
                                 ▼
                       ┌──────────────────┐
                       │    Docker Hub    │
                       │                  │
                       │  my-website      │
                       │     :latest      │
                       └────────┬─────────┘
                                │
                                ▼
                  ┌─────────────────────────────┐
                  │             CD              │
                  │                             │
                  │       SSH → AWS EC2         │
                  │             ↓               │
                  │    Create deployment dir   │
                  │             ↓               │
                  │    Create Compose file     │
                  │             ↓               │
                  │      Docker Pull            │
                  │             ↓               │
                  │   Docker Compose Up         │
                  └──────────────┬──────────────┘
                                 │
                                 ▼
                         ┌─────────────────┐
                         │     AWS EC2     │
                         │                 │
                         │ Docker Container│
                         │                 │
                         │    Port 80      │
                         └────────┬────────┘
                                  │
                                  ▼
                              🌍 Website
```

---

# 🛠️ Technologies Used

| Technology     | Purpose                         |
| -------------- | ------------------------------- |
| Git            | Version control                 |
| GitHub         | Source code repository          |
| GitHub Actions | CI/CD automation                |
| Docker         | Application containerization    |
| Docker Compose | Container deployment            |
| Docker Hub     | Container image registry        |
| AWS EC2        | Cloud server                    |
| SSH            | Secure server connection        |
| Linux          | Server operating system         |
| YAML           | CI/CD and Compose configuration |
| Bash           | Server automation               |

---

# 📁 Project Structure

The project uses the following structure:

```text
my-website/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
├── web/
│   ├── Dockerfile
│   ├── index.html
│   ├── style.css
│   └── ...
│
└── README.md
```

### Workflow files

`ci.yml`

Responsible for:

* Installing dependencies
* Running tests
* Building the application
* Building the Docker image
* Pushing the image to Docker Hub

`deploy.yml`

Responsible for:

* Connecting to AWS EC2
* Creating the deployment directory
* Creating `docker-compose.yml`
* Pulling the Docker image
* Stopping the old container
* Starting the new container

---

# 1️⃣ Create the Project

Create a project directory:

```bash
mkdir my-website
cd my-website
```

Create the website directory:

```bash
mkdir web
```

Place your website files inside:

```text
web/
├── index.html
├── style.css
└── ...
```

---

# 2️⃣ Create the Dockerfile

Inside the `web` directory:

```bash
cd web
nano Dockerfile
```

Example Dockerfile:

```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Explanation

```dockerfile
FROM nginx:alpine
```

Uses a lightweight Nginx image.

```dockerfile
COPY . /usr/share/nginx/html
```

Copies the website into the Nginx web directory.

```dockerfile
EXPOSE 80
```

Documents that the container serves traffic on port 80.

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

Runs Nginx in the foreground so the container stays alive.

---

# 3️⃣ Test the Docker Image Locally

From the project root:

```bash
docker build -t my-website ./web
```

Run the container:

```bash
docker run -d -p 8080:80 --name my-website my-website
```

Check the container:

```bash
docker ps
```

Open:

```text
http://localhost:8080
```

If the website loads, Docker is working correctly.

Stop the container:

```bash
docker stop my-website
```

Remove it:

```bash
docker rm my-website
```

---

# 4️⃣ Create a Docker Hub Repository

Create an account on Docker Hub if you do not already have one.

Create a repository called:

```text
my-website
```

Your image will eventually be available as:

```text
YOUR_DOCKER_USERNAME/my-website
```

For example:

```text
samueldev/my-website
```

---

# 5️⃣ Create a Docker Hub Access Token

Do not use your Docker Hub password in GitHub Actions.

Create a Docker Hub Personal Access Token with permission to push images.

Save the token securely.

It will be used as:

```text
DOCKER_TOKEN
```

---

# 6️⃣ Prepare the AWS EC2 Server

Create an EC2 instance running:

```text
Amazon Linux 2023
```

The instance needs a Security Group that allows:

```text
SSH     TCP 22
HTTP    TCP 80
HTTPS   TCP 443
```

For production environments, SSH access should preferably be restricted to trusted IP addresses rather than allowing the entire internet.

---

# 7️⃣ Connect to AWS EC2

Connect using SSH:

```bash
ssh -i your-key.pem ec2-user@YOUR_EC2_PUBLIC_IP
```

Example:

```bash
ssh -i my-server.pem ec2-user@18.200.10.20
```

---

# 8️⃣ Install Docker on EC2

On Amazon Linux:

```bash
sudo dnf update -y
```

Install Docker:

```bash
sudo dnf install docker -y
```

Start Docker:

```bash
sudo systemctl start docker
```

Enable Docker at boot:

```bash
sudo systemctl enable docker
```

Add `ec2-user` to the Docker group:

```bash
sudo usermod -aG docker ec2-user
```

Log out:

```bash
exit
```

Reconnect to EC2.

Test:

```bash
docker ps
```

Docker should now work without `sudo`.

---

# 9️⃣ Install Docker Compose

Verify whether Docker Compose is already available:

```bash
docker compose version
```

If available, you should see something similar to:

```text
Docker Compose version ...
```

The project uses the modern Docker Compose command:

```bash
docker compose
```

not:

```bash
docker-compose
```

---

# 🔐 10️⃣ Configure GitHub Secrets

Go to:

```text
GitHub Repository
        ↓
Settings
        ↓
Secrets and variables
        ↓
Actions
        ↓
New repository secret
```

Create the following secrets:

| Secret            | Value                               |
| ----------------- | ----------------------------------- |
| `DOCKER_USERNAME` | Your Docker Hub username            |
| `DOCKER_TOKEN`    | Docker Hub access token             |
| `EC2_HOST`        | EC2 public IPv4 address             |
| `EC2_USER`        | `ec2-user`                          |
| `EC2_SSH_KEY`     | Contents of your `.pem` private key |

---

## DOCKER_USERNAME

Example:

```text
samueldev
```

---

## DOCKER_TOKEN

Paste the Docker Hub Personal Access Token.

Do not commit this value to GitHub.

---

## EC2_HOST

Use your EC2 public IPv4 address:

```text
18.200.10.20
```

---

## EC2_USER

For Amazon Linux:

```text
ec2-user
```

---

## EC2_SSH_KEY

Open your `.pem` file and copy the entire contents.

Example:

```text
-----BEGIN RSA PRIVATE KEY-----
...
...
-----END RSA PRIVATE KEY-----
```

Paste the entire key into the GitHub secret.

Never commit the `.pem` file to the repository.

---

# 🔄 11️⃣ Continuous Integration

Create:

```text
.github/workflows/ci.yml
```

The CI pipeline is responsible for testing the application and publishing the Docker image.

```yaml
name: CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:

  test:
    name: Test Application
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Setup Node.js
        uses: actions/setup-node@v6
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build application
        run: npm run build


  docker:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest

    needs: test

    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Login to Docker Hub
        uses: docker/login-action@v4
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v4

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v7
        with:
          context: ./web
          file: ./web/Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/my-website:latest
            ${{ secrets.DOCKER_USERNAME }}/my-website:${{ github.sha }}
```

---

# 🔄 12️⃣ Continuous Deployment

Create:

```text
.github/workflows/deploy.yml
```

This workflow automatically deploys the Docker image to AWS after CI succeeds.

```yaml
name: CD - Deploy to AWS

on:
  workflow_run:
    workflows:
      - CI
    types:
      - completed

  workflow_dispatch:

jobs:

  deploy:
    name: Deploy to AWS EC2
    runs-on: ubuntu-latest

    if: >
      github.event_name == 'workflow_dispatch' ||
      github.event.workflow_run.conclusion == 'success'

    steps:

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.2.5
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}

          script: |
            set -e

            echo "================================="
            echo "Starting deployment"
            echo "================================="

            # Create deployment directory
            mkdir -p ~/my-website

            cd ~/my-website

            # Create Docker Compose configuration
            cat > docker-compose.yml << EOF
            services:
              website:
                image: ${{ secrets.DOCKER_USERNAME }}/my-website:latest
                container_name: my-website
                ports:
                  - "80:80"
                restart: unless-stopped
            EOF

            # Pull latest image
            docker pull ${{ secrets.DOCKER_USERNAME }}/my-website:latest

            # Stop old deployment
            docker compose down || true

            # Start new deployment
            docker compose up -d

            # Remove unused images
            docker image prune -f

            # Display running containers
            docker ps

            echo "================================="
            echo "Deployment completed successfully"
            echo "================================="
```

---

# 13️⃣ Docker Compose

The deployment workflow automatically creates this file on the EC2 server:

```yaml
services:
  website:
    image: YOUR_DOCKER_USERNAME/my-website:latest
    container_name: my-website
    ports:
      - "80:80"
    restart: unless-stopped
```

Therefore, you do not need to manually create the deployment directory.

The GitHub Actions deployment script creates:

```text
/home/ec2-user/my-website/
└── docker-compose.yml
```

automatically.

---

# 14️⃣ Push the Project to GitHub

Initialize Git:

```bash
git init
```

Add the remote repository:

```bash
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
```

Add files:

```bash
git add .
```

Commit:

```bash
git commit -m "Initial CI/CD pipeline"
```

Rename the branch:

```bash
git branch -M main
```

Push:

```bash
git push -u origin main
```

---

# 🚀 15️⃣ Automatic Deployment

After pushing to `main`, GitHub Actions starts the CI pipeline.

### CI

```text
Checkout
   ↓
Install dependencies
   ↓
Run tests
   ↓
Build application
   ↓
Build Docker image
   ↓
Push image to Docker Hub
```

If CI succeeds:

```text
CI ✅
 ↓
CD starts
```

Then:

```text
SSH → EC2
 ↓
Create ~/my-website
 ↓
Create docker-compose.yml
 ↓
docker pull
 ↓
docker compose down
 ↓
docker compose up -d
 ↓
Website running
```

---

# 🌍 16️⃣ Access the Website

Find the EC2:

```text
AWS Console
   ↓
EC2
   ↓
Instances
```

Copy the **Public IPv4 address**.

Open:

```text
http://YOUR_EC2_PUBLIC_IP
```

Example:

```text
http://18.200.10.20
```

Your website should now be running from the Docker container on AWS.

---

# 🔍 17️⃣ Verify the Deployment

SSH into EC2:

```bash
ssh -i your-key.pem ec2-user@YOUR_EC2_PUBLIC_IP
```

Check running containers:

```bash
docker ps
```

Check Compose:

```bash
cd ~/my-website
docker compose ps
```

View logs:

```bash
docker compose logs
```

Follow logs:

```bash
docker compose logs -f
```

---

# 🧪 18️⃣ Test the CI/CD Pipeline

Make a change to your website.

For example:

```bash
nano web/index.html
```

Save the changes.

Then:

```bash
git add .
git commit -m "Update website"
git push origin main
```

GitHub Actions automatically starts:

```text
CI
 ↓
Test
 ↓
Docker Build
 ↓
Docker Hub
 ↓
CD
 ↓
AWS EC2
 ↓
New Container
```

No manual deployment is required.

---

# 🔧 19️⃣ Troubleshooting

## Docker permission denied

If EC2 shows:

```text
permission denied while trying to connect to the Docker API
```

run:

```bash
sudo usermod -aG docker ec2-user
```

Then log out and reconnect.

Test:

```bash
docker ps
```

---

## Docker Compose not found

Check:

```bash
docker compose version
```

If it isn't available, install/configure the Docker Compose plugin for your EC2 distribution.

---

## Deployment directory doesn't exist

The deployment workflow automatically creates it:

```bash
mkdir -p ~/my-website
```

You can manually verify it:

```bash
ls -la ~/my-website
```

---

## Docker Hub 401 Unauthorized

If you receive:

```text
401 Unauthorized
access token has insufficient scopes
```

check:

* Docker Hub username
* Docker Hub Personal Access Token
* Token permissions
* Docker repository name
* GitHub `DOCKER_USERNAME` secret

The image should follow:

```text
YOUR_DOCKER_USERNAME/my-website
```

---

## Dockerfile not found

If your Dockerfile is inside:

```text
web/Dockerfile
```

the GitHub Actions Docker configuration must use:

```yaml
context: ./web
file: ./web/Dockerfile
```

---

## Website cannot be accessed

Check the container:

```bash
docker ps
```

Check logs:

```bash
docker logs my-website
```

Check EC2 port 80:

```bash
sudo ss -tulpn | grep :80
```

Also verify that the EC2 Security Group allows:

```text
HTTP
TCP
80
0.0.0.0/0
```

---

# 🔐 20️⃣ Security Considerations

This project uses GitHub Secrets for sensitive credentials.

Never put the following directly inside your repository:

```text
.pem files
Docker Hub passwords
Docker Hub tokens
AWS secret keys
SSH private keys
API keys
```

Never commit:

```text
*.pem
.env
credentials
secrets
```

A `.gitignore` file should be used.

Example:

```gitignore
.env
*.pem
*.key
node_modules/
```

---

# 📈 21️⃣ Future Improvements

This project can be extended with additional DevOps technologies.

Possible improvements include:

* HTTPS using Let's Encrypt
* Nginx reverse proxy
* Custom domain
* AWS Application Load Balancer
* AWS ECR instead of Docker Hub
* Terraform for infrastructure provisioning
* Ansible for server configuration
* Kubernetes for container orchestration
* Prometheus for monitoring
* Grafana for visualization
* Docker image vulnerability scanning
* Blue/Green deployment
* Rolling deployment
* Automated rollback
* Slack/Discord notifications

---

# 🎯 What This Project Demonstrates

This project demonstrates practical knowledge of:

* Linux server administration
* Git and GitHub
* Docker
* Docker Compose
* Container registries
* GitHub Actions
* Continuous Integration
* Continuous Deployment
* SSH
* AWS EC2
* Bash scripting
* Infrastructure automation
* Cloud deployment
* DevOps workflow design

---

# 📌 Final CI/CD Flow

```text
Developer
    │
    │ git push
    ▼
GitHub
    │
    ▼
┌───────────────┐
│      CI       │
├───────────────┤
│ Checkout      │
│ Install       │
│ Test          │
│ Build         │
│ Docker Build  │
│ Docker Push   │
└───────┬───────┘
        │
        ▼
   Docker Hub
        │
        ▼
┌───────────────┐
│      CD       │
├───────────────┤
│ SSH to EC2    │
│ Create folder │
│ Create Compose│
│ Pull image    │
│ Stop old app  │
│ Start new app │
└───────┬───────┘
        │
        ▼
     AWS EC2
        │
        ▼
   Docker Container
        │
        ▼
     🌍 Website
```

---

## 👨‍💻 Author

**Olayinka Olayiwola Samuel**

Electrical & Electronics Engineering Student
DevOps & Cloud Engineering Enthusiast

---

## ⭐ Project Goal

The goal of this project is to demonstrate how modern DevOps practices can automate the complete software delivery lifecycle — from **code commit to a running application in the cloud**.

Instead of manually building Docker images, pushing them to a registry, connecting to the server, stopping containers, and starting new ones, the entire process is automated using **GitHub Actions, Docker, Docker Hub, and AWS EC2**.
