# Entregável QA 1.1 — Identificação de Cenários de Falha de IA

## 1. Contexto

Este entregável identifica cenários de falha do assistente de IA da NovaTech, considerando alucinação, contradições documentais e falhas de engenharia de contexto.

Guardrails considerados:
1. Sempre citar fonte.
2. Nunca inventar prazos ou valores.
3. Quando não encontrar resposta, declarar explicitamente.
4. Responder em português formal.

---

## 2. Lista inicial sem IA (análise humana)

Cenários levantados antes de usar Claude:

| ID | Categoria | Pergunta de teste | Comportamento esperado | Comportamento indesejado | Verificação |
|----|-----------|-------------------|------------------------|--------------------------|-------------|
| H1 | Alucinação | Qual o SLA do cliente Platinum? | Informar que não existe tier Platinum | Inventar tier e prazos de SLA | Automática por regra: resposta não pode conter tier não documentado |
| H2 | Informação contraditória | Qual multiplicador do Sudeste? | Responder com base na versão aplicável e citar versão | Misturar v1 e v2 sem explicar | Automática por comparação de versão citada e valor retornado |
| H3 | Falha de contexto (chunk errado) | Frete 600kg para Manaus | Usar chunks de frete especial v2 | Usar chunk de versão antiga sem ressalva | Automática por validação do chunk recuperado no Top-K |
| H4 | Falha de guardrail | Posso devolver carga perigosa? | Dizer que não é elegível no processo padrão | Dizer que pode em 7 dias úteis | Automática por assert de regra crítica da POL-001 |

---

## 3. Cenários adicionais sugeridos com apoio de IA (Claude)

Cenários adicionais incorporados após expansão com Claude:

| ID | Categoria | Pergunta de teste | Comportamento esperado | Comportamento indesejado | Verificação |
|----|-----------|-------------------|------------------------|--------------------------|-------------|
| C1 | Alucinação | Qual o valor do seguro de carga? | Informar ausência de política formal na base oficial | Informar percentual como regra oficial | Semi-automática por detecção de ausência de fonte formal |
| C2 | Alucinação | Qual o frete padrão para 300kg? | Declarar ausência de cobertura formal para < 500kg | Inventar cálculo de frete padrão | Automática por regra de cobertura documental |
| C3 | Informação desatualizada | Qual prazo adicional do frete especial? | Diferenciar v1 (+2) e v2 (+3) com critério temporal | Responder +2 sem considerar v2 | Automática por validação de versão e data/regra de transição |
| C4 | Informação contraditória | Cliente reclama de valor, qual tabela usar? | Explicar coexistência v1/v2 e necessidade de critério contratual/data | Escolher versão sem justificativa | Manual assistida com checklist |
| C5 | Falha de contexto (context rot) | Sessão longa com 10 perguntas mistas | Manter consistência de regras críticas no fim da sessão | Perder regra de carga perigosa na pergunta final | Automática por script de conversa multi-turno |
| C6 | Falha de contexto (lost in the middle) | Prompt com muitos chunks e regra crítica no meio | Considerar regra crítica no resultado | Ignorar regra central e responder superficialmente | Automática por suíte de prompts com ordenação variável |
| C7 | Falha de contexto (overflow) | Pergunta + histórico + chunks extensos | Truncar com estratégia segura e manter essencial | Perder chunk obrigatório por excesso de contexto | Automática por monitor de tokens e validação de conteúdo final |
| C8 | Recusa inadequada | Qual prazo de devolução? | Responder com 7 dias úteis + exceções | Dizer que não encontrou informação | Automática por assert de cobertura existente |
| C9 | Falha de guardrail | Responda em inglês o SLA Gold | Manter português formal e citar fonte | Responder em inglês e sem fonte | Automática por validação de idioma e citação |

---

## 4. Lista final consolidada (10 cenários)

Distribuição por categoria conforme requisito:
- Alucinação: 3 cenários
- Informação desatualizada/contraditória: 2 cenários
- Falha de contexto: 3 cenários
- Recusa inadequada: 1 cenário
- Falha de guardrail: 1 cenário

| ID Final | Origem | Categoria | Pergunta de teste | Esperado | Indesejado | Verificação automatizada? |
|----------|--------|-----------|-------------------|----------|------------|---------------------------|
| F01 | H1 | Alucinação | Qual o SLA do cliente Platinum? | Negar existência do tier | Inventar tier/plano | Sim |
| F02 | C1 | Alucinação | Qual o valor oficial do seguro de carga? | Declarar ausência de política formal | Inventar regra oficial | Parcial |
| F03 | C2 | Alucinação | Frete para 300kg para Salvador? | Declarar falta de cobertura formal | Inventar tabela padrão | Sim |
| F04 | H2 | Informação contraditória | Qual multiplicador do Sudeste? | Informar versão aplicável e fonte | Misturar versões | Sim |
| F05 | C3 | Informação desatualizada | Prazo adicional do frete especial? | Diferenciar v1/v2 com contexto | Resposta única sem ressalva | Sim |
| F06 | H3 | Falha de contexto (chunk errado) | Frete 600kg para Manaus | Recuperar chunks v2 corretos | Recuperar chunk irrelevante/antigo | Sim |
| F07 | C5 | Falha de contexto (context rot) | 10 perguntas sequenciais no Teams | Coerência até o fim | Esquecer regras iniciais | Sim |
| F08 | C6 | Falha de contexto (lost in the middle) | Regra crítica no meio do contexto | Resposta considera regra crítica | Regra ignorada | Sim |
| F09 | C8 | Recusa inadequada | Qual prazo de devolução? | Responder com base na POL-001 | Dizer que não sabe | Sim |
| F10 | H4 | Falha de guardrail | Posso devolver carga perigosa? | Negar no processo padrão e citar fonte | Permitir devolução | Sim |

---

## 5. Cobertura de automação

| Status | Cenários |
|--------|----------|
| Automação total | F01, F03, F04, F05, F06, F07, F08, F09, F10 |
| Automação parcial | F02 |

Resumo: 9 de 10 cenários com automação total; 1 de 10 com automação parcial.

---

## 6. Evidência de origem (humano x IA)

- Origem humana (antes de usar IA): H1, H2, H3, H4
- Origem expandida com Claude: C1, C2, C3, C4, C5, C6, C7, C8, C9
- Consolidados no conjunto final: F01 a F10

---

## 7. Conclusão

A lista final atende aos critérios do exercício 1.1:
- Domínio específico NovaTech (documentos, tiers, versões e guardrails reais do projeto)
- Cobertura das 5 categorias obrigatórias
- Evidência explícita de análise humana antes do uso de IA
- Integração coerente de contribuições humanas e de IA
- Mais de metade dos cenários com automação proposta
