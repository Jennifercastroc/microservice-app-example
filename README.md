# Microservice App - PRFT Devops Training
  

### Team members  
- Jennifer Castro  
- Julian Motta  



## Overview  
This application is based on a project by **bortizf** and follows a **microservices architecture** implemented in different programming languages and frameworks.  

The project was modified to:  
- Implement **two cloud design patterns**: **API Gateway** and **Rate Limiting**.  
- Achieve deployment on **Azure** using:  
  - **1 infrastructure pipeline**.  
  - **5 development pipelines** (one for each microservice).  

 **Note:** The workshop is divided into two repositories:  
- **Development repository** (this one).  
- **Infrastructure repository** (contains Terraform + Ansible).  



## Implemented Architecture  
![Implemented Architecture](image)  



## Branching Strategy  

### Development (GitFlow)  
We chose **GitFlow** to manage development because it provides a clear separation between:  
- **Feature branches** (for isolated work).  
- **Develop branch** (safe integration point).  
- **Main branch** (stable production-ready code).  

This ensures each microservice can be developed independently while maintaining a stable codebase.  

### Operations (GitHub Flow)  
For **infrastructure and deployment**, we implemented **GitHub Flow**:  
- A single long-lived **main branch**.  
- Short-lived feature branches merged through **Pull Requests**.  

This approach supports **continuous deployment** while ensuring **infrastructure quality** through reviews.  

---

## Container Configuration  

Each microservice includes a **Dockerfile**. Example (frontend):  

```dockerfile
FROM node:8.17.0
WORKDIR /app

COPY package*.json ./  
RUN npm install        

COPY . .             
RUN npm run build     

EXPOSE 8080
CMD ["npm", "start"]
````

Integration of services is done with **docker-compose**. Example (users-api):

```yaml
users-api:
  build:
    context: ./users-api
  container_name: users-api
  environment:
    - SERVER_PORT=8083
    - JWT_SECRET=PRFT
  ports:
    - "8083:8083"
  depends_on:
    - auth-api
  networks:
    - app-network
```

Additional services:

* **Redis** → cache storage & messaging.
* **Zipkin** → distributed tracing.

---

## Implemented Patterns

### 1. API Gateway (Nginx)

Configured via **nginx.conf** to route requests.

* **Frontend (port 80)**.
* **Backend microservices (port 8000)**.

Example (frontend routing):

```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://frontend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### 2. Rate Limiting (Nginx)

Added to **Auth-API login endpoint** to prevent abuse.

```nginx
limit_req_zone $http_x_forwarded_for zone=login_limit:10m rate=5r/m;
limit_req_status 429;

location /login {
    limit_req zone=login_limit burst=3 nodelay;
    proxy_pass http://auth-api:8081;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

---

## Infrastructure (Infrastructure Repository)

* **Terraform** → creates an **Azure Virtual Machine**.
* **Ansible** → installs Docker, dependencies, and clones the repo.

### Example (Terraform inbound rule):

```hcl
security_rule {
  name                       = "HTTP"
  priority                   = 300
  direction                  = "Inbound"
  access                     = "Allow"
  protocol                   = "Tcp"
  destination_port_range     = "80"
  source_address_prefix      = "*"
  destination_address_prefix = "*"
}
```

### Example (Ansible inventory):

```ini
[azure_vm]
0.0.0.0 ansible_user=<user> ansible_ssh_pass=<password> 
```

Roles:

* `docker_install`
* `pip_install`
* `docker_compose`

---

## Pipelines and Scripts

We defined **8 pipelines**:

1. **Infrastructure-up** → deploys Azure VM (Terraform + Ansible).
2. **Infrastructure-down** → tears down Azure resources.
3. **Nginx pipeline**.
4. **5 pipelines** (one for each microservice).

---

## Execution

### Local

Run:

```bash
docker-compose up -d
```

Access via [http://localhost:80](http://localhost:80).

### Cloud (Azure Pipelines)

1. Create the **8 pipelines** from provided YAML.
2. Create a **variable group**: `variable-group-taller`.
3. Add variables:

   * `AZURE_ACCOUNT`
   * `RESOURCE_GROUP`
   * `VM_NAME`
   * `VM_PASSWORD`
   * `VM_USERNAME`
4. Run `Infrastructure-up` to deploy / `Infrastructure-down` to tear down.

When changes are made to a microservice, its pipeline will rebuild and redeploy the container automatically.



📌 **Maintainers**

* Jennifer Castro
* Julian Motta

