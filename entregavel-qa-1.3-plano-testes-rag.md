# Entregável QA 1.3 — Plano de Testes para Pipeline de RAG (NovaTech)

## 1. Objetivo

Definir e operacionalizar um plano de testes para validar a qualidade do pipeline de RAG no cenário NovaTech, cobrindo as 6 categorias obrigatórias:
- Ingestão
- Retrieval
- Geração
- Contexto
- Ponta a ponta
- Regressão

Meta de qualidade:
- Eliminar respostas incorretas por conflito documental
- Garantir rastreabilidade por fonte em 100% das respostas
- Evitar alucinação quando não houver cobertura documental

---

## 2. Escopo e fontes

Base oficial utilizada:
- anexo-a-documentacao-simulada-novatech.md
- anexo-b-chunks-referencia-rag.md
- POL-001-politica-devolucao.md
- PROC-042-frete-especial-v1.md
- PROC-042-v2-frete-especial-revisado.md
- SLA-2024-tabela-sla-clientes.md
- FAQ-atendimento.md (informal; não normativo)

---

## 3. Estratégia de avaliação (não binária)

Escala por caso de teste:
- **0** = Incorreto (erro factual, alucinação, sem fonte, inversão de regra)
- **1** = Parcialmente correto (acerta núcleo, mas omite ressalva crítica, fonte incompleta ou ambiguidade mal tratada)
- **2** = Correto (resposta adequada, fonte consistente, comportamento esperado)

Classificação do ciclo de execução:
- **Excelente:** média ≥ 1.8
- **Aceitável:** média entre 1.5 e 1.79
- **Reprovado:** média < 1.5

Critérios mínimos de aprovação do ciclo:
- Nenhum caso crítico com nota 0
- Cobertura completa das 6 categorias
- Ao menos 90% dos casos com nota ≥ 1

---

## 4. Matriz de categorias de teste

| Categoria | Objetivo | Critério principal |
|-----------|----------|--------------------|
| Ingestão | Verificar extração e metadados de documentos | Arquivos indexados corretamente com origem e versão |
| Retrieval | Verificar recuperação dos chunks corretos | Top-N com aderência ao mapa de cobertura do Anexo B |
| Geração | Verificar resposta final do assistente | Correção factual + citação de fonte + guardrails |
| Contexto | Verificar comportamento em contexto longo e concorrente | Sem context rot e sem perda de informação crítica |
| Ponta a ponta | Verificar fluxo completo pergunta → resposta | Resposta final utilizável no atendimento |
| Regressão | Evitar retorno de bugs após ajustes | Casos antigos permanecem corretos após mudanças |

---

## 5. Casos de retrieval com gabarito do Anexo B

Top-K recomendado para validação: K = 3 a 5.

| ID | Pergunta | Chunks esperados (mínimo) | Tipo |
|----|----------|--------------------------|------|
| R01 | Qual o prazo de devolução? | POL-001-A, POL-001-B | Regra formal |
| R02 | Posso devolver carga perigosa? | POL-001-B | Exceção crítica |
| R03 | Qual o SLA do cliente Gold? | SLA-2024-B | SLA |
| R04 | Qual o SLA do cliente Platinum? | SLA-2024-A | Anti-alucinação |
| R05 | Frete para 600kg para Manaus? | PROC-042v2-A, PROC-042v2-B | Frete especial |
| R06 | Frete para 300kg para Salvador? | Nenhum chunk plenamente aderente | Sem cobertura |
| R07 | Qual o multiplicador do Sudeste? | PROC-042v2-B | Conflito v1 x v2 |

Critério de aprovação por caso de retrieval:
- **Nota 2:** Todos os chunks essenciais presentes no Top-K
- **Nota 1:** Parte dos chunks essenciais presentes, sem erro grave
- **Nota 0:** Chunks errados no topo, ausência de chunk essencial ou confusão de versão

---

## 6. Casos de geração (qualidade da resposta)

| ID | Entrada | Resultado esperado | Guardrails |
|----|---------|--------------------|-----------| 
| G01 | Posso devolver carga perigosa? | Negar devolução no fluxo padrão com base na POL-001 e orientar Gestão de Riscos (ramal 4500) | 1, 2, 4 |
| G02 | Cliente Platinum: qual SLA? | Informar inexistência do tier Platinum; não inventar valores | 1, 2, 3, 4 |
| G03 | Frete 600kg para Norte | Aplicar lógica de frete especial > 500kg, multiplicador 1.8, citar PROC-042-v2 | 1, 2, 4 |
| G04 | Frete 300kg | Declarar ausência de regra formal para frete padrão < 500kg; escalar | 1, 3, 4 |
| G05 | Carga perigosa com frete expresso | Apontar que prática está apenas no FAQ informal e falta respaldo formal oficial | 1, 2, 4 |

