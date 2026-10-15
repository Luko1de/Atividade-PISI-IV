# Docker e Kubernetes: atividade prática

Atividade da disciplina **Projeto Interdisciplinar para Sistemas de Informação IV**.
Uma aplicação web simples em Python é empacotada em uma imagem Docker, executada em containers e depois orquestrada em um cluster Kubernetes local (Minikube).

**Aluno:** Lucas

## Estrutura do repositório

```
.
├── docker-k8s/          # Exercícios 1 a 9 (aplicação hello-app)
│   ├── app.py
│   ├── Dockerfile
│   ├── deployment.yaml
│   └── service.yaml
├── app-data/            # Exercício 10 (aplicação com nome e data atual)
│   ├── app.py
│   ├── Dockerfile
│   ├── deployment.yaml
│   └── service.yaml
└── README.md
```

## Pré-requisitos

- [Git](https://git-scm.com/)
- [Docker Desktop](https://www.docker.com/) em execução
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

Para conferir a instalação:

```powershell
docker --version
minikube version
kubectl version --client
```

## Parte 1: Docker

Dentro da pasta `docker-k8s`:

```powershell
# Construir a imagem
docker build -t lucas/hello-app:v1 .

# Rodar o container (acesse http://localhost:8080)
docker run -d -p 8080:8080 --name minha-app lucas/hello-app:v1

# Verificar
docker ps
docker images
```

Para rodar mais de um container ao mesmo tempo, mapeie portas diferentes do host para a porta 8080 do container:

```powershell
docker run -d -p 8082:8080 --name app1 lucas/hello-app:v1
docker run -d -p 8083:8080 --name app2 lucas/hello-app:v1
```

Para encerrar e remover:

```powershell
docker rm -f minha-app app1 app2
```

## Parte 2: Kubernetes com Minikube

### 1. Iniciar o cluster

```powershell
minikube start --driver=docker
kubectl get nodes
```

Deve aparecer 1 nó chamado `minikube` com status `Ready`.

### 2. Enviar a imagem ao cluster

Os manifestos usam `imagePullPolicy: Never`, então a imagem precisa estar dentro do Minikube:

```powershell
docker build -t lucas/hello-app:v1 .
minikube image load lucas/hello-app:v1
minikube image ls
```

> **Nota:** o guia da atividade sugere `minikube docker-env | Invoke-Expression`. Com o runtime containerd (padrão nesta versão do Minikube) esse comando falha no Windows com o erro `SSH_AGENT_START`, por isso foi usado o `minikube image load`. Se a imagem for reconstruída, é preciso rodar o `image load` de novo.

### 3. Criar o Deployment e o Service

```powershell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl get pods
kubectl get services
```

### 4. Acessar a aplicação

```powershell
minikube service hello-app-service
```

O comando abre o navegador. Em Windows com driver Docker, mantenha o terminal aberto enquanto usar a aplicação.

### 5. Escalar e depurar

```powershell
# Escalar: altere replicas para 4 no deployment.yaml e reaplique
kubectl apply -f deployment.yaml
kubectl get pods

# Depuração
kubectl describe pod NOME-DO-POD
kubectl logs NOME-DO-POD

# O Deployment recria um pod deletado
kubectl delete pod NOME-DO-POD
kubectl get pods
```

## Parte 3: Aplicação personalizada (exercício 10)

A pasta `app-data` contém uma aplicação que responde com o nome do aluno e a data e hora atuais.

```powershell
cd app-data
docker build -t lucas/data-app:v1 .
minikube image load lucas/data-app:v1
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
minikube service data-app-service
```

## Resumo dos conceitos

| Conceito | Descrição |
| --- | --- |
| Imagem | Molde somente leitura com o código e as dependências da aplicação |
| Container | Instância em execução criada a partir de uma imagem, isolada das demais |
| Pod | Menor unidade implantável no Kubernetes, geralmente com um container |
| Deployment | Mantém o número desejado de pods e recria os que falharem |
| Service | Endereço estável que distribui o tráfego entre os pods |

## Problemas comuns

| Sintoma | Causa provável | Solução |
| --- | --- | --- |
| `failed to connect to the docker API` | Docker Desktop não está rodando | Abrir o Docker Desktop e esperar o "Engine running" |
| Pod em `ErrImageNeverPull` | Imagem não está no Minikube | Rodar `minikube image load NOME-DA-IMAGEM` |
| Porta já em uso | Outro container usa a mesma porta do host | Usar outra porta em `-p HOST:8080` |
| Acentos quebrados na página | `app.py` salvo fora de UTF-8 | Salvar em UTF-8 e rebuildar a imagem |
