# Documentação Consolidada — Prática 1 (NovaTech)

## 1. Objetivo

Este documento consolida e organiza o conteúdo da pasta Prática 1 para servir como referência única de consulta para exercícios e avaliações.

Escopo da consolidação:
- Política de devolução
- Procedimentos de frete especial (v1 e v2)
- Regras de SLA por tier de cliente
- FAQ operacional informal
- Chunks de referência para simulação de RAG
- Contexto pedagógico do exercício da fase 1

---

## 2. Fontes Consideradas

| Arquivo | Tipo | Normativo? |
|---------|------|------------|
| POL-001-politica-devolucao.md | Política | Sim |
| PROC-042-frete-especial-v1.md | Procedimento | Sim (vigência a validar) |
| PROC-042-v2-frete-especial-revisado.md | Procedimento | Sim (versão mais recente) |
| SLA-2024-tabela-sla-clientes.md | Tabela contratual | Sim |
| FAQ-atendimento.md | FAQ informal | Não |
| anexo-a-documentacao-simulada-novatech.md | Consolidado referência | Referência |
| anexo-b-chunks-referencia-rag.md | Gabarito de chunks | Referência de teste |
| exercicio-fase-1-entendimento.md | Enunciado pedagógico | Não |

---

## 3. Hierarquia de Confiabilidade Recomendada

Para uso em atendimento e decisões operacionais, aplicar a seguinte prioridade:

1. Documento normativo/contratual oficial vigente (POL, SLA, PROC com vigência clara)
2. Procedimento oficial com ambiguidade de vigência (PROC coexistentes — verificar data)
3. FAQ do time de atendimento (apoio prático, sem força normativa)

Regras:
- Se houver conflito entre documento oficial e FAQ: prevalece o documento oficial.
- Se houver conflito entre dois documentos oficiais: responder com transparência e escalar para o dono do processo (Comercial/Compliance/Operações).

---

## 4. Resumo Consolidado por Tema

### 4.1 Devolução de Mercadorias (POL-001)

- **Prazo geral:** até 7 dias úteis após recebimento
- **Exceções não elegíveis no processo padrão:**
  - Cargas perigosas classes 1 a 6 ANTT
  - Cargas refrigeradas com quebra de cadeia de frio
  - Cargas com lacre violado (salvo exceção documentada na entrega)
- **Para exceções:** encaminhar Gestão de Riscos (ramal 4500)
- **Triagem do atendimento:** 4 horas úteis
- **Coleta reversa:** até 2 dias úteis após aprovação
- **Reembolso/crédito:** até 5 dias úteis após recebimento no CD
- **Custos:**
  - Erro/defeito da NovaTech: sem custo para o cliente
  - Desistência do cliente: frete reverso por conta do cliente
  - Fora do prazo: não elegível no fluxo padrão

### 4.2 Frete Especial Acima de 500kg (PROC-042 v1 e v2)

Fórmula base (comum a v1 e v2):
> Valor do frete = Valor base × Multiplicador regional × Fator de peso

**Diferenças relevantes entre versões:**

| Item | v1 | v2 |
|------|----|----|
| Fator de peso (faixas) | 1.0 / 1.2 / 1.5 | 1.0 / 1.15 / 1.4 |
| Sul | 1.2 | 1.3 |
| Sudeste | 1.0 | 1.1 |
| Centro-Oeste | 1.3 | 1.4 |
| Nordeste | 1.4 | 1.5 |
| Norte | 1.6 | 1.8 |
| Prazo adicional | +2 dias úteis | +3 dias úteis |
| Desconto | Negociação > 10 fretes/mês | 5% (8+/mês); 10% (15+/mês) |

Disposição transitória da v2:
- Chamados abertos antes de 01/12/2023 (em processamento): usar v1
- Chamados novos a partir de 01/12/2023: usar v2

### 4.3 SLA por Tipo de Cliente (SLA-2024)

Tiers válidos: **Gold**, **Silver**, **Standard**. Não existe tier Platinum.

