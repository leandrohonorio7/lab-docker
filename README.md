# Liquid - Demo App

## Arquitetura da aplicação

A aplicação foi construída usando o [Liquid Framework](https://github.com/Avanade/Liquid-Application-Framework), e tem como objetivo demonstrar suas capacidades e vantagens.


### Stack

- WSL2
- .Net Core 3.1.
- Asp .Net Core 3.1
- EntityFrameworkCore
- Oracle.EntityFrameworkCore
- Docker
- Para a base de dados foi usado uma imagem do Oracle.

## Configurando o ambiente

Está aplicação será excuta usando o WSL2 do Windows, rodando containers Docker diretamente no Linux, usando Visual Studio Code para desenvolvimento e debugging.

### Instalando o WSL2

Para instalar o WSL2 basta executar este comando no prompt do Windows:

```cmd
wsl.exe --install -d Ubuntu-20.04
```

Recomenda-se o uso da distribuição Ubuntu 20.04, após a instalação é necessario adicionar o usuário e senha.

#### Garanta que a versão instalada é a WSL2

Para isso execute o comando:

```cmd
wsl -l -v
``` 
Resultado:

```cmd
  NAME            STATE           VERSION
* Ubuntu-20.04    Running         2
```

Caso a versão seja diferente, atualize para a versão 2:

```cmd
wsl --set-version Ubuntu-20.04 2
```

### Instalando o Windows Terminal

Para interagir com o WSL2 devemos instalar o Windows Terminal, que está integrado ao WSL2.

[Windows Terminal](https://docs.microsoft.com/en-us/windows/terminal/install)

### Instalando o Docker e Docker Compose

Para trabalharmos com os containers é necessário instalar o Docker no WSL2.

Executar usando o Windows Terminal, é possível conectar diretamente no Linux para executar os comandos, para tal basta abrir o terminal e criar uma nova janela selecionando o Ubuntu.

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
sudo apt install build-essential
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

Ao reiniciar o WSL, o serviço do Docker não é inicializado automaticamente, para inicia-lo basta executar:

```bash
sudo service docker start
```

#### Executando comandos do Docker como usuário não 'root'

Para executar os comandos do docker sem 'sudo', basta executar os seguintes comandos:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

### Instalando o Git

Para instalar o Git basta executar:

```bash
sudo apt-get install git
```

Para evitar que seja necessário adicionar o usuário e senha do Git a cada operação, rode o seguinte comando:

```bash
git config --global credential.helper store
``` 

### Instalando o .Net Core SDK

Para podermos, desenvolver e executar a aplicação é necessário instalarmos o dotnet-sdk-3.1.

```bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```

### Acessando a aplicação no Windows

Como a aplicação é executada no WSL2 é necessário localizar o IP que está sendo exposto para que assim possamos fazer chamadas ao serviço.

Pata tal execute:

```bash
ip addr | grep eth0
``` 

Resultado:

![net tools](docs\images\net-tools.jpg "net tools")

O IP a ser usado deve ser o presente em 'eth0'.

### Limitando a quantidade de memória usada pelo WSL

No prompt de comando do Windows execute:

```cmd
notepad %USERPROFILE%\.wslconfig
```

Adicione o seguinte conteúdo e salve o arquivo:

```
[wsl2]
memory=6GB   # Limits VM memory in WSL2 up to 3GB
processors=4 # Makes the WSL2 VM use two virtual processors
```

Para que a alteração tenha efeito é necessário reiniciar o WSL2 (executar no prompt de comando do Windows):

```bash
wsl --shutdown

wsl --distribution Ubuntu-20.04
```

### VPN GlobalProtect

Para que o WSL2 funcione corretamente tendo acesso aos serviços que necessitam da VPN, os seguintes passos devem ser seguidos:

1- Alterar os arquivo wsl.conf:

```bash
sudo bash -c 'echo "[network]" > /etc/wsl.conf'
sudo bash -c 'echo "generateResolvConf = false" >> /etc/wsl.conf'
sudo bash -c 'echo "generateHosts = false" >> /etc/wsl.conf'
```

2 - Alterar o arquivo resolv.conf

```bash
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "nameserver 10.104.211.200" >> /etc/resolv.conf'
sudo bash -c 'echo "nameserver 10.104.91.200" >> /etc/resolv.conf'
sudo chattr +i /etc/resolv.conf
```

Inserimos no arquivo os IPs para resolução de DNS da VPN da UHG e o do Google para uso fora da VPN.

Caso não seja possível remover o arquivo, verifique se o mesmo não está marcado como imutável:  

```bash
lsattr /etc/resolv.conf
```

Resultado:

```bash
[root@cube ~]# lsattr /etc/resolv.conf
----i--------------- /etc/resolv.conf
```

Remova a flag:

```bash
sudo chattr -i /etc/resolv.conf
```

Agora basta executar os comandos para alterar o arquivo novamente.

3 - Alterar o arquivos host

Em alguns casos a resolução de DNS pode falhar, para corrigir este problema vamos adicionar manualmente as entradas.

Abra o arquivo:

```bash
sudo nano /etc/hosts
```

Em seguida adicione as novas entradas: 

```
10.104.28.209   github.amil.com.br
10.104.53.41    tfs.amil.com.br
10.105.48.34    container-registry-ebt.uhgbrasil.com.br
```

Agora basta salvar o arquivo.

> **Importante:** para cada novo serviço que use DNS, como por exemplo o endereço de uma base de dados, uma nova entrada deve ser adicionada contendo o IP e do DNS correspondente.

4 - Em seguida executar este script, no powershell como administrador, **este script deve ser executado após toda nova conexão com a VPN:**

```powershell
Get-NetIPInterface -InterfaceAlias "vEthernet (WSL)" | Set-NetIPInterface -InterfaceMetric 1
Get-NetAdapter | Where-Object {$_.InterfaceDescription -Match "PANGP Virtual Ethernet Adapter"} | Set-NetIPInterface -InterfaceMetric 6000
```

5 - Por fim executar o VPN Fix disponibilizado pela UHG, **este script deve ser executado após toda nova conexão com a VPN:**

## Configurando o Visual Studio Code

### Instalação 

Para abrir o Visual Studio Code basta executar na pasta que será aberta:

```bash
code .
```

Ao executar este comnando ele será instalado, caso ainda não esteja.

### Configurando as extensões

#### Extensões Locais

Estas extensões devem ser instaladas no Visual Studio Code localmente, ou seja, no Windows:

- Remote - Containers
- Remote - WSL
- Remote Development
- Visual Studio Keymap (Opcional)
- vscode-icons (Opcional)

Estas extensões devem ser instaladas no WSL:

- C#
- C# Extensions
- Docker
- GitLens (Opcional)
- .NET Core Test Explorer

Após abrir o Visual Studio Code no WSL2 é possível instalar as extensões.

Caso a extensão do Docker não funcione basta seguir este guia [Troubleshooting](https://github.com/microsoft/vscode-docker/wiki/Troubleshooting)

## Iniciando o desenvolvimento

### Criando o projeto

É recomendado que o fonte dos projetos esteja no file system do WSL, para isso navegue até a pasta raiz com o comando:

```bash
cd ~
```

Para organização dos fontes crie uma pasta chamada 'Workspace' na raiz:

```bash
mkdir Workspace
```

Em seguida crie a pasta do projeto:

```bash
cd Workspace
mkdir liquid-sample
```

Caso tenha problemas com permissões basta executar: 

```bash
sudo chown -R "username" .
```

Agora iremos criar a solução e os projetos:

```bash
cd liquid-sample
mkdir src

cd src

dotnet new sln -n Liquid.Arch.Demo

dotnet new classlib -o Liquid.Arch.Demo.Domain --framework "netcoreapp3.1"
dotnet new classlib -o Liquid.Arch.Demo.Application --framework "netcoreapp3.1"
dotnet new classlib -o Liquid.Arch.Demo.Data --framework "netcoreapp3.1"
dotnet new webapi -o Liquid.Arch.Demo.Api --framework "netcoreapp3.1"
dotnet new xunit -o Liquid.Arch.Demo.Tests --framework "netcoreapp3.1"
dotnet new worker -o Liquid.Arch.Demo.Worker --framework "netcoreapp3.1"

dotnet sln Liquid.Arch.Demo.sln add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj
dotnet sln Liquid.Arch.Demo.sln add Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj
dotnet sln Liquid.Arch.Demo.sln add Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj
dotnet sln Liquid.Arch.Demo.sln add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj
dotnet sln Liquid.Arch.Demo.sln add Liquid.Arch.Demo.Tests/Liquid.Arch.Demo.Tests.csproj
dotnet sln Liquid.Arch.Demo.sln add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj
```

Em seguida das referências entre os projetos:

```bash
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj reference Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj reference Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj
dotnet add Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj reference Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj
dotnet add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj reference Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj
dotnet add Liquid.Arch.Demo.Tests/Liquid.Arch.Demo.Tests.csproj reference Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj
dotnet add Liquid.Arch.Demo.Tests/Liquid.Arch.Demo.Tests.csproj reference Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj
dotnet add Liquid.Arch.Demo.Tests/Liquid.Arch.Demo.Tests.csproj reference Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj
dotnet add Liquid.Arch.Demo.Tests/Liquid.Arch.Demo.Tests.csproj reference Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj

dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj reference Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj reference Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj
```

Para fechar vamos adicionar os pacotes necessários:

```bash
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Liquid.WebApi.Http
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Liquid.Repository.EntityFramework
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Oracle.EntityFrameworkCore --version 5.21.3
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Liquid.Core.Telemetry.ElasticApm --version 2.0.0-preview-20211216-01

dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Serilog --version 2.10.0
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Serilog.AspNetCore --version 4.1.0
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Serilog.Exceptions --version 8.0.0
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Serilog.Sinks.Elasticsearch --version 8.4.1

dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Serilog.Enrichers.Environment --version 2.2.0
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Serilog.Sinks.Console --version 4.0.1
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Liquid.Repository.Mongo --version 2.1.0
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Elastic.Apm.EntityFrameworkCore --version 1.12.1
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Elastic.Apm.SerilogEnricher --version 1.5.3
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Elastic.CommonSchema.Serilog --version 1.5.3
dotnet add Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj package Liquid.Messaging.RabbitMq --version 2.0.0-preview-20210806-06

dotnet add Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj package Liquid.Domain
dotnet add Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj package Liquid.Repository.EntityFramework
dotnet add Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj package Liquid.Repository.Mongo --version 2.1.0

dotnet add Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj package Liquid.Domain
dotnet add Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj package Liquid.Repository.EntityFramework
dotnet add Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj package Liquid.Messaging.RabbitMq --version 2.0.0-preview-20210806-06

dotnet add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj package Liquid.Repository.EntityFramework
dotnet add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj package Oracle.EntityFrameworkCore --version 5.21.3
dotnet add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj package Microsoft.EntityFrameworkCore.Tools --version 5.0.11
dotnet add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj package Microsoft.EntityFrameworkCore.Design --version 5.0.11
dotnet add Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj package Microsoft.EntityFrameworkCore --version 5.0.11

dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog --version 2.10.0
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog.Exceptions --version 8.0.0
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog.Sinks.Elasticsearch --version 8.4.1
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog.Enrichers.Environment --version 2.2.0
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog.Sinks.Console --version 4.0.1
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Liquid.Repository.Mongo --version 2.1.0
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Elastic.CommonSchema.Serilog --version 1.5.3
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Microsoft.Extensions.Hosting --version 3.1.17
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Liquid.Core.Telemetry.ElasticApm --version 2.0.0-preview-20211216-01
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Liquid.Messaging.RabbitMq --version 2.0.0-preview-20210806-06
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog.Extensions.Hosting --version 4.2.0
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Serilog.Settings.Configuration --version 3.3.0
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Elastic.Apm.Extensions.Hosting --version 1.12.1
dotnet add Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj package Elastic.Apm.SerilogEnricher --version 1.5.3

dotnet add Liquid.Arch.Demo.Tests/Liquid.Arch.Demo.Tests.csproj package Moq --version 4.16.1

```


Caso ocorra este erro durante a criação dos projetos:

```
 Error: Invalid parameter(s):
   -f netcoreapp3.1
      'netcoreapp3.1' is not a valid value for -f (Framework) 
```

Basta executa este comando:

```bash
dotnet --info
```

Que irá retornar a seguinte lista:

```
This will show the list of frameworks installed 
.NET Core SDKs installed:
  2.1.202 [C:\\Program Files\\dotnet\\sdk]
  2.1.402 [C:\\Program Files\\dotnet\\sdk]
  2.1.503 [C:\\Program Files\\dotnet\\sdk]
  2.2.203 [C:\\Program Files\\dotnet\\sdk]
```

Caso existam mais de um SDK instalado é necessario setar a versão que precisamos:

```bash
dotnet new globaljson --sdk-version 3.1.416 
```

### Adicionando suporte ao debugging e ao Docker

Para adicionar o suporte ao debug é necessário criar uma pasta chamada '.vscode' dentro da pasta 'src'

```bash
mkdir .vscode
```

Crie um arquivo chamado 'launch.json' e adicione o seguinte conteúdo:

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Docker .NET Core Attach (Preview)",
            "type": "docker",
            "request": "attach",
            "platform": "netCore",
            "requireExactSource": false,
            "sourceFileMap": {
                "/src": "${workspaceFolder}"
            }            
        }

    ]
}
```

Agora vamos adicionar o arquivo do docker, para isso crie um arquivo chamado 'Dockerfile', dentro da pasta do projeto que contém a API e adicione o seguinte conteúdo:

```yml
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj", "Liquid.Arch.Demo.Api/"]
COPY ["Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj", "Liquid.Arch.Demo.Application/"]
COPY ["Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj", "Liquid.Arch.Demo.Domain/"]
COPY ["Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj", "Liquid.Arch.Demo.Data/"]

