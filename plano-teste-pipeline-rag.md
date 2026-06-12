# Plano de Testes — Pipeline de RAG NovaTech
## Versão 1.0
### Trilha AI First DGS | QA — Exercício 1.3

---

## 1. Visão Geral

Este documento descreve o plano de testes para o pipeline de RAG que alimentará o assistente de IA de atendimento da NovaTech. O pipeline segue a arquitetura:

```
Fontes (SharePoint, Confluence, planilhas)
    ↓ Extração e normalização
    ↓ Chunking
    ↓ Embedding (Azure AI Search)
    ↓ Armazenamento vetorial
                                    ← Pergunta do atendente
                                    ← Embedding da pergunta
                                    ← Busca por similaridade
                                    ← Chunks recuperados
                                    ← Montagem de contexto
                                    ← Geração (LLM)
                                    → Resposta com citação
```

**Premissa fundamental:** Testes de pipelines de IA não são binários. Um resultado pode ser parcialmente correto, correto com a versão errada do documento, ou correto mas incompleto. A rubrica de qualidade substitui o pass/fail simples em testes de geração.

---

## 2. Escopo dos Testes

### 2.1 Fontes indexadas (in-scope)
- POL-001: Política de Devolução v3.1
- PROC-042-v2: Frete Especial (versão vigente)
- SLA-2024: Tabela de SLA por Cliente
- FAQ-Atendimento (com marcação de "fonte informal")

### 2.2 Fontes em atenção especial
- PROC-042-v1: Deve ser indexado apenas como referência histórica com metadado `vigente: false` e `substituído_por: PROC-042-v2`

### 2.3 Out-of-scope
- Frete padrão (<500kg): sem documentação disponível — testes devem verificar que o assistente reconhece o gap
- PROC-088 (interceptação de carga), PROC-043 (frete de cargas perigosas): não disponíveis na base atual

---

## 3. Categorias de Teste

### Categoria 1 — Testes de Ingestão

**Objetivo:** Verificar que documentos foram corretamente extraídos, convertidos e indexados.

| ID | Caso de Teste | Entrada | Resultado Esperado | Método de Verificação |
|----|--------------|---------|-------------------|----------------------|
| ING-01 | Ingestão completa dos 5 documentos | 5 arquivos .md do Anexo A | Todos os 5 documentos presentes no índice com metadados corretos (nome, versão, data) | Query no Azure AI Search por documento; contar documentos retornados |
| ING-02 | Contagem de chunks por documento | Base indexada | POL-001: 4+ chunks; PROC-042-v2: 5+ chunks; SLA-2024: 5+ chunks | Query por filtro de fonte; contar chunks |
| ING-03 | Metadado de versão em documentos contraditórios | PROC-042-v1 e v2 indexados | PROC-042-v1 tem metadado `vigente: false`; PROC-042-v2 tem `vigente: true` | Inspecionar metadados dos chunks de cada versão |
| ING-04 | Atualização incremental | Novo documento adicionado à base | Documento disponível no índice em até 24h (requisito de atualização) | Adicionar documento de teste; medir tempo até disponibilidade |
| ING-05 | Preservação de tabelas | Documento com tabelas (SLA-2024, PROC-042) | Valores das tabelas preservados como texto estruturado nos chunks | Verificar se chunk contém "Gold", "Silver", "1.8", "1.3" etc. conforme o documento original |

---

### Categoria 2 — Testes de Retrieval

**Objetivo:** Verificar que os chunks corretos são recuperados para perguntas conhecidas.

Baseado no mapa de cobertura do Anexo B:

| ID | Pergunta | Chunks que DEVEM ser retornados | Chunks que NÃO devem aparecer no topo | Verificação |
|----|----------|--------------------------------|---------------------------------------|------------|
| RET-01 | "Qual o prazo de devolução?" | POL-001-A (seção 3.1), POL-001-B (seção 3.2) | Qualquer chunk de PROC ou SLA no top-1 | Top-3 chunks contém POL-001-A |
| RET-02 | "Posso devolver carga perigosa?" | POL-001-B (exceções) | FAQ-03 no top-1 (fonte informal) | Top-1 é POL-001-B, não FAQ |
| RET-03 | "Qual o SLA do cliente Gold?" | SLA-2024-B (chamados gerais), SLA-2024-C (incidentes críticos) | Qualquer chunk de PROC | Top-2 contém ambos os chunks de SLA |
| RET-04 | "Qual o SLA do cliente Platinum?" | SLA-2024-A (contém "não existem outros tiers") | Nenhum chunk que "invente" Platinum | Top-1 é SLA-2024-A com a negação explícita |
| RET-05 | "Frete para 600kg para Manaus?" | PROC-042v2-B (multiplicadores), PROC-042v2-A (fórmula) | PROC-042-B (versão antiga) no top-1 | Top-2 são chunks v2; v1 não aparece antes deles |
| RET-06 | "Qual o frete para 300kg para Salvador?" | Nenhum chunk relevante (gap documentado) | PROC-042v2-B inventando cobertura para <500kg | Pipeline retorna zero chunks ou chunks com score de similaridade abaixo do threshold |
| RET-07 | "O que acontece com carga danificada?" | FAQ-38 (único com essa informação) | Nenhum — mas deve indicar que é fonte informal | FAQ-38 retornado com metadado de fonte informal visível |

---

### Categoria 3 — Testes de Geração

**Objetivo:** Verificar que o LLM gera respostas adequadas dado os chunks corretos.

