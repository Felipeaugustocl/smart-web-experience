# Luminus Assistência Técnica

Site feito no Webflow para uma assistência técnica de celular e notebook (empresa fictícia), com código próprio em HTML, CSS e JavaScript e uma automação com IA no Zapier.

Trabalho da disciplina **Padrões Web para No Code e Low Code** (UniFECAF).
Aluno: Felipe Augusto Camargo Laosa

- Site publicado: https://unifecaf-trabalho.webflow.io
- Vídeo pitch: _(link do vídeo aqui)_

![Página inicial no desktop](prints/02-site-desktop-topo.png)

## O problema

A Luminus é uma loja pequena de bairro que só tinha Instagram. Todo cliente mandava mensagem perguntando a mesma coisa ("quanto fica pra trocar a tela?", "vocês consertam notebook?") e o dono passava o dia respondendo direct. Quem não recebia resposta rápido ia pra outra loja.

A ideia foi criar um site que já responde o básico na hora e organiza os pedidos de orçamento sozinho, sem precisar de programador fixo e sem gastar com servidor.

## O que o site faz

- Mostra os 6 serviços principais com faixa de preço e prazo
- **Assistente de triagem**: a pessoa escreve o problema do jeito dela ("deixei cair na piscina") e o site mostra na hora qual serviço é, quanto custa mais ou menos, o prazo, a urgência e uma dica do que fazer agora
- Botão "Quero o orçamento disso", que leva pro formulário com os campos já preenchidos
- Formulário de orçamento que dispara uma automação: a IA do Zapier lê o pedido, classifica a urgência, escreve uma resposta e manda por e-mail pro cliente, e o pedido vai pra uma planilha
- Mostra se a loja está aberta ou fechada no momento (pelo horário de Brasília)
- Perguntas frequentes que abrem e fecham
- Botão flutuante do WhatsApp com mensagem pronta
- Layout que funciona no celular, tablet e computador

## Tecnologias e ferramentas

| Ferramenta | Pra que usei |
|---|---|
| Webflow (plano grátis) | Estrutura das páginas, estilos, formulário nativo e publicação |
| HTML | Assistente, perguntas frequentes (`<details>`), campos extras do formulário |
| CSS | Variáveis de cor, animações, foco visível, ajustes de responsividade |
| JavaScript | Lógica do assistente, status aberto/fechado, máscara de telefone, animação ao rolar, botão do WhatsApp |
| Zapier (Webhooks + AI by Zapier) | Automação que roda quando chega um pedido |
| AI by Zapier | Lê o pedido, classifica e escreve a resposta pro cliente |
| Google Sheets | Guarda todos os pedidos |
| Gmail | Envia a resposta automática |

## Personalizações com código

O Webflow monta a estrutura visual (seções, cards, textos, menu, rodapé, formulário). Em cima disso coloquei 5 blocos de código (Code Embed), que estão na pasta [`codigo-personalizado/`](codigo-personalizado):

| Arquivo | Onde fica | O que faz |
|---|---|---|
| `1-estilos-globais.html` | começo da página | Variáveis CSS com as cores da marca, fonte, foco visível no teclado, link "pular para o conteúdo", animação de entrada com `@keyframes`, respeita `prefers-reduced-motion`, coloca `lang="pt-BR"` |
| `2-assistente.html` | seção Assistente | Formulário do assistente + lógica em JS que normaliza o texto (tira acento), procura palavras-chave, dá pontos pra cada serviço e decide a urgência. O resultado aparece numa área com `aria-live` pra leitor de tela anunciar |
| `3-duvidas.html` | seção Dúvidas | Perguntas com `<details>` e `<summary>`, que já funcionam no teclado sem JS |
| `4-scripts-gerais.html` | fim da página | Status aberto/fechado com `Intl.DateTimeFormat`, animação com `IntersectionObserver`, botão do WhatsApp, máscara do telefone, ano no rodapé |
| `5-campos-formulario.html` | dentro do formulário | Campos WhatsApp, Aparelho, Serviço e Descrição feitos na mão, mais um campo escondido com a triagem do assistente |

Algumas coisas que precisei fazer em código porque o Webflow não deixava (ou deixava só no plano pago):

- O plano grátis não deixa colocar código no `<head>` nem mudar o idioma da página, então isso foi pro primeiro embed
- O painel de estilos não aceita `@keyframes` nem `prefers-reduced-motion`
- Os campos que criei pelo painel saíam no site com nome "Field 3" e texto "Example Text", então refiz esses campos em HTML dentro do formulário

