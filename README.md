
# ExpenTrack - Automated Expense Tracker 📈

ExpenTrack is a smart, automated expense tracking application built on an event-driven microservices architecture. It intelligently parses bank transaction SMS messages using a Large Language Model (LLM) to automatically log expenses, eliminating the need for manual data entry.

This project demonstrates a robust backend system featuring secure authentication, asynchronous communication via message queues, and automated cloud deployment.

## 🌟 Key Features

* **🤖 AI-Powered SMS Parsing**: Utilizes an LLM (MistralAI) via LangChain to accurately extract expense details (amount, merchant, currency) from unstructured bank SMS messages.
* **🔐 Secure Authentication**: Implements JWT-based authentication and authorization with access and refresh tokens for secure API access.
* **🏛️ Microservices Architecture**: Decoupled services (Auth, User, Expense, AI) that communicate asynchronously using Kafka, ensuring high scalability and resilience.
* **📨 Event-Driven Communication**: Services react to events published on Kafka topics (e.g., `user_service`, `expense_service`), promoting loose coupling and independent service development.
* **☁️ Automated Cloud Deployment**: Features a CI/CD pipeline using GitHub Actions to automatically build, containerize, and deploy the authentication service to AWS via a CloudFormation template.
* **🐳 Containerized Environment**: The entire application stack is containerized with Docker and orchestrated using Docker Compose for consistent development and easy setup.

## 🏗️ System Architecture

The application is composed of four main microservices that communicate through a Kafka message broker.

1.  **Auth Service**: Handles user registration and login. Upon successful signup, it publishes user details to the `user_service` topic on Kafka.
2.  **User Service**: Subscribes to the `user_service` topic to create and manage user profile data in its own database.
3.  **DS Service (AI Service)**: Receives raw SMS text from the user. It uses an LLM to parse the text, extracts structured expense data, and publishes it to the `expense_service` topic.
4.  **Expense Service**: Subscribes to the `expense_service` topic to log the parsed expense details into the database, associating them with the correct user.


[![KuGknj4.md.png](https://iili.io/KuGknj4.md.png)](https://freeimage.host/i/KuGknj4)


## 🛠️ Tech Stack

| Category              | Technology                                                                                                                                                                                                                                                                          |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Backend Services** | ![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white) ![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=spring&logoColor=white) ![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) ![Flask](https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white) |
| **Database** | ![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)                                                                                                                                                                                    |
| **Messaging Queue** | ![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)                                                                                                                                                                 |
| **AI & Machine Learning** | ![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white) ![LangChain](https://img.shields.io/badge/LangChain-000000?style=for-the-badge&logo=langchain&logoColor=white)                                                               |
| **DevOps & Cloud** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)                               |

## 📦 Installation & Setup Guide

Follow these steps to get the project running locally.

### Prerequisites

* Git
* Docker and Docker Compose
* Java 21+
* Python 3.11+
* An OpenAI/MistralAI API Key

### 1. Clone the Repository

```bash
git clone https://github.com/al0nec0der/ExpenTrack.git
cd expentrack
````

### 2\. Configure Environment Variables

The `dsService` requires an API key for the LLM. Navigate to the `docker-compose.yml` file and replace the placeholder value.

In `docker-compose.yml`, find the `dsservice` definition and update the `OPENAI_API_KEY`:

```yaml
services:
  # ... other services
  dsservice:
    build: ./dsService
    container_name: dsservice
    restart: on-failure
    ports:
      - "8010:8010"
    environment:
      OPENAI_API_KEY: your_actual_api_key # <--- REPLACE THIS
      KAFKA_HOST: kafka
      KAFKA_PORT: 9092
```

### 3\. Build and Run the Services

Use Docker Compose to build the images and start all the containers.

```bash
docker-compose up --build
```

The services will be available at the following ports:

  * **Auth Service**: `http://localhost:9898`
  * **User Service**: `http://localhost:9810`
  * **Expense Service**: `http://localhost:9811`
  * **DS Service**: `http://localhost:8010`

## 🚀 Usage (API Endpoints)

You can interact with the application using any API client like Postman or `curl`.

### 1\. Sign Up a New User

This will create a new user in the `authservice` and publish an event to `userservice` to create a corresponding user profile.

```bash
curl --location 'http://localhost:9898/auth/v1/signup' \
--header 'Content-Type: application/json' \
--data '{
    "username": "johndoe",
    "password": "password123",
    "first_name": "John",
    "last_name": "Doe",
    "phone_number": 1234567890,
    "email": "john.doe@example.com"
}'
```

**Response:**
You will receive an `access_token`, `refresh_token`, and `user_id`. Save these for the next steps.

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiJ9...",
    "token": "a1b2c3d4-e5f6-7890-gh12-ijklmnopqrst",
    "user_id": "z9y8x7w6-v5u4-3210-ab98-fedcba987654"
}
```

### 2\. Submit an SMS for Expense Parsing

Use the `access_token` and `user_id` from the signup step to authenticate your request with the `dsService`.

```bash
# Replace YOUR_USER_ID and YOUR_ACCESS_TOKEN with the values from the previous step
export USER_ID="z9y8x7w6-v5u4-3210-ab98-fedcba987654"
export ACCESS_TOKEN="eyJhbGciOiJIUzI1NiJ9..."

curl --location 'http://localhost:8010/v1/ds/message' \
--header "x-user-id: $USER_ID" \
--header "Authorization: Bearer $ACCESS_TOKEN" \
--header 'Content-Type: application/json' \
--data '{
    "message": "Transaction alert: You have spent INR 450.50 at STARBUCKS with your credit card."
}'
```

**Response:**
The `dsService` will return the parsed data and publish it to Kafka for the `expenseService` to consume and save.

```json
{
    "amount": "450.50",
    "merchant": "STARBUCKS",
    "currency": "INR",
    "user_id": "z9y8x7w6-v5u4-3210-ab98-fedcba987654"
}
```

### 3\. Get All Expenses for a User

Retrieve all logged expenses for the user from the `expenseService`.

```bash
curl --location 'http://localhost:9811/expense/v1/getExpense' \
--header "X-User-Id: $USER_ID" \
--header "Authorization: Bearer $ACCESS_TOKEN"
```

**Response:**

```json
[
    {
        "external_id": "c4d5e6f7-...",
        "amount": 450.50,
        "user_id": "z9y8x7w6-v5u4-3210-ab98-fedcba987654",
        "merchant": "STARBUCKS",
        "currency": "INR",
        "created_at": "2025-09-14T14:00:00.000+00:00"
    }
]
```

## ☁️ Deployment

The `authservice` is configured for automated deployment to AWS. The workflow is defined in `.github/workflows/deploy.yml` and uses the infrastructure-as-code template in `authservice/cloudformation-template.yaml`.

**Pipeline Steps:**

1.  **On push to `main`**: The GitHub Actions workflow is triggered.
2.  **AWS Login**: Authenticates with AWS using repository secrets.
3.  **Build & Push to ECR**: Builds the service's Docker image and pushes it to Amazon ECR.
4.  **Deploy CloudFormation Stack**: Deploys the infrastructure defined in the CloudFormation template, which includes an Application Load Balancer (ALB), an Auto Scaling Group, and EC2 launch templates to run the containerized service.

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## 📞 Contact

  * **GitHub**: [github.com/al0nec0der](https://www.google.com/search?q=https://github.com/al0nec0der)
  * **LinkedIn**: [linkedin.com/in/codewithteja](https://linkedin.com/in/codewithteja)

<!-- end list -->
