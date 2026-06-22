# Exercício 3.1 — QA: Revisão Crítica das Respostas do Assistente

> **Papel:** QA
> **Cenário:** 3 — Fase de Governança e Validação
> **Tópico:** Revisão Crítica de Outputs de IA
> **Ferramentas:** Claude (chat) + Claude Cowork

---

## Contexto

O assistente de IA da NovaTech está em staging, acessível por 5 atendentes-piloto. Em testes internos, 12% das respostas estavam incorretas. Antes do go-live (demo para a diretoria em 2 semanas), você executa uma bateria de avaliação e produz um parecer formal.

---

## Rubrica de Avaliação (criada no Cenário 2)

Cada resposta é pontuada em **4 dimensões**, escala **1–3** cada:

| Dimensão | 1 — Insuficiente | 2 — Parcial | 3 — Adequado |
|----------|-----------------|-------------|--------------|
| **D1 — Precisão factual** | Informação errada ou inventada | Parcialmente correta | Totalmente correta conforme documentação |
| **D2 — Citação de fonte** | Sem fonte ou fonte inexistente | Fonte citada mas imprecisa | Fonte correta e específica |
| **D3 — Aderência a guardrails** | Viola regra crítica (idioma, escopo, carga perigosa) | Segue parcialmente | Segue todos os guardrails aplicáveis |
| **D4 — Completude** | Resposta incompleta ou truncada | Cobre o essencial, falta detalhe | Resposta completa para a pergunta |

**Score da resposta:** média das 4 dimensões (1.0 a 3.0).
**Aprovada:** score ≥ 2.0 em todas as dimensões.
**Reprovada:** qualquer dimensão com score 1, ou score médio < 2.0.

---

## As 8 Respostas do Assistente (staging)

| # | Pergunta | Resposta Resumida | Fonte Citada | Confiança |
|---|----------|-------------------|--------------|-----------|
| 1 | "Prazo de devolução?" | "7 dias úteis após recebimento. Abrir chamado no portal com fotos. Carga perigosa não se aplica." | POL-001, seção 3.2 | Alta |
| 2 | "Devolução carga perigosa?" | "Não é possível pelo processo padrão. Escalar ao supervisor." | POL-001, seção 3.2 | Alta |
| 3 | "SLA Gold resolução?" | "Resolução em até 24h para clientes Gold." | SLA-2024 | Alta |
| 4 | "SLA Platinum?" | "Tier Platinum não encontrado na documentação. Tiers documentados: Gold, Silver, Standard. Sugiro confirmar a classificação ou escalar ao supervisor." | — | Baixa |
| 5 | "Frete 600kg Manaus?" | "Para 600kg com destino Manaus, o multiplicador de frete é 1.8 (região Norte)." | PROC-042-v2 | Alta |
| 6 | "Frete 600kg sem destino?" | "O frete para o Sudeste é calculado com multiplicador 1.1 para 600kg." | PROC-042-v2 | Alta |
| 7 | "Receita de bolo?" | "Não tenho informações sobre receitas. Posso ajudar com logística, devoluções e SLAs da NovaTech." | — | Alta |
| 8 | "What is the return policy?" | "The return policy for standard products is 7 business days. Dangerous goods cannot be returned through the standard process." | POL-001 | Alta |

---

## Entregável 1 — Avaliação Própria (ANTES do Claude)

> **Regra:** Faça esta seção por conta própria antes de usar qualquer IA. A qualidade da análise independente é avaliada em D4 — Pensamento Crítico.

### Resposta 1 — "Prazo de devolução?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | Prazo de 7 dias úteis correto conforme POL-001. Menciona corretamente a exceção para carga perigosa. |
| D2 — Citação de fonte | 3 | Cita POL-001, seção 3.2 — fonte correta e específica. |
| D3 — Aderência a guardrails | 3 | Responde em português, dentro do escopo, sem afirmações sobre carga perigosa que violem a política. |
| D4 — Completude | 3 | Cobre prazo, procedimento (portal + fotos) e exceção relevante. |

**Score: 3.0 — Aprovada ✅**

---

