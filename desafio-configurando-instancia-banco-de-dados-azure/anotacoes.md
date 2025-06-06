# Configuração de Instância de Banco de Dados no Azure: Portal e Azure CLI

> **Objetivo:** Documentar detalhadamente o processo de provisionamento de uma instância de banco de dados no Microsoft Azure, demonstrando tanto a interface gráfica do **Azure Portal** quanto a **Azure CLI**. Estas anotações servem de referência rápida para estudantes e profissionais que pretendem replicar o procedimento em ambientes de teste ou produção.

## Índice
1. [Introdução](#1-introdução)
2. [Pré‑requisitos](#2-pré‑requisitos)
3. [Escolha do Serviço de Banco de Dados](#3-escolha-do-serviço-de-banco-de-dados)
4. [Provisionamento via Azure Portal](#4-provisionamento-via-azure-portal)
   1. [Passo a Passo](#41-passo-a-passo)
   2. [Configurações Importantes](#42-configurações-importantes)
5. [Provisionamento via Azure CLI](#5-provisionamento-via-azure-cli)
   1. [Instalação e Login](#51-instalação-e-login)
   2. [Passo a Passo](#52-passo-a-passo)
6. [Validação e Conexão](#6-validação-e-conexão)
7. [Boas Práticas de Segurança e Custos](#7-boas-práticas-de-segurança-e-custos)
8. [Solução de Problemas Comuns](#8-solução-de-problemas-comuns)
9. [Limpeza de Recursos](#9-limpeza-de-recursos)
10. [Referências](#10-referências)

---

## 1. Introdução
A criação de bancos de dados gerenciados no Azure simplifica tarefas de manutenção como backup, atualização de sistema operacional, alta disponibilidade e escalabilidade. Esta aula foca nos serviços **Azure Database for PostgreSQL**, **MySQL** e **SQL Database**, embora o fluxo básico seja similar para outros mecanismos.

## 2. Pré‑requisitos
- Conta ativa no Microsoft Azure (Subscription válida).
- Permissão de **Contributor** ou superior no Resource Group.
- **Azure CLI** versão ≥ 2.60 instalada localmente.
- Ferramenta cliente para o SGBD escolhido (p.ex. *psql*, *mysql* ou *sqlcmd*).

## 3. Escolha do Serviço de Banco de Dados
| Cenário | Serviço Recomendado | Observações |
|---------|--------------------|-------------|
| Aplicações .NET tradicionais | **Azure SQL Database** | Compatibilidade nativa com SQL Server. |
| Aplicações open‑source que usam PostgreSQL | **Azure Database for PostgreSQL – Flexible Server** | Suporte a extensões e fine‑tuning de parâmetros. |
| Workloads baseados em LAMP | **Azure Database for MySQL – Flexible Server** | Baixa latência e escalonamento vertical simples. |

> **Nota:** Avalie SLAs, limites de armazenamento, vCores e modelo de cobrança (provisionado vs. burstable) antes de prosseguir.

## 4. Provisionamento via Azure Portal
### 4.1 Passo a Passo
1. **Acesse** o [Azure Portal](https://portal.azure.com) e selecione **Create a resource**.
2. **Procure** por *Azure Database for PostgreSQL* (ou o serviço desejado) e clique em **Create**.
3. **Configurações Básicas**:
   - **Subscription**: escolha a assinatura adequada.
   - **Resource Group**: crie ou selecione um existente (ex.: `rg‑db‑demo`).
   - **Server name**: nome único (ex.: `pg‑bootcamp‑dio`).
   - **Region**: selecione a região mais próxima dos usuários.
   - **Workload type**: *Development* ou *Production*.
   - **Version**: escolha a versão do SGBD (ex.: PostgreSQL 16).
   - **Compute + storage**: defina vCores, memória e armazenamento.
   - **Administrator account**: usuário e senha fortes.
4. **Networking**:
   - Ative **Public access** ou configure **Private endpoint**.
   - Adicione regras de firewall (ex.: `0.0.0.0 – 0.0.0.0` para testes rápidos, depois restrinja).
5. **Data Encryption**: mantenha TDE habilitado (padrão) ou forneça sua própria chave (Customer‑managed).
6. **Review + create**: valide e clique em **Create**. O deployment leva ~5 min.

### 4.2 Configurações Importantes
- **Zona de Disponibilidade (AZ)**: recomendável habilitar para alta disponibilidade.
- **Backup Retention**: ajuste de 7 a 35 dias (padrão 7 dias).
- **Maintenance Window**: personalize para horários de menor uso.

## 5. Provisionamento via Azure CLI
### 5.1 Instalação e Login
```bash
# Instalação (Ubuntu/Debian)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
# Login interativo
az login
# Definir assinatura padrão (opcional)
az account set --subscription "<SUBSCRIPTION_ID>"
```

### 5.2 Passo a Passo
```bash
# Variáveis
RG="rg-db-demo"
LOCATION="eastus"
SERVER="pg-bootcamp-dio"
ADMIN_USER="azureadmin"
ADMIN_PASS="$(openssl rand -base64 16)"  # substitua conforme necessário
SKU="Standard_D2ds_v5"
VOLUME="Flexible"

# 1. Criar Resource Group
az group create --name $RG --location $LOCATION

# 2. Criar servidor PostgreSQL Flexible
az postgres flexible-server create \
  --resource-group $RG \
  --name $SERVER \
  --location $LOCATION \
  --admin-user $ADMIN_USER \
  --admin-password $ADMIN_PASS \
  --sku-name Standard_D2ds_v5 \
  --storage-size 100 \
  --tier Burstable \
  --version 16

# 3. Configurar acesso à rede (AllowAll para testes)
az postgres flexible-server firewall-rule create \
  --resource-group $RG \
  --name $SERVER \
  --rule-name AllowAll \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 255.255.255.255

# 4. Exibir cadeia de conexão
az postgres flexible-server show --resource-group $RG --name $SERVER --query fullyQualifiedDomainName -o tsv
```

> **Dica:** Para MySQL e SQL Database, substitua o comando `az postgres` por `az mysql flexible-server` ou `az sql db` conforme a documentação.

## 6. Validação e Conexão
1. **Instale** a ferramenta cliente apropriada (ex.: `sudo apt install postgresql-client`).
2. **Conecte‑se**:
   ```bash
   psql "host=<FQDN> port=5432 dbname=postgres user=$ADMIN_USER password=$ADMIN_PASS sslmode=require"
   ```
3. **Crie** um banco de dados de teste e uma tabela simples para confirmar.
4. **Teste** tempos de resposta usando `
   \timing` e consultas de exemplo.

## 7. Boas Práticas de Segurança e Custos
- **Princípio do Menor Privilégio:** crie usuários específicos para cada aplicação.
- **Network Security:** prefira **Private Link** ou **Service Endpoints** em produção.
- **Auditoria e Monitoramento:** habilite **Azure Monitor** e **Diagnostic Settings**.
- **Escalonamento Automático:** configure *scale up* ou *scale out* conforme carga.
- **Reserva de Instância:** para workloads previsíveis, utilize planos de 1 ou 3 anos para reduzir custos.

## 8. Solução de Problemas Comuns
| Erro | Causa Provável | Resolução |
|------|----------------|-----------|
| `psql: could not connect to server` | Porta 5432 bloqueada | Ajustar regras de firewall ou NSG. |
| `FATAL: password authentication failed` | Senha incorreta | Redefinir senha via portal ou CLI. |
| Latência alta inesperada | Região distante | Recriar instância em região mais próxima. |

## 9. Limpeza de Recursos
```bash
# Remover servidor e RG (irá deletar tudo!)
az group delete --name $RG --yes --no-wait
```
> **Atenção:** A exclusão é irreversível; faça backup antes.

## 10. Referências
- **Documentação Oficial:** <https://learn.microsoft.com/azure>
- **Azure CLI Reference:** <https://learn.microsoft.com/cli/azure>
- **Melhores Práticas de Banco de Dados:** <https://learn.microsoft.com/azure/architecture/best-practices/data>
