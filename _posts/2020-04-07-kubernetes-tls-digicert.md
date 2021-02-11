---
layout: post
title: Kubernetes TLS com certificado DigiCert
date: 2020-04-07 00:00:00 +0300
author: Kleber Tertulino
tags: [Kubernetes, TLS]
---

Recentemente tive que atualizar um certificado que estava para expirar, o certificado antigo já esta pronto nos formatos necessários, já o novo certificado emitido pela DigiCert não estava funcionava simples assim. Quando abria o site pelo navegador o certificado era tido válido porém ao testar com o **Postman** ou mesmo com um simples **curl** apresenta um erro na CA.

O pacote de arquivos disponibilizado contia os seguintes arquivos:

- **Arquivo chave:** tls.key
- **Arquivo de certificado x.509:** certificado.crt

Para resolver o problema tive que juntar o conteúdo do certificado disponibilizado pela DigiCert com o do certificado intermediário disponível no site **GeoTrust_RSA_CA_2018.pem**.

Com os arquivos no mesmo diretório executei os seguintes comandos:

```shell
cp certificado.crt tls.crt
cat GeoTrust_RSA_CA_2018.pem >> tls.crt
```

Depois foi só criar o secret com o comando:

```yaml
kubectl create secret tls nome --key tls.key --cert tls.crt
```

E posteriormente, utilizar essa chave no Ingress

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
---
spec:
tls:
  - secretName: nome
backend:
  serviceName: servico
  servicePort: 80
```

Espero ter ajudado.

Um grande abraço!

Kleber Tertulino
