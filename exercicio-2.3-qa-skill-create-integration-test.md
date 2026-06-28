# Exercício 2.3 — QA: Skill create-integration-test

**Papel:** QA
**Cenário:** 2 — Fase de Estruturação do Trabalho
**Ferramentas usadas:** Claude (chat) + Claude Cowork
**Data:** 2026-06-14
**Versão:** 1.0
**Status:** Entregável final

---

## Objetivo

Definir uma skill prescritiva para geração de testes de integração, com template, exemplos DO/DON'T, anti-padrões e checklist rápido de revisão.

## Entradas consideradas

- Cenário completo
- Testing Standards (resultado do exercício 2.1)
- Anti-padrões e boas práticas de testes gerados por IA

## Rastreabilidade

- **Exercício relacionado:** 2.3
- **Artefato principal:** exercicio-2.3-skill-create-integration-test.md
- **Artefatos complementares:** exercicio-2.3-qa-skill-create-integration-test.md

---

# SKILL.md — create-integration-test

**Nível:** Artifact
**Localização no repositório:** `/skills/artifact/create-integration-test.md`
**Dependências (ler antes desta skill):**

- `/skills/foundation/typescript-conventions.md` — convenções de TypeScript (strict mode, imports, naming)
- `/skills/foundation/error-handling.md` — padrão de erros customizados do projeto
- `/skills/domain/testing-patterns.md` — padrões de mocking (msw), fixtures e estrutura geral

---

## Quando usar esta skill

Use quando precisar criar um **teste de integração** para um endpoint Azure Function do projeto NovaTech.

**Frase-ativação:** "Crie um teste de integração para [endpoint/handler]" ou "Escreva testes para [função]".

**NÃO use para:**

- Testes unitários de funções puras (sem I/O).
- Testes do pipeline de ingestão (cenários diferentes de mocking).
- Testes do bot do Teams (usa Bot Framework Test Adapter).

---

## Template base

Substitua os placeholders `[UPPERCASE]` conforme o contexto:

```typescript
import { describe, it, expect, beforeAll, afterAll, afterEach } from 'vitest'
import { setupServer } from 'msw/node'
import { http, HttpResponse } from 'msw'
import { [HANDLER_NAME] } from '../../src/functions/[MODULE]/handler'
import { [MOCK_HELPERS] } from '../mocks/handlers'
import { chunks } from '../fixtures/chunks'
import { queries } from '../fixtures/queries'

// [SERVER_SETUP]: Servidor msw para interceptar chamadas externas
const server = setupServer()
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

describe('[MODULE_NAME]', () => {
  describe('[BEHAVIOR_GROUP]', () => {
    it('should [EXPECTED_BEHAVIOR] when [CONDITION]', async () => {
      // Arrange
      server.use(
        [MOCK_AZURE_SEARCH_HANDLER],
        [MOCK_AZURE_OPENAI_HANDLER]
      )
      const request = buildRequest({ [INPUT_FIELDS] })

      // Act
      const response = await [HANDLER_NAME](request)

      // Assert
      expect(response.statusCode).toBe([EXPECTED_STATUS])
      const body = JSON.parse(response.body)
      expect(body.[FIELD]).toBe([EXPECTED_VALUE])
    })
  })
})
```

---

## Exemplos completos

### ✅ DO — Teste de integração bem escrito

```typescript
import { describe, it, expect, beforeAll, afterAll, afterEach } from "vitest";
import { setupServer } from "msw/node";
import { handler } from "../../src/functions/query/handler";
import {
  mockAzureSearchResponse,
  mockAzureOpenAICompletion,
} from "../mocks/handlers";
import { chunks } from "../fixtures/chunks";
import { queries } from "../fixtures/queries";

const server = setupServer();
beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe("QueryHandler", () => {
  describe("dangerous goods guardrail", () => {
    it("should return explicit rejection when query involves dangerous goods return", async () => {
      // Arrange
      server.use(
        mockAzureSearchResponse([chunks.pol001B]),
        mockAzureOpenAICompletion(
          "Cargas perigosas (classes 1-6 ANTT) não são elegíveis para devolução pelo processo padrão. " +
            "Entre em contato com a Gestão de Riscos pelo ramal 4500.",
        ),
      );
      const request = buildRequest({ question: queries.dangerousGoodsReturn });

      // Act
      const response = await handler(request);

      // Assert
      expect(response.statusCode).toBe(200);
      const body = JSON.parse(response.body);
      expect(body.answer).toMatch(
        /não.*elegível|processo especial|Gestão de Riscos/i,
      );
      expect(body.answer).not.toMatch(/7 dias úteis/);
      expect(body.source_document).toMatch(/POL-001/);
    });
  });

  describe("no-match fallback", () => {
    it("should return fallback message and null source_document when no chunk matches", async () => {
      // Arrange
      server.use(
        mockAzureSearchResponse([]), // nenhum chunk relevante
        mockAzureOpenAICompletion(
          "Não encontrei informação sobre isso na documentação disponível.",
        ),
      );
      const request = buildRequest({ question: queries.unknownCoverage });

      // Act
      const response = await handler(request);

      // Assert
      expect(response.statusCode).toBe(200);
      const body = JSON.parse(response.body);
      expect(body.answer).toMatch(/não encontr|não tenho informação/i);
      expect(body.source_document).toBeNull();
    });
  });
});
```

---

### ❌ DON'T — Teste com problemas comuns gerados por IA