RUN dotnet restore "Liquid.Arch.Demo.Api/Liquid.Arch.Demo.Api.csproj"
COPY . .
WORKDIR "/src/Liquid.Arch.Demo.Api"
RUN dotnet build "Liquid.Arch.Demo.Api.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Liquid.Arch.Demo.Api.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Liquid.Arch.Demo.Api.dll"]
```

E o dockerfile para a Worker:
```yml
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj", "Liquid.Arch.Demo.Worker/"]
COPY ["Liquid.Arch.Demo.Application/Liquid.Arch.Demo.Application.csproj", "Liquid.Arch.Demo.Application/"]
COPY ["Liquid.Arch.Demo.Domain/Liquid.Arch.Demo.Domain.csproj", "Liquid.Arch.Demo.Domain/"]
COPY ["Liquid.Arch.Demo.Data/Liquid.Arch.Demo.Data.csproj", "Liquid.Arch.Demo.Data/"]

RUN dotnet restore "Liquid.Arch.Demo.Worker/Liquid.Arch.Demo.Worker.csproj"
COPY . .
WORKDIR "/src/Liquid.Arch.Demo.Worker"
RUN dotnet build "Liquid.Arch.Demo.Worker.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Liquid.Arch.Demo.Worker.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Liquid.Arch.Demo.Worker.dll"]
```

Em seguida adicionaremos o arquivo 'docker-compose.yml', responsável por facilitar e organizar os vários containers que uma aplicação pode usar, este arquivo deve ser adicionado dentro da pasta 'src'.


```yaml
version: '3.6'

