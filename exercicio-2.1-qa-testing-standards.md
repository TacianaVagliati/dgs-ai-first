# Exercício 2.1 — QA: Testing Standards para o AGENTS.md

> **Papel:** QA  
> **Cenário:** 2 — Fase de Estruturação do Trabalho  
> **Ferramenta usada:** Claude (chat)  
> **Data:** 2026-06-14  
> **Versão:** 1.0

---

## Entregável 1 — Seção "Testing Standards" do AGENTS.md

```markdown
## Testing Standards (QA)

> Esta seção é lida por agentes de IA (Copilot, Claude Code) antes de gerar qualquer código de teste.
> Todas as regras são prescritivas. Agentes DEVEM seguir sem adaptação criativa.

### Nomenclatura

- DEVE usar `describe('NomeDoModulo', () => { ... })` para agrupar testes por módulo.
- DEVE usar `it('should [comportamento] when [condição]')` para descrever cada caso de teste.
- Nomes DEVEM estar em inglês.
- NÃO DEVE usar nomes genéricos como `'works'`, `'test'`, `'funciona'`, ou `'it should work'`.

Exemplos corretos:
```typescript
describe('QueryHandler', () => {
  it('should return source_document field when query matches SLA document', async () => { ... })
  it('should return explicit rejection when query involves dangerous goods return', async () => { ... })
  it('should return fallback message when no chunk matches the query', async () => { ... })
})
```

### Estrutura obrigatória: Arrange / Act / Assert

Todo teste DEVE ter as três seções separadas e comentadas:

```typescript
it('should include source_document in response for valid query', async () => {
  // Arrange
  const query = 'Qual o SLA de resposta para cliente Gold?'
  const mockChunks = [chunks.sla2024B] // de /tests/fixtures/chunks.ts
  server.use(mockAzureSearchResponse(mockChunks))

  // Act
  const response = await handler(buildRequest({ question: query }))

  // Assert
  expect(response.statusCode).toBe(200)
  const body = JSON.parse(response.body)
  expect(body.source_document).toMatch(/SLA-2024/)
  expect(body.answer).toContain('2h úteis')
})
```

### Assertions

- DEVE fazer assertions específicas ao comportamento (campos concretos, valores esperados).
- NÃO DEVE usar `toBeDefined()` ou `toBeTruthy()` como única assertion.
- NÃO DEVE usar `toEqual({})` ou `toMatchObject({})` sem propriedades concretas.
- Para respostas JSON: DEVE verificar ao menos `statusCode`, `body.answer`, e `body.source_document`.

### Mocking

- DEVE usar `msw` (Mock Service Worker) para interceptar chamadas HTTP externas:
  - Azure OpenAI (completions e embeddings)
  - Azure AI Search (query endpoint)
- NÃO DEVE fazer chamadas reais a serviços externos em testes.
- NÃO DEVE usar `jest.spyOn` ou `vi.spyOn` em módulos internos sem comentário explicativo.
- Handlers msw DEVEM estar em `/tests/mocks/handlers.ts`.

```typescript
// CORRETO
import { server } from '../mocks/server'
import { mockAzureSearchResponse } from '../mocks/handlers'
server.use(mockAzureSearchResponse([chunks.pol001B]))

// INCORRETO
vi.spyOn(searchService, 'query').mockResolvedValue(fakeResult)
```

### Fixtures

- Fixtures reutilizáveis DEVEM estar em `/tests/fixtures/`:
  - `chunks.ts` — chunks do corpus NovaTech (POL-001, PROC-042v2, SLA-2024)
  - `queries.ts` — perguntas reais do domínio de logística
  - `expected-responses.ts` — respostas esperadas para queries conhecidas
- Dados de teste DEVEM ser do domínio NovaTech (carga perigosa, SLA Gold, frete Manaus, devoluções).
- NÃO DEVE usar dados genéricos como `"test"`, `"hello"`, `{ question: "qualquer coisa" }`.

```typescript
// /tests/fixtures/queries.ts — CORRETO
export const queries = {
  slaGold: 'Qual o prazo de resposta para cliente Gold em chamado crítico?',
  dangerousGoodsReturn: 'Posso devolver carga perigosa (líquido inflamável)?',
  freteManaus: 'Qual o multiplicador de frete para 600kg entregue em Manaus?',
  unknownCoverage: 'Qual o frete padrão para 300kg para Salvador?',
}
```

### Proibições absolutas

- NÃO DEVE acessar serviços reais (Azure, APIs externas) em nenhum teste.
- NÃO DEVE ter testes que dependem da ordem de execução (cada teste DEVE ser independente).
- NÃO DEVE usar `console.log` nos testes.
- NÃO DEVE hardcodar valores que estão nas fixtures (importar das fixtures).
- NÃO DEVE testar implementação interna (métodos privados, estado interno de classes).
```

