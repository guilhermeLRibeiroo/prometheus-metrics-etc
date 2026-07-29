# Forum API — Observabilidade com Prometheus e Grafana

API REST de fórum construída em **Java/Spring Boot**, com o foco principal do projeto sendo a instrumentação e observabilidade da aplicação: métricas expostas via Actuator/Micrometer, coletadas pelo Prometheus e visualizadas em dashboards no Grafana — tudo containerizado.

## Arquitetura

```
                        ┌──────────────┐
                        │    Client    │
                        └──────┬───────┘
                               │
                        ┌──────▼────────┐
                        │  Nginx (proxy)│  :80
                        └──────┬────────┘
                               │
                        ┌──────▼────────┐         ┌──────────┐
                        │  Forum API    │───────▶ │  Redis  │  (cache)
                        │ (Spring Boot) │         └──────────┘
                        └──────┬────────┘
                               │
                        ┌──────▼───────┐
                        │    MySQL     │
                        └──────────────┘

     ┌──────────────┐        ┌──────────────┐
     │  Prometheus  │◄───────│  Actuator/   │
     │   :9090      │ scrape │  Micrometer  │ (dentro da Forum API)
     └──────┬───────┘        └──────────────┘
            │
     ┌──────▼───────┐
     │   Grafana    │  :3000
     └──────────────┘
```

## Stack Técnica

- **Java / Spring Boot 2.3**
- **Spring Security + JWT** — autenticação própria (`jjwt`)
- **Spring Data JPA** — persistência
- **MySQL** — banco de dados principal
- **Redis** — camada de cache
- **Spring Boot Actuator + Micrometer** — exposição de métricas da aplicação no formato Prometheus
- **Prometheus** — coleta e armazenamento de métricas (scrape da API)
- **Grafana** — dashboards e visualização das métricas coletadas
- **Nginx** — proxy reverso na frente da API
- **Docker Compose** — orquestração de todos os serviços (API, MySQL, Redis, Nginx, Prometheus, Grafana, client)
- **Swagger (Springfox)** — documentação interativa da API
- **H2** — banco em memória para testes

## Serviços (docker-compose)

| Serviço | Função | Porta exposta |
|---|---|---|
| `client-forum-api` | Cliente/frontend consumindo a API | — |
| `proxy-forum-api` | Nginx como proxy reverso, ponto de entrada | 80 |
| `app-forum-api` | API Spring Boot (fórum) | interna |
| `mysql-forum-api` | Banco de dados principal | interna |
| `redis-forum-api` | Cache | interna |
| `prometheus-forum-api` | Coleta de métricas (scrape do Actuator) | 9090 |
| `grafana-forum-api` | Dashboards das métricas coletadas | 3000 |

A API expõe um health check via Actuator (`/actuator/health`), usado pelo próprio docker-compose para aguardar a aplicação subir antes de liberar dependências.

## Como rodar o projeto

### Pré-requisitos
- Docker e Docker Compose

### Passo a passo

```bash
docker-compose up -d
```

Isso sobe, em ordem de dependência: Redis → MySQL → API (Spring Boot) → Nginx (proxy) → Prometheus → Grafana → Client.

Depois de tudo no ar:
- **API (via proxy):** `http://localhost` (porta 80)
- **Prometheus:** `http://localhost:9090` — visualize as métricas expostas pela aplicação (endpoint `/actuator/prometheus`)
- **Grafana:** `http://localhost:3000` — monte dashboards a partir do Prometheus como data source

## O que este projeto demonstra

- Instrumentação de uma aplicação Spring Boot para expor métricas customizadas e de infraestrutura (JVM, HTTP, pool de conexões) via Actuator/Micrometer.
- Pipeline de observabilidade ponta a ponta: aplicação → coleta (Prometheus) → visualização (Grafana).
- Ambiente multi-serviço totalmente containerizado, com healthcheck controlando a ordem de subida.

## Possíveis evoluções

- Adicionar alertas no Prometheus (Alertmanager) com base em métricas de erro/latência.
- Dashboards pré-provisionados no Grafana (hoje presume-se configuração manual).
- Métricas de negócio customizadas além das técnicas (ex: número de posts criados, usuários ativos).

---

Projeto de estudo focado em observabilidade de aplicações backend, usando uma API de fórum como aplicação de exemplo.
