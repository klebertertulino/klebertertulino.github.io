---
layout: post
title: Quatro IAs, o mesmo projeto
date: YYYY-MM-DD HH:MM
author: Kleber Tertulino
tags: [ia, desenvolvimento, devops, chatops]
---

Em junho de 2026 eu tinha um plano escrito para uma versão enxuta do Context, um assistente de incidentes que funciona dentro do Slack. Em vez de implementar o plano uma vez, entreguei o mesmo plano para quatro ferramentas: Claude, Codex, Cursor e Gemini. Cada uma gerou o projeto do zero, numa pasta separada.

<!-- TODO: contar por que decidi fazer isso (curiosidade, escolher uma ferramenta, ver se o plano estava claro o bastante...) -->

### O que foi pedido

O plano descrevia um serviço pequeno, sem a plataforma completa do Context:

- O Slack envia um comando, uma menção ou um alerta.
- Um servidor HTTP confirma o recebimento em menos de 3 segundos e coloca o trabalho numa fila SQS.
- Um worker, no mesmo container, coleta evidências da AWS e do GitHub, sempre com acesso somente leitura.
- Um LLM resume o incidente com base nessas evidências.
- A resposta volta para a thread no Slack.

Também pedia Python, Docker, configuração em YAML, provedor de LLM plugável e deploy na AWS.

### Onde as quatro concordaram

Nenhuma fugiu da arquitetura. Todas usaram Python com FastAPI, boto3, fila SQS com DLQ, ECS/Fargate atrás de um load balancer e Terraform para a infraestrutura. Todas criaram plugins para OpenAI, Anthropic, Google e Ollama.

Isso diz mais sobre o plano do que sobre as ferramentas. Quando a arquitetura já está decidida, o que muda é o detalhe.

### Onde elas divergiram

| | Claude | Codex | Cursor | Gemini |
|---|---|---|---|---|
| Arquivos Python | 43 | 29 | 56 | 19 |
| Linhas de Python | ~2.500 | ~2.000 | ~3.300 | ~2.300 |
| Arquivos de teste | 5 | 6 | 4 | 4 |
| Estado das execuções | DynamoDB (prod) / SQLite (local) | SQLite em EFS | S3 (jobs) + DynamoDB (métricas) | S3 (prod) / arquivos locais |
| Dependências | pyproject | pyproject | pyproject + Makefile | requirements.txt |

Algumas diferenças que chamaram atenção:

**Claude** separou bem as responsabilidades. Cada fonte de evidência (alarme e logs do CloudWatch, ECS, Lambda, RDS, GitHub) virou um módulo próprio, e o ambiente local sobe com LocalStack e Ollama, sem precisar de nenhuma chave de API.

**Codex** foi o mais enxuto e o único que deixou pronto um ciclo de verificação: `pytest`, `ruff`, `mypy`, `docker build` e `terraform fmt`. Também foi o que mais carregou o contexto original. A variável da fila ainda se chama `CONTEXT_LITE_QUEUE_URL`.

**Cursor** foi além do pedido. Além do serviço, criou um site em Astro e um guia de estudo de quatro semanas para aprender o código camada por camada.

**Gemini** teve a estrutura mais simples (poucos arquivos, maiores) e foi o único que criou a própria VPC no Terraform. As outras três assumem que a rede já existe. Também foi a única que usou as threads de assistente do Slack (`assistant_thread_started`).

<!-- TODO: qual delas rodou de primeira? Os testes passaram? Quanto tempo/quantas interações cada uma levou? -->

### O que eu aprendi

<!-- TODO: suas conclusões. Sugestões do que dá para tirar dos dados acima, se fizer sentido para você:
- O plano foi o que garantiu a consistência. Sem ele, as quatro arquiteturas provavelmente seriam bem diferentes.
- "Ir além do pedido" (site, guia) pode ser bom ou ruim, depende do que você queria.
- Número de linhas não diz quem fez melhor.
-->

No fim, o projeto seguiu outro caminho. O Context continuou como produto e a parte de estudo virou o Lino, que eu escrevo à mão. As quatro versões do Insights foram para o arquivo.

<!-- TODO: confirmar se quer mencionar o Lino aqui -->

Um grande abraço!  
Kleber Tertulino
