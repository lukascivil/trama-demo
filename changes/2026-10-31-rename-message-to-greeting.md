---
id: 2026-10-31-rename-message-to-greeting
title: Renomear message para greeting no GET /hello
status: open
touches: ["GET /hello"]
steps:
  - order: 1
    repo: trama-demo-api
    action: Adicionar greeting ao GET /hello, mantendo message
    branch: trama/rename-message-to-greeting
    pr: https://github.com/lukascivil/trama-demo-api/pull/1
    state: open
  - order: 2
    repo: trama-demo-web
    action: Ler greeting em vez de message
    branch: trama/rename-message-to-greeting
    pr: https://github.com/lukascivil/trama-demo-web/pull/1
    state: open
  - order: 3
    repo: trama-demo-api
    action: Remover message do GET /hello
    state: pending
---

## Por quê

Pedido: "renomear message para greeting no GET /hello". O campo da resposta traz a saudação, então deve se chamar `greeting`, não `message`.

## Impacto

`GET /hello` é fornecido por `trama-demo-api` e consumido só por `trama-demo-web` (`trama uses "GET /hello"`). Os dois cards estão em dia (`trama status`).

`trama-demo-api` (provedor):

- `internal/api/api.go:13-15`: `HelloResponse` declara `Message string` com a tag `json:"message"`.
- `internal/api/api.go:20`: `Hello` responde `HelloResponse{Message: "Hello, World"}`.
- `internal/api/api.go:42`: registra a rota `GET /hello`.
- `internal/api/api_test.go:32-38`: o teste decodifica com o próprio `api.HelloResponse` e confere `body.Message`.
- `README.md:9` e `CLAUDE.md:7`: documentam a resposta `{"message": "Hello, World"}`.

`trama-demo-web` (consumidor):

- `src/api/hello.ts:3`: `type HelloResponse = { message: string }`.
- `src/api/hello.ts:5`: o comentário diz que `fetchHello` devolve a `message`.
- `src/api/hello.ts:15`: `return body.message`, a leitura real do campo.
- `src/api/hello.test.ts:7-8` e `src/api/hello.test.ts:14`: os mocks de `fetch` respondem `{ message: "Hello, World" }`.
- Fora do impacto: `src/App.tsx:6`, `src/App.tsx:14` e `src/App.tsx:21` usam `message` como nome local da string que `fetchHello` devolve, sem ler a resposta; `src/App.tsx:15` é `err.message`.

Quebra ou compatível:

- Renomear direto quebra `trama-demo-web`: `body.message` passaria a ser `undefined` e a página mostraria um título vazio.
- Adicionar `greeting` mantendo `message` é compatível: é um campo novo.

Testes que não pegariam a quebra:

- `trama-demo-api`: o teste decodifica com `api.HelloResponse`, então renomear o campo e a tag juntos continua passando.
- `trama-demo-web`: os testes mockam `fetch` com o formato antigo e continuariam passando contra uma API que já não manda `message`.

Relações pendentes: nenhuma; `trama uses` não lista consumidores `pending in`.

Cards: depois dos merges, `/trama:map trama-demo-api` (o `shape` muda) e `/trama:map trama-demo-web` (os `fields_used` mudam).

## Plano

Ordem expandir → migrar → contrair:

1. `trama-demo-api`: devolver `greeting` ao lado de `message`, marcando `message` como obsoleto; o teste passa a conferir as chaves do JSON, sem o tipo do próprio pacote; README e `CLAUDE.md` atualizados.
2. `trama-demo-web`: ler `greeting` (tipo, função e testes). O pull request abre como rascunho e só entra depois do merge e deploy do passo 1.
3. `trama-demo-api`: remover `message`, depois do merge e deploy do passo 2. Fica para depois.

## Execução

Ainda não começou.

## Pendente

- Passo 3, quando `trama-demo-web` com `greeting` estiver em produção.
