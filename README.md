# Projeto de CI/CD com GitOps: FastAPI, GitHub Actions e Argo CD

Este projeto demonstra a implementação de um pipeline completo de Integração Contínua (CI) e Entrega Contínua (CD) para uma aplicação web simples desenvolvida em FastAPI. O fluxo é totalmente automatizado utilizando os princípios de GitOps, onde o Git atua como a única fonte de verdade para a infraestrutura e as implantações.

---

## 📖 Índice
- [Objetivo](#objetivo)  
- [🚀 Tecnologias Utilizadas](#-tecnologias-utilizadas)  
- [⚙️ Arquitetura e Fluxo de Trabalho](#%EF%B8%8F-arquitetura-e-fluxo-de-trabalho)  
- [📋 Passo a Passo da Implementação](#-passo-a-passo-da-implementação)  

---

## Objetivo
Automatizar todo o ciclo de vida de uma aplicação, desde o push de um novo código no repositório até sua implantação em um cluster Kubernetes. O processo abrange:

- Build automático de uma imagem Docker.
- Push da imagem para um registro de contêineres (Docker Hub).
- Atualização automática dos manifestos de implantação do Kubernetes.
- Sincronização e deploy contínuo no cluster Kubernetes através do Argo CD.

---

## 🚀 Tecnologias Utilizadas

**Aplicação:**
- **FastAPI**: Framework web moderno e de alta performance para a criação de APIs em Python.

**Contêineres:**
- **Docker**: Plataforma para desenvolver, empacotar e executar aplicações em contêineres.
- **Docker Hub**: Registro de contêineres na nuvem para armazenar e distribuir as imagens Docker.

**CI/CD e Automação:**
- **GitHub Actions**: Ferramenta de automação integrada ao GitHub para criar pipelines de CI/CD, build, testes e deploy.

**Orquestração e GitOps:**
- **Kubernetes**: Plataforma de orquestração de contêineres para automatizar a implantação, o dimensionamento e a gestão de aplicações.
- **Rancher Desktop**: Ferramenta para executar um ambiente Kubernetes e de contêineres localmente.
- **Argo CD**: Ferramenta declarativa de Entrega Contínua para Kubernetes, baseada nos princípios de GitOps.

**Ambiente de Desenvolvimento:**
- **GitHub Codespaces**: Ambiente de desenvolvimento em nuvem acessível diretamente do navegador.
- **Ubuntu + kubectl**: Gerenciamento do cluster Kubernetes fornecido pelo Rancher Desktop via linha de comando.

---

## ⚙️ Arquitetura e Fluxo de Trabalho

O projeto é dividido em dois repositórios principais:

1. **hello-app**: Contém o código Python, Dockerfile e workflow do GitHub Actions.
2. **hello-manifests**: Contém os manifestos Kubernetes (`deployment.yaml`, `service.yaml`) que descrevem o estado desejado da aplicação. o arquivo `namespace.yaml` e o `kustomization.yaml` para ditar a ordem de execução dos arquivos.

**Fluxo de trabalho automatizado:**

1. Desenvolvedor faz `git push` para a branch `main` do repositório `hello-app`.
2. GitHub Actions é acionado.
3. A action constrói uma nova imagem Docker e envia para o Docker Hub com a tag do commit SHA.
4. A action clona o repositório `hello-manifests`, atualiza `deployment.yaml` com a nova tag da imagem e abre um Pull Request.
5. Após o merge do PR, o Argo CD detecta a mudança.
6. Argo CD sincroniza o estado do cluster Kubernetes com o estado do repositório, atualizando a aplicação.

---

## 📋 Passo a Passo da Implementação

### 1. Preparação dos Repositórios
- **hello-app**: Contém `main.py`, `Dockerfile`, `requiriments.txt`,`chaves publicas e pivadas` e workflow em `.github/workflows/ci-cd.yaml`.
- **hello-manifests**: Contém pasta `k8s/` com os manifestos `deployment.yaml`, `service.yaml`, `namespace.yaml` e o `kustomization.yaml`.

### main.py
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Meu novo novo novo novo novo novo novo novo:: Hello from Hello-App! V8"}
```

### 2. Configuração da Autenticação
- **Chave SSH**:
  - Gerar par de chaves: `ssh-keygen`.
  - Adicionar chave pública como Deploy Key com permissão de escrita no repositório `hello-manifests`.
  - Adicionar chave privada como Secret (`SSH_PRIVATE_KEY`) no repositório `hello-app`.
- **Segredos do Docker Hub**:
  - Nome de usuário (`DOCKER_USERNAME`) e Access Token (`DOCKER_PASSWORD`) adicionados como Secrets no repositório `hello-app`.

### 3. Pipeline do GitHub Actions
O arquivo `.github/workflows/ci-cd.yaml` foi configurado para:

- Fazer login no Docker Hub usando os segredos.
- Construir e enviar a imagem Docker para o Docker Hub (`latest` e SHA do commit).
- Usar a chave SSH para clonar `hello-manifests`.
- Atualizar tag da imagem diretamente na main em `k8s/deployment.yaml` e fazer push.
- (Opcional) Abrir Pull Request com alterações.

### 4. Configuração do Argo CD
- Instalar Argo CD no cluster Kubernetes local (Rancher Desktop).
- Criar aplicação na UI do Argo CD:
  - Repository URL: `hello-manifests`
  - Path: `k8s/`
  - Destination Cluster: `https://kubernetes.default.svc`
  - Namespace: `hello-app`
  - Sync Policy: Automatic, com prune e self-heal.
  - Aceeso pelo:
    ```bash
     kubectl -n argocd port-forward svc/argocd-server 8081:443
    ```

### 5. Teste da Aplicação
- Acessar aplicação via `kubectl port-forward`:

```bash
kubectl port-forward svc/hello-service 8082:80 -n hello-app
```



### 6. Atualização da mensagem diretamente no arquivo main.py do repositório ou por um editor de texto
- Após a alteração verificar o a seção Actions do Github;
- Verificação da saúde e sincronização do ArgoCD;
- Coneção do pod com a porta definida;
- Atualização da mensagem corretamente.


## 🖼️ Evidências de Execução do Projeto

### 1. Alteração no Código
![Mensagem alterada no main.py](https://github.com/VitorYuji25/hello-app/raw/main/imagens/mensagem_main.py.png)

### 2. Pipeline do GitHub Actions
![Workflow do GitHub Actions com Sucesso](https://github.com/VitorYuji25/hello-app/raw/main/imagens/GitHub_Actions_Sucesso.png)

### 3. Atualização da Imagem no DockerHub
![Atualização no DockerHub](https://github.com/VitorYuji25/hello-app/raw/main/imagens/Atualizacao_DockerHub.png)

### 4. Sincronização do ArgoCD
![Sincronização do ArgoCD](https://github.com/VitorYuji25/hello-app/raw/main/imagens/ArgoCD_Sync.png)

### 5. Pod Funcional no Kubernetes
![Imagem do Pod funcional](https://github.com/VitorYuji25/hello-app/raw/main/imagens/Imagem_pod_funcional.png)

### 6. Resultado Final
![Saída da mensagem atualizada](https://github.com/VitorYuji25/hello-app/raw/main/imagens/Saida_mensagem_atualizada.png)