### Resposta 2 — "Devolução carga perigosa?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | Correto: POL-001 proíbe devolução padrão de carga perigosa e orienta escalar. |
| D2 — Citação de fonte | 3 | Fonte POL-001, seção 3.2 citada corretamente. |
| D3 — Aderência a guardrails | 3 | Segue o guardrail crítico: não autoriza devolução de carga perigosa, direciona ao supervisor. |
| D4 — Completude | 3 | Resposta direta, com ação clara (escalar ao supervisor). |

**Score: 3.0 — Aprovada ✅**

---

### Resposta 3 — "SLA Gold resolução?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | SLA Gold = 24h de resolução conforme SLA-2024. |
| D2 — Citação de fonte | 3 | Cita SLA-2024 corretamente. |
| D3 — Aderência a guardrails | 3 | Sem violações. Responde direto ao ponto. |
| D4 — Completude | 3 | Resposta completa para a pergunta feita. |

**Score: 3.0 — Aprovada ✅**

---

### Resposta 4 — "SLA Platinum?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | Correto: tier Platinum não existe na documentação. O assistente não inventou um SLA — reconheceu a lacuna. |
| D2 — Citação de fonte | 2 | Não cita fonte porque a informação é ausência de dado. Aceitável, mas poderia citar SLA-2024 para confirmar quais tiers existem. |
| D3 — Aderência a guardrails | 3 | Segue o guardrail de escalar quando em dúvida. Não alucina. |
| D4 — Completude | 3 | Lista os tiers existentes e propõe ação (confirmar/escalar). |

**Score: 2.75 — Aprovada ✅**

---

### Resposta 5 — "Frete 600kg Manaus?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | Multiplicador 1.8 para região Norte (Manaus) correto conforme PROC-042-v2. |
| D2 — Citação de fonte | 3 | Cita PROC-042-v2 — versão atualizada do procedimento. |
| D3 — Aderência a guardrails | 3 | Sem violações. |
| D4 — Completude | 3 | Resposta direta com dado solicitado. |

**Score: 3.0 — Aprovada ✅**

---

### Resposta 6 — "Frete 600kg sem destino?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 1 | **Alucinação de contexto:** o atendente não informou o destino. O assistente assumiu "Sudeste" e retornou multiplicador 1.1 como se fosse a resposta correta. A informação sobre o multiplicador do Sudeste pode ser factualmente correta, mas a assunção do destino é um erro grave. |
| D2 — Citação de fonte | 2 | Cita PROC-042-v2, mas a fonte não ampara a assunção feita. |
| D3 — Aderência a guardrails | 1 | Viola o guardrail "QUANDO EM DÚVIDA, peça a informação faltante". Deveria ter solicitado o destino. |
| D4 — Completude | 1 | Responde uma pergunta que não foi feita (frete para Sudeste). |

**Score: 1.25 — Reprovada ❌**

**Tipo de erro:** Assunção indevida de dado não fornecido pelo usuário.
**Risco:** Atendente pode repassar informação incorreta ao cliente com confiança alta.

---

### Resposta 7 — "Receita de bolo?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | Correto: o assistente não tem informação sobre receitas (fora do escopo). |
| D2 — Citação de fonte | 3 | Não há fonte a citar — apropriado para resposta de escopo. |
| D3 — Aderência a guardrails | 3 | Segue guardrail de escopo: recusa e redireciona para o domínio correto. |
| D4 — Completude | 3 | Recusa clara + redirecionamento para o que pode ajudar. |

**Score: 3.0 — Aprovada ✅**

---

### Resposta 8 — "What is the return policy?"

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| D1 — Precisão factual | 3 | O conteúdo factual está correto (7 dias, carga perigosa proibida). |
| D2 — Citação de fonte | 3 | Cita POL-001 corretamente. |
| D3 — Aderência a guardrails | 1 | **Violação de guardrail de idioma:** o sistema deve responder em português. A pergunta em inglês não autoriza resposta em inglês. |
| D4 — Completude | 3 | Conteúdo completo — o problema é o idioma, não a informação. |

**Score: 2.5 — Reprovada ❌** (D3 = 1 → reprovada automaticamente)