| ID | Cenário | Chunks fornecidos | Resposta Esperada | Verificação com Rubrica |
|----|---------|-------------------|-------------------|------------------------|
| GER-01 | Resposta correta com chunk correto | POL-001-A, POL-001-B | "7 dias úteis; cargas perigosas não elegíveis — contatar ramal 4500" | D1≥3, D2≥2, D3=3, D4≥2 |
| GER-02 | Resposta com documento contraditório | PROC-042-v1-B e PROC-042v2-B juntos | Usar v2 (1.3 Sul, 1.8 Norte etc.) e indicar qual versão foi usada | D1≥2 (usa versão correta), D2=3 |
| GER-03 | Pergunta sem chunk relevante | Nenhum chunk recuperado | "Não encontrei informação sobre esse tema na base. Recomendo contato com [área específica]." | D3=3 (guardrail de "não inventar" respeitado) |
| GER-04 | Pergunta com chunk de fonte informal | FAQ-38 (carga danificada) | Responder com a informação E indicar que a fonte é informal/não-validada | D2≥2, D3≥2 (menciona limitação da fonte) |

---

### Categoria 4 — Testes de Contexto

**Objetivo:** Verificar que o gerenciamento de contexto não degrada a qualidade das respostas.

| ID | Cenário | Método | Resultado Esperado | Como Verificar |
|----|---------|--------|-------------------|----------------|
| CTX-01 | Context rot — sessão longa | Fazer 8 perguntas consecutivas no Teams; comparar resposta 1 com resposta 8 para a mesma pergunta | Resposta 8 tão precisa quanto resposta 1 | Diferença de score (rubrica) entre respostas 1 e 8 deve ser ≤0.3 |
| CTX-02 | Lost in the middle | Montar contexto com chunk correto na posição central (3ª de 5 chunks); fazer pergunta que depende desse chunk | Resposta usa o chunk central corretamente | Verificar se a informação do chunk central está presente na resposta |
| CTX-03 | Orçamento de contexto | Montar prompt máximo (system prompt + 10 chunks + histórico de 5 turnos) | Nenhum chunk é truncado silenciosamente; prompt total dentro do limite do modelo | Medir tokens totais antes de enviar; verificar se último chunk está íntegro |
| CTX-04 | Pergunta multi-domínio | "Qual o SLA e o procedimento de devolução para cliente Gold com carga de 700kg?" | Resposta combina SLA-2024 (SLA Gold) + POL-001 (devolução) + PROC-042-v2 (frete especial) corretamente | Verificar se os 3 domínios estão presentes na resposta com fontes corretas |

---

### Categoria 5 — Testes de Ponta a Ponta

**Objetivo:** Verificar o fluxo completo desde a pergunta até a resposta apresentada ao atendente.

| ID | Pergunta | Resultado Esperado E2E | Score Mínimo Aceitável |
|----|----------|------------------------|----------------------|
| E2E-01 | "Qual o prazo de devolução?" | "7 dias úteis após recebimento. Exceção: cargas perigosas classes 1–6 ANTT — contatar ramal 4500. Fonte: POL-001, seção 3.1 e 3.2." | 2.5 |
| E2E-02 | "Qual o SLA do cliente Platinum?" | "O tier Platinum não existe na NovaTech. Os tiers disponíveis são Gold, Silver e Standard. Fonte: SLA-2024, seção 1." | 2.5 |
| E2E-03 | "Frete especial para 1.500kg para Porto Alegre (Sul)?" | Fórmula com multiplicador 1.3 (Sul, v2) e fator de peso 1.15 (1.001–3.000kg, v2). Fonte: PROC-042-v2. | 2.5 |
| E2E-04 | "Qual o frete para 400kg para Curitiba?" | "Não encontrei informações sobre frete para cargas abaixo de 500kg na base disponível. Recomendo contato com o Comercial." | 2.5 |
| E2E-05 | "Posso devolver carga perigosa?" | "Não. Cargas perigosas não são elegíveis para devolução pelo processo padrão (POL-001, seção 3.2). Contatar Gestão de Riscos — ramal 4500." | 2.5 |

---

### Categoria 6 — Testes de Regressão

**Objetivo:** Verificar que mudanças no prompt ou na base não degradam respostas que antes estavam corretas.

| Gatilho | Testes de Regressão a Executar |
|---------|-------------------------------|
| Atualização do system prompt | E2E-01 a E2E-05 completos |
| Adição de novo documento à base | RET-01 a RET-07 + E2E para o domínio do novo documento |
| Atualização de documento existente (ex: nova versão da PROC-042) | RET-05, RET-06, GER-02, E2E-03 |
| Mudança no modelo de LLM | Todos os testes E2E + CTX-01 a CTX-04 |
| Mudança no modelo de embedding | Todos os testes RET + E2E-01 a E2E-05 |

---

## 4. Critérios de Aceite do Pipeline

| Métrica | Valor Mínimo Aceitável |
|---------|----------------------|
| Taxa de recuperação correta (top-1 é o chunk esperado) | ≥ 85% |
| Score médio E2E (rubrica 4D) | ≥ 2.5 |
| Taxa de respostas reprovadas (<2.0) em lote aleatório | ≤ 5% |
| Tempo de resposta (pergunta → resposta) | ≤ 10 segundos |
| Tempo de ingestão de novo documento até disponibilidade | ≤ 24 horas |
| Taxa de violação de guardrails (D3=1) | 0% em testes conhecidos |

---

## 5. Ambientes e Responsabilidades

| Ambiente | Finalidade | Responsável |
|----------|-----------|-------------|
| Dev | Testes de ingestão e retrieval durante desenvolvimento | Dev |
| QA | Testes de geração, contexto e E2E | QA |
| Staging | Testes de regressão antes de releases | QA + Tech Lead |
| Produção | Monitoramento contínuo (amostragem 5% dos chamados) | QA + Operações |