services:

    liquid-demo-api:
        image: liquid-demo-api:latest
        container_name: liquid-demo-api
        build:
            context: .
            dockerfile: ./Liquid.Arch.Demo.Api/Dockerfile
        environment:
            - ASPNETCORE_ENVIRONMENT=Development    
            - ASPNETCORE_URLS=http://+:80   
            - ConnectionStrings__OracleConnection=Data Source=database:1521/ORCLCDB.LOCALDOMAIN;User Id=demo_admin;Password=demopass;Load Balancing=True;Max Pool Size=50;Connection Timeout=60;
            - ElasticApm__ServerUrl=http://elasticapm:8200
            - ElasticsearchUri=http://elasticsearch:9200/
            - IndexFormat=application-index-{0:yyyy.MM.dd}
            - Liquid__databases__mongo__dbSettings__0__ConnectionString=mongodb://mongo-database:27017
            - Liquid__Messaging__RabbitMq__Producer__QueueSettings__ConnectionString=amqp://guest:guest@rabbitmq:5672
        ports:
            - "80:80"
            - "443:443"
        networks:
            - demo-network
        depends_on:
            - database
            - rabbitmq

    liquid-demo-worker:
        image: liquid-demo-worker:latest
        container_name: liquid-demo-worker
        restart: always
        build:
            context: .
            dockerfile: ./Liquid.Arch.Demo.Worker/Dockerfile
        environment:
            - ASPNETCORE_ENVIRONMENT=Development    
            - ElasticApm__ServerUrl=http://elasticapm:8200
            - ElasticsearchUri=http://elasticsearch:9200/
            - IndexFormat=application-index-{0:yyyy.MM.dd}
            - Liquid__databases__mongo__dbSettings__0__ConnectionString=mongodb://mongo-database:27017
            - Liquid__messaging__rabbitMq__consumer__QueueSettings__connectionString=amqp://guest:guest@rabbitmq:5672
        networks:
            - demo-network
        depends_on:
            - mongo
            - rabbitmq

    database:
        image: store/oracle/database-enterprise:12.2.0.1
        container_name: database
        ports:
            - 1521:1521
            - 5500:5500
        volumes:
            - data:/ORCL
        networks:
            - demo-network

    mongo:
        image: mongo:5.0.6
        container_name: mongo-database        
        restart: always
        volumes:
          - mongodata:/mongo
        ports:
          - 27017:27017
        networks:
          - demo-network

    rabbitmq:
        image: rabbitmq:3-management
        hostname: "demo-rabbit"
        container_name: rabbit-local-server
        ports:
            - "5672:5672"
            - "15672:15672"
        networks:
            - demo-network
        labels:
            name: "rabbit-local-server"     