```typescript
// ❌ PROBLEMA 1: Nome genérico — não descreve o comportamento
test("query handler works", async () => {
  // ❌ PROBLEMA 2: Sem mock — chama Azure real (falha no CI, custa dinheiro)
  const result = await handler({ body: '{"question": "test"}' });

  // ❌ PROBLEMA 3: Dado genérico "test" — não exercita lógica de domínio
  // ❌ PROBLEMA 4: Assertion vaga — passa mesmo se retornar erro 500
  expect(result).toBeDefined();
});

// ❌ PROBLEMA 5: Spy em módulo interno — testa implementação, não comportamento
test("search service is called", async () => {
  const spy = vi.spyOn(searchService, "query");
  await handler(buildRequest({ question: "SLA Gold?" }));
  expect(spy).toHaveBeenCalled(); // ❌ Se refatorar o nome do método, o teste quebra
});

// ❌ PROBLEMA 6: Mock permissivo demais — qualquer resposta passa
vi.mock("../../src/services/search", () => ({
  search: vi.fn().mockResolvedValue({ results: [] }), // ❌ Não valida que os chunks corretos foram recuperados
}));
```

**Por que cada problema é real:** LLMs tendem a gerar testes que compilam e passam sem realmente validar o comportamento. Os anti-padrões acima fazem o teste passar com uma implementação vazia ou quebrada.

---

## Anti-padrões específicos de testes gerados por IA

| Anti-padrão                            | Como identificar                                            | Como corrigir                                                                                    |
| -------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `toBeDefined()` como única assertion   | `expect(result).toBeDefined()` sem assertions adicionais    | Adicionar assertions sobre campos concretos: `statusCode`, `body.answer`, `body.source_document` |
| Dado genérico na pergunta              | `"question": "test"` ou `"question": "hello"`               | Usar fixtures de queries do domínio: `queries.slaGold`, `queries.dangerousGoodsReturn`           |
| Spy em método interno                  | `vi.spyOn(searchService, 'query')` como assertion principal | Usar `msw` para interceptar HTTP e verificar resposta final                                      |
| Mock permissivo que não valida chunks  | `mockResolvedValue({ results: [] })` para qualquer query    | Usar `mockAzureSearchResponse([chunks.específico])` — chunk correto para cada cenário            |
| Sem tratamento de `onUnhandledRequest` | `setupServer()` sem `{ onUnhandledRequest: 'error' }`       | Configurar `onUnhandledRequest: 'error'` para detectar chamadas reais não mockadas               |
| Testes com ordem dependente            | `it` que usa estado do teste anterior                       | Usar `afterEach(() => server.resetHandlers())` e estado local por teste                          |

---

## Dependências declaradas

Antes de usar esta skill, o agente DEVE ler:

1. **`/skills/foundation/typescript-conventions.md`** — para nomear variáveis, usar imports corretos, e seguir strict mode.
2. **`/skills/foundation/error-handling.md`** — para saber como o projeto trata erros e o que testar em casos de falha.
3. **`/skills/domain/testing-patterns.md`** — para entender como usar `msw`, como estruturar fixtures, e onde ficam os arquivos de mock.

Sem essas skills, o agente pode gerar testes com imports inexistentes, mocks incompatíveis com a versão do msw do projeto, ou fixtures com estrutura errada.

---

# Checklist de Revisão de Testes

**Estimativa:** ~90 segundos por teste
**Uso:** QA revisa cada teste gerado por IA antes do merge

---

## Bloco 1 — Nomenclatura (20 segundos)

- [ ] O `describe` usa o nome do módulo/handler (ex: `QueryHandler`, `FeedbackHandler`)?
- [ ] O `it` começa com `'should'` e descreve comportamento + condição?
- [ ] Nenhum nome genérico como `'works'`, `'test'`, `'funciona'`?

---

## Bloco 2 — Estrutura (20 segundos)

- [ ] Os comentários `// Arrange`, `// Act`, `// Assert` estão presentes?
- [ ] O bloco `// Assert` tem ao menos uma assertion sobre conteúdo concreto (campo, valor, regex)?
- [ ] `toBeDefined()` NÃO é a única assertion?

---

## Bloco 3 — Dados de Teste (20 segundos)

- [ ] A pergunta no `// Arrange` é do domínio NovaTech (carga perigosa, SLA Gold, frete, devolução)?
- [ ] Os chunks no mock são os corretos para o cenário (ex: `chunks.pol001B` para carga perigosa)?
- [ ] Nenhum dado genérico como `"test"`, `"hello"`, `"anything"`?

---

## Bloco 4 — Mocking (20 segundos)

- [ ] `msw` está configurado com `setupServer()`, `beforeAll`, `afterEach`, `afterAll`?
- [ ] `onUnhandledRequest: 'error'` está configurado no `setupServer`?
- [ ] Nenhum `vi.spyOn` em módulo interno como principal mecanismo de validação?
- [ ] Nenhuma chamada real a `process.env.AZURE_` sem mock?

---

## Bloco 5 — Guardrails do domínio (10 segundos — apenas para testes de VC-03)

- [ ] Testes de carga perigosa + devolução verificam que "7 dias úteis" NÃO aparece na resposta?
- [ ] Testes de tier inexistente verificam que SLAs inventados NÃO aparecem?

---

**Resultado:** Se algum item estiver marcado como ❌, o teste retorna para reescrita antes do merge.

---

## Conclusão

Esta skill foi definida para guiar a geração de testes de integração com padrão consistente, reduzindo anti-padrões frequentes de código gerado por IA.

### Critérios de qualidade atendidos

- Template acionável com placeholders claros
- Exemplos DO/DON'T e anti-padrões verificáveis
- Checklist rápido para revisão objetiva em QA

### Observações finais

A skill deve permanecer alinhada ao AGENTS.md (Testing Standards) e às dependências Foundation/Domain antes de qualquer revisão de versão.

---

_Fim do entregável._
