
## 🎯 Backend Implementation – Service Bootstrap (Java + Spring Boot + JPA + Docker)

## ✅ Objective

Create a **Java Spring Boot microservice** to serve as the backend for a **Transaction Insight Service**.  
This deliverable includes a **running project skeleton**, **Dockerized setup**, and **database containerization**.  
No business logic or APIs (except a health check) will be implemented at this stage.

---

## 🏗️ Scope of Work

1. **Generate a Spring Boot microservice project**:
   - Java 17+
   - Spring Boot (latest stable version)
   - Maven as the build tool

2. **Include the following dependencies**:
   - Spring Web (for REST APIs)
   - Spring Data JPA
   - PostgreSQL Driver
   - Lombok (optional but recommended)

3. **Prepare the folder structure**, aligned with **microservices architecture principles**.

4. **Dockerized Setup**:
   - Multi-stage Dockerfile for the Spring Boot app.
   - docker-compose.yml with:
     - Spring Boot service (`app`)
     - PostgreSQL service (`db`) with:
       - POSTGRES_DB: `transactiondb`
       - POSTGRES_USER: `transaction_user`
       - POSTGRES_PASSWORD: `secure_password`
       
5. **Implement an initial health check API**:
   - `GET /health` → Response: `{"status": "UP"}`.

6. **Add application configuration files**:
   - `application.yml` (base config)
   - `application-dev.yml` (development profile)
   - `application-prod.yml` (production profile)

7. **Database Schema**:
   - `transactions` table capturing transaction details.
   - `transaction_insights` table capturing insights linked to transactions.

8. **Separate Dockerfiles if needed**, ensuring clean containerization:
   - One Dockerfile for the app.
   - Docker Compose manages both containers (app + db).

---

## 📁 Expected Folder Structure

```
   transaction-insight-service/
   ├── src/
   │   ├── main/
   │   │   ├── java/com/example/transactioninsight/
   │   │   │   ├── TransactionInsightApplication.java
   │   │   │   ├── config/
   │   │   │   ├── controller/
   │   │   │   │   └── HealthController.java
   │   │   │   ├── service/
   │   │   │   ├── repository/
   │   │   │   └── model/
   │   │   └── resources/
   │   │       ├── application.yml
   │   │       ├── application-dev.yml
   │   │       ├── application-prod.yml
   │   │       └── schema.sql
   │   └── test/java/com/example/transactioninsight/
   │       └── HealthControllerTest.java
   ├── Dockerfile
   ├── docker-compose.yml
   ├── .dockerignore
   ├── .gitignore
   ├── README.md
   └── pom.xml
```
---
## 🗄️ Database Schema

### transactions
```sql
CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    source_account VARCHAR(50) NOT NULL,
    destination_account VARCHAR(50) NOT NULL,
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    transaction_date TIMESTAMP WITHOUT TIME ZONE NOT NULL,
    status VARCHAR(20) NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITHOUT TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITHOUT TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_transactions_source_account ON transactions (source_account);
CREATE INDEX idx_transactions_destination_account ON transactions (destination_account);
CREATE INDEX idx_transactions_transaction_date ON transactions (transaction_date);
CREATE INDEX idx_transactions_status ON transactions (status);
```

### transaction_insights
```sql
CREATE TABLE transaction_insights (
    id UUID PRIMARY KEY,
    transaction_id UUID NOT NULL,
    insight_type VARCHAR(50) NOT NULL,
    insight_value TEXT NOT NULL,
    created_at TIMESTAMP WITHOUT TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITHOUT TIME ZONE DEFAULT NOW(),
    CONSTRAINT fk_transaction FOREIGN KEY(transaction_id) REFERENCES transactions(id) ON DELETE CASCADE
);

CREATE INDEX idx_insights_transaction_id ON transaction_insights (transaction_id);
CREATE INDEX idx_insights_type ON transaction_insights (insight_type);
```

---

## ⚙️ Configuration Details

### Environment Variables  
For **PostgreSQL database container**, use:

| Variable              | Example Value         | Description                          |
|-----------------------|-----------------------|--------------------------------------|
| `POSTGRES_DB`         | `transactiondb`       | Name of the database                 |
| `POSTGRES_USER`       | `transaction_user`    | DB username                          |
| `POSTGRES_PASSWORD`   | `secure_password`     | DB user password                     |

For **Spring Boot service**, use:

| Variable                       | Example Value                                              |
|--------------------------------|------------------------------------------------------------|
| `SPRING_DATASOURCE_URL`        | `jdbc:postgresql://db:5432/transactiondb`                 |
| `SPRING_DATASOURCE_USERNAME`   | `transaction_user`                                        |
| `SPRING_DATASOURCE_PASSWORD`   | `secure_password`                                         |
| `SPRING_PROFILES_ACTIVE`       | `dev`                                                     |

---

## 🐳 Docker & Docker Compose

1. **Dockerfile (Spring Boot App)**  
   - Multi-stage (optional) for efficiency.
   - Expose port `8080` by default.

2. **docker-compose.yml**  
   - Defines two services:
     - `app`: Spring Boot application container.
     - `db`: PostgreSQL database container.
   - Ensure both services are on the same network.
   - Schema initializes automatically on startup.

---

## 🔗 Endpoints

| Method | Endpoint  | Description                  |
|--------|-----------|------------------------------|
| GET    | `/health` | Health check (returns 200 OK)|

---

## 📜 Deliverables

- ✅ Working Spring Boot microservice skeleton.
- ✅ Dockerfile for app container.
- ✅ docker-compose.yml for multi-container orchestration.
- ✅ Health check endpoint responding to `GET /health`.
- ✅ README.md with instructions:
  - How to run the project.
  - How to verify health status.
  - How to connect to the DB container.