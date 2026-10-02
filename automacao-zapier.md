# Automação no Zapier (formulário → IA → planilha → e-mail)

Eu montei este Zap para receber pedidos de orçamento da Luminus, usar IA para analisar as informações, salvar tudo em uma planilha e enviar uma resposta por e-mail.

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
- No Webflow, eu cadastrei um webhook do tipo `form_submission` e apontei para a URL gerada pelo Zapier. Assim, cada pedido enviado pelo formulário vai direto para o Zap.
- Eu não usei o gatilho "Webflow > New Form Submission" porque ele só mostrava um exemplo genérico ("Joe Bloggs", apenas nome e e-mail). Ele também não puxava os campos que eu criei em HTML. Com o webhook, todos os campos chegam corretamente.

Os campos chegam dentro de `payload.data`: `Name`, `Email`, `WhatsApp`, `Aparelho`, `Servico`, `Descricao`, `Triagem do site`. Também chega `payload.submittedAt`, que traz a data.

## Passo 2: AI by Zapier

Neste passo, eu usei a ferramenta de IA do próprio Zapier. Gostei dessa opção porque dá para definir os campos de saída. Desse jeito, a IA já devolve cada informação separada, e eu não precisei escrever código para tratar a resposta. Também não precisei criar nenhuma chave de API.

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
- Eu configurei cada coluna para receber o campo do webhook com o mesmo nome. As três últimas colunas recebem `categoria`, `urgencia` e `resposta` do passo de IA.
- Colunas (linha 1): `Data | Nome | Email | WhatsApp | Aparelho | Servico | Descricao | Triagem do site | Categoria IA | Urgencia IA | Resposta enviada`

## Passo 4: Gmail

- Evento: **Send Email**
- Para: `{Payload Data Email}`
- Assunto: `Luminus | Recebemos seu pedido de orçamento`
- Corpo: `{resposta}`
- Nome do remetente: `Luminus Assistência Técnica`

## Teste

1. Eu enviei um pedido pelo site.
2. Verifiquei o Zap rodando no histórico (Zap history).
3. Conferi a nova linha na planilha e o e-mail na caixa de entrada.

## Resultado do teste

No teste final, feito pelo próprio site (assistente → "Bateria fraca" → formulário), o Webflow enviou o webhook, o Zap executou os 4 passos e a IA classificou o pedido como **Troca de bateria / urgência média**.

Nos testes anteriores, a IA classificou "Deixei cair na piscina e agora a tela não acende" como **Aparelho molhado / urgência alta**. Também classificou "notebook lento, esquenta demais, ventoinha faz barulho" como **Limpeza e pasta térmica / urgência baixa**. As duas linhas foram para a planilha e os e-mails foram enviados.

![Resposta da IA](prints/08-zapier-ia-resposta.png)
![Pedido enviado no site](prints/10-pedido-enviado.png)
![Planilha](prints/09-planilha-pedidos.png)
