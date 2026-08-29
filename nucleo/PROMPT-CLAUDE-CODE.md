# Prompts de handoff para o Claude Code

Dois prompts prontos. O primeiro constrói a fase 1 do sistema. O segundo roda o
modelo de livros num PDF, sem construir nada — serve para validar o formato antes.

---

## Prompt A — construir a fase 1

> Copie tudo do bloco abaixo e cole numa sessão nova do Claude Code, aberta na
> pasta do repositório `humancore-cit-formulario` (o plano está em `nucleo/`).
> Se preferir um repositório novo, copie a pasta `nucleo/` para lá antes.

```
Vou construir com você um sistema pessoal de estudo chamado Núcleo. O plano
completo já existe neste repositório — leia estes arquivos antes de qualquer
coisa e trate-os como a especificação:

- nucleo/index.html — o plano: escopo, arquitetura, telas, fases e custos
- nucleo/modelo-livro/prompts.md — o modelo de conversão de livros (fase 3)
- nucleo/modelo-livro/schema.json — o formato de saída desse modelo
- index.html (raiz) — formulário existente, de onde vem a identidade visual

CONTEXTO
É um sistema de uso pessoal, de um usuário só (eu). Não é produto, não vai ser
vendido, não terá outros usuários e nada será publicado. Ele grava e transcreve
as aulas e reuniões que eu assisto, estrutura o conteúdo em resumo/decisões/
ações, guarda minhas anotações e mantém uma biblioteca com busca por
significado. Interface toda em português do Brasil.

O QUE CONSTRUIR AGORA — SOMENTE A FASE 1
1. Envio de arquivo: áudio, vídeo e PDF, com barra de progresso e status.
2. Transcrição em segundo plano, com marcação de tempo, retomando de falha.
3. Estruturação do conteúdo com um modelo de linguagem, saída validada por
   schema: resumo em uma frase, resumo em camadas, decisões, ações e perguntas
   em aberto — cada item apontando o minuto de origem na transcrição.
4. Caderno: anotação livre minha, presa ao trecho e ao horário que a motivou.
   O caderno não gera tarefa nem cartão de revisão. É só anotação.
5. Biblioteca: lista de tudo, filtros por tipo e busca por significado que
   responde citando as fontes.
6. Exportar qualquer item em Markdown.

DECISÕES JÁ TOMADAS — não reabra sem me perguntar
- Next.js (App Router) + TypeScript, hospedado na Vercel.
- Supabase para Postgres, pgvector, storage de arquivos.
- Sem sistema de contas: uma senha única no .env protege o app inteiro.
- Transcrição atrás de uma interface trocável (ex.: `Transcritor`), com duas
  implementações desde já: (a) API na nuvem, (b) whisper rodando localmente por
  linha de comando. A escolha vem de variável de ambiente. Isso não é opcional:
  é o que me deixa zerar o custo por hora depois.
- Modelo de linguagem: Claude, via SDK oficial `@anthropic-ai/sdk`, com o modelo
  configurável por tipo de conteúdo. Padrão `claude-sonnet-5`; `claude-haiku-4-5`
  para conteúdo simples e `claude-opus-5` para conteúdo denso. Use structured
  outputs para a saída obedecer ao schema.
- Toda afirmação gerada carrega a origem (minuto na transcrição). Item sem
  origem rastreável não entra na tela.
- Identidade visual: siga a de nucleo/index.html — verde #17a06b, fundo escuro
  no topo, cartões claros, tipografia Familjen Grotesk + IBM Plex Mono.

O QUE NÃO FAZER
- Nada de bot entrando em reunião. Aula online entra como arquivo gravado.
- Nada de baixar vídeo de plataforma de curso paga.
- Sem app de celular (é fase 2), sem processar livros (fase 3), sem tarefas com
  prazo e agenda (fase 4). Não adiante fase.
- Sem contas, permissões, times, cobrança ou qualquer coisa de produto.

COMO TRABALHAR
- Antes de escrever código, me mostre em uma tela só o plano de implementação:
  modelo de dados, rotas e a ordem das entregas. Espere meu ok.
- Trabalhe em commits pequenos, cada um com o app funcionando.
- Peça as chaves de API que faltarem em vez de inventar valores. Nunca commite
  chave nenhuma; use .env.local e deixe um .env.example.
- Se alguma decisão do plano não fizer sentido na hora de implementar, diga
  antes de mudar por conta própria.

PRONTO QUANDO
Eu consigo, rodando `npm run dev` na minha máquina: subir a gravação de uma
aula, ver a transcrição com marcação de tempo, ler o resumo com as ações e
conferir cada uma no minuto de origem, escrever uma anotação presa a um trecho,
achar isso depois pela busca e exportar tudo em Markdown.

Comece lendo os arquivos e me apresentando o plano de implementação.
```

---

## Prompt B — testar o modelo de livros num PDF

> Use este antes de construir qualquer coisa, com um livro que você **já leu** —
> é a única forma de julgar se a saída está boa.

```
Leia nucleo/modelo-livro/prompts.md e nucleo/modelo-livro/schema.json deste
repositório. Eles definem um modelo de seis passos para converter um livro em
framework executável, com sete regras duras.

Aplique esse modelo, na íntegra e na ordem, ao arquivo que vou indicar:
CAMINHO_DO_PDF

Meu contexto, para os passos 4 e 6:
- Sou personal trainer e vou aplicar isto no meu trabalho
- Objetivo com este livro: DESCREVA_EM_UMA_FRASE
- Tenho cerca de TEMPO minutos por semana para aplicar
- Restrições: DESCREVA_SE_HOUVER

Regras que eu quero ver respeitadas, sem exceção:
- Todo item com página e citação literal de até 25 palavras
- Nenhum número que não esteja escrito no livro
- Critério de feito observável em cada passo e cada tarefa
- Máximo de 7 princípios, 5 protocolos e 12 tarefas
- Rode o passo 5 (verificação adversarial) de verdade e me mostre o que foi
  removido e por quê
- o_que_o_livro_nao_responde preenchido

Entregue o framework no formato do schema e, depois, uma versão em Markdown
legível para eu ler. Ao final, me diga a sua avaliação: densidade acionável de
0 a 10 e quantos itens ficaram sem fonte.
```

---

## Depois de rodar

Volte aqui com o resultado. Com o framework do livro em mãos, ajustamos o modelo
ao seu jeito de trabalhar antes de automatizar qualquer coisa — e, se você me
mandar as imagens dos prompts que já usa, comparo campo a campo com este.