Guardrails obrigatórios na validação:
- **1:** Sempre citar fonte
- **2:** Nunca inventar prazo/valor/tier
- **3:** Quando não houver base, declarar ausência e indicar escalonamento
- **4:** Responder em português formal

---

## 7. Casos de contexto (context engineering)

### 7.1 Context rot (sessão longa)

Cenário: rodada com 8 a 12 perguntas consecutivas misturando SLA, devolução e frete.

Esperado:
- Manter regra de não elegibilidade para carga perigosa até a última pergunta
- Não perder referência de versão PROC-042 v1/v2
- Não inventar tier Platinum após múltiplas rodadas

### 7.2 Lost in the middle

Cenário: prompt com muitos chunks, chunk crítico posicionado no meio.

Esperado:
- Resposta ainda considera o chunk crítico
- Sem inversão de regra por prioridade errada de contexto

### 7.3 Competição de contexto

Cenário: histórico da conversa sugere informação informal do FAQ enquanto chunk oficial diverge.

Esperado:
- Prevalecer documento oficial
- Sinalizar conflito ao atendente

---

## 8. Casos de ingestão

Checklist de validação:
- [ ] Todos os arquivos obrigatórios indexados
- [ ] Metadado de origem presente (nome do documento)
- [ ] Metadado de versão presente quando aplicável (PROC v1 e v2 distinguíveis)
- [ ] Separação explícita entre documento oficial e FAQ informal
- [ ] Atualização de índice sem duplicação indevida

---

## 9. Testes ponta a ponta

Fluxo validado:
1. Pergunta do atendente
2. Recuperação de chunks (Top-K)
3. Montagem de contexto
4. Resposta com citação de fonte
5. Decisão de atendimento (responder diretamente ou escalar)

Critérios:
- Resposta clara e acionável
- Alinhamento com documento oficial
- Transparência explícita em caso de ambiguidades ou conflito de versões

---

## 10. Regressão

Suíte mínima fixa de regressão (executar a cada mudança de prompt, chunking, reranking ou base documental):

| ID | Cenário crítico |
|----|----------------|
| RG01 | Devolução carga perigosa — negar no processo padrão |
| RG02 | Tier Platinum — recusar como inexistente |
| RG03 | Frete 600kg Norte — aplicar v2, multiplicador 1.8 |
| RG04 | Frete 300kg sem cobertura — declarar ausência |
| RG05 | Conflito PROC-042 v1 x v2 — diferenciar versões com transparência |

---

## 11. Template operacional de execução

| Caso | Categoria | Resultado | Nota (0–2) | Observação | Responsável | Status |
|------|-----------|-----------|------------|------------|-------------|--------|
| R01 | Retrieval | Aprovado | 2 | Chunks corretos no Top-3 | QA | Feito |
| R02 | Retrieval | Aprovado | 2 | POL-001-B recuperado no Top-1 | QA | Feito |
| R03 | Retrieval | Parcial | 1 | SLA-2024-B no Top-4 | QA | Feito |
| R04 | Geração | Reprovado | 0 | Inventou tier Platinum | QA | Aberto |
| C01 | Contexto | Parcial | 1 | Perdeu referência de versão na rodada 9 | QA | Aberto |

---

## 12. Priorização de risco

| Prioridade | Tipo de falha |
|------------|---------------|
| Alta | Alucinação de tier inexistente |
| Alta | Inversão de regra de devolução de carga perigosa |
| Alta | Mistura de valores entre PROC v1 e v2 sem transparência |
| Média | Omissão de fonte |
| Média | Resposta parcialmente correta sem ressalva de ambiguidade |
| Baixa | Pequenas falhas de redação sem impacto de decisão |

---

## 13. Critério de saída (go/no-go)

**Go:**
- Média geral ≥ 1.8
- Zero casos críticos com nota 0
- Cobertura das 6 categorias concluída
- Regressão mínima 100% executada

**No-go:**
- Qualquer caso crítico com nota 0
- Ausência de cobertura em contexto ou regressão
- Persistência de falha em tier inexistente ou carga perigosa

---

## 14. Plano de melhoria contínua

- Revisar mensalmente casos com novos documentos publicados
- Atualizar suíte de regressão com incidentes reais do atendimento
- Monitorar taxa de respostas sem cobertura documental
- Monitorar taxa de conflito de versão (v1 x v2)
- Reavaliar Top-K e estratégia de reranking trimestralmente

---

## 15. Aderência ao exercício QA 1.3

Este plano entrega:
- As 6 categorias exigidas (ingestão, retrieval, geração, contexto, ponta a ponta, regressão)
- Casos de retrieval com gabarito do Anexo B (7 pares pergunta → chunks esperados)
- Cobertura explícita de context rot, lost in the middle e competição de contexto
- Avaliação não binária com graus de qualidade (0/1/2)
- Template operacional reutilizável com status, responsável e evidência esperada
