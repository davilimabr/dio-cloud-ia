## Anotações do LAB – Azure Speech Studio & Language Studio 📝

> **Objetivo geral:** Explorar na prática como **Azure Speech Studio** e **Azure Language Studio** podem ser combinados para criar soluções de **análise de fala** e **processamento de linguagem natural**, passando pelos principais recursos de cada serviço e como eles se integram a bots e agentes conversacionais.

---

### 1. Análise de Texto e Resposta a Perguntas

| Conceito                                        | Pontos-chave                                                                                                                                                                                                                                                                                                                   |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Text Analytics**                              | • Detecta idioma, sentimento, opiniões, entidades, frases-chave, PII, e recursos médicos.<br>• Usa modelos Transformer gerenciados ou customizados. ([learn.microsoft.com][1])                                                                                                                                                 |
| **Question Answering (CQA)**                    | • Evolução do QnA Maker, agora nativo no AI Language.<br>• Permite importar FAQs, URLs ou arquivos para gerar base de conhecimento.<br>• Suporta respostas exatas e geração de respostas curtas.<br>• QnA Maker será aposentado em 31 mar 2025 → migrar para CQA! ([learn.microsoft.com][2], [microsoftlearning.github.io][3]) |
| **Conversational Language Understanding (CLU)** | • Classificação de intenção + extração de entidades em linguagem coloquial.<br>• Totalmente integrado ao AI Foundry; pode alimentar bots ou fluxos de voz. ([devblogs.microsoft.com][4])                                                                                                                                       |

> **Dica prática:** Crie um projeto **Question Answering** no Language Studio, alimente com um PDF de FAQ interno e teste perguntas variadas (sinônimos, erros ortográficos) para avaliar a cobertura.

---

### 2. Serviço de Bot do Azure

* **Azure AI Bot Service + Copilot Studio** oferece ambiente low-code/pro-code para implantar bots em Teams, WebChat, Slack, etc. ([azure.microsoft.com][5])
* **Bot Framework Composer** permite fluxos de diálogo avançados e integração com CLU/CQA.
* Bots ficam hospedados como **App Service**; monitoramento e variáveis de ambiente são configurados no portal. ([learn.microsoft.com][6])

> **Hands-on**:
>
> 1. Crie um bot no Copilot Studio.
> 2. Adicione uma **habilidade** de “Question Answering” apontando para o recurso AI Language criado no passo anterior.
> 3. Publique e teste pelo Web Chat embutido.

---

### 3. Compreensão da Linguagem Coloquial

* **CLU** detecta intenções (“marcar reunião”, “cancelar reserva”) e extrai entidades (“amanhã”, “14h”, “Rio”).
* Alta tolerância a gírias e erros – ideal para atendimento ao cliente.
* Treine conjuntos de exemplo e valide métricas (precisão/recall). ([devblogs.microsoft.com][4])

> **Boas práticas:**
> • Use pelo menos 15 frases por intenção na fase inicial.
> • Faça *review* contínuo de frases reais para melhorar o modelo.

---

### 4. Links Úteis

