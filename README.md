# React Application with Docker & CI/CD

A modern React application with containerized development and production environments, featuring automated testing and deployment to AWS Elastic Beanstalk.

## 🚀 Features

- **Docker Development Environment** - Consistent development setup across all machines
- **Multi-stage Production Build** - Optimized production Docker image with nginx
- **Automated CI/CD Pipeline** - GitHub Actions workflow for testing and deployment
- **AWS Elastic Beanstalk Deployment** - Automated deployment to scalable cloud infrastructure
- **Automated Testing** - Tests run automatically on every push to main branch

## 📋 Prerequisites

- **Node.js** 20.x or higher
- **Docker** Desktop installed and running
- **AWS Account** with Elastic Beanstalk setup
- **GitHub Repository** with Actions enabled

## 🛠️ Tech Stack

- **Frontend**: React
- **Container Runtime**: Docker
- **Web Server**: nginx (production)
- **CI/CD**: GitHub Actions
- **Cloud Platform**: AWS Elastic Beanstalk
- **Infrastructure**: Docker on AWS Linux

## 📁 Project Structure

```
.
├── Dockerfile              # Production build (React + nginx)
├── Dockerfile.dev          # Development build
├── Dockerrun.aws.json      # AWS EB configuration
├── .github/
│   └── workflows/
│       └── deploy.yml      # CI/CD pipeline
├── src/                    # React source code
├── public/                 # Static assets
├── package.json            # Dependencies
└── README.md              # This file
```

## 🏃 Getting Started

### Local Development (without Docker)

```bash
# Install dependencies
npm install

# Start development server
npm start

# Run tests
npm test

# Build for production
npm run build
```

### Local Development (with Docker)

```bash
# Build development image
docker build -f Dockerfile.dev -t react-app-dev .

# Run development container
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app my-frontend-app

# Access app at http://localhost:3000
```

### Production Build (with Docker)

```bash
# Build production image
docker build -t react-app-prod .

# Run production container
docker run -p 80:80 react-app-prod

# Access app at http://localhost
```

## 🔧 Docker Configuration

### Development Dockerfile (`Dockerfile.dev`)

- Uses Node.js 20 Alpine image
- Installs all dependencies including dev dependencies
- Mounts source code for hot-reloading
- Runs `npm start` for development server

### Production Dockerfile (`Dockerfile`)

**Stage 1: Build**
- Builds optimized React production bundle
- Runs `npm run build`

**Stage 2: Serve**
- Uses nginx Alpine image for minimal size
- Serves static files from `/usr/share/nginx/html`
- Exposes port 80

## 🚀 CI/CD Pipeline

The GitHub Actions workflow automatically:

1. **Test Stage** (on push to main):
   - Checks out code
   - Builds Docker image using `Dockerfile.dev`
   - Runs all tests inside container with `CI=true`

2. **Deploy Stage** (after tests pass):
   - Checks out code
   - Creates deployment package (zip file)
   - Deploys to AWS Elastic Beanstalk
   - AWS builds production image and starts container

### Workflow Trigger

```yaml
on:
  push:
    branches:
      - main
```

## ☁️ AWS Elastic Beanstalk Setup

### Prerequisites

1. **Create EB Application and Environment**:
   - Go to AWS Console → Elastic Beanstalk
   - Create new application
   - Create environment with Docker platform

2. **Create IAM User for GitHub Actions**:
   ```bash
   # User name: github-actions-deployer
   # Policies needed:
   - AWSElasticBeanstalkFullAccess
   - AmazonS3FullAccess (or limited to EB bucket)
   ```

3. **Add GitHub Secrets**:
   - Repository → Settings → Secrets and variables → Actions
   - Add `AWS_ACCESS_KEY_ID`
   - Add `AWS_SECRET_ACCESS_KEY`

### Dockerrun.aws.json Configuration

```json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": ".",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": 80,
      "HostPort": 80
    }
  ]
}
```

## 🔐 Environment Variables

### GitHub Secrets (Required)

| Secret Name | Description |
|------------|-------------|
| `AWS_ACCESS_KEY_ID` | IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key |

### AWS Configuration (in workflow)

| Variable | Description | Example |
|----------|-------------|---------|
| `application_name` | EB application name | `react-frontend` |
| `environment_name` | EB environment name | `react-frontend-env` |
| `region` | AWS region | `us-east-1` |

## 📝 Useful Commands

### Docker Commands

```bash
# Build development image
docker build -f Dockerfile.dev -t react-app-dev .

# Build production image
docker build -t react-app-prod .

# Run tests in container
docker run -e CI=true react-app-dev npm test

# Remove all containers and images (cleanup)
docker system prune -a

# View running containers
docker ps

# View all containers (including stopped)
docker ps -a

# Stop a container
docker stop <container-id>

# Remove a container
docker rm <container-id>
```

### AWS CLI Commands

```bash
# List EB applications
aws elasticbeanstalk describe-applications

# List EB environments
aws elasticbeanstalk describe-environments --application-name your-app-name

# Check environment health
aws elasticbeanstalk describe-environment-health --environment-name your-env-name --attribute-names All

# View recent events
aws elasticbeanstalk describe-events --environment-name your-env-name --max-records 10
```

### GitHub Actions

```bash
# View workflow runs
gh run list

# View specific workflow run
gh run view <run-id>

# Re-run failed workflow
gh run rerun <run-id>
```

## 🐛 Troubleshooting

### Docker Issues

**Problem**: Container fails to start
```bash
# Check logs
docker logs <container-id>

# Check if port is already in use
lsof -i :3000  # or :80 for production
```

**Problem**: Changes not reflecting in dev container
```bash
# Make sure you're mounting the volume correctly
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app my-frontend-app
```

### AWS Deployment Issues

**Problem**: Deployment fails
```bash
# Check EB environment logs in AWS Console
# Or use CLI:
aws elasticbeanstalk describe-events --environment-name your-env-name

# Check if IAM permissions are correct
aws iam get-user-policy --user-name github-actions-deployer --policy-name your-policy-name
```

**Problem**: Application not accessible after deployment
```bash
# Check environment health
aws elasticbeanstalk describe-environment-health --environment-name your-env-name --attribute-names All

# Verify Dockerrun.aws.json port mappings
# Make sure nginx is running on port 80 in container
```

### GitHub Actions Issues

**Problem**: Tests fail in CI but pass locally
```bash
# Run tests with CI flag locally
docker run -e CI=true react-app-dev npm test

# Check if all dependencies are in package.json
```

**Problem**: AWS credentials invalid
```bash
# Verify secrets are set correctly in GitHub
# Test credentials locally:
aws sts get-caller-identity --profile github-actions
```

## 📊 Deployment Flow

```
Developer pushes to main
        ↓
GitHub Actions triggered
        ↓
Run Tests in Docker container
        ↓
    Tests Pass?
        ↓ (Yes)
Create deployment package
        ↓
Upload to AWS S3
        ↓
AWS Elastic Beanstalk
        ↓
Build production Docker image
        ↓
Deploy to environment
        ↓
Application live! 🎉
```

## 🔒 Security Best Practices

- ✅ Never commit AWS credentials to repository
- ✅ Use GitHub Secrets for sensitive data
- ✅ Create dedicated IAM user for CI/CD with minimal permissions
- ✅ Rotate AWS access keys regularly
- ✅ Use `.gitignore` to exclude sensitive files
- ✅ Enable MFA on AWS root account
- ✅ Review IAM policies periodically

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [React Documentation](https://react.dev/)
- [nginx Documentation](https://nginx.org/en/docs/)

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👨‍💻 Author

**Kiran**

---

**Happy Coding! 🚀**