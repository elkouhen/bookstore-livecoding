## Run only database in docker

## bookstore-pg


```bash 
docker run --rm -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgres -e POSTGRES_DB=postgres -p 5432:5432 postgres:13.2-alpine
```

## bookstore-api

```bash 
mvn spring-boot:run
```

## bookstore-gui 

```bash 
npm run serve
```