networks:
  demo-network: 
    external: 
      name: default-applications

volumes:
  data:
  mongodata:

```

No arquivo foram definidos os serviços necessários para esta aplicação, cada aplicação terá configurações diferentes.

### Observabilidade

Para facilitar a localização e tratamento de erros, vamos usar o Elastic APM, em conjunto com o Liquid para termos logs e metricas 
da aplicação, para isso usaremos um docker compose que contém todas as dependências necessárias.

docker-compose-observability.yml
```yml
version: '3.6'

services:

    elasticsearch:
        container_name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.15.1
        ports:
          - 9200:9200
        networks:
          - demo-network
        volumes:
          - elasticdata:/usr/share/elasticsearch/data
        environment:
          - discovery.type=single-node
          - http.port=9200
          - http.cors.enabled=true
          - http.cors.allow-origin=http://grafana:3000,http://127.0.0.1:3000
          - http.cors.allow-headers=X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
          - http.cors.allow-credentials=true
          - bootstrap.memory_lock=true      
          - xpack.monitoring.enabled=true
          - xpack.watcher.enabled=false
          - 'ES_JAVA_OPTS=-Xms512m -Xmx512m'
        healthcheck:
          interval: 20s
          retries: 10
          test: curl -s http://localhost:9200/_cluster/health | grep -vq '"status":"red"'

    kibana:
        container_name: kibana
        image: docker.elastic.co/kibana/kibana:7.15.1
        ports:
          - 5601:5601
        networks:
          - demo-network
        environment:
          ELASTICSEARCH_URL: http://elasticsearch:9200
        depends_on:
          - elasticsearch
        healthcheck:
          interval: 10s
          retries: 20
          test: curl --write-out 'HTTP %{http_code}' --fail --silent --output /dev/null http://localhost:5601/api/status

    grafana:
        container_name: grafana
        image: grafana/grafana:8.3.5
        ports:
          - 3000:3000
        networks:
          - demo-network
        volumes:
          - grafanadata:/var/lib/grafana

    elasticsearch-apm:
        container_name: elasticapm
        image: docker.elastic.co/apm/apm-server:7.15.1
        depends_on:
          - elasticsearch
          - kibana
        cap_add: ["CHOWN", "DAC_OVERRIDE", "SETGID", "SETUID"]
        cap_drop: ["ALL"]
        ports:
          - 8200:8200
        networks:
          - demo-network
        command: >
          apm-server -e
            -E apm-server.rum.enabled=true
            -E setup.kibana.host=kibana:5601
            -E setup.template.settings.index.number_of_replicas=0
            -E apm-server.kibana.enabled=true
            -E apm-server.kibana.host=kibana:5601
            -E output.elasticsearch.hosts=["elasticsearch:9200"]
        healthcheck:
          interval: 10s
          retries: 12
          test: curl --write-out 'HTTP %{http_code}' --fail --silent --output /dev/null http://localhost:8200/

