# Rubrica de Avaliação de Respostas do Assistente de IA — NovaTech
## Versão 2.0 (Definitiva)
### Trilha AI First DGS | QA — Exercício 1.2

---

## Instruções de Uso

Esta rubrica avalia a qualidade de respostas geradas pelo assistente de IA da NovaTech em 4 dimensões. Cada dimensão recebe um score de 1 a 3. O score final é a **média das 4 dimensões** (1.0 a 3.0).

**Score final ≥ 2.5** → Resposta aprovada para uso em produção  
**Score final 2.0–2.4** → Aprovada com ressalvas — verificação humana recomendada  
**Score final < 2.0** → Reprovada — não deve ser apresentada ao cliente sem revisão

---

## Dimensão 1 — Precisão Factual

*A informação contida na resposta é correta e consistente com os documentos oficiais da NovaTech?*

| Score | Descrição |
|-------|-----------|
| **3** | Todas as informações estão corretas e alinhadas com os documentos oficiais (POL, PROC, SLA). Nenhum valor, prazo ou regra foi distorcido. Quando há documentos contraditórios na base (ex: PROC-042 v1 vs v2), a versão correta/mais recente foi utilizada. |
| **2** | A maior parte das informações está correta, mas há imprecisão em detalhe não-crítico (ex: citar a seção errada do documento correto, ou omitir uma condição de exceção). A informação central responde à pergunta sem causar dano ao cliente. |
| **1** | Contém pelo menos uma informação incorreta ou fabricada que, se usada pelo atendente, levaria a erro no atendimento. Ex: afirmar que carga perigosa pode ser devolvida, inventar valores de SLA para tier inexistente, citar multiplicador da versão desatualizada como definitivo. |

---

## Dimensão 2 — Citação de Fonte

*A resposta indica a origem da informação de forma verificável?*

| Score | Descrição |
|-------|-----------|
| **3** | Cita explicitamente o documento de origem com identificador (ex: "POL-001, seção 3.2" ou "SLA-2024, seção 2"). A citação é correta — o documento e a seção referenciados realmente contêm a informação citada. |
| **2** | Cita o documento mas sem número de seção, ou com seção imprecisa (ex: cita "POL-001" sem especificar a seção). A citação é verificável, mas exige esforço adicional do atendente. |
| **1** | Não cita qualquer fonte, ou cita um documento que não contém a informação fornecida (citação fantasma). Viola guardrail básico do assistente. |

---

## Dimensão 3 — Aderência aos Guardrails

*A resposta respeita todas as regras de comportamento definidas para o assistente?*

Guardrails vigentes:
1. Sempre citar fonte
2. Nunca inventar prazos ou valores não documentados
3. Quando não encontrar resposta, dizer explicitamente e sugerir escalação
4. Responder em português formal e acessível

| Score | Descrição |
|-------|-----------|
| **3** | Todos os guardrails são respeitados. Quando a informação não está disponível na base, o assistente comunica explicitamente a limitação e sugere o próximo passo (ex: "não encontrei essa informação — recomendo contato com o Comercial"). |
| **2** | Respeita a maioria dos guardrails, mas viola um de forma não-crítica (ex: linguagem levemente informal, ou omissão de sugestão de escalação em caso de ausência de resposta). |
| **1** | Viola guardrail crítico: inventa prazo/valor não documentado, responde em idioma errado, ou omite completamente a citação de fonte em resposta factual. |

---

## Dimensão 4 — Completude

*A resposta endereça de forma suficiente o que o atendente precisa para resolver o chamado?*

| Score | Descrição |
|-------|-----------|
| **3** | A resposta cobre todos os aspectos relevantes da pergunta, incluindo exceções e condições que impactam o caso do cliente. O atendente consegue usar a resposta diretamente sem buscar informação adicional. Ex: para pergunta sobre devolução, menciona prazo geral E exceções para carga perigosa. |
| **2** | A resposta cobre o ponto principal da pergunta mas omite informação complementar relevante (ex: menciona o prazo mas não o procedimento de abertura de chamado). O atendente consegue responder ao cliente, mas pode precisar de uma busca adicional. |
| **1** | A resposta é incompleta de forma que o atendente não consegue resolver o chamado sem busca adicional significativa. Ex: responde "depende do tipo de carga" sem especificar quais tipos ou o que verificar. |

---

## Planilha de Score

Use a tabela abaixo para registrar a avaliação de cada resposta:

| # | Pergunta (resumo) | D1 Precisão | D2 Fonte | D3 Guardrails | D4 Completude | Score Final | Aprovação |
|---|-------------------|-------------|----------|---------------|---------------|-------------|-----------|
| 1 | | | | | | | |
| 2 | | | | | | | |
| 3 | | | | | | | |
| 4 | | | | | | | |
| 5 | | | | | | | |

**Score Final** = (D1 + D2 + D3 + D4) / 4

**Aprovação:** ✅ Aprovada (≥2.5) | ⚠️ Com ressalvas (2.0–2.4) | ❌ Reprovada (<2.0)

---

## Histórico de Versões

| Versão | Data | Mudança |
|--------|------|---------|
| v1.0 | Inicial | Rubrica com 3 dimensões: precisão, fonte, completude |
| v2.0 | Atual | Adicionada Dimensão 3 (Guardrails) separada; revisão dos critérios de D1 para incluir contradições entre versões de documentos; refinamento das descrições de score 2 |
