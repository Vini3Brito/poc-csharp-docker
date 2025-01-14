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
# dump build stage
FROM postgres:11-alpine as dumper

COPY test_dump.sql /docker-entrypoint-initdb.d/

RUN ["sed", "-i", "s/exec \"$@\"/echo \"skipping...\"/", "/usr/local/bin/docker-entrypoint.sh"]

ENV POSTGRES_USER=postgres
ENV POSTGRES_PASSWORD=postgres
ENV PGDATA=/data

RUN ["/usr/local/bin/docker-entrypoint.sh", "postgres"]

# final build stage
FROM postgres:11-alpine

COPY --from=dumper /data $PGDATA
```

Buildar img
``` Console
docker build . -t NOMEIMAGEMAQUI:latest -f Dockerfile.postgres
```

Subir container
``` Console
docker run -d --rm -p 5432:5432 -e POSTGRES_PASSWORD=PGPWDAQUI -e POSTGRES_USER=PGUSERAQUI --name NOMECONTAINERAQUI NOMEIMAGEMAQUI:latest
```

Criar arquivo Makefile
```Makefile
default: all

.PHONY: default all fetch_dump

date := `date '+%Y-%m-%d'`
TARGET_IMAGE ?= my_app

all: check_vars fetch_dump generate_image push_to_registry clean finished

check_vars:
	@test -n "$(DB_ENDPOINT)" || (echo "You need to set DB_ENDPOINT environment variable" >&2 && exit 1)
	@test -n "$(DB_NAME)" || (echo "You need to set DB_NAME environment variable" >&2 && exit 1)
	@test -n "$(DESTINATION_REPOSITORY)" || (echo "You need to set DESTINATION_REPOSITORY environment variable" >&2 && exit 1)

fetch_dump: DB_USER ?= postgres
fetch_dump:
	@echo ""
	@echo "====== Fetching remote dump ======"
	@PGPASSWORD="$(DB_PASSWORD)" pg_dump -h $(DB_ENDPOINT) -d $(DB_NAME) -U $(DB_USER) > dump.sql

generate_image:
generate_image:
	@docker build . -t $(TARGET_IMAGE):latest -t $(DESTINATION_REPOSITORY)/$(TARGET_IMAGE):latest -t $(DESTINATION_REPOSITORY)/$(TARGET_IMAGE):$(date)

push_to_registry:
	@echo ""
	@echo "====== Pushing image to repository ======"
	@docker push $(DESTINATION_REPOSITORY)/$(TARGET_IMAGE)

clean:
	@echo ""
	@echo "====== Cleaning used files ======"
	@rm -f dump.sql

finished:
	@echo ""
	@echo "Finished with success. Pushed image to $(DESTINATION_REPOSITORY)/$(TARGET_IMAGE)"
```


Rodar Makefile
```console
make DB_ENDPOINT=127.0.0.1 DB_USER=postgres DB_PASSWORD=postgres DB_NAME=my_db TARGET_IMAGE=myapp-data DESTINATION_REPOSITORY=gcr.io/my_project
```