networks:
  demo-network: 
    external: 
      name: default-applications 

volumes:
  elasticdata:
  grafanadata:
```

Agora basta executar os seguintes comandos:

```bash
docker network create default-applications
```
Este comando cria um rede comum para ser usada pelas aplicações, assim é possível que várias aplicações locais usem a mesma infra.

Com a rede criada basta executar o compose.
```bash
docker-compose -f docker-compose-observability.yml up -d
```

### Adicionando as configurações necessarias ao Liquid Framework

#### Configurando o arquivo 'Startup.cs'

Iremos adicionar as configurações do Liquid Framework, isto dará suporte para que possamos criar nossa API
usando o Liquid.

Configura o suporte a múltiplos environments:
```csharp
public Startup(IConfiguration configuration, IHostEnvironment hostEnvironment)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(hostEnvironment.ContentRootPath)
        .AddJsonFile("appsettings.json", true, true)
        .AddJsonFile($"appsettings.{hostEnvironment.EnvironmentName}.json", true, true)
        .AddEnvironmentVariables()
    Configuration = builder.Build();
}
```

Configura os Handler e os repositórios:

```csharp
  public void ConfigureServices(IServiceCollection services)
  {

      services.AddHttpContextAccessor();
      services.AddLiquidConfiguration();

      services.AddLiquidHttp(typeof(AddDataHandler).Assembly);

      Action<DbContextOptionsBuilder> dbContextOptionsBuilder = (options) =>
      {
          var connectionString = Configuration["ConnectionStrings:OracleConnection"];

          options.UseOracle(connectionString,
              b => b.MigrationsAssembly(typeof(DemoContext).Assembly.FullName));
      };

      services.AddDbContext<DemoContext>(dbContextOptionsBuilder);

      services.AddLiquidEntityFramework<DemoContext, ParentData, int>(dbContextOptionsBuilder);
      services.AddLiquidEntityFramework<DemoContext, ChildItem, int>(dbContextOptionsBuilder);

      services.AddLiquidRabbitMqProducer<AddDataMessage>("Liquid:Messaging:rabbitMq:producer", true);

      services.AddLiquidMongoWithTelemetry<DataItemEntity, int>(options =>
      {
          options.DatabaseName = "demo-database";
          options.CollectionName = "DataItemCollection";
          options.ShardKey = "Id";
      });

      services.AddLiquidElasticApmTelemetry(Configuration);
      services.AddControllers();
      services.AddHttpClient();
  }