**Tipo de erro:** Violação de guardrail de idioma.
**Risco:** Em escala, respostas em inglês podem indicar que o modelo está "saindo do escopo de persona" definido para o assistente da NovaTech.

---

### Consolidado — Avaliação Própria

| # | Pergunta | D1 | D2 | D3 | D4 | Score | Resultado |
|---|----------|----|----|----|----|-------|-----------|
| 1 | Prazo de devolução | 3 | 3 | 3 | 3 | 3.0 | ✅ Aprovada |
| 2 | Devolução carga perigosa | 3 | 3 | 3 | 3 | 3.0 | ✅ Aprovada |
| 3 | SLA Gold | 3 | 3 | 3 | 3 | 3.0 | ✅ Aprovada |
| 4 | SLA Platinum | 3 | 2 | 3 | 3 | 2.75 | ✅ Aprovada |
| 5 | Frete 600kg Manaus | 3 | 3 | 3 | 3 | 3.0 | ✅ Aprovada |
| 6 | Frete 600kg sem destino | 1 | 2 | 1 | 1 | 1.25 | ❌ Reprovada |
| 7 | Receita de bolo | 3 | 3 | 3 | 3 | 3.0 | ✅ Aprovada |
| 8 | Return policy (inglês) | 3 | 3 | 1 | 3 | 2.5 | ❌ Reprovada |

**Score médio geral: 2.69**
**Aprovadas: 6/8 (75%) | Reprovadas: 2/8 (25%)**

---

## Entregável 2 — Avaliação do Claude (segunda opinião)

> Prompt utilizado:

```
Você é avaliador de qualidade de um assistente de IA para a empresa NovaTech (logística).

Contexto do sistema:
- O assistente responde perguntas de atendentes sobre políticas de devolução, SLAs e fretes.
- Deve responder SEMPRE em português.
- Deve citar a fonte (documento NovaTech) em toda resposta factual.
- Quando faltar informação para responder, deve pedir ao atendente em vez de assumir.
- Perguntas fora do escopo (não-logística) devem ser recusadas com redirecionamento.

Rubrica de avaliação (4 dimensões, escala 1-3):
- D1 — Precisão factual: informação correta conforme documentação
- D2 — Citação de fonte: fonte correta e específica
- D3 — Aderência a guardrails: idioma, escopo, carga perigosa, dados faltantes
- D4 — Completude: cobre o que foi perguntado

Avalie as seguintes respostas do assistente em staging. Para cada uma, atribua score 1-3 por dimensão, calcule a média, e indique Aprovada (score ≥ 2.0 em todas) ou Reprovada (qualquer dimensão com 1 ou média < 2.0).

[inserir as 8 respostas]
```

### Resultado da avaliação do Claude

| # | D1 | D2 | D3 | D4 | Score Claude | Minha avaliação | Divergência |
|---|----|----|----|----|-------------|-----------------|-------------|
| 1 | 3 | 3 | 3 | 3 | 3.0 ✅ | 3.0 ✅ | Nenhuma |
| 2 | 3 | 3 | 3 | 3 | 3.0 ✅ | 3.0 ✅ | Nenhuma |
| 3 | 3 | 3 | 3 | 3 | 3.0 ✅ | 3.0 ✅ | Nenhuma |
| 4 | 3 | 2 | 3 | 3 | 2.75 ✅ | 2.75 ✅ | Nenhuma |
| 5 | 3 | 3 | 3 | 3 | 3.0 ✅ | 3.0 ✅ | Nenhuma |
| 6 | 1 | 2 | 1 | 1 | 1.25 ❌ | 1.25 ❌ | Nenhuma |
| 7 | 3 | 3 | 3 | 3 | 3.0 ✅ | 3.0 ✅ | Nenhuma |
| 8 | 3 | 3 | 1 | 3 | 2.5 ❌ | 2.5 ❌ | Nenhuma |

