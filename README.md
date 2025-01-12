# PoC de Uso do Docker em aplicações C#

> Baseado no [artigo da M$](https://learn.microsoft.com/pt-br/dotnet/core/docker/build-container?tabs=windows&pivots=dotnet-8-0) e conteúdo ministrado nas aulas de Engineering Software Development na FIAP pelo professor André Pontes Sampaio.

Necessário:
- Ter uma conta no [Docker Hub](https://hub.docker.com).
- Ter Docker instalado ([Docker Desktop](https://www.docker.com/products/docker-desktop/) ou alternativas).

## Criar Imagem

### Exemplo de Dockerfile

Criar Dockerfile na raiz do projeto

```Python Console
touch Dockerfile
```

Conteúdo do arquivo

```Dockerfile
# Alterar as imagens (FROM) de acordo com a versão do SO e dotnet

FROM mcr.microsoft.com/dotnet/sdk:8.0@sha256:35792ea4ad1db051981f62b313f1be3b46b1f45cadbaa3c288cd0d3056eefb83 AS build
WORKDIR /App

# Copy everything
COPY . ./
# Restore as distinct layers
RUN dotnet restore
# Build and publish a release
RUN dotnet publish -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0@sha256:6c4df091e4e531bb93bdbfe7e7f0998e7ced344f54426b7e874116a3dc3233ff
WORKDIR /App
COPY --from=build /App/out .
ENTRYPOINT ["dotnet", "NOMEPROJETOAQUI.dll"]
```
 

### Criar uma imagem do contêiner


```Python Console
docker build -t NOMEIMAGEMAQUI . 
```

OBS: O ponto no final é o PATH do diretório que contém o Dockerfile 

### Executar o contêiner local

Comando para executar o contêiner: 

```Python Console
docker run --name NOMESERVICOAQUI -it -p 5050:5000 -d NOMEIMAGEMAQUI
```

- `-it` permite interagir com container
- `-p` define a porta
- `-d` roda container no background

## Publicar imagem

### Fazer o login no docker (hub) 

```Python Console
docker logout 

docker login -u SEUUSUARIOAQUI 
```

### Salvar seu container no Hub.Docker.com

```Python Console
docker tag NOMEIMAGEMAQUI SEUUSUARIOAQUI/REPOIMAGEMAQUI:versãoxpto

docker push SEUUSUARIOAQUI/REPOIMAGEMAQUI:versãoxpto
```

O ideal é fazer `git tag` + `docker tag`.

No futuro, você pode rodar a solução em qualquer ambiente através do comando:

```Python Console
docker run --name NOMESERVICOAQUI -p 5005:5000 -d SEUUSUARIOAQUI/REPOIMAGEMAQUI 
```

---

Placeholders usados:
 - REPOIMAGEMAQUI
 - NOMEIMAGEMAQUI
 - SEUUSUARIOAQUI
 - NOMESERVICOAQUI
 - NOMEPROJETOAQUI
