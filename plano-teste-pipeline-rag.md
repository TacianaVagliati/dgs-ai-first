# Plano de Teste — Pipeline RAG NovaTech

## Visão Geral

- **Escopo:** Validação do pipeline Documento → Embedding → Retrieval → LLM Generation
- **Ambiente:** Desenvolvimento + Produção (Staging)
- **Data:** 2026-06-03

---

## 1. Objetivo

Validar o pipeline de RAG da NovaTech quanto a precisão, rastreabilidade e segurança de resposta para atendimento ao cliente.

---

## 2. Guardrails Definidos

1. Citar fonte de origem sempre
2. Nunca inventar prazos ou valores não contemplados na documentação
3. Na ausência dos dados, informar explicitamente
4. Responder em português formal

---

## 3. Fontes de Verdade

| Documento | Tipo | Normativo? |
|-----------|------|------------|
| POL-001-politica-devolucao.md | Política | Sim |
| PROC-042-frete-especial-v1.md | Procedimento | Sim (vigência a validar) |
| PROC-042-v2-frete-especial-revisado.md | Procedimento | Sim (versão mais recente) |
| SLA-2024-tabela-sla-clientes.md | Tabela contratual | Sim |
| FAQ-atendimento.md | FAQ informal | Não |
| anexo-a-documentacao-simulada-novatech.md | Consolidado | Referência |
| anexo-b-chunks-referencia-rag.md | Gabarito de chunks | Referência de teste |

---

## 4. Estratégia de Qualidade

Escala de avaliação por caso:
- **0:** Incorreta (erro factual, alucinação, sem fonte, inversão de regra)
- **1:** Parcialmente correta (acerta núcleo, omite ressalva crítica ou fonte incompleta)
- **2:** Correta (resposta adequada, fonte consistente, comportamento esperado)

Critérios de aprovação do ciclo:
- Nenhum caso crítico com nota 0
- Cobertura completa das 6 categorias
- 90% dos casos com nota ≥ 1

---

## 5. Visão Geral do Pipeline

1. **Ingestão:** Extração e normalização dos documentos
2. **Chunking:** Segmentação em blocos para indexação vetorial
3. **Embedding e indexação:** Geração de embeddings e armazenamento no Azure AI Search
4. **Retrieval:** Recuperação dos chunks relevantes (Top-K)
5. **LLM Generation:** Resposta final com citação de fonte

---

## 6. Testes de Ingestão

### 6.1 Validação de Extração

| Caso | Entrada | Resultado esperado | Evidência | Status |
|------|---------|--------------------|-----------|--------|
| ING-01 | POL-001 em markdown | Texto completo extraído sem perda de seções | Log de ingestão | Planejado |
| ING-02 | PROC-042-v2 em markdown | Tabela de multiplicadores preservada | Snapshot do parser | Planejado |

### 6.2 Validação de Chunking

| Caso | Regra de chunk | Resultado esperado | Evidência | Status |
|------|----------------|--------------------|-----------|--------|
| CHK-01 | Chunk por seção + overlap | Seções críticas intactas (3.2 POL-001, 2.1 PROC) | Amostra de chunks | Planejado |
| CHK-02 | Limite de tamanho | Sem truncar regras essenciais | Relatório de tamanho | Planejado |

### 6.3 Validação de Indexação

| Caso | Validação | Resultado esperado | Evidência | Status |
|------|-----------|--------------------|-----------|--------|
| IDX-01 | Metadados de origem | Documento e seção presentes no índice | Consulta no índice | Planejado |
| IDX-02 | Versão documental | v1 e v2 distinguíveis por filtro | Consulta por filtro | Planejado |

---

## 7. Testes de Retrieval

Matriz de casos com gabarito do Anexo B:

