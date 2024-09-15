
- Create `jar` file
  ```sh
  ./mvnw clean package
  ```
- To create `docker image` from `dockerFile`
  ```sh
   docker build -t <imageName>:latest . 
  ``` 
>`ImageName` format is YourDockerUserName/imageName
>
  eg
  ```sh
  docker build -t sumandevkota/myproject:latest  . 
  ```

- To check the `storage` if the `container is not running becasue of error` 
  ```sh 
   docker run -it --entrypoint /bin/sh <container_name>
   ```

- Sample of `dockerfile`
  ```docker
  FROM eclipse-temurin:17-jdk-focal

  WORKDIR /projectmgmt

  COPY target/projectManager-0.0.1-SNAPSHOT.jar  /projectmgmt/projectmanager01.jar

  COPY src/main/resources/certs /projectmgmt/certs


  EXPOSE 8080

  ENTRYPOINT ["java", "-jar", "/projectmgmt/projectmanager01.jar"]

  ```

>Note: <mark><mark><mark><mark><mark><mark>projectmgmt</mark></mark></mark></mark></mark></mark> is the name of project...you can give any name.
>
>Copying `certs` folder from `resources/certs` to `/projectmgmt/certs`

- Create a separate `application-docker.properties` file for using it with docker.
  >
  - `Original application.properties`
  >
    ```properties
      spring.application.name=projectManager

      #postgreSQL
      spring.datasource.url=jdbc:postgresql://localhost:5432/projectmanager

      spring.datasource.username=suman
      spring.datasource.password=password
      spring.jpa.database=POSTGRESQL
      spring.jpa.show-sql=true
      spring.jpa.hibernate.ddl-auto=create-drop
      spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect


      #jwt-private and public key paths
      jwt.rsa-private-key=classpath:certs/privateKey.pem
      jwt.rsa-public-key=classpath:certs/publicKey.pem

      #jwt exipration time
      myapp.jwt.expiration=15
      myapp.jwt.refresh.expiration=30


      logging.level.org.springframework.security=DEBUG
    ```
  - New `application-docker.properties`
    ```properties
    spring.application.name=projectManager

    #postgreSQL
    spring.datasource.url=jdbc:postgresql://postgres:5432/projectmanager

    spring.datasource.username=suman
    spring.datasource.password=password
    spring.jpa.database=POSTGRESQL
    spring.jpa.show-sql=true
    spring.jpa.hibernate.ddl-auto=create-drop
    spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect


    #jwt-private and public key paths
    jwt.rsa-private-key=certs/privateKey.pem
    jwt.rsa-public-key=certs/publicKey.pem

    #jwt exipration time
    myapp.jwt.expiration=15
    myapp.jwt.refresh.expiration=30


    logging.level.org.springframework.security=DEBUG
    ```
>Note: path `jwt.rsa-private-key=certs/privateKey.pem` and`jwt.rsa-public-key=certs/publicKey.pem` changed see in `application-docker.properties` as both our `jar` and `certs` is in same folder.

- Complete `docker-compose.yaml` will look like:
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
        POSTGRES_DB: projectmanager
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
    project-custom-manager:
      image: sumandevkota/projectisawesome
      container_name: project-ok
      ports:
        - "8080:8080"
      environment:
        SPRING_PROFILES_ACTIVE: docker
      depends_on:
        - postgres
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

- To check volumes
  - Use `docker volume ls`
  - `docker volume inspect <volumeName>` to check information about the specific volume
