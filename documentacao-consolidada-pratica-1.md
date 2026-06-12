# Documentação Consolidada — Prática 1
## Trilha AI First DGS | Cenário-Âncora 1 — Fase de Entendimento e Contexto
### Papel: QA | Participante: Taciana Vagliati

---

## 1. Contexto do Projeto

A **NovaTech** é uma empresa de médio porte do setor de logística com 1.200 funcionários. A equipe de atendimento ao cliente (45 pessoas) gasta em média **12 minutos por chamado** buscando informações em três fontes dispersas:

| Fonte | Quantidade | Formato | Atualização | Responsável |
|-------|-----------|---------|-------------|-------------|
| SharePoint | ~800 docs | PDF, DOCX | Mensal | Operações, Compliance |
| Confluence | ~400 páginas | HTML/Wiki | Semanal | TI, Comercial |
| Pasta de rede | ~50 planilhas | XLSX | Mensal | Comercial |

**Meta:** Reduzir o tempo de busca de 12 para menos de 2 minutos por chamado usando um assistente de IA integrado ao Microsoft Teams + SharePoint.

**Volume:** 320 chamados/dia | ~60% envolvem consulta a documentação.

---

## 2. Base de Documentação Oficial da NovaTech

### 2.1 POL-001 — Política de Devolução (v3.1 | 15/01/2024)

- **Prazo geral:** 7 dias úteis após recebimento confirmado no tracking (excluem-se sábados, domingos e feriados nacionais).
- **Categorias NÃO elegíveis** para devolução padrão:
  - Cargas perigosas classes 1–6 ANTT (Resolução ANTT nº 5.947/2021)
  - Cargas refrigeradas com cadeia de frio rompida (>30 min fora da faixa)
  - Cargas com lacre violado sem documentação no ato da entrega
- **Para exceções:** contatar Gestão de Riscos — ramal 4500.
- **Procedimento:** abertura de chamado no Portal do Cliente (portal.novatech.com.br) com CT-e + 3 fotos + motivo.
- **Triagem:** 4 horas úteis. Coleta reversa: até 2 dias úteis após aprovação. Reembolso: até 5 dias úteis após recebimento no CD.
- **Custos:** erro da NovaTech → sem custo; desistência do cliente → frete reverso por conta do cliente; prazo expirado → encaminhar ao Comercial.

### 2.2 PROC-042 v1 — Frete Especial (v1.0 | 03/03/2023)

> ⚠️ Documento SEM indicação formal de vigência. Coexiste com PROC-042-v2.

**Fórmula:** `Valor base × Multiplicador regional × Fator de peso`

| Região | Multiplicador |
|--------|--------------|
| Sul | 1.2 |
| Sudeste | 1.0 |
| Centro-Oeste | 1.3 |
| Nordeste | 1.4 |
| Norte | 1.6 |

**Fatores de peso (v1):** 1.0 (500–1.000kg) | 1.2 (1.001–3.000kg) | 1.5 (>3.000kg)

**Prazo adicional:** +2 dias úteis

### 2.3 PROC-042-v2 — Frete Especial Revisado (v2.0 | 10/11/2023)

> ⚠️ Sem indicação formal de que substitui o v1. Ambos coexistem no SharePoint.

| Região | Multiplicador |
|--------|--------------|
| Sul | 1.3 |
| Sudeste | 1.1 |
| Centro-Oeste | 1.4 |
| Nordeste | 1.5 |
| Norte | 1.8 |

**Fatores de peso (v2):** 1.0 (500–1.000kg) | 1.15 (1.001–3.000kg) | 1.4 (>3.000kg)

**Prazo adicional:** +3 dias úteis

**Regra de transição:** chamados abertos antes de 01/12/2023 → usar v1; novos a partir de 01/12/2023 → usar v2.

**Descontos de volume (v2):** ≥8 fretes/mês → 5% sobre multiplicador | ≥15 fretes/mês → 10%.

### 2.4 SLA-2024 — Tabela de SLA por Cliente (v2024.1 | 02/01/2024)

| Tier | Critério |
|------|---------|
| Gold | Contrato anual >R$ 500.000 OU >200 operações/mês |
| Silver | Contrato anual R$ 100.000–500.000 OU 50–200 operações/mês |
| Standard | Todos os demais |

> ⚠️ **NÃO existem outros tiers.** Tier Platinum não existe na NovaTech.

| Métrica | Gold | Silver | Standard |
|---------|------|--------|----------|
| Resposta — chamados gerais | 2h úteis | 4h úteis | 8h úteis |
| Resolução — chamados gerais | 24h úteis | 48h úteis | 72h úteis |
| Resposta — incidentes críticos | 30 min | 1h | 2h |
| Resolução — incidentes críticos | 4h | 8h | 24h |

**Incidente crítico:** carga >R$100K com status desconhecido há >6h | carga perigosa com irregularidade | >5 chamados do mesmo cliente em 24h sobre o mesmo problema | risco à segurança de pessoas.

**SLA não pausa** para incidentes críticos de clientes Gold.

### 2.5 FAQ-Atendimento (documento informal — NÃO validado)

> ⚠️ Criado informalmente pelo time ao longo de 2 anos. Use com cautela — pode conter informações desatualizadas.

Pontos relevantes:
- Carga perigosa: orientar ramal 4500 (Gestão de Riscos). Não dizer que é impossível.
- Duas versões PROC-042 coexistem — usar v2 como padrão, mas contratos antigos podem usar v1.
- Tier Platinum não existe — cliente pode estar confundindo com programa descontinuado em 2022.
- Seguro de carga: 0,3% (padrão) e 0,8% (perigosa) — confirmar com Comercial para contratos anteriores a 2023.
- Carga danificada: registrar em até 48h, encaminhar para sinistros@novatech.com.br.

---

## 3. Contradições Identificadas na Base

| ID | Documentos | Contradição |
|----|-----------|------------|
| C1 | PROC-042 v1 vs v2 | Multiplicadores regionais diferentes (ex: Norte: 1.6 vs 1.8) |
| C2 | PROC-042 v1 vs v2 | Fator de peso diferente (ex: 1.001–3.000kg: 1.2 vs 1.15) |
| C3 | PROC-042 v1 vs v2 | Prazo adicional: +2 dias vs +3 dias |
| C4 | FAQ-32 vs docs formais | FAQ diz que carga perigosa pode ter frete expresso "com autorização", mas não há documento formal |

## 4. Gaps Identificados

| ID | Descrição |
|----|-----------|
| G1 | Não há documento formal sobre tratamento de carga danificada em trânsito (só FAQ) |
| G2 | Seguro de carga: apenas no FAQ, sem documento formal |
| G3 | Frete padrão (<500kg): não há documento na base |
| G4 | Processo da Gestão de Riscos para cargas perigosas: não documentado |

---

## 5. Resumo dos Exercícios QA — Cenário 1

| Exercício | Título | Entregável Principal |
|-----------|--------|---------------------|
| 1.1 | Identificação de cenários de falha de IA | Lista de 10+ cenários em 5 categorias |
| 1.2 | Design de critérios de aceitação | Rubrica + template + avaliação das 5 respostas |
| 1.3 | Plano de testes para pipeline de RAG | Plano estruturado em 6 categorias + artefato Cowork |