| Caso | Pergunta | Chunk esperado (Anexo B) | Top-K retornado | Resultado esperado | Status |
|------|----------|--------------------------|-----------------|--------------------|--------|
| RET-01 | Qual o prazo de devolução? | POL-001-A, POL-001-B | A preencher | Chunks corretos no Top-K | Planejado |
| RET-02 | Quanto custa frete para 600kg para Manaus? | PROC-042v2-A, PROC-042v2-B | A preencher | Chunks v2 priorizados | Planejado |
| RET-03 | Qual o SLA do cliente Platinum? | SLA-2024-A | A preencher | Recuperar regra de inexistência do tier | Planejado |
| RET-04 | Posso devolver carga perigosa? | POL-001-B | A preencher | Recuperar regra de não elegibilidade | Planejado |
| RET-05 | Qual o multiplicador de frete para o Sudeste? | PROC-042v2-B | A preencher | Recuperar valor 1.1 da v2 | Planejado |

---

## 8. Testes de Avaliação de Retrieval e Resposta

| Caso | Critério | Resultado esperado | Medida |
|------|----------|--------------------|--------|
| EVAL-01 | Precisão de retrieval | Chunks essenciais presentes no Top-K | Precision@K |
| EVAL-02 | Cobertura de chunks críticos | Recuperar chunk correto em perguntas armadilha | Recall de chunks críticos |
| EVAL-03 | Coerência da resposta | Resposta aderente ao documento oficial | Nota rubrica v2 |

---

## 9. Testes de Geração (LLM)

| Caso | Pergunta | Resultado esperado | Guardrails | Status |
|------|----------|--------------------|------------|--------|
| LLM-01 | Qual o SLA do cliente Platinum? | Informar inexistência do tier | 1, 2, 3, 4 | Planejado |
| LLM-02 | Posso devolver carga perigosa? | Negar no processo padrão e orientar Gestão de Riscos | 1, 2, 4 | Planejado |
| LLM-03 | Qual o multiplicador para o Sudeste? | Informar 1.1 da v2 com fonte | 1, 2, 4 | Planejado |

---

## 10. Teste Ponta a Ponta (E2E)

| Fluxo | Entrada | Saída esperada | Evidência | Status |
|-------|---------|----------------|-----------|--------|
| E2E-01 | Pergunta de devolução | Resposta correta com fonte e guardrails | Log de sessão | Planejado |
| E2E-02 | Pergunta de frete especial | Resposta com regra de cálculo correta | Log de sessão | Planejado |
| E2E-03 | Pergunta sem cobertura formal | Declarar ausência de cobertura | Log de sessão | Planejado |

---

## 11. Testes de Regressão

Regressão mínima obrigatória — reexecutar a cada mudança de prompt, chunking ou base documental:

| Caso | Cenário crítico | Versão anterior | Versão atual | Houve regressão? | Ação |
|------|-----------------|-----------------|--------------|------------------|------|
| REG-01 | Tier Platinum | A preencher | A preencher | A preencher | A preencher |
| REG-02 | Carga perigosa devolução | A preencher | A preencher | A preencher | A preencher |
| REG-03 | Sudeste 1.1 v2 | A preencher | A preencher | A preencher | A preencher |
| REG-04 | Frete 300kg sem cobertura | A preencher | A preencher | A preencher | A preencher |
| REG-05 | Conflito PROC-042 v1 x v2 | A preencher | A preencher | A preencher | A preencher |

---

## 12. Métricas e KPIs

| Métrica | Definição | Meta inicial |
|---------|-----------|--------------|
| Taxa de resposta com fonte válida | % de respostas com citação correta | ≥ 95% |
| Taxa de alucinação | % de respostas com informação inventada | ≤ 2% |
| Precision@K | Relevância dos Top-K chunks | ≥ 0.80 |
| Recall de chunks críticos | Recuperação de chunks obrigatórios | ≥ 0.90 |
| Tempo médio de resposta | Tempo total pergunta → resposta | ≤ 5s |
| Conformidade com guardrails | % de respostas aderentes aos 4 guardrails | ≥ 98% |

---

## 13. Evidências Esperadas

- Matriz CSV de rastreamento atualizada
- CSV de respostas avaliadas com rubrica v2 aplicada
- Log de execução por ciclo de regressão