---

## Entregável 2 — Teste Reescrito: Antes e Depois

### Teste original (gerado pelo Copilot sem guidance)

```typescript
// Teste gerado pelo Copilot sem guidance
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

### Problemas identificados

| Problema | Impacto |
|----------|---------|
| Nome genérico `'query endpoint works'` | Não descreve comportamento. Falha não informa o que quebrou. |
| Dados genéricos `"test"` | Não exercita lógica de domínio. Poderia passar com qualquer resposta. |
| `expect(result).toBeDefined()` como única assertion | Passa mesmo se o endpoint retornar `500` ou resposta vazia. |
| Sem arrange/act/assert | Difícil de ler e manter. |
| Sem mock | Chamada real ao Azure (ou falha por ausência de credenciais no CI). |

### Teste reescrito seguindo os Testing Standards

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { setupServer } from 'msw/node'
import { handler } from '../../src/functions/query/handler'
import { mockAzureOpenAICompletion, mockAzureSearchResponse } from '../mocks/handlers'
import { chunks } from '../fixtures/chunks'
import { queries } from '../fixtures/queries'

const server = setupServer()
beforeAll(() => server.listen())
afterAll(() => server.close())

describe('QueryHandler', () => {
  it('should return answer with source_document when query matches SLA Gold document', async () => {
    // Arrange
    server.use(
      mockAzureSearchResponse([chunks.sla2024B, chunks.sla2024C]),
      mockAzureOpenAICompletion('O SLA de resposta para cliente Gold é de até 2h úteis.')
    )
    const request = {
      method: 'POST',
      body: JSON.stringify({ question: queries.slaGold }),
      headers: { 'Content-Type': 'application/json' },
    }

    // Act
    const response = await handler(request as any)

    // Assert
    expect(response.statusCode).toBe(200)
    const body = JSON.parse(response.body)
    expect(body.answer).toContain('2h úteis')
    expect(body.source_document).toMatch(/SLA-2024/)
  })
})
```

### O que melhorou

1. **Nome descritivo** — o `it(...)` descreve exatamente o comportamento e a condição testada.
2. **Dados de domínio** — usa `queries.slaGold` (do fixture) em vez de `"test"`.
3. **Mock explícito** — `msw` intercepta Azure Search e OpenAI; sem chamada real.
4. **Arrange/Act/Assert** — estrutura clara facilita manutenção e leitura.
5. **Assertions específicas** — verifica `statusCode`, conteúdo da `answer` e presença de `source_document`.

---

## Entregável 3 — Critérios de Review de Testes Gerados por IA

Os 3 critérios abaixo são objetivos: dois QAs analisando o mesmo teste chegam à mesma conclusão.

### Critério 1 — Presença de assertion específica ao comportamento

**Aprovado:** O teste faz ao menos uma assertion sobre conteúdo concreto da resposta (valor de campo, status code, texto esperado).  
**Reprovado:** A única assertion é `toBeDefined()`, `toBeTruthy()`, ou equivalente genérico.

**Como verificar:** Leia o bloco `// Assert`. Se remover a lógica de negócio do handler, o teste ainda passaria? Se sim, reprovado.

---

### Critério 2 — Ausência de chamadas reais a serviços externos

**Aprovado:** Todas as chamadas HTTP (Azure OpenAI, Azure AI Search) são interceptadas por `msw`.  
**Reprovado:** O teste usa credenciais reais, faz fetch sem mock, ou importa `AzureOpenAI` diretamente no teste sem stub.

**Como verificar:** Busque `process.env.AZURE_`, `new AzureOpenAI`, ou `fetch(` nos imports e no corpo do teste. Se encontrar sem mock correspondente, reprovado.

---

### Critério 3 — Dados de teste do domínio NovaTech

**Aprovado:** Perguntas e chunks nos testes são do domínio de logística (carga perigosa, SLA Gold, frete especial, devolução, Manaus/Norte).  
**Reprovado:** Dados genéricos como `"test"`, `"hello"`, `{ question: "anything" }` sem justificativa.

**Como verificar:** Leia o bloco `// Arrange`. Se a pergunta não faz referência ao domínio NovaTech, reprovado.
