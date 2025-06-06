# Relatório de Aprendizado – Computação em Nuvem

## Índice
1. [Introdução](#introdução)
2. [Introdução à Computação em Nuvem](#introdução-à-computação-em-nuvem)
   1. [Modelos de Serviço](#21-modelos-de-serviço)
   2. [Modelos de Implantação](#22-modelos-de-implantação)
3. [Como a Computação em Nuvem Funciona](#como-a-computação-em-nuvem-funciona)
4. [Benefícios da Computação em Nuvem](#benefícios-da-computação-em-nuvem)
5. [Criando Máquinas Virtuais no Microsoft Azure](#criando-máquinas-virtuais-no-microsoft-azure)
   1. [Pré‑requisitos](#51-pré‑requisitos)
   2. [Procedimento via Portal Azure](#52-procedimento-via-portal-azure)
   3. [Procedimento via Azure CLI](#53-procedimento-via-azure-cli)
   4. [Boas Práticas de Provisionamento](#54-boas-práticas-de-provisionamento)
6. [Conclusão](#conclusão)
7. [Referências e Leituras Recomandadas](#referências-e-leituras-recomendadas)

---

## Introdução

Este documento consolida o conhecimento adquirido nas últimas aulas do curso de Computação em Nuvem. O conteúdo foi organizado de forma a apresentar conceitos fundamentais, benefícios práticos e um guia passo a passo para a criação de máquinas virtuais no Microsoft Azure, plataforma estudada em laboratório.

---

## Introdução à Computação em Nuvem

A computação em nuvem é um paradigma de entrega de recursos de TI — incluindo servidores, armazenamento, redes, software e análises — pela internet, sob demanda e com tarifação baseada no consumo. Em vez de investir pesado em infraestrutura local, as organizações podem provisionar recursos de forma elástica, pagando apenas pelo que utilizam.

### 2.1 Modelos de Serviço

| Modelo | Descrição | Responsabilidades do Provedor | Responsabilidades do Cliente |
|--------|-----------|--------------------------------|------------------------------|
| IaaS (Infrastructure as a Service) | Fornece recursos básicos de computação (VMs, redes, armazenamento) | Hardware, rede, host, virtualização | Sistema operacional, runtime, dados, aplicações |
| PaaS (Platform as a Service) | Oferece ambiente pronto para desenvolvimento e implantação | IaaS + sistema operacional e runtime | Dados e aplicações |
| SaaS (Software as a Service) | Disponibiliza aplicações completas, acessadas via navegador | Toda a pilha, incluindo aplicação | Configuração limitada do aplicativo e uso |

### 2.2 Modelos de Implantação

- **Nuvem Pública**: Infraestrutura compartilhada e operada por um provedor de serviços.
- **Nuvem Privada**: Recursos exclusivos para uma organização, podendo residir on‑premises ou em data center de terceiro.
- **Nuvem Híbrida**: Combinação de nuvens públicas e privadas, permitindo portabilidade de cargas de trabalho.
- **Multi‑cloud**: Uso de dois ou mais provedores públicos, reduzindo dependência de um único fornecedor.

---

## Como a Computação em Nuvem Funciona

1. **Virtualização**  
   A nuvem explora a virtualização para abstrair hardware físico em múltiplas máquinas virtuais isoladas.

2. **Pools de Recursos**  
   CPU, memória, armazenamento e rede são agrupados em grandes clusters geridos por hipervisores.

3. **Orquestração e Automação**  
   Softwares de orquestração alocam dinamicamente recursos conforme políticas de escalonamento.

4. **Multitenancy**  
   Vários clientes coexistem na mesma infraestrutura de forma segura, com isolamento lógico.

5. **Redundância e Alta Disponibilidade**  
   Dados são replicados entre regiões e zonas de disponibilidade, minimizando o impacto de falhas.

6. **Governança e Segurança**  
   Controles de identidade, criptografia, monitoramento e conformidade regulatória são incorporados à camada de serviço.

---

## Benefícios da Computação em Nuvem

| Benefício | Explicação |
|-----------|-----------|
| Escalabilidade sob demanda | Aumenta ou reduz recursos automaticamente de acordo com a carga. |
| Redução de custos de capital | Dispensa aquisição e manutenção de hardware próprio. |
| Rapidez na implantação | Provisionamento de servidores em minutos, acelerando experimentação e time‑to‑market. |
| Modelo pay‑as‑you‑go | Pagamento proporcional ao uso efetivo, evitando desperdício. |
| Alta disponibilidade | Infraestrutura distribuída globalmente com SLAs robustos. |
| Recuperação de desastres | Serviços de backup e replicação geográfica simplificam planos de contingência. |
| Inovação contínua | Provedores lançam novos serviços e atualizações sem intervenção do cliente. |

---

## Criando Máquinas Virtuais no Microsoft Azure

### 5.1 Pré‑requisitos

- Conta Microsoft ou conta corporativa com assinatura Azure ativa.  
- Permissões para criar recursos ou grupo de recursos.  
- Estratégia de nomeação padronizada para facilitar gestão.

### 5.2 Procedimento via Portal Azure

1. **Acessar o portal**  
   Visite `https://portal.azure.com` e autentique‑se.
2. **Criar recurso**  
   Clique em ‘Create a resource’ e selecione ‘Virtual Machine’.
3. **Configurações básicas**  
   - Grupo de recursos  
   - Nome da VM  
   - Região (ex.: Brazil South)  
   - Imagem (ex.: Ubuntu 22.04 LTS)  
   - Tamanho (ex.: Standard B2s)  
4. **Autenticação**  
   Configure usuário e senha fortes ou carregue chave pública SSH.  
5. **Discos**  
   Escolha entre discos SSD premium ou padrão.  
6. **Rede**  
   Defina VNet, subnet e regras de porta, como abrir a porta 22 para SSH.  
7. **Segurança**  
   Habilite Azure Defender e configure regras de Security Center, se necessário.  
8. **Tags e revisão**  
   Adicione tags para governança de custos e clique em ‘Review + Create’.  
9. **Criação**  
   Após validação, clique em ‘Create’ e aguarde a conclusão do deployment.  
10. **Acesso**  
   Copie o IP público e conecte‑se via SSH ou RDP conforme o sistema operacional.  

### 5.3 Procedimento via Azure CLI

```bash
# Autenticação
az login

# Selecionar assinatura
az account set --subscription "<ID_da_Assinatura>"

# Variáveis
RESOURCE_GROUP="rg-lab-cloud"
LOCATION="brazilsouth"
VM_NAME="vm-ubuntu-22"
IMAGE="UbuntuLTS"
SIZE="Standard_B2s"

# Criação do grupo de recursos
az group create --name $RESOURCE_GROUP --location $LOCATION

# Criação da VM
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image $IMAGE \
  --size $SIZE \
  --admin-username aluno \
  --generate-ssh-keys
```

### 5.4 Boas Práticas de Provisionamento

- **Grupos de Recursos**: agrupar recursos relacionados para facilitar exclusão e políticas.  
- **Métricas e Logs**: habilitar Monitor e Azure Log Analytics.  
- **Tagueamento**: usar chaves como `environment=lab` e `owner=<nome>` para controle de custos.  
- **Políticas de Segurança**: limitar portas públicas, aplicar NSG e usar redes privadas sempre que possível.  
- **Desligamento Automatizado**: configurar auto-shutdown em ambientes de teste para economizar.

---

## Conclusão

A computação em nuvem transforma a forma como organizações consomem recursos de TI, oferecendo elasticidade, economia e rapidez. Ao dominar a criação de máquinas virtuais no Azure, consolidamos os conhecimentos práticos que sustentam esses benefícios, preparando-nos para arquitetar soluções escaláveis, seguras e economicamente viáveis.

---

## Referências e Leituras Recomandadas

- Microsoft Learn – Fundamentos de Azure  
- NIST SP 800‑145 – Definition of Cloud Computing  
- Documentation: Azure Virtual Machines  
- Armbrust, M. et al. – A view of cloud computing, Communications of the ACM, 2010  
- Erl, T.; Puttini, R.; Mahmood, Z. – Cloud Computing: Concepts, Technology & Architecture
