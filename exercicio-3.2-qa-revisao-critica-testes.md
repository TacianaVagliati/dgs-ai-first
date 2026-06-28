# Exercício 3.2 — QA: Revisão Crítica dos Testes Gerados por IA

> **Papel:** QA  
> **Cenário:** 3 — Fase de Governança e Validação  
> **Tópico:** Revisão Crítica de Outputs de IA  
> **Ferramenta:** Claude (chat)

---

## Contexto

O Copilot gerou testes de integração para o projeto NovaTech. O Tech Lead pediu que você revise antes do merge — testes que "passam" mas não verificam o comportamento certo dão falsa segurança e são piores do que não ter testes.

**Referência:** O projeto usa **Vitest** como framework de testes (ver AGENTS.md / Anexo C). Não usa Jest.

---

## Os 3 Testes Gerados pelo Copilot

```typescript
// Teste 1 — assertions vagas
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});

// Teste 2 — dados irreais
describe('query endpoint edge cases', () => {
  it('should handle empty question', async () => {
    const res = await request(app).post('/api/query').send({ question: '' });
    expect(res.status).toBe(400);
  });
});

// Teste 3 — mock que mascara bug
describe('feedback endpoint', () => {
  it('should save feedback', async () => {
    const mockCreate = jest.fn().mockResolvedValue({ id: '123' });
    const res = await request(app).post('/api/feedback').send({
      queryId: 'q1', rating: 5, comment: 'great'
    });
    expect(res.status).toBe(200);
    expect(mockCreate).toHaveBeenCalled();
  });
});
```

---

## Entregável 1 — Revisão Própria (ANTES do Claude)

> **Regra:** Faça esta análise antes de usar qualquer IA. Para cada teste: o que testa, o que falha em testar, e qual o risco se passar com o código errado.

---

### Teste 1 — assertions vagas

**O que testa:**
- Que o endpoint `/api/query` responde com status 200.
- Que `res.body` não é `undefined` ou `null`.

**O que falha em testar:**
- Não verifica se o conteúdo da resposta está correto. `res.body` poderia ser `{}`, `{ error: "falha interna" }`, ou `{ answer: "receita de bolo" }` — o teste passaria em todos os casos.
- Não verifica se `source_document` está presente (campo obrigatório do structured output).
- Não verifica se `answer` contém a informação esperada sobre prazo de devolução.
- Não verifica se `confidence_score` foi retornado.

**Classificação dos problemas:**

| Problema | Tipo |
|----------|------|
| `expect(res.body).toBeDefined()` como única assertion de conteúdo | Testing Standards — assertion vaga |
| Nome do teste `'should return a response'` não descreve comportamento esperado | Testing Standards — nomenclatura |
| Pergunta `'prazo devolução'` não usa fixtures do domínio | Testing Standards — dados de teste |
| Sem verificação do conteúdo factual da resposta | Bug potencial — resposta errada passa |

**Risco se o teste passar com código errado:**
O response-validator poderia estar retornando a mensagem de fallback padrão em vez da resposta real, e o teste não detectaria. O endpoint poderia estar alucinando ou retornando resposta de outro documento — o teste passaria com 200 e `body` definido em qualquer situação.

---

### Teste 2 — dados irreais

**O que testa:**
- Que o endpoint retorna 400 para pergunta vazia.

**O que falha em testar:**
- É um teste de validação de input — válido e necessário. Mas é o único edge case coberto, e não exercita nenhuma lógica de domínio.
- Não há nenhum teste com pergunta real de logística (frete, devolução, SLA, carga perigosa) para verificar que o assistente recupera os chunks corretos e retorna a informação certa.
- Ausência de testes de domínio significa que nenhuma regressão no pipeline RAG seria detectada pelos testes.

**Classificação dos problemas:**

| Problema | Tipo |
|----------|------|
| Nenhum teste exercita o domínio NovaTech | Testing Standards — fixtures de domínio ausentes |
| Cobertura só de validação de input, não de comportamento real | Bug potencial — falha de retrieval não detectada |
| Pergunta `''` não usa fixtures (`queries.emptyQuestion`) | Testing Standards — dados hardcoded |

**Risco se o teste passar com código errado:**
O pipeline de RAG poderia estar recuperando chunks errados, retornando alucinações, ou quebrando silenciosamente para perguntas reais — e nenhum teste detectaria, porque o único teste de comportamento verifica a rejeição de input vazio.

