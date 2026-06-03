# Entregável QA 1.2 — Design de Critérios de Aceitação para Respostas de IA

## 1. Contexto

Este entregável define critérios objetivos para classificar respostas do assistente da NovaTech e aplica a avaliação ao lote de 5 respostas simuladas do exercício.

Fonte de verdade usada para validação:
- POL-001-politica-devolucao.md
- PROC-042-v2-frete-especial-revisado.md
- SLA-2024-tabela-sla-clientes.md
- anexo-a-documentacao-simulada-novatech.md

---

## 2. Avaliação manual inicial (antes da rubrica)

Classificação própria realizada antes de construir a rubrica com IA.

| # | Pergunta | Julgamento manual | Justificativa baseada em fonte |
|---|----------|-------------------|-------------------------------|
| 1 | Qual o prazo de devolução? | Correta | POL-001 define 7 dias úteis e exceções para cargas perigosas classes 1 a 6 — resposta alinhada |
| 2 | Quanto custa frete para 600kg para Manaus? | Correta | PROC-042-v2 aplica frete especial > 500kg; Norte com multiplicador 1.8 — sem invenção de valor absoluto |
| 3 | Qual o SLA do cliente Platinum? | **Incorreta** | SLA-2024 define apenas Gold, Silver e Standard; Platinum **não existe** — alucinação de tier e valores |
| 4 | Posso devolver carga perigosa? | **Incorreta** | POL-001 seção 3.2: cargas perigosas classes 1 a 6 NÃO são elegíveis no processo padrão — regra invertida |
| 5 | Qual o multiplicador de frete para o Sudeste? | Correta | PROC-042-v2 indica Sudeste = 1.1 — versão correta citada |

---

## 3. Rubrica de avaliação (4 dimensões, escala 1–3)

### D1 — Precisão Factual
- **1:** Contém erro material, alucinação ou inversão de regra
- **2:** Núcleo correto, mas com omissão relevante (ex.: cita prazo sem citar exceção crítica)
- **3:** Factual e semanticamente correta, alinhada à fonte oficial

### D2 — Citação e Rastreabilidade de Fonte
- **1:** Sem fonte, fonte errada ou não verificável
- **2:** Cita documento correto, mas seção insuficiente ou ambígua
- **3:** Cita documento e trecho/seção adequados (ex.: "POL-001 seção 3.2")

### D3 — Aderência a Guardrails
- **1:** Inventa informação, não declara ausência de cobertura, ou viola idioma/forma
- **2:** Parcialmente aderente, com pequenas falhas (ex.: cita fonte mas sem seção)
- **3:** Totalmente aderente aos 4 guardrails

### D4 — Completude e Utilidade para Atendimento
- **1:** Resposta incompleta ou não acionável pelo atendente
- **2:** Parcialmente acionável (responde o básico mas omite exceção relevante)
- **3:** Resposta completa, clara e aplicável imediatamente ao atendimento

**Score final:** média simples das 4 dimensões.

| Score médio | Classificação |
|-------------|---------------|
| 2.5 a 3.0 | Aprovada com alta confiança |
| 2.0 a 2.4 | Aprovada com ressalva |
| abaixo de 2.0 | Reprovada |

---

## 4. Template reutilizável de avaliação (formato Cowork)

Campos obrigatórios para rastreio operacional:

| ID | Pergunta | Resposta | Fonte citada | D1 Precisão | D2 Fonte | D3 Guardrails | D4 Completude | Média | Status | Observações |
|----|----------|----------|--------------|-------------|----------|---------------|---------------|-------|--------|-------------|
| QXX | ... | ... | ... | 1–3 | 1–3 | 1–3 | 1–3 | calc | Aprov/Reprov | ... |

Campos mínimos adicionais:
- Avaliador
- Data
- Versão do prompt
- Versão da base documental
- Ação corretiva recomendada

---

## 5. Aplicação da rubrica nas 5 respostas

| # | D1 | D2 | D3 | D4 | Média | Resultado |
|---|----|----|----|----|-------|-----------|
| 1 | 3 | 2 | 3 | 3 | 2.8 | Aprovada com alta confiança |
| 2 | 3 | 2 | 3 | 2 | 2.5 | Aprovada com alta confiança |
| 3 | 1 | 1 | 1 | 1 | 1.0 | Reprovada |
| 4 | 1 | 1 | 1 | 1 | 1.0 | Reprovada |
| 5 | 3 | 2 | 3 | 2 | 2.5 | Aprovada com alta confiança |

Justificativas das respostas críticas:
- **Resposta 3:** Reprovada por alucinação de tier inexistente (Platinum) e valores de SLA inventados. Violação de guardrails 2 e 3.
- **Resposta 4:** Reprovada por inversão explícita da regra da POL-001 seção 3.2 — carga perigosa não é elegível no processo padrão.

---

## 6. Verificação das armadilhas obrigatórias

| Armadilha | Identificada? | Justificativa |
|-----------|---------------|---------------|
| #3 — SLA Platinum | **Sim** | Tier Platinum não existe na SLA-2024 (apenas Gold, Silver e Standard) — corretamente marcada como incorreta |
| #4 — Devolução de carga perigosa | **Sim** | POL-001 seção 3.2 estabelece não elegibilidade — resposta do assistente inverteu a regra |

---

## 7. Ações de melhoria recomendadas

1. Incluir validação pré-resposta para tiers permitidos (Gold/Silver/Standard) — rejeitar qualquer menção a Platinum.
2. Incluir regra dura para exceções de devolução (carga perigosa não elegível no fluxo padrão).
3. Exigir estrutura mínima de citação com documento + seção (D2 = 3 requer "POL-001 seção X", não apenas "POL-001").
4. Em conflito documental entre versões, responder com transparência e solicitar validação de vigência.

---

## 8. Conclusão

O processo atende ao exercício 1.2:
- Avaliação manual realizada antes da construção da rubrica
- Rubrica objetiva com 4 dimensões e escala 1–3 descrita concretamente
- Template reutilizável para lotes futuros (não one-off)
- Aplicação completa às 5 respostas com justificativas baseadas nos documentos do Anexo A
- Identificação correta das duas armadilhas obrigatórias com referência explícita à fonte
