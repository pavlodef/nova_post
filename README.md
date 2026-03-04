# nova_post

Microservices backend simulating a delivery platform (loosely inspired by Nova Poshta). Four independent FastAPI services communicate via RabbitMQ, each with its own PostgreSQL + MongoDB database. Observability stack included: Loki + Promtail + Grafana.

## Services

| Service | Port | Description |
|---------|------|-------------|
| `auth_service` | 8000 | User registration, JWT auth, token validation |
| `branch_service` | 8001 | Post office branches management |
| `courier_service` | 8002 | Courier management, listens to RabbitMQ events |
| `shipment_service` | 8003 | Shipment lifecycle, publishes events to RabbitMQ |

## Stack

Python 3.11, FastAPI, SQLAlchemy, Alembic, PostgreSQL, MongoDB (Motor), RabbitMQ, Grafana, Loki, Promtail, Docker Compose

## Quick Start

```bash
git clone https://github.com/pavlodef/nova_post.git
cd nova_post

# copy and fill env files for each service
for s in auth_service branch_service courier_service shipment_service; do
  cp $s/.env.example $s/.env
done

docker compose up --build
```

| UI | URL |
|----|-----|
| RabbitMQ management | http://localhost:15672 (admin/admin) |
| Grafana | http://localhost:3000 (admin/admin) |
| Auth API docs | http://localhost:8000/docs |
| Branch API docs | http://localhost:8001/docs |
| Courier API docs | http://localhost:8002/docs |
| Shipment API docs | http://localhost:8003/docs |

## Environment Variables

Each service has its own `.env`. Copy from `.env.example` inside each service folder:

```env
SECRET_KEY=your_secret_key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

DB_NAME=service_db
DB_PORT=5432
DB_HOST=localhost
DB_USER=postgres
DB_PASSWORD=your_password

MONGO_DB_USER=your_user
MONGO_DB_PASSWORD=your_password
MONGO_DB_URL=mongodb+srv://<user>:<password>@<cluster>.mongodb.net/
```

## Project Structure

```
nova_post/
├── auth_service/       # JWT auth + user management
├── branch_service/     # branch CRUD
├── courier_service/    # courier management + RabbitMQ consumer
├── shipment_service/   # shipment tracking + RabbitMQ publisher
├── promtail-config.yml # log collection config
└── docker-compose.yml  # full stack
```

## Architecture

```
 Client
   │
   ├── auth_service     (JWT)
   ├── branch_service
   ├── courier_service  ◄── RabbitMQ ◄── shipment_service
   └── shipment_service

Each service → own PostgreSQL + MongoDB
All services → Loki (via Promtail) → Grafana
```