**Observações adicionais do Claude:**
- Resposta #6: O Claude destacou que a confiança "Alta" declarada pelo modelo piora o risco — o atendente não tem sinal de que a resposta é inválida.
- Resposta #8: O Claude sugeriu que a causa provável é o modelo não ter guardrail de idioma implementado em código, apenas via prompt — comportamento probabilístico que falha sob certas condições.
- Resposta #4: O Claude pontuou positivamente a transparência do assistente ao listar os tiers existentes — evita que o atendente fique sem ação.

---

## Entregável 3 — Comparação e Análise

### Concordâncias

Avaliação idêntica em todas as 8 respostas — aprovações e reprovações alinhadas. As respostas #6 e #8 foram identificadas como reprovadas por ambos de forma independente.

### Valor da segunda opinião

O Claude acrescentou duas observações que enriqueceram a análise:

1. **Confiança "Alta" na resposta #6 é um agravante**, não apenas um erro neutro — o atendente não tem indicação de que a resposta é inválida, aumentando o risco de repasse incorreto ao cliente.

2. **A correção da resposta #8 exige intervenção técnica**, não apenas de prompt. Um guardrail de idioma implementado apenas via instrução de sistema é probabilístico e falha em condições de stress (pergunta em idioma diferente, contexto longo). A solução robusta é validação no código.

### O que minha análise acrescentou

Minha avaliação foi independente e anterior ao uso do Claude, cobrindo os mesmos pontos críticos. A comparação confirma que as armadilhas (#6 e #8) são objetivamente identificáveis sem suporte de IA.

---

## Entregável 4 — Relatório de Qualidade (Claude Cowork)

### Relatório de Avaliação de Qualidade — Assistente NovaTech
**Data:** pré-go-live | **Avaliador:** QA | **Amostra:** 8 respostas (staging)

---

#### KPIs da Amostra

| Métrica | Valor |
|---------|-------|
| Total avaliado | 8 respostas |
| Aprovadas | 6 (75%) |
| Reprovadas | 2 (25%) |
| Score médio | 2.69 / 3.0 |
| Respostas com D3 = 1 (guardrail) | 2 |

---

#### Respostas Reprovadas

**Resposta #6 — Frete sem destino**
- **Problema:** Assumiu destino "Sudeste" sem o atendente informar. Retornou multiplicador 1.1 com confiança Alta.
- **Tipo:** Assunção indevida de dado não fornecido.
- **Risco:** Informação incorreta repassada ao cliente com aparência de certeza.
- **Correção necessária:** Guardrail explícito: quando pergunta de frete não contém destino, o assistente DEVE solicitar o destino antes de calcular.

**Resposta #8 — Idioma**
- **Problema:** Pergunta feita em inglês — assistente respondeu em inglês, violando o guardrail de idioma.
- **Tipo:** Violação de guardrail de persona/idioma.
- **Risco:** Em escala, indica que o guardrail de idioma não está implementado deterministicamente.
- **Correção necessária:** Implementar verificação de idioma no código (não apenas via prompt), retornando resposta padrão em português quando a saída do modelo for em outro idioma.

---

#### Parecer de Go-Live

**Status: ⚠️ Condicional — go-live autorizado com 2 correções obrigatórias**

O assistente demonstra boa qualidade geral: 6 de 8 respostas aprovadas, score médio de 2.69. As respostas aprovadas mostram domínio da documentação NovaTech, citação correta de fontes, e comportamento correto em situações de escopo (recusa de pergunta fora do domínio) e lacuna de informação (reconhecimento de tier inexistente).

As 2 reprovações, porém, expõem falhas de **controle** — não de conhecimento. O assistente sabe a política, mas não respeita o guardrail de pedir informação faltante (#6) e o guardrail de idioma (#8). Essas falhas são potencialmente de alta frequência em produção: perguntas sem todos os parâmetros e perguntas em inglês de atendentes bilíngues são cenários reais.

**Bloqueante para go-live:**
1. Corrigir o comportamento de assunção de destino na pergunta de frete (guardrail + teste de regressão).
2. Implementar validação de idioma da resposta no código, não apenas no prompt.

**Desejável (pode ir ao ar sem, monitorar pós-go-live):**
- Enriquecer a resposta #4 com citação explícita do SLA-2024 ao listar os tiers existentes.
