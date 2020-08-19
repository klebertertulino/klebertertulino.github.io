---
layout: post
title: WebSocket no Ingress do Kubernetes
date: 2020-08-19 14:34
author: Kleber Tertulino
img: web.jpg
tags: [Kubernetes, Ingress]
summary: Como ativar WebSocket no Ingress do Kubernetes
---

Para expor um serviço através de um ingress controller é necessário incluir uma anotação nas definições do recurso para que o mesmo identifique qual dos serviços expostos devem ser tratado como WebSocket.

### O que é WebSocket

Pelas palavras da Wikipedia:

> **WebSocket** é uma tecnologia que permite a comunicação bidirecional por canais full-duplex sobre um único soquete Transmission Control Protocol (TCP). Ele é projetado para ser executado em navegadores e servidores web que suportem o HTML5, mas pode ser usado por qualquer cliente ou servidor de aplicativos.

Exemplo de serviço com WebSocket

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: meu-ingress
  annotations:
    nginx.org/websocket-services: "ws-svc"
spec:
  rules:
    - host: meu.exemplo.com
      http:
        paths:
          - path: /login
            backend:
              serviceName: login-svc
              servicePort: 80
          - path: /carrinho
            backend:
              serviceName: carrinho-svc
              servicePort: 80
          - path: /ws
            backend:
              serviceName: ws-svc
              servicePort: 8008
```

No exemplo acima incluí uma anotação informando que o “ws-svc” é um serviço WebSocket.

Simples e efetivo, não é mesmo?

Um grande abraço!

Kleber Tertulino
