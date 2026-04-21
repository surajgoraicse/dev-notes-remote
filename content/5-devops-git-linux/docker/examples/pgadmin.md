

```bash
docker run -d \
  --name pgadmin \
  --network server_default \
  -e PGADMIN_DEFAULT_EMAIL=surajgoraicse@gmail.com \
  -e PGADMIN_DEFAULT_PASSWORD=surajgoraicse \
  -p 5050:80 \
  dpage/pgadmin4
```


## connect to docker postgres

- both postgres and pgadmin must run on the same network.
- **Host name/address:** `server-db-1` this should match the name show in `docker ps` (container name)
- **Port:** `5432`  pgadmin will directly connect to the postgres internal port running in docker instead of host machine port.
- username and password as defined in postgres docker config.
- maintenance database : `postgres` initial default db, use `postgres` as it is automatically created.

![[Pasted image 20260417130430.png]]


The connection string : 
1. Internal container topology : `postgres://postgres:mysecretpassword@server-db-1:5432/swags.me?sslmode=disable`
2. External host topology : `postgres://postgres:mysecretpassword@localhost:5434/swags.me?sslmode=disable`
