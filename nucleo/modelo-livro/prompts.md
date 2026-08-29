# Modelo de conversão de livro em framework executável — v1

Seis passos. Cada um é uma chamada separada ao modelo, com uma tarefa só.
Passo único e prompt gigante produz resumo genérico; é o erro mais comum.

Placeholders entre `{{ }}` são preenchidos pelo sistema.

---

## Passo 1 — Triagem

**Entra:** sumário, prefácio, introdução e conclusão.
**Sai:** tipo do livro e mapa de onde está o conteúdo aplicável.
**Serve para:** não gastar processamento igual em capítulo denso e em capítulo de história.

```
Você vai preparar a leitura de um livro que será convertido em material aplicável.

Recebe apenas: sumário, prefácio, introdução e conclusão.

TAREFAS
1. Classifique o livro em: protocolar, conceitual, narrativo, tecnico ou misto.
   - protocolar: ensina procedimentos com passos e números
   - conceitual: ensina um modo de pensar, sem passo a passo
   - narrativo: ensina por histórias e casos
   - tecnico: referência acadêmica, denso em evidência
2. Escreva a tese provável do livro em uma frase.
3. Para cada capítulo do sumário, atribua densidade acionável de 0 a 10 e diga em
   uma linha o que se espera extrair dele.
4. Aponte os capítulos que podem ser lidos em varredura rápida.
5. Liste os termos próprios do autor que devem ser preservados na saída final.

REGRAS
- Baseie-se somente nos trechos recebidos. Onde não houver base, escreva "indeterminado".
- Não resuma o livro. Você está planejando a leitura, não fazendo a leitura.

SAÍDA: JSON no formato TRIAGEM.

<material>
{{sumario_prefacio_introducao_conclusao}}
</material>
```

---

## Passo 2 — Extração (roda uma vez por bloco do livro)

**Entra:** um bloco de texto com marcação de página, mais o resultado da triagem.
**Sai:** unidades brutas, cada uma com citação literal e página.
**Serve para:** garantir que tudo que vier depois esteja preso ao texto original.

```
Você está extraindo unidades aplicáveis de um trecho de livro.

CONTEXTO: livro do tipo {{tipo_livro}}, tese provável: {{tese}}.
O trecho vem marcado com [p.N] indicando a página.

Para cada unidade encontrada, devolva:
- tipo: principio | procedimento | parametro | evidencia | erro | metrica | heuristica | definicao | decisao
- enunciado: a ideia em suas palavras, no máximo 30 palavras
- citacao_literal: até 25 palavras copiadas exatamente do trecho, que sustentam o enunciado
- pagina: o número da marcação mais próxima
- confianca: alta se o autor afirma diretamente, media se você juntou dois trechos, baixa se é interpretação

REGRAS
- Extraia apenas o que está neste trecho. Não use conhecimento externo sobre o tema.
- Não resuma o trecho. Ou a unidade é aplicável, ou não entra.
- Não repita a mesma ideia em duas unidades.
- Números, faixas e doses só entram se aparecerem escritos no texto.
- Se o trecho for só história, exemplo ou digressão, devolva lista vazia. Lista vazia
  é uma resposta correta e esperada em boa parte dos blocos.

SAÍDA: array JSON de UNIDADE. Nada além do JSON.

<trecho>
{{bloco_com_marcacao_de_pagina}}
</trecho>
```

---

## Passo 3 — Consolidação

**Entra:** todas as unidades do livro inteiro.
**Sai:** unidades agrupadas, sem repetição, ordenadas por importância.
**Serve para:** o autor repete a mesma ideia em cinco capítulos; isso vira força, não cinco itens.

```
Você recebe todas as unidades extraídas de um livro, com id, tipo, página e citação.

TAREFAS
1. Agrupe as unidades que dizem a mesma coisa. O grupo mantém TODAS as páginas como fonte.
2. Ordene os grupos por importância, calculada como:
   recorrência no livro x centralidade em relação à tese x aplicabilidade prática.
3. Marque contradições internas: pontos onde o próprio livro se contradiz.
4. Descarte exemplos isolados que não sustentam nenhuma regra.

REGRAS
- Não crie unidades novas. Não reescreva enunciados além do necessário para fundir duplicatas.
- Se duas unidades se parecem mas têm condição de uso diferente, mantenha separadas.

SAÍDA: JSON no formato CONSOLIDADO.

<unidades>
{{unidades_de_todos_os_blocos}}
</unidades>
```

---

## Passo 4 — Síntese do framework

**Entra:** consolidado + triagem + perfil do leitor.
**Sai:** o framework preenchido no schema.
**Serve para:** é o passo que gera o material. Os limites de quantidade são o coração do prompt.