```

Aqui estamos configurando todas as dependências, ou seja, os cartuchos do Liquid, que estamos usando na aplicação, aqui também configuramos a telemetria com o Elastic APM usando o cartucho do Liquid. 

Adiciona as configurações do Liquid:

```csharp
  public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
  {

      app.UseLiquidConfigure();
      app.UseLiquidElasticApm(Configuration);
      app.UseRouting();
      app.UseAuthorization();
      app.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
      });
  }
```
Por fim é necessário adicionar as chaves de configuração no arquivo 'appsegttings.json':

```json
"Liquid": {
    "databases": {
      "mongo": {
        "DbSettings": [
          {
            "databaseName": "demo-database",
            "connectionString": "mongodb://mongo-database:27017"
          }
        ]
      }
    },
    "swagger": {
      "name": "v1",
      "host": "",
      "schemes": [
        "http",
        "https"
      ],
      "title": "Liquid.Arch.Demo",
      "version": "v1",
      "description": "Liquid UHG Demo",
      "SwaggerEndpoint": {
        "url": "/swagger/v1/swagger.json",
        "name": "Liquid.Arch.Demo.Api"
      }
    },
    "culture": {
      "defaultCulture": "pt-BR"
    },
    "httpScopedContext": {
      "keys": [
        {
          "keyName": "Connection",
          "required": true
        },
        {
          "keyName": "Accept",
          "required": true
        }
      ]
    },
    "HttpScopedLogging": {
      "keys": [
        {
          "keyName": "Connection",
          "required": true
        },
        {
          "keyName": "Accept",
          "required": true
        }
      ]
    },
    "messaging": {
      "rabbitMq": {
        "producer": {
          "compressMessage": false,
          "exchange": "demo-exchange",
          "queueSettings": {
            "connectionString": ""
          },
          "advancedSettings": {
            "durable": true,
            "autoDelete": false,
            "autoAck": true,
            "expiration": "3600000"
          }
        }
      }
    }
  },
  "MongoDbSettings": {
    "DefaultDatabaseSettings": {
      "DatabaseName": "demo-database"
    },
    "Entities": {
      "DataItemEntity": {
        "CollectionName": "DataItemCollection",
        "ShardKey": "Id"
      }
    }
  }
