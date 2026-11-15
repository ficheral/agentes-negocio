# Arquitetura

Como montar tecnicamente, com o que existe em 2026 e sem exagero.

## Princípio

**Comece com o menor número de peças que resolve.** Cada peça nova é manutenção eterna. O cliente paga pela entrega, não pela elegância da stack.

## A stack mínima (que aguenta produção)

```
┌─────────────────────────────────────────────┐
│  Interface: Next.js (ou só WhatsApp)        │
├─────────────────────────────────────────────┤
│  Orquestração: seu código + tool calls      │
├─────────────────────────────────────────────┤
│  Modelo: API (barato/rápido) ou local       │
├─────────────────────────────────────────────┤
│  Conhecimento: Postgres + pgvector (RAG)    │
├─────────────────────────────────────────────┤
│  Integração: MCP servers / REST do cliente  │
├─────────────────────────────────────────────┤
│  Observabilidade: logs + evals + trilha     │
└─────────────────────────────────────────────┘
```

## Decisão 1: modelo na nuvem ou local

| | Nuvem (API) | Local (aberto) |
|---|---|---|
| Custo | por token | zero marginal |
| Qualidade | melhor | boa o suficiente |
| Dado sai da rede | **sim** | **não** |
| Setup | minutos | horas |
| Quando usar | cliente sem restrição | dado sensível, ou volume alto |

**Regra prática:** comece na nuvem para validar. Migre para local quando (a) o cliente exigir, ou (b) o custo de token passar de R$ 500/mês — aí o local se paga.

Modelos abertos que rodam bem em 2026: **Gemma 4** (12B em 16 GB de VRAM, 31B em 24 GB quantizado), servidos via **Ollama** ou **llama.cpp**. Para dado sensível, é o caminho.

## Decisão 2: como o agente acessa o sistema do cliente

Este é o ponto que separa demo de produção.

**Opção A — MCP (Model Context Protocol)**
Padrão que se consolidou. Você escreve um servidor MCP que expõe as operações do sistema do cliente como ferramentas, e o agente as chama. Vantagem: o agente não precisa saber SQL nem API; ele vê "buscar_titulo", "registrar_cobranca".

```python
# servidor MCP mínimo, conceito
@tool("buscar_titulos_vencidos")
def buscar_titulos(dias: int) -> list[dict]:
    return db.query("SELECT ... WHERE vencimento < now() - %s", dias)
```

**Opção B — integração direta escrita à mão**
Mais rápido no começo, mais frágil depois. Aceitável para o primeiro cliente.

**Nunca:** dar ao agente acesso direto ao banco do cliente com permissão de escrita. Um erro do modelo vira estrago real.

## Decisão 3: RAG sobre o dado proprietário

É o que dá a vantagem. O modelo é o mesmo para todos; o seu índice não.

Três regras que evitam 80% dos problemas:

1. **Chunk por estrutura, não por tamanho** — quebre por seção de documento, não a cada 500 caracteres
2. **Guarde a origem** — toda resposta precisa poder citar de onde veio
3. **Reindexe por evento, não por cron** — quando o documento muda, não três da manhã

`pgvector` no próprio Postgres resolve sem serviço extra até uns 500 mil documentos.

## Decisão 4: em que o agente pode confiar

Classifique cada ação antes de construir:

| Ação | Exemplo | Autonomia |
|---|---|---|
| **Leitura** | consultar título, ler contrato | total |
| **Escrita reversível** | criar rascunho, registrar log | total, com auditoria |
| **Escrita com efeito externo** | enviar e-mail ao cliente, emitir nota | **humano aprova** |
| **Financeira** | cobrar, estornar, pagar | **humano aprova, sempre** |

A regra: **quanto mais irreversível, mais humano no meio.** Nunca dê ao agente uma ação que não pode ser desfeita sem uma confirmação.

## Decisão 5: como saber se está funcionando

Sem isso, o cliente descobre o erro antes de você.

**Evals:** um conjunto de 30 a 50 casos reais com a resposta esperada. Roda a cada mudança de prompt ou modelo. Se a taxa de acerto cair, você reverte antes do cliente reclamar.

```
casos/cobranca-001.json  →  {"entrada": "...", "esperado": "escalar para humano"}
casos/cobranca-002.json  →  {"entrada": "...", "esperado": "enviar lembrete"}
```

**Métrica que importa:** taxa de acerto nos evals + taxa de correção humana em produção. Se o humano corrige mais de 20% dos casos, o agente ainda não está pronto.

## Custo real por cliente pequeno

| Item | R$/mês |
|---|---|
| Servidor (1 VM, 4 vCPU) | 80 a 150 |
| Modelo na nuvem (uso moderado) | 100 a 400 |
| Banco gerenciado (ou na mesma VM) | 0 a 60 |
| **Total** | **180 a 610** |

Contra receita de R$ 1.500 a 3.000/mês por cliente. A margem é o que permite cobrar por resultado.

## Ordem de construção (não fure)

1. Ler e relatar (só leitura, sem ação) — 2 dias
2. Propor a ação, humano executa — 2 dias
3. Executar ação reversível sozinho — 2 dias
4. Executar ação com efeito externo, com aprovação — 2 dias
5. Só então: aumentar autonomia

Quem começa pelo passo 5 entrega um agente que erra e perde o cliente na primeira semana.

## O que não construir

- Painel de administração elaborado (ninguém usa)
- Multi-tenant no primeiro cliente (você não sabe ainda o que é "tenant")
- Sistema próprio de autenticação (use o do cliente)
- Orquestrador visual (código é mais rápido de mudar)
