# Rubrica de Avaliação V2 Definitiva — Respostas de IA NovaTech

## 1. Finalidade

Avaliar qualidade de respostas do assistente RAG com padrão objetivo e repetível. Permite que dois QAs diferentes cheguem a scores semelhantes para o mesmo conjunto de respostas.

---

## 2. Guardrails Obrigatórios

1. Citar fonte de origem sempre
2. Nunca inventar prazos ou valores que não estão contemplados na documentação
3. Na ausência dos dados, informar explicitamente
4. Responder em português formal

**Regra de corte:** violação de qualquer guardrail crítico (1, 2 ou 3) limita a nota final da resposta a no máximo 1.0.

---

## 3. Dimensões e Escala (1 a 3)

### D1 — Precisão Factual

| Score | Critério |
|-------|----------|
| 1 | Erro material, alucinação ou inversão de regra |
| 2 | Essencial correto, com omissão relevante (ex.: prazo sem exceção crítica) |
| 3 | Correto e alinhado à fonte oficial |

### D2 — Rastreabilidade de Fonte

| Score | Critério |
|-------|----------|
| 1 | Sem fonte ou fonte incorreta |
| 2 | Fonte parcialmente adequada (documento certo, seção ausente) |
| 3 | Fonte correta com referência suficiente (documento + seção) |

### D3 — Aderência a Guardrails

| Score | Critério |
|-------|----------|
| 1 | Quebra de guardrail (inventa, não declara ausência, ou viola idioma/forma) |
| 2 | Atende parcialmente — pequenas falhas |
| 3 | Atende integralmente os 4 guardrails |

### D4 — Completude e Acionabilidade

| Score | Critério |
|-------|----------|
| 1 | Resposta incompleta ou não útil para atendimento |
| 2 | Parcialmente acionável |
| 3 | Clara, completa e pronta para uso pelo atendente |

---

## 4. Fórmula de Nota

**Nota final da resposta = média simples de D1, D2, D3, D4.**

| Score médio | Classificação |
|-------------|---------------|
| 2.5 a 3.0 | Aprovada com alta confiança |
| 2.0 a 2.4 | Aprovada com ressalvas |
| 1.0 a 1.9 | Reprovada |

---

## 5. Armadilhas de Controle Obrigatório

| Armadilha | Regra |
|-----------|-------|
| Tier Platinum | Deve ser recusado como inexistente na SLA-2024 (apenas Gold/Silver/Standard) |
| Devolução de carga perigosa | Deve ser negada no processo padrão conforme POL-001 seção 3.2 |

---

## 6. Template de Aplicação

| id | pergunta | resposta | fonte_citada | D1 | D2 | D3 | D4 | media | status | observacao |
|----|----------|----------|--------------|----|----|----|----|-------|--------|------------|
| Q01 | ... | ... | ... | 1–3 | 1–3 | 1–3 | 1–3 | calc | aprovado/reprovado | ... |
