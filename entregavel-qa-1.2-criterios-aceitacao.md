# Entregável QA — Exercício 1.2
## Design de Critérios de Aceitação para Respostas de IA
### Trilha AI First DGS | Cenário 1 | Papel: QA

---

## Parte 1 — Avaliação Manual (elaborada antes da rubrica)

Avaliação das 5 respostas com base na documentação oficial do Anexo A, antes de criar a rubrica formal.

---

### Resposta 1

**Pergunta:** "Qual o prazo de devolução?"  
**Resposta do assistente:** "O prazo é de 7 dias úteis, exceto para cargas perigosas classes 1 a 6 da ANTT."  
**Fonte citada:** POL-001, seção 3.2

**Avaliação:** ✅ **CORRETA** (com observação menor)

**Justificativa:** A informação está correta — o prazo geral de 7 dias úteis está em POL-001 seção 3.1 e a exceção para cargas perigosas em POL-001 seção 3.2. A citação da fonte está parcialmente correta (menciona o documento certo), mas aponta apenas para a seção 3.2 (exceções) quando o prazo geral vem da seção 3.1. Isso é uma imprecisão de citação, não um erro factual. A resposta é suficiente para orientar o atendente.

---

### Resposta 2

**Pergunta:** "Quanto custa frete para 600kg para Manaus?"  
**Resposta do assistente:** "O frete especial para cargas acima de 500kg para a região Norte tem multiplicador de 1.8 sobre o valor base."  
**Fonte citada:** PROC-042-v2, seção 2

**Avaliação:** ✅ **CORRETA**

**Justificativa:** O multiplicador para a região Norte (que inclui Manaus/AM) na versão vigente PROC-042-v2 é de fato 1.8 (seção 2.1). A carga de 600kg está na faixa de frete especial (>500kg). O assistente usou a versão correta do documento. A resposta é factualmente precisa, cita a fonte correta e usa a versão mais recente. Uma melhoria possível seria mencionar o fator de peso (1.0 para 500–1.000kg) para dar ao atendente a fórmula completa.

---

### Resposta 3

**Pergunta:** "Qual o SLA do cliente Platinum?"  
**Resposta do assistente:** "O cliente Platinum tem resposta em até 1h e resolução em até 12h."  
**Fonte citada:** SLA-2024

**Avaliação:** ❌ **INCORRETA — Alucinação de tier e valores**

**Justificativa:** O tier "Platinum" **não existe** na NovaTech. O documento SLA-2024 seção 1 é explícito: "Não existem outros tiers além dos três listados acima" (Gold, Silver, Standard). O assistente alucionou tanto o tier quanto os valores de SLA (1h/12h não correspondem a nenhum tier real). Ao citar "SLA-2024" como fonte, cria falsa impressão de legitimidade para uma informação inventada. Esta é a falha mais grave dos 5 casos — um atendente que confiar nessa resposta passará informação contratualmente incorreta ao cliente.

---

### Resposta 4

**Pergunta:** "Posso devolver carga perigosa?"  
**Resposta do assistente:** "Sim, cargas perigosas podem ser devolvidas em até 7 dias úteis."  
**Fonte citada:** POL-001, seção 3.2

**Avaliação:** ❌ **INCORRETA — Inversão de regra**

**Justificativa:** POL-001 seção 3.2 é a seção de **exceções**, e a primeira exceção listada é exatamente que cargas perigosas **NÃO** são elegíveis para devolução pelo processo padrão. O assistente inverteu completamente a regra — o documento que ele cita diz exatamente o contrário do que ele afirmou. Para cargas perigosas, o procedimento correto é encaminhar ao setor de Gestão de Riscos (ramal 4500). Esta é uma falha crítica de inversão de regra, agravada pelo fato de que a fonte citada contradiz a resposta.

---

### Resposta 5

**Pergunta:** "Qual o multiplicador de frete para o Sudeste?"  
**Resposta do assistente:** "O multiplicador regional para o Sudeste é 1.1."  
**Fonte citada:** PROC-042-v2, seção 2

**Avaliação:** ✅ **CORRETA**

**Justificativa:** O multiplicador para o Sudeste na PROC-042-v2 (versão vigente) é 1.1. O assistente usou a versão correta (v2) em vez da desatualizada (v1, que teria 1.0). A citação está correta. Esta resposta demonstra que o pipeline está recuperando e usando o documento mais recente para perguntas sobre multiplicadores regionais.

---

## Parte 2 — Rubrica de Avaliação (elaborada com o Claude)

Ver arquivo separado: **`rubrica-avaliacao-v2-definitiva.md`**

A rubrica resultante tem 4 dimensões (Precisão Factual, Citação de Fonte, Aderência aos Guardrails, Completude), escala 1–3, com critérios objetivos derivados dos documentos da NovaTech.

---

## Parte 3 — Pontuações Aplicadas às 5 Respostas

| # | Pergunta | D1 Precisão | D2 Fonte | D3 Guardrails | D4 Completude | Score Final | Aprovação |
|---|----------|-------------|----------|---------------|---------------|-------------|-----------|
| 1 | Prazo de devolução | 3 | 2 | 3 | 2 | **2.5** | ✅ Aprovada |
| 2 | Frete 600kg para Manaus | 3 | 3 | 3 | 2 | **2.75** | ✅ Aprovada |
| 3 | SLA cliente Platinum | 1 | 1 | 1 | 1 | **1.0** | ❌ Reprovada |
| 4 | Devolução carga perigosa | 1 | 1 | 1 | 1 | **1.0** | ❌ Reprovada |
| 5 | Multiplicador Sudeste | 3 | 3 | 3 | 3 | **3.0** | ✅ Aprovada |

---

## Análise dos Resultados

**Respostas aprovadas (3/5):** Respostas 1, 2 e 5 estão factualmente corretas. A Resposta 1 tem imprecisão menor de citação (seção errada); a Resposta 2 poderia incluir o fator de peso para ser mais completa.

**Respostas reprovadas (2/5):** 
- **Resposta 3** (Platinum): alucinação pura — tier inventado, valores inventados, citação falsa. Risco contratual alto.
- **Resposta 4** (carga perigosa): inversão de regra com citação real — o documento citado contradiz diretamente a resposta. Risco operacional e de segurança.

**Padrão identificado:** O assistente comete os erros mais graves justamente quando a documentação é contra-intuitiva (exceção que proíbe, ao invés de permitir) ou quando a informação simplesmente não existe na base (tier inexistente). Esses são os casos que precisam de guardrails mais fortes no pipeline.

---

## Recomendações para o Pipeline

1. Adicionar instrução explícita no system prompt: "Se um tier de cliente não constar em SLA-2024, informe que não existe e liste os tiers disponíveis."
2. Adicionar instrução: "Para perguntas sobre elegibilidade de devolução, sempre verifique a seção 3.2 de POL-001 antes de confirmar que a carga pode ser devolvida."
3. Implementar verificação determinística pós-geração: checar se a resposta contém termos como "Platinum" ou "Premium" que não existem na base.
