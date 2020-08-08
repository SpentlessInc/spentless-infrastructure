# Sheet Without Shit. Infrastructure

### How to run development env?
1. Create new directory: sws-infrastructure.
2. Go to created directory: `cd sws-infrastructure`
3. Clone SWS services:
    ```shell script
    git clone git@github.com:SheetWithoutShit/sws-server.git
    git clone git@github.com:SheetWithoutShit/sws-collector.git
    git clone git@github.com:SheetWithoutShit/sws-postgres.git
    ```
4. Create docker-compose.yml with the following configurations:
    ```yaml
   version: '3.7'

    services:
      ngrok:
        container_name: sws-ngrok
        image: wernight/ngrok
        env_file:
          - .env
        environment:
          - NGROK_PORT=collector:5010
        ports:
          - 4040:4040
        restart: always
    
      postgres:
        image: postgres:10
        container_name: sws-postgres
        working_dir: /docker-entrypoint-initdb.d
        env_file:
          - .env
        ports:
          - 5432
        volumes:
          - ./sws-postgres:/docker-entrypoint-initdb.d/
          - postgres-data:/var/lib/postgresql/data/
        restart: always
    
      redis:
        image: redis:5
        container_name: sws-redis
        ports:
          - 6379
        env_file:
          - .env
        restart: always
    
      collector:
        container_name: sws-collector
        build: ./sws-collector
        command: adev runserver run.py --app-factory=init_app --port=5010
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./sws-collector/collector:/collector
        depends_on:
          - postgres
          - redis
        ports:
          - 5010:5010
        restart: always
    
      server:
        container_name: sws-server
        build: ./sws-server
        command: adev runserver run.py --app-factory=init_app --port=5000
        stdin_open: true
        tty: true
        env_file:
          - .env
        volumes:
          - ./sws-server/server:/server
        depends_on:
          - postgres
          - redis
          - ngrok
        ports:
          - 5000:5000
        restart: always
    
    volumes:
      postgres-data:
   ```
5. Create .env file with the following variables:
    ```shell script
    MONOBANK_WEBHOOK_SECRET=

    POSTGRES_HOST=postgres
    POSTGRES_PORT=5432
    POSTGRES_USER=
    POSTGRES_PASSWORD=
    POSTGRES_DB=
    
    REDIS_HOST=redis
    REDIS_PORT=6379
    
    SERVER_SECRET=
    JWT_EXP_DAYS=7
    JWT_ALGORITHM=HS256
   ```
6. Run docker compose up command: `docker-compose up`