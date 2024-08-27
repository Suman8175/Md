# PostgreSQL in docker

```yaml
services:
  postgres:
    image: postgres
    container_name: postgres_db
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: suman
      POSTGRES_PASSWORD: password
      POSTGRES_DB: jpamapping
    volumes:
      - postgres:/var/lib/postgresql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - network_establish
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin_db
    restart: unless-stopped
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin4@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-admin}
      PGADMIN_DEFAULT_SERVER_PORT: 5432  # Define port for PostgreSQL
    volumes:
      - pgadmin:/var/lib/pgadmin
    networks:
      - network_establish
networks:
  network_establish:
    name: network_establish
    driver: bridge
volumes:
  postgres:
  pgadmin:
```
***init.sql***
```sql
CREATE DATABASE projectManager;
```

  * In application.properties

    ```properties
    spring.datasource.url=jdbc:postgresql://localhost:5432/projectManager
    spring.datasource.username=suman
    spring.datasource.password=password
    spring.jpa.database=POSTGRESQL
    spring.jpa.show-sql=true
    spring.jpa.hibernate.ddl-auto=create-drop
    spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
    ```