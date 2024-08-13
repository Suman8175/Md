# Using redis via docker


## Step1 : ***Run docker desktop***


## Step2: ***Run docker command***

```yaml
version: '3.8'

services:
  redis-stack:
    image: redis/redis-stack:latest
    container_name: redis-stack
    ports:
      - "6379:6379"
      - "8001:8001"
    restart: always

```
- We can run localhost:8001 to visualize 8001 whereas redis runs on port 6379.

## Step3: ***Run redis through cmd***
- Run docker command to get container id
    ```sh
    docker ps
    ```
    >Get container id from it
-Run dollowing command in cmd
    ```sh
    docker exec -it <containerid> bash
    ```
    >-it stands for interactive mode
    >
    >bash is accessing bash of the container

- Use redis command 
    ```sh
    redis-cli ping
    ```
    or you can do 
    ```sh
    redis-cli
    ```
    and
    ```sh
    ping
    ```

