# PlaceRental — API Gateway (Ocelot)

Este repositório contém o **API Gateway** da solução *PlaceRental*, construído em **.NET 8** e baseado no **Ocelot**. Ele faz parte de uma **arquitetura distribuída** (estilo microserviços), onde múltiplos serviços independentes expõem APIs próprias e o gateway atua como **ponto único de entrada** para o cliente.

## Contexto: arquitetura distribuída

Em uma arquitetura distribuída, funcionalidades do domínio são separadas em serviços autônomos (ex.: *Users*, *Places*), com deploy, escala e evolução independentes. Para reduzir acoplamento no cliente e centralizar preocupações transversais, usamos um **API Gateway**.

## Papel deste projeto (API Gateway)

Este gateway:

- expõe endpoints “upstream” (para o cliente) e roteia para endpoints “downstream” (serviços internos)
- centraliza o roteamento em um único lugar via `ocelot.json`
- pode ser estendido para responsabilidades típicas de gateway (ex.: autenticação, autorização, rate limit, cache, agregações, circuit breaker) conforme necessidade do sistema

## Rotas configuradas

As rotas estão em `PlaceRental.APIGateway/ocelot.json`.

### Users service

- **Upstream**: `/users/{everything}`
- **Downstream**: `/api/users/{everything}`
- **Destino**: `https://localhost:7293`

### Places service

- **Upstream**: `/places/{everything}`
- **Downstream**: `/api/places/{everything}`
- **Destino**: `https://localhost:7224`

## Pré-requisitos

- **.NET SDK 8.x**
- Os serviços downstream (*Users* e *Places*) rodando localmente nas portas esperadas:
  - Users: `https://localhost:7293`
  - Places: `https://localhost:7224`

## Como executar

Na raiz do repositório:

```bash
dotnet restore
dotnet run --project "PlaceRental.APIGateway/PlaceRental.APIGateway.csproj" --launch-profile https
```

O gateway ficará disponível (conforme `launchSettings.json`) em:

- `https://localhost:7290`
- `http://localhost:5006`

## Como testar o roteamento

Exemplos (substitua `{everything}` pelo caminho real do serviço):

- Gateway → Users:
  - `GET https://localhost:7290/users/{everything}`
  - será roteado para `GET https://localhost:7293/api/users/{everything}`

- Gateway → Places:
  - `GET https://localhost:7290/places/{everything}`
  - será roteado para `GET https://localhost:7224/api/places/{everything}`

## Observações importantes

- **Swagger**: o gateway habilita Swagger em ambiente `Development`. Ele documenta os controllers do próprio gateway (se existirem). Rotas do Ocelot podem não aparecer automaticamente como endpoints “convencionais” de controller.
- **HTTPS local**: se houver erro de certificado em ambiente local, confirme que o dev-certificate está confiável:

```bash
dotnet dev-certs https --trust
```

## Estrutura do projeto (alto nível)

- `PlaceRental.APIGateway/Program.cs`: bootstrap do ASP.NET Core + configuração do Ocelot
- `PlaceRental.APIGateway/ocelot.json`: tabela de roteamento do gateway
- `PlaceRental.APIGateway/appsettings*.json`: logging e configurações padrão


