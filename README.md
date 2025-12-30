# 🐳 Baleia GitOps - CI Pipeline com Argo Workflows

Este repositório contém a definição de infraestrutura e pipelines de CI/CD para o projeto Baleia, utilizando **Argo Workflows** rodando em um cluster **Kubernetes (Kind)** local.

## 🛠 Tecnologias Utilizadas
- **Kubernetes (Kind):** Orquestração local.
- **Argo Workflows:** Motor de pipeline nativo de container.
- **Kaniko:** Build de imagens Docker dentro do cluster (sem Docker-in-Docker).
- **Docker Hub:** Registry de imagens.

## 🚀 O Pipeline (CI)
O arquivo `workflows/baleia-pipeline.yaml` define um fluxo que:
1. **Baixa o código-fonte:** Utiliza uma estratégia de download via ZIP (HTTP) para contornar latências de rede.
2. **Gera Versionamento:** Cria uma tag dinâmica baseada em Data/Hora (`YYYYMMDD-HHMMSS`).
3. **Build & Push:** Compila a aplicação Go (Multi-stage build) e envia para o Docker Hub.

## 🔥 Desafios Enfrentados e Soluções (Troubleshooting)

Durante a implementação deste laboratório em ambiente local (Linux Mint + Kind), enfrentei e resolvi desafios complexos de infraestrutura:

### 1. Instabilidade de Rede e MTU (`connection reset by peer`)
* **Problema:** O download de imagens Docker e o `git clone` falhavam consistentemente com reset de conexão devido à fragmentação de pacotes na rede local.
* **Solução:** Ajuste manual do MTU (Maximum Transmission Unit) do nó do cluster para `1200`.
    ```bash
    docker exec -it kind-control-plane ip link set eth0 mtu 1200
    ```

### 2. Limite de Arquivos Abertos (`too many open files`)
* **Problema:** O Argo e o Git tentavam monitorar muitos arquivos simultaneamente, estourando o limite padrão do Kernel do Linux.
* **Solução:** Tuning do Kernel via Sysctl.
    ```bash
    sudo sysctl fs.inotify.max_user_watches=524288
    ```

### 3. Erro de Sistema de Arquivos no Kaniko (`device or resource busy`)
* **Problema:** O Kaniko falhava ao tentar limpar diretórios de sistema (`/proc`, `/sys`) entre os estágios do Dockerfile.
* **Solução:** Configuração de flags `--ignore-path` no template do Argo para ignorar diretórios protegidos pelo Kernel.

### 4. Timeout no Protocolo Git
* **Problema:** O protocolo Git sofria timeouts devido à instabilidade da conexão.
* **Solução:** Substituição da etapa de `git clone` por `wget` (Download do código via ZIP), garantindo robustez no download do código-fonte via HTTPS simples.

---
**Status do Projeto:** ✅ Pipeline de Build e Push concluído com sucesso.

<img width="1012" height="411" alt="image" src="https://github.com/user-attachments/assets/ffa1b2f1-e54a-4f76-9bf2-fc99fa12062b" />

<img width="1363" height="684" alt="image" src="https://github.com/user-attachments/assets/79f33d29-331f-4b8c-9373-cc876a84d268" />
<img width="1355" height="687" alt="image" src="https://github.com/user-attachments/assets/78b68e9a-2a22-4b55-84e2-2d695d183020" />
<img width="1350" height="453" alt="image" src="https://github.com/user-attachments/assets/6feed717-1a23-4542-876a-332e45ff38b1" />

