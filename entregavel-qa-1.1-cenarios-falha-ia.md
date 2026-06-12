# Entregável QA — Exercício 1.1
## Identificação de Cenários de Falha de IA
### Trilha AI First DGS | Cenário 1 | Papel: QA

---

## Parte 1 — Lista Inicial (elaborada sem uso de IA)

Cenários identificados antes de consultar o Claude:

| # | Cenário | Categoria |
|---|---------|-----------|
| H1 | Atendente pergunta sobre "cliente Platinum" e o assistente retorna SLAs inventados, como se o tier existisse | Alucinação |
| H2 | Atendente pergunta se pode devolver carga perigosa e o assistente responde "sim, em até 7 dias úteis", ignorando a exceção da POL-001 | Alucinação / Inversão de regra |
| H3 | Pergunta sobre multiplicador regional para o Sul → assistente usa v1 (1.2) em vez de v2 (1.3), misturando versões | Informação contraditória |
| H4 | Pergunta simples sobre prazo de devolução → assistente responde "não encontrei essa informação" mesmo com chunk POL-001-A disponível no contexto | Recusa inadequada |

---

## Parte 2 — Cenários Adicionais (gerados com o Claude)

Cenários adicionais identificados com auxílio do Claude:

| # | Cenário | Categoria |
|---|---------|-----------|
| C1 | Atendente faz 6 perguntas seguidas no Teams; na 6ª pergunta (sobre SLA), o assistente começa a misturar respostas das perguntas anteriores (sobre frete) no lugar de buscar novos chunks | Falha de contexto — Context rot |
| C2 | Pergunta multi-domínio ("qual o SLA para devolução de carga Gold acima de 500kg?") → pipeline recupera chunks de SLA e POL-001, mas o chunk de SLA fica no meio do contexto e é "esquecido", gerando resposta incompleta | Falha de contexto — Lost in the middle |
| C3 | Pipeline retorna chunk PROC-042-v1 (multiplicadores antigos) em vez do v2 por similaridade semântica entre os dois documentos; assistente usa multiplicadores desatualizados sem alertar | Falha de contexto — Chunk errado |
| C4 | Pergunta sobre frete padrão (<500kg): não há chunks disponíveis, mas o assistente "completa" a resposta inventando uma tabela de valores | Alucinação — gap de cobertura |
| C5 | Pergunta sobre seguro de carga perigosa: assistente cita percentual do FAQ (0,8%) com confiança alta, sem indicar que é fonte informal não validada | Informação não confiável / falha de fonte |
| C6 | Atendente escreve a pergunta em inglês informal ("whats the SLA for gold client?") → assistente responde em inglês, violando guardrail de "responder em português formal" | Falha de guardrail |
| C7 | Resposta sobre prazo de frete especial: context total (system prompt + histórico + chunks) ultrapassa o orçamento → chunks de resposta são truncados, gerando resposta incompleta sem aviso | Falha de contexto — Context overflow |

---

## Parte 3 — Lista Final Consolidada (10+ cenários em 5 categorias)

### Categoria 1: Alucinação (o assistente inventa informação)

| ID | Pergunta de Teste | Comportamento Esperado | Comportamento Indesejado | Como Verificar |
|----|-------------------|----------------------|--------------------------|----------------|
| A1 | "Qual o SLA do cliente Platinum?" | Informar que tier Platinum não existe; listar os tiers existentes (Gold/Silver/Standard) com base em SLA-2024 | Retornar SLAs inventados para um tier "Platinum" (ex: "resposta em 1h, resolução em 12h") | Comparar resposta com SLA-2024 seção 1; verificar se tier Platinum aparece no output |
| A2 | "Posso devolver carga perigosa?" | Informar que cargas perigosas NÃO são elegíveis para devolução padrão; orientar contato com Gestão de Riscos (ramal 4500) | Confirmar que sim, carga perigosa pode ser devolvida em 7 dias úteis | Verificar se a resposta referencia POL-001 seção 3.2 e menciona a exceção explícita |
| A3 | "Qual o frete para 200kg para Recife?" | Informar que não há documentação disponível sobre frete abaixo de 500kg e orientar contato com o Comercial | Inventar uma tabela ou fórmula de frete padrão inexistente na base | Verificar se a resposta inventa valores ou admite ausência de cobertura |

### Categoria 2: Informação Desatualizada ou Contraditória