* Speech Studio Visão geral → [https://aka.ms/speech-studio-docs](https://aka.ms/speech-studio-docs) ([learn.microsoft.com][7])
* Language Studio Visão geral → [https://aka.ms/language-studio-docs](https://aka.ms/language-studio-docs) ([learn.microsoft.com][1])
* Guia Bot Service → [https://aka.ms/azure-bot-service](https://aka.ms/azure-bot-service) ([learn.microsoft.com][8])
* Exercício oficial Question Answering → [https://aka.ms/mslearn-ai-language-qna](https://aka.ms/mslearn-ai-language-qna) ([microsoftlearning.github.io][3])

---

### 5. Conhecendo o Estudio de Fala (Speech Studio)

| Recurso                          | Descrição rápida                                                                                                                                      |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Speech-to-Text em tempo real** | Upload de áudio ou microfone; suporte a português (BR) e vários dialetos. Possível ajuste de modelo acústico ou de idioma. ([learn.microsoft.com][7]) |
| **Text-to-Speech (Neural TTS)**  | + de 400 vozes; novas vozes HD (feb 2025) com entonação emocional. ([techcommunity.microsoft.com][9])                                                 |
| **Speech Translation**           | Tradução bidirecional de fala com reconhecimento simultâneo.                                                                                          |
| **Speaker Recognition**          | Identificação/verificação de locutor para segurança ou personalização.                                                                                |
| **Conteinerização**              | Recursos podem rodar localmente via containers para latência/offline.                                                                                 |

> **Passo-a-passo sugerido:**
>
> 1. Abra Speech Studio → **Criar projeto Speech-to-Text** → carregue amostra WAV.
> 2. Observe transcrição, ajuste *locale* e veja diferença com/profundo.
> 3. Gere voz Neural do mesmo texto no módulo TTS.

---

### 6. Conhecendo o Language Studio

* Interface web que agrupa Text Analytics, CLU, CQA, Tradução, Resumo, Redação PII etc. ([learn.microsoft.com][1])
* Permite *playground* sem código + exportação para **REST** ou **SDKs** (.NET, Python, JS).
* Possui métricas de qualidade embutidas (precisão, cobertura de entidades).

> **Tip:** Use o botão **“View Code”** para copiar a chamada REST ou snippet de SDK após cada teste e acelerar a prototipação.

---

### 7. Entendendo o Desafio

> **Meta do LAB:**
> Construir um **bot de FAQ por voz** que:
>
> 1. Reconheça a pergunta do usuário em português falado (Speech-to-Text).
> 2. Classifique a intenção ou busque resposta exata via CQA no AI Language.
> 3. Devolva resposta sintetizada em voz Neural via Text-to-Speech.
> 4. Implemente fallback quando confiança < 70 % e registre *telemetry*.

**Critérios de sucesso**

| Critério                   | Indicador                                           |
| -------------------------- | --------------------------------------------------- |
| Taxa de resposta correta   | ≥ 85 % em conjunto de 20 perguntas-teste            |
| Latência ponta-a-ponta     | < 3 s média                                         |
| Cobertura de intenções/FAQ | 100 % do escopo fornecido                           |
| Uso de boas práticas Azure | Variáveis em Key Vault, logs no App Insights, CI/CD |

> **Próximos passos:** Dividir em grupos:
> • **Grupo A:** modelos Speech.
> • **Grupo B:** CQA/CLU.
> • **Grupo C:** orquestração Bot + DevOps.


[1]: https://learn.microsoft.com/en-us/azure/ai-services/language-service/overview?utm_source=chatgpt.com "What is Azure AI Language - Learn Microsoft"
[2]: https://learn.microsoft.com/en-us/azure/ai-services/qnamaker/overview/overview?utm_source=chatgpt.com "What is QnA Maker service? - Azure - Learn Microsoft"
[3]: https://microsoftlearning.github.io/mslearn-ai-language/Instructions/Exercises/02-qna.html?azure-portal=true&utm_source=chatgpt.com "Create a Question Answering Solution | Azure AI Language Exercises"
[4]: https://devblogs.microsoft.com/foundry/enhancing-conversational-agents-with-azure-ai-language-conversational-language-understanding-and-custom-question-answering/?utm_source=chatgpt.com "Enhancing Conversational Agents with Azure AI Language ..."
[5]: https://azure.microsoft.com/en-us/products/ai-services/ai-bot-service?utm_source=chatgpt.com "Azure AI Bot Service"
[6]: https://learn.microsoft.com/en-us/azure/bot-service/bot-service-manage-overview?view=azure-bot-service-4.0&utm_source=chatgpt.com "Manage a bot - Bot Service | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-studio-overview?utm_source=chatgpt.com "Speech Studio overview - Azure AI services | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0&utm_source=chatgpt.com "Azure AI Bot Service documentation - Learn Microsoft"
[9]: https://techcommunity.microsoft.com/blog/azure-ai-services-blog/azure-ai-speech-text-to-speech-feb-2025-updates-new-hd-voices-and-more/4387263?utm_source=chatgpt.com "Azure AI Speech text to speech Feb 2025 updates: New HD voices ..."