## Recurso inteligente

São duas partes.

**1. Triagem no próprio site (JavaScript).** Funciona na hora e sem internet lenta atrapalhar. Testei com 10 frases diferentes e ele acertou as 10 (depois de corrigir "estufou", que no começo ele não reconhecia).

**2. Automação com IA (Zapier).** Quando alguém envia o formulário:

```
Webflow (webhook do formulário) → Webhooks by Zapier → AI by Zapier (classifica e escreve a resposta)
   → Google Sheets (salva o pedido) → Gmail (manda a resposta pro cliente)
```

No passo de IA eu defini três campos de saída (`categoria`, `urgencia` e `resposta`), assim a IA já devolve tudo separado e os próximos passos usam direto.

O passo a passo completo, com o prompt, está em [`automacao-zapier.md`](automacao-zapier.md).

No teste, a IA classificou "Deixei cair na piscina e agora a tela não acende" como aparelho molhado com urgência alta e escreveu o e-mail abaixo. O pedido foi salvo na planilha junto com a classificação.

![Resposta da IA no Zapier](prints/08-zapier-ia-resposta.png)

![Pedido enviado no site](prints/10-pedido-enviado.png)

![Planilha com os pedidos](prints/09-planilha-pedidos.png)
_A linha 4 veio de um pedido real feito no site: o assistente indicou troca de bateria e a IA do Zapier marcou urgência média._

Pensei em chamar uma IA direto do JavaScript do site, mas pra isso a chave da API ia ficar no código-fonte da página e qualquer pessoa conseguiria ver. Com a IA rodando dentro do Zapier não tem chave exposta.

## Prints

| | |
|---|---|
| ![Site no celular](prints/03-site-celular-topo.png) | ![Assistente no celular](prints/06-assistente-celular.png) |
| Página inicial no celular | Assistente no celular |

![Assistente funcionando](prints/04-assistente-funcionando.png)
_Assistente classificando "Deixei cair na piscina e agora a tela não acende"_

![Formulário preenchido](prints/05-formulario-preenchido-pelo-assistente.png)
_Formulário já preenchido depois de clicar em "Quero o orçamento disso"_

![Página inteira](prints/01-site-desktop-completo.png)

## Como testar

1. Abra https://unifecaf-trabalho.webflow.io
2. Vá em **Assistente** e escreva um problema, por exemplo:
   - "a tela trincou e tem listras verdes"
   - "meu notebook tá lento e esquentando"
   - "a bateria estufou"
   - ou clique em um dos exemplos prontos
3. Clique em **Quero o orçamento disso** e veja o formulário preenchido
4. Coloque nome, e-mail de verdade e WhatsApp e envie
5. Em alguns minutos chega um e-mail com a resposta escrita pela IA (pode cair no spam)

Também dá pra testar a responsividade diminuindo a janela ou abrindo no celular, e a acessibilidade navegando só com a tecla Tab.

## Lighthouse (celular)

Rodei o Lighthouse no site publicado, simulando celular. Na primeira vez a acessibilidade deu 97 por causa do contraste do texto amarelo pequeno em cima dos títulos. Escureci a cor (de `#9a6d00` pra `#7a5600`) e rodei de novo:

| Desempenho | Acessibilidade | Boas práticas | SEO |
|---|---|---|---|
| 92 | 100 | 100 | 100 |

![Resultado do Lighthouse](prints/07-lighthouse.png)

## Estrutura da pasta

```
projeto/
├── README.md
├── automacao-zapier.md         passo a passo do Zap
├── codigo-personalizado/       os 5 embeds colocados no Webflow
├── prints/                     evidências
└── teste-local.html            página que usei pra testar o código antes de subir
```

## Limitações

- É uma empresa fictícia, então os preços, depoimentos e o endereço são inventados
- O plano grátis do Webflow mostra o selo "Made in Webflow" e tem limite de 50 envios de formulário
- O gatilho pronto "Webflow" do Zapier não puxava os campos que fiz em HTML, então troquei por um webhook do Webflow ligado no "Webhooks by Zapier"
- A triagem do site é por palavras-chave, então se a pessoa escrever algo muito diferente ele cai em "Diagnóstico na loja"
- O botão do WhatsApp não tem número porque a loja não existe