| Tipo | Gold | Silver | Standard |
|------|------|--------|----------|
| Chamados gerais — resposta | 2h úteis | 4h úteis | 8h úteis |
| Chamados gerais — resolução | 24h úteis | 48h úteis | 72h úteis |
| Incidentes críticos — resposta | 30 min | 1h | 2h |
| Incidentes críticos — resolução | 4h | 8h | 24h |

Penalidades por violação no mês:
- 1ª ocorrência: registro interno
- 2ª ocorrência: crédito de 5%
- 3ª ou mais: crédito de 10% + reunião obrigatória

### 4.4 FAQ de Atendimento

O FAQ reflete experiência prática e acelera respostas operacionais, mas não é documento normativo.

Uso recomendado:
- Como apoio para contexto e linguagem de atendimento
- Nunca como única base para decisões críticas de regra, prazo ou valor
- Sempre cruzar com POL/PROC/SLA antes de resposta final ao cliente

---

## 5. Contradições Identificadas

1. PROC-042 v1 e v2 coexistem sem descontinuação formal explícita.
2. Fatores de peso diferentes entre v1 e v2.
3. Multiplicadores regionais diferentes entre v1 e v2 para todas as regiões.
4. Prazo adicional do frete especial diferente (+2 vs +3 dias úteis).
5. Critérios de desconto diferem entre v1 e v2.
6. FAQ cita práticas sem respaldo formal para alguns temas (ex.: frete expresso para carga perigosa).

---

## 6. Lacunas de Documentação

1. Política formal para carga danificada em trânsito não documentada em POL/PROC.
2. Regras formais de seguro de carga ausentes em documento oficial (apenas no FAQ).
3. Regras de frete padrão abaixo de 500kg não aparecem nos procedimentos analisados.
4. Fluxo detalhado de tratamento interno da Gestão de Riscos não documentado.

---

## 7. Diretriz Operacional para Respostas

Fluxo recomendado:
1. Validar se a pergunta está coberta por POL, PROC ou SLA.
2. Se houver duas versões conflitantes, explicitar o conflito e aplicar critério temporal/contratual.
3. Se a pergunta só estiver no FAQ, responder com ressalva de que é prática informal e indicar validação.
4. Se não houver cobertura documental, informar ausência e escalar.

Modelos de resposta segura:
- "Com base no documento X, a regra é Y."
- "Há coexistência de versões para este tema (v1 e v2). Para aplicar corretamente, precisamos validar data de abertura do chamado/contrato vigente."
- "Não encontrei regra formal nos documentos oficiais para este caso; recomendo escalar para [área responsável]."

---

## 8. Referência para RAG e Avaliação de Prompts

Do Anexo B (chunks de referência):
- Mapa de cobertura: pergunta → chunks esperados
- Armadilhas propositais para teste de alucinação, conflito de versões e uso indevido de FAQ

Aplicações recomendadas:
- Testar recuperação de chunks corretos por pergunta
- Medir quando o assistente confunde v1 e v2
- Verificar se o assistente recusa inventar resposta quando não há cobertura

---

## 9. Checklist de Governança

Para manter qualidade da base de conhecimento:
- [ ] Definir status formal de vigência para PROC-042 v1 e v2
- [ ] Criar política oficial para carga danificada
- [ ] Publicar procedimento formal de seguro de carga
- [ ] Publicar procedimento de frete padrão (< 500kg)
- [ ] Formalizar processo de exceção da Gestão de Riscos
- [ ] Estabelecer cadência de revisão e responsável por documento

---

## 10. Conclusão Executiva

A Prática 1 contém base suficiente para os casos frequentes de devolução, frete especial e SLA, mas apresenta risco operacional por coexistência de versões e dependência de FAQ informal em temas críticos.

Recomendação principal:
- Operar com precedência de documentos oficiais
- Explicitar conflitos quando existirem
- Escalar casos sem cobertura formal
- Priorizar saneamento documental para reduzir inconsistências no atendimento e em soluções de IA com RAG
