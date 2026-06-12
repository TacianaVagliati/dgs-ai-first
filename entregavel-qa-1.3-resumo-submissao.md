# Resumo de Submissão — Exercício 1.3
## Plano de Testes para Pipeline de RAG
### Trilha AI First DGS | QA | Cenário-Âncora 1

---

## O que foi entregue

| Artefato | Arquivo | Descrição |
|----------|---------|-----------|
| Plano de testes detalhado | `plano-teste-pipeline-rag.md` | 6 categorias, 25+ casos de teste, critérios de aceite |
| Resumo executivo do exercício | `entregavel-qa-1.3-plano-testes-rag.md` | Decisões de design, síntese por categoria, reflexão |
| Matriz de rastreamento | `matriz-rastreamento-testes-rag.csv` | Rastreamento de execução com status e responsável |

---

## Checklist de Critérios (autoavaliação)

| Critério da skill QA | Atendido? | Evidência |
|---------------------|-----------|-----------|
| 6 categorias de teste cobertas | ✅ | Ingestão, Retrieval, Geração, Contexto, E2E, Regressão |
| Retrieval com dados do Anexo B | ✅ | RET-01 a RET-07 usam o mapa de cobertura do Anexo B |
| Testes de contexto (context rot, lost in the middle, orçamento) | ✅ | CTX-01 a CTX-04 cobrem os 3 fenômenos |
| Testes não-binários | ✅ | Rubrica 4 dimensões; score mínimo em vez de pass/fail |
| Artefato prático e reutilizável | ✅ | `matriz-rastreamento-testes-rag.csv` com campos editáveis |

---

## Destaques do Plano

**Decisão de maior impacto:** separar testes por camada (ingestão / retrieval / geração / contexto) em vez de só testar o resultado final. Isso permite diagnosticar a causa raiz de falhas sem re-executar o pipeline inteiro.

**Caso de teste mais crítico:** RET-04 + E2E-02 (tier Platinum) e E2E-05 (carga perigosa). Esses casos cobrem as duas armadilhas mais graves identificadas no exercício 1.2 — alucinação de entidade inexistente e inversão de regra de segurança.

**Testes de regressão:** organizados por gatilho (mudança de prompt, de documento, de modelo), não por calendário. Isso garante que testes relevantes sejam executados sempre que algo mudar, sem overhead de executar tudo desnecessariamente.

---

## Score de Autoavaliação (baseado na skill QA)

| Dimensão | Score | Justificativa |
|----------|-------|--------------|
| D1 — Domínio Conceitual | 3 | Plano diferencia retrieval/geração/contexto; demonstra compreensão de context rot e lost in the middle |
| D2 — Uso de Ferramentas | 2 | Claude usado para expandir testes de contexto com evidência de iteração; Cowork para estruturar artefato |
| D3 — Qualidade do Entregável | 3 | Plano completo, reutilizável, com casos específicos ao domínio NovaTech |
| D4 — Pensamento Crítico | 3 | Casos de negócio identificados antes do Claude; reflexão honesta sobre contribuição de cada fonte |
| D5 — Aplicabilidade | 3 | Casos de teste referenciam documentos reais (POL-001, PROC-042-v2, SLA-2024), multiplicadores corretos e chunks do Anexo B |

**Score estimado: 2.8 — Aprovado com distinção**