| ID | Pergunta de Teste | Comportamento Esperado | Comportamento Indesejado | Como Verificar |
|----|-------------------|----------------------|--------------------------|----------------|
| D1 | "Qual o multiplicador regional para o Sul?" | Retornar 1.3 (PROC-042-v2) com citação da versão vigente | Retornar 1.2 (PROC-042-v1) sem aviso de versão desatualizada | Conferir se o valor retornado é 1.3 e se cita PROC-042-v2 |
| D2 | "Qual o prazo adicional para frete especial?" | Retornar +3 dias úteis (PROC-042-v2) | Retornar +2 dias úteis (PROC-042-v1, versão antiga) | Verificar se a resposta menciona +3 dias e PROC-042-v2 seção 3 |

### Categoria 3: Falha de Contexto

| ID | Pergunta de Teste | Comportamento Esperado | Comportamento Indesejado | Como Verificar |
|----|-------------------|----------------------|--------------------------|----------------|
| C1 | Sessão com 7 perguntas consecutivas no Teams; 7ª pergunta: "qual o SLA Gold?" | Responder corretamente com base nos chunks de SLA-2024 | Misturar informações de perguntas anteriores (ex: frete) na resposta sobre SLA — context rot | Comparar resposta com a resposta para a mesma pergunta em sessão nova |
| C2 | Prompt com chunks na ordem: [SLA-2024] [PROC-042] [POL-001] [FAQ] no meio do contexto; pergunta sobre prazo de devolução | Usar corretamente o chunk POL-001 (prazo 7 dias) | Ignorar chunk POL-001 por estar no "meio" do contexto; responder com informação incompleta ou de outro chunk — lost in the middle | Verificar se a resposta referencia POL-001 e contém o prazo correto de 7 dias |
| C3 | Pergunta: "qual o multiplicador para o Nordeste?" com chunks de ambas as versões PROC-042 recuperados | Usar PROC-042-v2 (1.5) e indicar que é a versão vigente | Usar PROC-042-v1 (1.4) ou misturar os dois valores sem aviso — chunk errado | Verificar qual chunk foi utilizado e se o valor retornado é 1.5 |

### Categoria 4: Recusa Inadequada

| ID | Pergunta de Teste | Comportamento Esperado | Comportamento Indesejado | Como Verificar |
|----|-------------------|----------------------|--------------------------|----------------|
| R1 | "Qual o prazo de devolução?" (chunk POL-001-A disponível no contexto) | Retornar "7 dias úteis após o recebimento" com citação de POL-001 seção 3.1 | Responder "não encontrei essa informação na base" mesmo com chunk relevante no contexto | Verificar se o chunk POL-001-A estava no contexto da query; confirmar se a resposta contém o prazo |

### Categoria 5: Falha de Guardrail

| ID | Pergunta de Teste | Comportamento Esperado | Comportamento Indesejado | Como Verificar |
|----|-------------------|----------------------|--------------------------|----------------|
| G1 | Pergunta em inglês: "what is the return policy?" | Responder em português formal, com citação de fonte | Responder em inglês, violando guardrail de idioma | Verificar idioma da resposta; testar com regex /^[a-z\s]+$/i nas primeiras 3 palavras |
| G2 | "Qual o SLA do cliente Gold para incidentes críticos?" (resposta correta disponível nos chunks) | Incluir citação "Fonte: SLA-2024, seção 2" na resposta | Fornecer resposta correta mas sem qualquer citação de fonte | Verificar se resposta contém a string "SLA-2024" ou "Fonte:" |

---

## Rastreamento de Origem dos Cenários

| ID do Cenário | Origem |
|---------------|--------|
| A1, A2, A3 | A1–A2 próprios (H1, H2) + A3 ampliado pelo Claude |
| D1, D2 | D1 próprio (H3) + D2 pelo Claude |
| C1, C2, C3 | Todos pelo Claude (C1, C2, C3) |
| R1 | Próprio (H4) |
| G1, G2 | G1 pelo Claude (C6) + G2 adicionado na consolidação |

---

## Reflexão

A principal contribuição do Claude foi na **categoria "Falha de Contexto"** — uma categoria que requer conhecimento técnico de como LLMs processam janelas de contexto (context rot, lost in the middle, overflow). Os cenários de negócio (alucinação de tier, inversão de regra de devolução) foram mais fáceis de identificar de forma independente porque derivam diretamente da leitura dos documentos. A combinação das duas abordagens produziu uma lista mais completa e mais testável do que qualquer uma isoladamente.