---

### Teste 3 — mock que mascara bug

**O que testa:**
- Que o endpoint `/api/feedback` responde com status 200.
- Que `mockCreate` foi chamado.

**O que falha em testar:**
- O `mockCreate` **não está conectado ao código real**. O mock é declarado mas nunca injetado na aplicação — `request(app)` usa o handler real, não o mock. Isso significa que `expect(mockCreate).toHaveBeenCalled()` sempre falhará (o mock nunca é chamado) — ou sempre passará se houver outro `CosmosClient` mocado globalmente sem que o autor perceba.
- Não verifica que a validação de input funciona (ex: `queryId` obrigatório).
- Não verifica que dados sensíveis (`attendantEmail`) não são logados.

**Classificação dos problemas:**

| Problema | Tipo |
|----------|------|
| `jest.fn()` em projeto que usa Vitest | Violação do AGENTS.md — framework errado |
| Mock não conectado ao handler real (mock fantasma) | Bug potencial — teste passa sem testar nada |
| `jest.fn()` sem `vi.fn()` pode silenciosamente não mockar nada | Bug potencial — resultado imprevisível |
| Sem validação de input obrigatório | Testing Standards — edge cases de domínio |
| `attendantEmail` presente no body sem teste de que não é logado | Problema de segurança — dado pessoal |

**Risco se o teste passar com código errado:**
Este é o teste mais perigoso dos três. O mock desconectado cria a ilusão de cobertura: o teste "passa" sem verificar que o feedback foi salvo, sem verificar a validação de input, e sem detectar que dados sensíveis estão sendo logados. Um desenvolvedor vendo testes verdes pode fazer merge de código com bugs sérios de segurança.

---

### Ponto de atenção: Jest vs Vitest

O Teste 3 usa `jest.fn()`. O projeto usa **Vitest** (definido no AGENTS.md). Isso é uma inconsistência com o AGENTS.md que demonstra que o Copilot gerou código sem ler as convenções do projeto. No Vitest, usa-se `vi.fn()` — `jest.fn()` pode funcionar em alguns ambientes com compatibilidade ativada, mas não é garantido e gera confusão. Todo teste gerado por IA deve ser revisado quanto ao framework correto.

---

## Entregável 2 — Revisão do Claude (segunda opinião)

> Prompt utilizado:

```
Você é um QA sênior revisando testes de integração gerados pelo Copilot para o projeto NovaTech.

Contexto:
- O projeto usa TypeScript com Vitest (não Jest) como framework de testes.
- O AGENTS.md proíbe: console.log, dados pessoais em logs, require dinâmico, as any sem Zod.
- Assertions devem ser específicas ao domínio (nunca apenas toBeDefined ou toBeTruthy).
- Dados de teste devem ser do domínio NovaTech (logística, fretes, SLAs, devoluções).

Para cada teste abaixo:
1. O que ele realmente testa.
2. O que ele falha em testar.
3. Qual o risco se o teste passar com código incorreto.
4. Classifique cada problema: violação do AGENTS.md, problema de segurança, ou bug potencial.

[inserir os 3 testes]
```

### Resultado da revisão do Claude

**Teste 1:** Claude identificou os mesmos problemas — assertion vaga, ausência de verificação do `source_document`, e nome do teste não-descritivo. Acrescentou que a ausência de mock do Azure OpenAI/Search significa que o teste faz chamada real (ou falha por falta de credenciais), o que viola o padrão de isolamento.

**Teste 2:** Claude concordou que o teste é necessário mas insuficiente. Acrescentou que, do ponto de vista de cobertura de domínio, seria prioritário adicionar um teste que exercite a query de carga perigosa (caso de alto risco) antes do go-live.

**Teste 3:** Claude identificou os mesmos problemas. Destaque adicional: chamou o mock desconectado de "phantom mock" — o mock existe mas não intercepta nada, tornando `expect(mockCreate).toHaveBeenCalled()` uma assertion sobre o vácuo. Também identificou o `jest.fn()` como violação do AGENTS.md.

---

## Entregável 3 — Comparação

### Concordâncias

As análises foram alinhadas nos pontos principais:
- Teste 1: assertion vaga, sem verificação de conteúdo.
- Teste 2: edge case válido mas sem cobertura de domínio.
- Teste 3: mock fantasma, jest/Vitest, dado sensível sem controle.

