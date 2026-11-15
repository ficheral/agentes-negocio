# Agentes de Negócio

Playbook e implementações para vender **resultado** usando agentes de IA — não ferramenta, não curso, não hora técnica.

> A pergunta que este repositório responde: se qualquer um cria software em minutos com IA, o que ainda dá dinheiro?

## A resposta em uma frase

**O modelo de IA virou commodity. A distribuição, o dado proprietário e a confiança não.**

Quem vende "mais um agente que responde perguntas" está competindo com o OpenAI. Quem vende "eu reduzo sua inadimplência em 20%" tem um negócio.

## O que mudou (e por que a abordagem antiga morreu)

| Antes | Agora |
|---|---|
| Construir software era caro → vendia-se o software | Construir é barato → vender software perdeu valor |
| Curso ensinava o que ninguém sabia | O modelo já sabe → curso genérico virou commodity |
| Cobrava-se por hora ou por assento | Cobra-se por resultado entregue |
| O diferencial era a tecnologia | O diferencial é o acesso ao problema real |

O gargalo saiu da **construção** e foi para a **confiança e a distribuição**. Isso muda tudo sobre como conquistar cliente.

## O que as grandes empresas realmente colocaram em produção

Não é chatbot. É **agente que executa etapa de processo com supervisão humana**:

1. **Um processo específico**, não "atendimento geral"
2. **Integrado ao sistema de registro** (ERP, CRM, sistema interno) — é isso que faz o agente trabalhar
3. **Human-in-the-loop** nos pontos de decisão com consequência financeira
4. **Avaliação contínua** — conjunto de casos com resposta esperada, medida a cada mudança
5. **Guardrails e auditoria** — trilha de tudo que o agente decidiu

O padrão técnico que se consolidou em 2025-2026: **tool use via MCP** (Model Context Protocol), RAG sobre documentos proprietários, e modelos abertos rodando dentro da rede para dado sensível.

## Os documentos

| Arquivo | O que tem |
|---|---|
| [00-TESE.md](00-TESE.md) | Por que a venda de software morreu e o modelo de resultado venceu |
| [01-PLAYBOOK-AQUISICAO.md](01-PLAYBOOK-AQUISICAO.md) | Como conquistar cliente quando construir é grátis — o funil real |
| [02-PRODUTOS.md](02-PRODUTOS.md) | 4 produtos de agente que vendem, com preço e escopo |
| [03-ARQUITETURA.md](03-ARQUITETURA.md) | Como montar tecnicamente: stack, MCP, RAG, evals, custo |
| [04-90-DIAS.md](04-90-DIAS.md) | Plano de execução, semana a semana |

## Regra de ouro

> Não venda o agente. Venda o **número que muda** — e cobre uma fração dele.

O cliente não compra "IA". Ele compra: menos tempo, menos erro, menos gente contratada, mais receita recuperada. Se você não consegue nomear esse número em uma frase, o produto ainda não está pronto.
