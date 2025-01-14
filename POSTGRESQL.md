# Criando imagem no Docker de 


Placeholders:
- NOMEDATABASEAQUI
- NOMECONTAINERAQUI
- NOMEIMAGEMAQUI
- PGUSERAQUI
- PGPWDAQUI


Fazer o dump do banco atual
``` Console
pg_dump -v NOMEDATABASEAQUI > /var/lib/postgresql/database_dump.sql
```

Subir container
``` Dockerfile
## filename: Dockerfile.postgres
FROM postgres:11-alpine

COPY database_dump.sql /docker-entrypoint-initdb.d/
```

Buildar img
``` Console
docker build . -t NOMEIMAGEMAQUI:latest -f Dockerfile.postgres
```

Subir container
``` Console
docker run -d --rm -p 5432:5432 -e POSTGRES_PASSWORD=PGPWDAQUI -e POSTGRES_USER=PGUSERAQUI --name NOMECONTAINERAQUI NOMEIMAGEMAQUI:latest
```

