# Automação no Zapier (formulário → IA → planilha → e-mail)

Zap: **Luminus | Pedido de orçamento com IA**

```
Webflow (webhook de envio de formulário)
   └─> Webhooks by Zapier (Catch Hook)
         └─> AI by Zapier (lê o pedido, classifica e escreve a resposta)
               ├─> Google Sheets (salva o pedido na planilha)
               └─> Gmail (manda a resposta pro cliente)
```

## Passo 1: gatilho
- App: **Webhooks by Zapier**, evento **Catch Hook**
- No Webflow cadastrei um webhook do tipo `form_submission` apontando pra URL que o Zapier gerou. Assim todo pedido do formulário vai direto pro Zap.
- Por que não usei o gatilho "Webflow > New Form Submission": ele só mostrava um exemplo genérico ("Joe Bloggs", só nome e e-mail) e não puxava os campos que eu fiz em HTML. Com o webhook chegam todos os campos.

Campos que chegam (dentro de `payload.data`): `Name`, `Email`, `WhatsApp`, `Aparelho`, `Servico`, `Descricao`, `Triagem do site`, e também `payload.submittedAt` com a data.

## Passo 2: AI by Zapier
Usei o passo de IA do próprio Zapier. A vantagem é que nele dá pra definir os campos de saída, então a IA já devolve tudo separado e não precisei de código pra tratar a resposta. Também não precisei criar chave de API nenhuma.

**Campos de saída** (todos obrigatórios):

| Campo | Descrição |
|---|---|
| `categoria` | Um destes: Troca de tela, Troca de bateria, Conector de carga, Aparelho molhado, Formatação e otimização, Limpeza e pasta térmica, Diagnóstico na loja |
| `urgencia` | baixa, media ou alta |
| `resposta` | Texto do e-mail para o cliente, pronto para enviar |

**Modelo:** Standard (1 tarefa por execução, é suficiente pra esse caso).

**Prompt** (os campos entre chaves são os dados do pedido, inseridos com `/`):

```
Você é o atendente da Luminus Assistência Técnica, uma loja pequena de conserto de celular e notebook no centro de São Paulo. Leia o pedido de orçamento abaixo e preencha os campos de saída.

categoria: escolha só uma destas: Troca de tela, Troca de bateria, Conector de carga, Aparelho molhado, Formatação e otimização, Limpeza e pasta térmica, Diagnóstico na loja.
urgencia: escreva baixa, media ou alta. É alta quando o aparelho molhou, a bateria estufou, tem cheiro de queimado ou a pessoa pode perder dados.
resposta: um e-mail curto (até 120 palavras) para o cliente, em português do Brasil, simpático e simples, chamando pelo primeiro nome. Explique o que provavelmente aconteceu, o que fazer agora e diga que o diagnóstico na loja é grátis. Use as faixas de preço abaixo, mas não prometa valor exato. Assine como Equipe Luminus.

Faixas de preço: tela R$ 180 a R$ 650; bateria R$ 120 a R$ 350; conector R$ 90 a R$ 220; aparelho molhado R$ 150 a R$ 380; formatação R$ 120 a R$ 180; limpeza R$ 130 a R$ 200.

Dados do pedido
Nome: {Payload Data Name}
Aparelho: {Payload Data Aparelho}
Serviço que o cliente marcou: {Payload Data Servico}
Triagem automática do site: {Payload Data Triagem Do Site}
Descrição do cliente: {Payload Data Descricao}
```

## Passo 3: Google Sheets
- Evento: **Create Spreadsheet Row**
- Planilha: **Luminus - Pedidos de orçamento**, aba `Pedidos`
- Cada coluna recebe o campo do webhook com o mesmo nome; as três últimas recebem `categoria`, `urgencia` e `resposta` do passo de IA
- Colunas (linha 1): `Data | Nome | Email | WhatsApp | Aparelho | Servico | Descricao | Triagem do site | Categoria IA | Urgencia IA | Resposta enviada`

## Passo 4: Gmail
- Evento: **Send Email**
- Para: `{Payload Data Email}`
- Assunto: `Luminus | Recebemos seu pedido de orçamento`
- Corpo: `{resposta}`
- Nome do remetente: `Luminus Assistência Técnica`

## Teste
1. Enviar um pedido pelo site
2. Ver o Zap rodar no histórico (Zap history)
3. Conferir a linha nova na planilha e o e-mail na caixa de entrada

## Resultado do teste

Teste final feito pelo próprio site (assistente → "Bateria fraca" → formulário): o Webflow mandou o webhook, o Zap rodou os 4 passos e a IA classificou como **Troca de bateria / urgência média**.

Nos testes anteriores, a IA classificou "Deixei cair na piscina e agora a tela não acende" como **Aparelho molhado / urgência alta** e "notebook lento, esquenta demais, ventoinha faz barulho" como **Limpeza e pasta térmica / urgência baixa**. As duas linhas foram pra planilha e os e-mails foram enviados.

![Resposta da IA](prints/08-zapier-ia-resposta.png)
![Pedido enviado no site](prints/10-pedido-enviado.png)
![Planilha](prints/09-planilha-pedidos.png)
