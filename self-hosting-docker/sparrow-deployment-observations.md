Points 
Working on the sparrow deployment through Docker. What i figure out is mentioned below:
    - Modified the docker-compose file (Syntax)
    - No need to create docker image from scratch as we have docker image on GitHub we can pull from there
    - Need to modify the .env which we have
    - Restart policy need to add
    - change in .env and set : KAFKA_BROKER=kafka:9092
    - change in docker-compose file: 
      from: -KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092,EXTERNAL://kafka:9094
      to : - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094 
REASON: 
**Key Changes:**

1. **Advertise Kafka's internal listener** as `PLAINTEXT://kafka:9092` for container-to-container communication.
2. **Advertise the external listener** as `EXTERNAL://localhost:9094`, so it's reachable from the host machine.
3. Expose both ports `9092` and `9094` to ensure external access.


    -  For AMD64 we dont have docker images in dockerhub so it not work in AMD architecture
