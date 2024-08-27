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
##

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
    ><mark>-it </mark> stands for interactive mode
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
##
## Step4: ***Redis commands***
- ### String

    > <mark> SET </mark> is used to put data in redis
    >
    > <mark> GET </mark> is used to fetch data in redis

    - To set value
    ```sh
    SET name:1 Suman
    ```
    - To set value with spaces
    ```sh
    SET name:1 "Suman Devkota"
    ```
    - To set value only if key doesn't exist
    ```sh
    set name:3 "Suman" nx   
    ```
    ><mark>nx</mark> makes sure it doesn't replace old value as set by default replace old values with same key if inserted with same key twice
    - To set value only key is present previously (like edit)
    ```sh
    set name:3 "Suman" nx  
    ```
    - To set multiple value in one command
    ```sh
    mset logo:1 "Nepal" logo:2 "Ares" logo:3 "Wera"
    ```
    - To get multiple value in one command
    ```sh
    mget logo:1 logo:2 logo:3
    ```
    - To get all keys with same name like ```name:1```,```name:2```,```name:3``` all starts with key ```name:```
    ```sh
    KEYS name:*
    ```

- ### Expire
    - To exipre a data type use
    ```sh
    EXPIRE  <key> <seconds>
    ```
    <mark> key </mark> is any key you want to expire
    >
    <mark> seconds </mark> pass no of seconds like ```10``` ```20``` ```30```
- ### JSON
     > Note : We need to install RedisJSON Module to get json support in redis
   >
   >Hit command ``` info modules ``` to get modules list ..if you have RedisJSON you should see it in modules
   >

    - To create a json value use command
    ```sh
    JSON.SET obj1 $ '{"name":"Suman", "age": 42}'
    ```
    - To fetch JSON
    ```sh
    JSON.GET obj1
    ```
    - To fetch value from key in JSON
    ```sh
    JSON.GET obj1 $.<keyName>
    ```
    - To update value from JSON
    > ```nx``` or ```xx``` are not supported in JSON so if we put new key and value new key and value are created in existing JSON
    ```sh
    JSON.SET obj1 $.name '"Suman2"'
    ```
    - To remove key-value from json
    ```sh
    JSON.DEL obj1 $.age
    ```
- ### List
    - To treat list as queue(means left push)
    ```sh
    lpush <listName> "Hello"
    ```
    - To treat list as stack (means right push)
    ```sh
    rpush <listName> "Hi"
    ``` 
    - To pop  elements from queue (Left insert-Right pop)
    ```sh
    rpop <listName>
    ```
    - To pop  elements from stack (Left insert-Left pop or Right insert-Right Pop)
    ```sh
    lpop <listName>
    ```
    - To find length of list
    ```sh
    llen <listName>
    ```
    - To show data in list  ```0 -1``` gives all data do ```0 0``` for first,```0 1``` for two data,and so on
    ```sh
    lrange <listName> 0 -1
    ```
    - To block if no data in list ```60``` is no of seconds it blocks if no data in list(it waits until the given time...if message comes within time it gives the message and exit waiting time)
    ```
    blpop <listName> 60
    ```
- ### Set
    >Set donot contains duplicate value

    - To add in set
    ```sh
    sadd <setName> <value>
    ```
    - To remove value  from set
    ```sh
    sadd <setName> <value>
    ```
    - To find if the value is present in set or not
    ```sh
    sismember <setName> <valueToBeChecked>
    ```
- ### Hash
    - To add hash(field:value) in redis
    ```sh
    HSET bike:1 model Deimos brand Ergonom type 'Enduro bikes' price 4972
    ```
    visaulaize it as

    Field         | Value
    ------------- | -------------
    model         | Deimos
    brand         | Ergonom
    types         | Enduro bikes
    price         | 4972

    - To get value from field in hash
    ```sh
    HGET bike:1 model
    ```
    - To get all field value of hash
    ```
    HGETALL bike:1
    ```
    - To fetch multiple value at once
    ```sh
     HMGET bike:1 model price
    ```
- ## Geo
    - To add geographic data 
    ```sh
    GEOADD <keyName> <longitude> <latitude> <placeName>
    ```
    eg:
    ```sh
     GEOADD hotels:available 84.011033 28.178437 hotel:shangrilla
     GEOADD hotels:available 84.010750 28.183530 hotel:tal
     GEOADD hotels:available 84.003836 28.184770 hotel:basketball
     GEOADD hotels:available 84.003385 28.192871 hotel:kalika
     GEOADD hotels:available 84.001238 28.204640 hotel:amarsingh
     GEOADD hotels:available 83.999735 28.212154 hotel:gandakiHospital
    ```
    - To find all available places within given radius
    ```sh
    GEOSEARCH <keySearch> FROMLONLAT <longitude> <latitude> BYRADIUS <noOfKm> km WITHDIST
    ```
    eg
    ```sh
    GEOSEARCH hotels:available FROMLONLAT 84.013233 28.178437 BYRADIUS 1 km WITHDIST
    ```