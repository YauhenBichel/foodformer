# MSDS - MLOps course - Foodformer <img src="./images/foodformer_logo.jpeg" alt="foodformer_logo" width="20"/>

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/downloads/release/python-31011/)
[![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nico-usf/foodformer)

Home of the Foodformer MLOps project for the [MLOps course the Masters of Science in Data Science of the University of San Francisco](https://catalog.usfca.edu/preview_course_nopop.php?catoid=38&coid=562876).

The goal of this project is to create a food classification app powered by a Vision Transformer. The repo showcases several MLOps concepts:
- Model development on GPU instances with AWS SageMaker
- Experiment tracking and artifacts management with Weights & Biases
- API development (FastAPI, Docker) and deployment (AWS Fargate, AWS ECR)
- Setting up a demo environment with GradIO
- Continuous Deployment with GitHub Actions
- Operational and functional monitoring with Grafana
- Load testing with Locust

Here is the complete architecture diagram with tools icons:

<img src="./images/architecture_foodformer.svg" width="600" height="600" alt="Architecture Diagram">

## Development

To setup this repo locally, run `./setup.sh`, it will simply install the dependencies and pre-commit hooks. 

I recommend creating a virtual environment, see intructions [in the FAQ section](#faq).


## Testing the API

You can use API platforms like Postman or Insomnia, the Swagger interface of the API (http://localhost:8080/docs), or the command-line tool `curl`:

- for the healthcheck endpoint: `curl http://localhost:8080`
- for a post endpoint called `predict`:

```bash
curl -X 'POST' \
  'http://0.0.0.0:8080/predict' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@image.jpg;type=image/jpeg'
```

## Deployment to AWS Fargate

Fargate is a serverless deployment solution for Docker containers. Deploying a Docker image to Fargate requires uploading the image to a registry like AWS ECR. While redeployments are automated through GH Actions, initially creating the Fargate service requires following the instructions below.

### Build and push the Docker image

In a terminal:

- Build the dockerfile: `docker build -t foodformer .`
- Fetch and store your AWS account id and AWS region: `export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text) && export AWS_REGION='us-east-2'`
- Authenticate with AWS ECR: `aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com` -> You should see the following message: "Login Succeeded"
- Create a repo in ECR: `aws ecr create-repository --repository-name foodformer --image-scanning-configuration scanOnPush=true --region $AWS_REGION`
- Tag and push you Docker image: `docker tag foodformer:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/foodformer` followed by `docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/foodformer` (this command will take a while, it's uploading the entire Docker image to ECR).

### Deploy API container with Fargate

Follow [this guide](https://app.tango.us/app/workflow/Creating-and-Deploying-an-ECS-Cluster-for-Foodformer-Application-7ede665f523044ec93f0239ad24f41a5) to create the required services in the AWS Console.


## FAQ

### How to create a virtual environment?

With [PyEnv](https://github.com/pyenv/pyenv)) you can run the following (MacOS):

```bash
brew install pyenv
pyenv init
pyenv install -s 3.10.10
pyenv virtualenv 3.10.10 foodformer
pyenv activate foodformer
pyenv local foodformer
```

Pyenv will automatically load the correct virtual environment when you `cd` into this directory, by reading the `.python-version` file created by the `pyenv local` step.