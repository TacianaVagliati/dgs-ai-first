# Entregável QA — Exercício 1.3
## Plano de Testes para Pipeline de RAG
### Trilha AI First DGS | Cenário 1 | Papel: QA

---

## Resumo Executivo

Este entregável apresenta o plano de testes para o pipeline de RAG do assistente de IA da NovaTech, cobrindo as 6 categorias exigidas: ingestão, retrieval, geração, contexto, ponta a ponta e regressão.

O plano foi desenvolvido em conjunto com o Claude, que ajudou a expandir os testes de contexto e a estruturar os critérios de aceite. O documento de referência completo está em `plano-teste-pipeline-rag.md`.

---

## Decisões de Design do Plano

### Por que testes não são binários neste pipeline?

Um pipeline de RAG tem 3 camadas que podem falhar independentemente: **retrieval** (recuperou o chunk certo?), **montagem** (o contexto foi montado corretamente?) e **geração** (o LLM interpretou o contexto corretamente?). Uma resposta "errada" pode ser resultado de falha em qualquer camada, e a severidade é diferente:

- Chunk errado recuperado → falha de retrieval (correto no prompt, errado na busca)
- Chunk certo mas resposta errada → falha de geração ou instrução (correto na busca, errado na saída)
- Resposta correta mas sem citação → falha de guardrail (correto no conteúdo, errado no formato)

Tratar tudo como pass/fail mascara a causa raiz. A rubrica de 4 dimensões permite diagnosticar em qual camada o problema ocorreu.

---

## Resumo das 6 Categorias

### 1. Testes de Ingestão (5 casos — ING-01 a ING-05)

Verificam que os documentos foram extraídos, chunkeados e indexados corretamente. Destaque para **ING-03** (metadado de versão): o PROC-042-v1 deve ser marcado como `vigente: false` para que o pipeline saiba priorizar a v2.

### 2. Testes de Retrieval (7 casos — RET-01 a RET-07)

Baseados no mapa de cobertura do Anexo B. Casos críticos:
- **RET-04:** Para "cliente Platinum", o chunk retornado deve ser SLA-2024-A (que contém a negação explícita do tier)
- **RET-05:** Para frete >500kg, deve retornar chunks da v2, não da v1
- **RET-06:** Para frete <500kg, deve retornar zero chunks — gap documentado

### 3. Testes de Geração (4 casos — GER-01 a GER-04)

Testam o LLM isolado dos problemas de retrieval (chunks fornecidos diretamente). Caso crítico: **GER-04** testa se o assistente sinaliza quando a fonte é o FAQ informal.

### 4. Testes de Contexto (4 casos — CTX-01 a CTX-04)

Categoria mais técnica. Testa:
- **CTX-01 (Context rot):** Sessão de 8 perguntas — resposta 8 deve ter qualidade similar à resposta 1
- **CTX-02 (Lost in the middle):** Chunk correto no meio do contexto deve ser utilizado
- **CTX-03 (Orçamento):** Prompt máximo não pode truncar chunks silenciosamente
- **CTX-04 (Multi-domínio):** Pergunta que cruza SLA + devolução + frete deve usar os 3 documentos

### 5. Testes de Ponta a Ponta (5 casos — E2E-01 a E2E-05)

Fluxo completo: pergunta real → resposta com score mínimo 2.5. Os 5 casos cobrem os cenários mais críticos identificados na análise de documentação:
- E2E-02 testa explicitamente a alucinação de tier Platinum
- E2E-05 testa a inversão de regra de carga perigosa

### 6. Testes de Regressão

Acionados por 5 gatilhos: atualização de prompt, novo documento, atualização de documento, mudança de LLM, mudança de embedding. Cada gatilho tem sua própria lista de testes a executar.

---

## Critérios de Aceite Resumidos

| Métrica | Mínimo |
|---------|--------|
| Retrieval correto (top-1) | ≥ 85% |
| Score médio E2E | ≥ 2.5 |
| Taxa de respostas reprovadas | ≤ 5% |
| Tempo de resposta | ≤ 10s |
| Violação de guardrail (D3=1) em testes conhecidos | 0% |

---

## Reflexão: IA como par de revisão no desenvolvimento do plano

O uso do Claude para expandir o plano foi especialmente útil nos **testes de contexto** (Categoria 4). A lista inicial não cobria context rot e lost in the middle — o Claude identificou esses casos e propôs métodos de verificação (comparar resposta 1 vs resposta 8 em sessão longa, verificar se chunk central é utilizado). 

Em contrapartida, os testes de negócio (RET-04 para Platinum, E2E-05 para carga perigosa) foram identificados de forma independente — o conhecimento dos documentos da NovaTech foi determinante para detectar as armadilhas específicas do domínio.

---

## Artefato Estruturado

Ver arquivo `plano-teste-pipeline-rag.md` para o plano completo com tabelas de casos de teste, critérios e responsabilidades.

Ver arquivo `matriz-rastreamento-testes-rag.csv` para rastreamento de execução.