```

Para mais informações sobre a configuração e sobre Liquid, consulte o repositório [oficial](https://github.com/Avanade/Liquid.Repository).

## Executando a aplicação

Para executar a aplicação basta executarmos o docker compose, na posta src:

```bash
docker-compose up --build
```

O parâmetro'--build' força a recriação da imagem da API.

Para verificar se a aplicação está executando, rode o comando:

```bash
docker container ls
```

Agora já é possível fazer uma chamada a API, usando o IP recuperado pelo comando 'ifconfig'.

## Debugando a aplicação

Com a aplicação em execução, basta adicionar o breakpoint no ponto desejado e iniciar o debugger.

![Debugger](docs\images\debugger.jpg "Debugger")

Ao executar o código o breakpoint será acionado:

![Breakpoint](docs\images\breakpoint.jpg "Breakpoint")

Ao executar a aplicação pela primeira vez é necessário 'atachar' o debugger ao container:

![Attach debugger](docs\images\attach-debugger.jpg "Attach debugger")

## Verficando os logs

Após a execução do docker compose de observabilidade é possível abrir a interface da Kibana, e monitorar todas as metricas, logs e erros da aplicação.
Para isso acesse o dashboard da Kibana: http://[Ip do WSL]:5601/.

Agora é necessário, configurar a Kibana para ler os logs do indice da aplicação.

### Configurando o índice

Acesse:

http://[Ip do WSL]:5601/app/management/kibana/indexPatterns

![Index Creation](docs\images\kibana-index-creation.jpg "Index Creation")

Clique em 'Create Index Patterns', e adicione o índice que esta configurado no docker-compose, adicionando '*' para pegar as variações de data, 
por exemplo, 'application-index-*', e adicione o campo de data para o índice.

![Create Index Pattern](docs\images\create-index-pattern.jpg "Create Index Pattern")

Com o índice criado, vamos configurar o Kibana para ler os dados dele, para isso acesse:

http://[Ip do WSL]:5601/app/logs/stream

Clique em 'Settings' e selecione em 'Log index pattern' o índice que foi criado, em seguida salve as alterações.

![Log Settings](docs\images\log-settings.jpg "Log Settings")

Pronto, após estas configurações os logs e metricas estarão disponíveis em:

http://[Ip do WSL]:5601/app/observability/overview

## Usando a imagem do Oracle

Nesta aplicação usamos uma imagem oficial para demonstrar o uso do Liquid Framework com o Oracle.

Para ter acesso a imagem é necessário fazer um cadastro no site do Docker hub, para tal é necessario se autenticar no 
site, em seguida acessar [Oracle Database Enterprise Edition](https://hub.docker.com/_/oracle-database-enterprise-edition) e clicar em 'Proceed to Checkout':

![Oracle Checkout](docs\images\oracle-checkout.jpg "Oracle Checkout")

Então basta preencher os dados solicitados, e se autenticar no docker com o comando (executar no WSL):

```bash
docker login
```
Após isto será possível fazer o 'pull' da imagem.

Usando o IP do WSL, o mesmo usado para fazer chamadas é possível conectar na base de dados usando o Oracle SQL Developer:

![Oracle SQL Developer](docs\images\oracle-sql-developer.jpg "Oracle SQL Developer")

Onde Hostname é o IP do WSL, e a senha é padrão para a imagem: 'Oradoc_db1'.

### Configurando o Oracle para uso

Para que possamos usar o banco conectando a aplicação é necessário criar um usuário, para tal basta executa os comandos na base:

```sql
alter session set "_ORACLE_SCRIPT"=true;
CREATE USER demo_admin IDENTIFIED BY demopass;
GRANT ALL PRIVILEGES TO demo_admin;
``` 

Agora basta adicionar este usuário na string de conexão:

```
Data Source=database:1521/ORCLCDB.LOCALDOMAIN;User Id=demo_admin;Password=demopass;Load Balancing=True;Max Pool Size=50;Connection Timeout=60
```




detalhamento das camadas e como criar um endpoint basico


https://support.tools/post/fix-stuck-resolv-conf/


certificate could not be found or is out of date.
liquid-demo-api    | To generate a developer certificate run 'dotnet dev-certs https'. To trust the certificate (Windows and macOS only) run 'dotnet dev-certs https --trust'.

sudo apt install openssl