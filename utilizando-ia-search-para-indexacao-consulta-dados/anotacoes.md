## Anotações do LAB – Azure Cognitive Search: AI Search para Indexação & Consulta de Dados 📝

> **Objetivo geral:** aplicar o ciclo **Ingestão → Indexação → Exploração** para minerar conhecimento com **Azure AI Search** (antigo *Cognitive Search*), entendendo como criar pipelines sem-código ou por API, enriquecer documentos com IA e executar buscas cognitivas eficientes.

---

### 1. Mineração de conhecimento

| Fase                          | O que acontece?                                                                                                                                          | Ferramentas                                                                               | Dicas                                                                                                                           |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Ingestão**                  | Conectar fontes (Blob, ADLS, SQL, Cosmos DB). Indexer executa *document cracking* para extrair texto e metadados. ([learn.microsoft.com][1])             | Import Data Wizard ou SDK/REST (`datasources`, `indexers`).                               | Garanta permissões (“**Storage Blob Data Contributor**” p/ MSI do serviço). ([learn.microsoft.com][2])                          |
| **Enriquecimento (opcional)** | Skillset aplica OCR, detecção de linguagem, entidades, emoções; pode gravar em **Knowledge Store**. ([learn.microsoft.com][3], [learn.microsoft.com][4]) | Skills pré-construídos (Cognitive Services) + Custom Skills (Azure Functions/Containers). | Adicione *multiservice key* se exceder os 20 transações/dia grátis. ([learn.microsoft.com][5])                                  |
| **Indexação**                 | Define esquema, cria índice físico, carrega dados. ([learn.microsoft.com][6], [learn.microsoft.com][7])                                                  | Portal (“Indexes”) ou REST `PUT /indexes/my-index`.                                       | Use sufixos `_s`, `_t`, `_ss` p/ campos filterable, searchable, sortable.                                                       |
| **Exploração**                | Consultas full-text, filtros, facets, semantic & vector search.                                                                                          | Azure AI Search REST (`search=*`), SDKs, Postman, Search Explorer.                        | Habilite **Semantic Ranker** para reescrita de queries + ranking profundo. ([learn.microsoft.com][8], [learn.microsoft.com][9]) |

---

### 2. Solução de Pesquisa Cognitiva do Azure

* **Azure AI Search** = motor *enterprise-ready* para conteúdo heterogêneo; suporta até 100 indexes/SKU, escalonamento horizontal com réplicas & partitions. ([learn.microsoft.com][10])
* Pricing tiers: Free (3 indexes), Basic, Standard S1-S3 (vector search incluso a partir de S1).
* Segurança: requisições assinadas por **API-Key** ou **AAD**; dados criptografados em repouso.

> **Hands-on:** crie serviço Free, importe o *hotels-sample* via wizard e consulte no Search Explorer.

---

### 3. Enriquecimento de IA

| Item                | Exemplo prático                                                                                                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Skillset**        | `EntityRecognitionSkill`, `SentimentSkill`, `OCRSkill`, `CustomWebApiSkill`.                                                                                               |
| **Pipeline**        | Data Source → Indexer → **Skillset** → Index/Knowledge Store. ([learn.microsoft.com][3])                                                                                   |
| **Knowledge Store** | Salva tabelas/Blobs normalizados para Power BI ou Synapse. ([learn.microsoft.com][4])                                                                                      |
| **Boas práticas**   | • Encadear outputs com `outputs`: `sourceFieldMappings`.<br>• Testar skillset isoladamente (Portal > Skillset > Test).<br>• Monitorar falhas de skill em *Indexer Status*. |

---

### 4. Conhecendo o Desafio do laboratório

> **Cenário:** sua empresa possui milhares de PDFs de contratos. Você precisa habilitar pesquisa semântica sobre cláusulas específicas.

**Checkpoint da sessão**

1. **Fonte**: Blob Storage “contracts/”.
2. **Skillset mínimo**: OCR + Language Detection + KeyPhrase.
3. **Index**: campos `content`, `contractNumber`, `parties_ss`, `effectiveDate_dt`.
4. **Query Demo**: `search=cláusula de rescisão AND parties:ACME`.

---

### 5. Buscas cognitivas

* **Full-Text** – oper. `search`, term boost `^`, filtros OData.
* **Facets & Aggregations** – `facet=parties,count:5` devolve top 5 partes.
* **Semantic Search** – habilita `queryType=semantic` (+ `captions`), usa modelos Bing para ranker L2. ([learn.microsoft.com][8])
* **Vector Search (preview GA)** – armazena embeddings OpenAI no mesmo índice, realiza `vectorQueries`. Combinaçao híbrida = BM25 + Vectors + Semantic.
* **Query rewrite** – auto-expand sinônimos e intents (nov/24 preview). ([learn.microsoft.com][8])

> **Tip rápido:** sempre defina um **semanticConfiguration** no índice para escolher campos mais relevantes; não exige rebuild. ([learn.microsoft.com][9])

---

### 6. Entendendo o Desafio – Critérios de Sucesso

| Métrica                               | Alvo                                                   |
| ------------------------------------- | ------------------------------------------------------ |
| **Cobertura indexada**                | 100 % dos documentos no container                      |
| **Latência média**                    | ≤ 2 s por consulta                                     |
| **Precisão Top-3 (avaliação manual)** | ≥ 85 %                                                 |
| **Observabilidade**                   | Logs de indexer & queries exportados para App Insights |

**Entrega final**

1. Repositório Git com scripts REST/SDK (`datasource.json`, `skillset.json`, `index.json`, `indexer.json`).
2. Painel Power BI apontando para Knowledge Store (opcional).
3. Documento markdown explicando decisões de schema, skills e tuning.

[1]: https://learn.microsoft.com/en-us/azure/search/search-indexer-overview?utm_source=chatgpt.com "Azure AI Search - Indexer overview - Learn Microsoft"
[2]: https://learn.microsoft.com/en-us/answers/questions/1805090/error-while-connecting-to-data-in-the-azure-ai-sea?utm_source=chatgpt.com "Error while connecting to data in the 'Azure AI Search' - Microsoft Q&A"
[3]: https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-intro?utm_source=chatgpt.com "AI enrichment concepts - Azure AI Search - Learn Microsoft"
[4]: https://learn.microsoft.com/en-us/azure/search/knowledge-store-concept-intro?utm_source=chatgpt.com "Knowledge store concepts - Azure AI Search - Learn Microsoft"
[5]: https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-image-scenarios?utm_source=chatgpt.com "Extract text from images by using AI enrichment - Azure AI Search"
[6]: https://learn.microsoft.com/en-us/azure/search/search-how-to-create-search-index?utm_source=chatgpt.com "Create an index - Azure AI Search - Learn Microsoft"
[7]: https://learn.microsoft.com/en-us/rest/api/searchservice/create-index?utm_source=chatgpt.com "Create Index (Azure AI Search REST API) - Learn Microsoft"
[8]: https://learn.microsoft.com/en-us/azure/search/whats-new?utm_source=chatgpt.com "What's new in Azure AI Search | Microsoft Learn"
[9]: https://learn.microsoft.com/en-us/azure/search/semantic-how-to-configure?utm_source=chatgpt.com "Configure semantic ranker - Azure AI Search | Microsoft Learn"
[10]: https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search?utm_source=chatgpt.com "What's Azure AI Search? - Learn Microsoft"
