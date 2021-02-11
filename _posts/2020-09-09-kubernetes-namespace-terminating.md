---
layout: post
title: Excluindo um namespace travado no estado "Terminating"
date: 2020-09-09 22:31
author: Kleber Tertulino
tags: [Kubernetes, Namespaces, base-de-conhecimento]
---

As vezes ao excluir um namespace, por exemplo: **foo**. O namespace fica travado no estado "_Terminating_":

Exemplo:

```
kubectl delete namespace foo
Error from server (Conflict): Operation cannot be fulfilled on namespaces "foo": The system is ensuring all content is removed from this namespace.  Upon completion, this namespace will automatically be purged by the system.
```

Após algum tempo é possível verificar que o namespace "foo" está com o estado "_Terminating_":

```
kubectl get namespace foo
NAME    STATUS        AGE
foo   Terminating     3d
```

## Solução

Para remover o namespace devemos executar os seguintes passos:

1. Gerar um arquivo de "dump" do namespace no formato json;
2. Editar o arquivo tmp.json;
3. Executar o kubectl proxy;
4. Atualizar o namespace através da API do Kubernetes;
5. Verificar o resultado.

---

#### 1. Gerar um "dump" em formato json do "namespace"

```
kubectl get namespace foo -o json > tmp.json
```

#### 2. Editar o arquivo tmp.json;

Edite o arquivo **tmp.json** com o seu editor favorito, removendo a string **"kubernetes"** das chaves **_finalizers_**, veja abaixo o antes e o depois:

**Antes**

```
 "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
```

**Depois**

```
 "spec": {
        "finalizers": [

        ]
    },
```

#### 3. Executar o kubectl proxy;

O kubectl proxy cria um servidor proxy entre o Servidor de API do Kubernetes e o localhost. Para maiores detalhes sobre o comando proxy verifique a [documentação do kubectl](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#proxy)
**Obs.:** o & é para executar o comando em background, se preferir pode executar em uma outra janela do terminal

```
kubectl proxy &

```

Você deve receber um output como:

```
Starting to serve on 127.0.0.1:8001
```

#### 4. Atualizar o namespace através da API do Kubernetes;

Agora execute o comando abaixo lembrando de substituir o **"foo"** pelo namespace que deseja remover:

```
curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://localhost:8001/api/v1/namespaces/foo/finalize

```

#### 5. Verificar o resultado.

Ao executar novamente o comando, o namespace **foo** não é mais encontrado.

```
kubectl get namespace foo
Error from server (NotFound): namespaces "foo" not found
kubectl get namespaces
```
