# Docker Compose(d) Databases

## Quick Start

- Install docker desktop and make sure its up and running
- Run the command below

```bash
docker compose --profile mysql up -d
docker compose --profile postgres up -d
docker compose --profile mongo up -d
docker compose --profile elasticsearch up -d
docker compose --profile redis up -d
```

## Connecting

All databases run on their corresponding default ports:

- MySQL - 3306
- Postgresql - 5432
- MongoDB - 27017

All databases either do not require authentication or use app_dev / app_dev as username / password.
