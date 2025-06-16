## Anotações do LAB – Copilot, OpenAI & IA Generativa Responsável 📝

> **Objetivo geral:** Explorar, em um ambiente hands-on, como **Microsoft Copilot** e os serviços **OpenAI/Azure OpenAI** podem ser utilizados com **boas práticas de IA responsável**, cobrindo filtros de conteúdo, políticas de uso e criação de soluções generativas.

---

### 1. IA Generativa Responsável

| Tópico                      | Pontos-chave                                                                                                                                                           |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Princípios Microsoft**    | Fairness, Reliability & Safety, Privacy & Security, Inclusiveness, Transparency, Accountability ([learn.microsoft.com][1], [learn.microsoft.com][2])                   |
| **Responsible AI Standard** | Framework interno aplicado a todos os produtos de IA; exige avaliações de impacto, governança de dados e monitoramento contínuo ([learn.microsoft.com][2])             |
| **Boas práticas para devs** | 1) Definir casos de uso claros.<br>2) Avaliar vieses nos dados.<br>3) Implementar *human-in-the-loop* em decisões críticas.<br>4) Registrar telemetria para auditoria. |
| **OpenAI Usage Policies**   | Restringem spam, desinformação, extração de dados pessoais, conteúdo ilícito, etc. Quebra = suspensão da chave API. ([openai.com][3])                                  |

---

### 2. Introdução – Filtros de Conteúdo OpenAI

* **Content Filtering Pipeline:** toda *prompt* + *completion* passam por modelos de classificação que sinalizam violência, sexualidade, ódio, autolesão e conteúdo ilícito. ([learn.microsoft.com][4])
* **Níveis de severidade** (low / medium / high); respostas em violação → bloqueadas ou truncadas.
* **Customização (preview):** clientes podem ajustar limites ou desligar filtros mediante aprovação (“managed customers”). ([learn.microsoft.com][5])
* **Recomendações:** logar `content_filter_results`, criar fallback, explicar bloqueios ao usuário sem vazar detalhes técnicos.

> **Mini-experimento:** envie três prompts de teor crescente (neutro → controverso) para notar mudanças nas flags de severidade.

---

### 3. Microsoft Copilot – Visão Atual (Jun 2025)

| Recurso                       | Descrição & Valor                                                                                                                                                                                                     |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Copilot Chat (M365 / Web)** | Assistente unificado em Word, Excel, PowerPoint, Outlook; gera, resume e automatiza tarefas. Última atualização inclui **Copilot Tuning** (low-code para ajustar o modelo com dados da empresa). ([microsoft.com][6]) |
| **Copilot Vision (Windows)**  | Novo recurso experimental que “enxerga” a janela compartilhada e fornece insights contextuais em tempo real. ([theverge.com][7])                                                                                      |
| **Copilot Actions**           | Automatiza fluxos web (preencher formulários, extrair dados) dentro do navegador; disponível via Copilot Labs. ([microsoft.com][8])                                                                                   |
| **Setups Enterprise**         | Integração com Microsoft Purview para proteção de dados, multi-agent orchestration e conectores Graph. ([microsoft.com][6])                                                                                           |

> **Hands-on sugerido:**
>
> 1. Ative Copilot no Word → “Resume este documento em bullet points de 3 linhas”.
> 2. Use Copilot Vision: compartilhe um PDF técnico e peça “explique esta tabela”. Observe a latência e a precisão.

---

### 4. Possibilidades do Microsoft Learning

* **Cursos e Trilhas:** “AI Skills Challenge”, “Responsible AI” e labs de **Azure AI Foundry Models**.
* **Credenciais:** *Microsoft Certified: Azure AI Engineer Associate* agora inclui módulo de **Copilot Studio** (jun/2025).
* **Recursos Gratuitos:** sandbox de 1 h para testar Azure OpenAI, quick-starts, GitHub repos oficiais.

> **Dica:** Complete o *Microsoft Learn Collection* “Build solutions with Azure OpenAI Service” antes de tentar desligar filtros de conteúdo.

---

### 5. Entendendo o Desafio

> **Meta do LAB:** Construir um **Assistente de Produtividade** que:
>
> 1. Receba comandos em linguagem natural via Copilot Studio.
> 2. Use Azure OpenAI (GPT-4o) para gerar respostas, respeitando content filter.
> 3. Execute ações no M365 (ex.: criar rascunho de e-mail, resumir planilha).
> 4. Faça *log* de prompts/completions + resultados do filtro em App Insights.

| Critério de sucesso                | Alvo                                                   |
| ---------------------------------- | ------------------------------------------------------ |
| **Taxa de conclusão sem bloqueio** | ≥ 90 % das instruções previstas                        |
| **Tempo médio resposta**           | < 4 s                                                  |
| **Aderência às políticas**         | 0 violações **OpenAI** ou **MSFT Responsible AI**      |
| **Observabilidade**                | Telemetria de chamadas + métricas de filtro exportadas |

**Divisão de equipes sugerida**

* **Grupo A:** Orquestração Copilot & Graph.
* **Grupo B:** Prompt engineering + segurança (filtros, políticas).
* **Grupo C:** Observabilidade, dashboards e documentação.

[1]: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/strategy/inform/ai?utm_source=chatgpt.com "AI Considerations for Your Cloud Strategy - Learn Microsoft"
[2]: https://learn.microsoft.com/en-us/azure/machine-learning/concept-responsible-ai?view=azureml-api-2&utm_source=chatgpt.com "What is Responsible AI - Azure Machine Learning | Microsoft Learn"
[3]: https://openai.com/policies/usage-policies/?utm_source=chatgpt.com "Usage policies - OpenAI"
[4]: https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter?utm_source=chatgpt.com "Azure OpenAI in Azure AI Foundry Models content filtering"
[5]: https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/content-filters?utm_source=chatgpt.com "Configure content filters (preview) - Azure OpenAI | Microsoft Learn"
[6]: https://www.microsoft.com/en-us/microsoft-365/blog/2025/05/19/introducing-microsoft-365-copilot-tuning-multi-agent-orchestration-and-more-from-microsoft-build-2025/?utm_source=chatgpt.com "Introducing Microsoft 365 Copilot Tuning, multi-agent orchestration ..."
[7]: https://www.theverge.com/news/685963/microsoft-copilot-vision-windows-launch?utm_source=chatgpt.com "Microsoft's new Copilot Vision can 'see' your apps on Windows"
[8]: https://www.microsoft.com/en-us/microsoft-copilot/blog/2025/06/04/release-notes-june-4-2025/?utm_source=chatgpt.com "Release Notes: June 4, 2025 | Microsoft Copilot Blog"