```
Você vai montar o framework executável de um livro, para uma pessoa específica aplicar.

LIVRO: {{titulo}}, tipo {{tipo_livro}}.
QUEM VAI APLICAR: {{perfil_do_leitor}}
OBJETIVO DECLARADO: {{objetivo}}
RESTRIÇÕES: {{restricoes}}

Preencha o schema FRAMEWORK_V1 usando exclusivamente as unidades consolidadas.

LIMITES RÍGIDOS
- No máximo 7 princípios.
- No máximo 5 protocolos.
- No máximo 12 tarefas somadas no plano de 30 dias.
Estes limites existem para forçar escolha. Se sobrar coisa boa de fora, ela vai para
perguntas_de_revisao, não para dentro dos limites.

REGRAS DE PREENCHIMENTO
- Todo item afirmativo carrega fonte com página e citação literal. Item sem fonte não entra.
- Preserve o vocabulário do autor nos nomes; explique com suas palavras nos enunciados.
- Todo passo e toda tarefa terminam em criterio_de_feito observável por outra pessoa.
  "Entender melhor" não é critério. "Planilha preenchida com 6 alunos" é.
- Números só onde o livro deu número. Faltou número? Campo vazio e uma entrada em
  o_que_o_livro_nao_responde.
- Livro conceitual ou narrativo: deixe protocolos vazio e preencha heuristicas.
  Não invente passo a passo que o autor não escreveu.
- O plano de 30 dias precisa caber em {{tempo_semanal_min}} minutos por semana e
  respeitar as restrições acima.
- o_que_o_livro_nao_responde é obrigatório e nunca vem vazio. Nenhum livro cobre tudo.

SAÍDA: JSON válido conforme FRAMEWORK_V1. Nada além do JSON.

<consolidado>
{{consolidado}}
</consolidado>
```

---

## Passo 5 — Verificação adversarial

**Entra:** o framework + as unidades originais.
**Sai:** veredito item a item.
**Serve para:** este passo é o que separa material confiável de texto plausível. Não pule.

```
Você é um revisor cético. Seu trabalho é achar problema, não elogiar.

Recebe um framework gerado a partir de um livro e as unidades originais com citações.

Para CADA item do framework, responda:
1. A citação anexada sustenta mesmo a afirmação, ou foi esticada?
2. Todo número presente aparece no livro, ou apareceu do nada?
3. O item é acionável e verificável, ou é conselho vago?
4. Veredito: manter | reescrever | remover, com o motivo em uma linha.

Depois:
- Liste tudo que foi afirmado sem base nas unidades.
- Complete o_que_o_livro_nao_responde com as lacunas que você percebeu.
- Dê confianca_global e a contagem de itens_sem_fonte.

REGRAS
- Não conserte os itens. Apenas julgue.
- Na dúvida entre manter e remover, remova. Framework menor e confiável vale mais
  que framework completo e duvidoso.

SAÍDA: JSON no formato VERIFICACAO.

<framework>{{framework}}</framework>
<unidades>{{consolidado}}</unidades>
```

---

## Passo 6 — Aplicação ao seu contexto

**Entra:** framework verificado + seu perfil real.
**Sai:** as tarefas que entram no painel de ação, com prazo.
**Serve para:** o mesmo livro gera plano diferente para quem tem 6 alunos e para quem tem 60.

```
Você vai converter um framework já verificado em tarefas para o painel de ação.

PERFIL: {{perfil_do_leitor}}
AGENDA DISPONÍVEL: {{tempo_semanal_min}} minutos por semana
JÁ EM ANDAMENTO: {{acoes_abertas_no_painel}}

TAREFAS
1. Escolha o que aplicar primeiro pelo critério: maior efeito com menor custo de mudança.
2. Distribua nas 4 semanas respeitando dependências entre tarefas.
3. Para cada tarefa: título no imperativo, esforço em minutos, dia sugerido e
   critério de feito observável.
4. Aponte o que do livro NÃO deve ser aplicado agora, e por quê.
5. Se algo colidir com uma ação já aberta no painel, diga qual e proponha ordem.

REGRAS
- Máximo 12 tarefas. Se não coube, não entra: entra em "aplicar depois".
- Nenhuma tarefa maior que 90 minutos. Quebre em partes.
- Toda tarefa aponta o item do framework e a página de origem.

SAÍDA: JSON no formato PLANO_APLICADO.
```

---

## As sete leis do modelo

1. **Sem citação, não entra.** Todo item carrega página e trecho literal.
2. **Citação curta.** Até 25 palavras. O framework serve para aplicar e conferir, nunca para substituir o livro.
3. **Vocabulário do autor nos nomes**, linguagem própria nas explicações.
4. **Número só se o livro deu número.** Faltou, vira pergunta em aberto.
5. **Critério de feito observável** em todo passo e toda tarefa.
6. **Limites rígidos**: 7 princípios, 5 protocolos, 12 tarefas. Sem limite vira sumário.
7. **Vazio é resposta válida.** Livro narrativo sem protocolo devolve protocolos vazio e heurísticas preenchidas.

## Aceite: o framework está bom quando

- `itens_sem_fonte` é zero.
- Você consegue abrir o livro em 3 páginas citadas ao acaso e confirmar o que está escrito.
- Existe pelo menos uma tarefa que você faria ainda esta semana.
- `o_que_o_livro_nao_responde` tem algo que você realmente quer saber.
- Nenhum item soa como frase de efeito genérica que caberia em qualquer livro do tema.
- Passados 30 dias, dá para responder se funcionou olhando as métricas — sem depender de memória.