### O que o Claude acrescentou

1. **Teste 1:** A ausência de mock dos serviços externos (Azure) é um problema de infra de teste — o teste não é isolado e pode falhar ou acertar por motivos errados.
2. **Teste 2:** Sugestão prioritária: um teste de carga perigosa antes do go-live, dado o risco de negócio.
3. **Teste 3:** Nomenclatura "phantom mock" é útil para comunicar o problema ao time.

### O que minha análise acrescentou

A análise de segurança do `attendantEmail` no Teste 3 foi mais explícita na revisão própria — o risco de dado pessoal sendo logado sem controle (conforme violação identificada no feedback-handler do Dev 3.2) é um ponto que o QA deve conectar ao histórico do projeto.

### Honestidade sobre divergências

Não houve divergência de avaliação — as duas análises identificaram os mesmos problemas. A ausência de divergência é esperada porque os problemas são objetivos (framework errado, assertion vaga, mock desconectado). Exercícios com mais subjetividade tipicamente geram divergências mais ricas.

---

## Entregável 4 — Teste 1 Reescrito

### Versão original

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

**Problemas:** assertion vaga, nome genérico, sem mock, sem verificação de conteúdo.

---

### Versão reescrita

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { setupServer } from 'msw/node'
import request from 'supertest'
import { app } from '../../src/app'
import { mockAzureOpenAICompletion, mockAzureSearchResponse } from '../mocks/handlers'
import { chunks } from '../fixtures/chunks'
import { queries } from '../fixtures/queries'

const server = setupServer()

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))
afterAll(() => server.close())

describe('query endpoint — política de devolução', () => {
  it('should return correct return policy with source document when asked about return deadline', async () => {
    // Arrange
    server.use(
      mockAzureSearchResponse([chunks.pol001ReturnPolicy]),
      mockAzureOpenAICompletion({
        answer: 'O prazo de devolução para produtos standard é de 7 dias úteis após o recebimento. O cliente deve abrir chamado no portal e anexar fotos.',
        source_document: 'POL-001',
        confidence_score: 0.92
      })
    )

    // Act
    const res = await request(app)
      .post('/api/query')
      .send({ question: queries.returnDeadline })

    // Assert
    expect(res.status).toBe(200)
    expect(res.body.answer).toContain('7 dias úteis')
    expect(res.body.source_document).toMatch(/POL-001/)
    expect(res.body.confidence_score).toBeGreaterThan(0)
  })

  it('should block response and return safe fallback when source_document is missing', async () => {
    // Arrange — modelo retorna resposta sem campo obrigatório
    server.use(
      mockAzureSearchResponse([chunks.pol001ReturnPolicy]),
      mockAzureOpenAICompletion({
        answer: 'O prazo de devolução é 7 dias.',
        // source_document ausente — deve ser rejeitado pelo response-validator
        confidence_score: 0.85
      })
    )

    // Act
    const res = await request(app)
      .post('/api/query')
      .send({ question: queries.returnDeadline })

    // Assert — resposta padrão segura retornada, não a do modelo
    expect(res.status).toBe(200)
    expect(res.body.answer).toContain('Não foi possível recuperar a informação com a fonte verificada')
    expect(res.body.source_document).toBeUndefined()
  })
})
```

### Melhorias aplicadas

| Melhoria | Justificativa |
|----------|---------------|
| `import { describe, it, expect } from 'vitest'` | Framework correto conforme AGENTS.md |
| `setupServer` + `mockAzureOpenAICompletion` | Isolamento: sem chamadas reais a serviços externos |
| Nome descritivo com `should [comportamento] when [condição]` | Padrão Testing Standards: falha indica exatamente o que quebrou |
| `expect(res.body.answer).toContain('7 dias úteis')` | Verifica conteúdo factual, não apenas existência |
| `expect(res.body.source_document).toMatch(/POL-001/)` | Verifica campo obrigatório do structured output |
| `expect(res.body.confidence_score).toBeGreaterThan(0)` | Verifica presença e validade do score |
| Segundo caso: `source_document` ausente → fallback | Testa o guardrail do response-validator diretamente |
| `queries.returnDeadline` de fixtures | Dado de domínio reutilizável, não hardcoded |
| Arrange / Act / Assert comentados | Padrão Testing Standards para legibilidade |
