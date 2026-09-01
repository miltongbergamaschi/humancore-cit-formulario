# Pesquisa de preços e limites — 01/09/2026

Números levantados por busca na web. Onde a página oficial não pôde ser aberta
diretamente daqui, a fonte é secundária e está marcada — vale reconferir na
tabela do fornecedor antes de contratar.

## 1. Transcrição por API (por hora de áudio)

| Serviço | Modelo | Preço | Diarização | Crédito inicial |
|---|---|---|---|---|
| AssemblyAI | Universal-2 | US$ 0,15/h | +US$ 0,02/h | camada gratuita generosa (~185h) ou US$ 50 em crédito |
| OpenAI | gpt-4o-mini-transcribe | US$ 0,18/h (US$ 0,003/min) | não incluída | — |
| AssemblyAI | Universal-3.5 Pro | US$ 0,21/h | +US$ 0,02/h | idem |
| Deepgram | Nova-3 (pré-gravado) | US$ 0,26 a 0,29/h | +US$ 0,12/h | US$ 200 |
| OpenAI | whisper-1 / gpt-4o-transcribe | US$ 0,36/h (US$ 0,006/min) | não incluída | — |

**Leitura:** a faixa de US$ 0,16 a 0,36 por hora que estava no plano se confirma.
O crédito inicial do Deepgram (US$ 200) sozinho cobre centenas de horas — na
prática, os primeiros meses saem de graça mesmo na nuvem.

## 2. Modelos Claude — organizar o conteúdo

Tabela de 24/06/2026, por milhão de tokens:

| Modelo | Entrada | Saída | Custo por hora de aula |
|---|---|---|---|
| Haiku 4.5 | US$ 1 | US$ 5 | ~US$ 0,03 |
| Sonnet 5 | US$ 2 | US$ 10 | ~US$ 0,10 |
| Opus 5 | US$ 5 | US$ 25 | ~US$ 0,30 |

Conta: ~15 mil tokens de transcrição por hora de aula, duas passadas.

## 3. Whisper rodando no seu computador

| Situação | Desempenho | Requisito |
|---|---|---|
| Placa de vídeo dedicada (ex.: RTX 3050) | ~6,5× o tempo real — 1h de aula em ~9 min | ~3 GB de VRAM com quantização int8; 6 GB roda folgado |
| Mac com chip Apple e Metal | ~10× o tempo real — 1h em ~6 min | 16 GB de RAM recomendados para o modelo grande |
| Só processador, sem placa de vídeo | **mais lento que o tempo real** — 1h de aula leva ~2,5h | inviável para uso real |

- Projetos: `faster-whisper` (placa NVIDIA) e `whisper.cpp` (Mac, Metal).
- Modelo grande (large-v3) é o que tem qualidade comparável às APIs pagas. Os
  modelos pequenos são bem piores — e a diferença aparece mais em português.
- **Não confirmado:** números de erro (WER) específicos para português por
  tamanho de modelo. Separar quem falou offline exige ferramenta adicional
  (pyannote) e complica bastante o setup.

## 4. Planos gratuitos — o achado que muda a arquitetura

| Limite | Vercel Hobby | Supabase Free |
|---|---|---|
| Corpo de requisição | **4,5 MB** | — |
| Duração de função | **10 segundos** | — |
| Banda / egress | 100 GB/mês | 5 GB/mês |
| Banco | — | 500 MB |
| Arquivos | — | 1 GB total, **50 MB por arquivo** |
| Outros | 1M invocações, 4 CPU-horas | pausa o projeto após 7 dias sem uso |

**Resposta direta:** subir 1 GB de áudio e processar por 20 minutos dentro do
plano gratuito é **impossível**. Nem perto.

**Consequência:** o áudio não deve subir para a nuvem. O trabalho pesado —
guardar o arquivo e transcrever — fica no seu computador; a nuvem recebe só o
texto. Uma transcrição de 1h ocupa ~60 KB, então os 500 MB de banco comportam
milhares de aulas e o plano gratuito passa a servir com folga.

Atenção ao detalhe do Supabase: **o projeto é pausado após 7 dias sem uso**, e
uso pessoal fica parado com facilidade. É incômodo, não impeditivo.

## 5. Capturar o áudio do computador (aula online)

- **Mac:** instalar o BlackHole (gratuito), criar um Dispositivo de Saída
  Múltipla no Configuração de Áudio e MIDI juntando alto-falante + BlackHole, e
  gravar tendo o BlackHole como entrada. Para pegar sua voz junto, um
  Dispositivo Agregado combinando BlackHole + microfone.
- **Windows:** usar o Stereo Mix quando existir; quando não existir (o caso da
  maioria dos notebooks hoje), instalar o VB-Audio Cable, ou gravar com o
  Audacity em modo WASAPI loopback.

## 6. Gravar no celular em segundo plano

- Expo tem `enableBackgroundRecording`, que ativa o modo de áudio em segundo
  plano no iOS. Funciona, mas há relatos documentados de gravação sendo
  encerrada silenciosamente depois de um tempo variável.
- Para gravação longa e confiável, a recomendação é sair do fluxo gerenciado do
  Expo e usar módulo nativo.
- **Não confirmado:** as regras atuais de instalar no próprio aparelho sem loja
  (TestFlight e perfil de desenvolvedor). Confirmar antes da fase 2.

## 7. YouTube — o segundo achado

Não existe caminho oficial para baixar a legenda automática de um vídeo de
outra pessoa. A API do YouTube só entrega legenda de vídeo que **você** é dono
ou tem permissão de editar. O endpoint `timedtext`, muito usado, é
**não documentado e o uso pode violar os termos**.

**Consequência:** cai a ideia de "vídeo do YouTube tem transcrição de graça".
Para aula do YouTube, o caminho limpo é o mesmo dos outros: capturar o áudio
enquanto você assiste e transcrever no seu sistema.

## 8. Zoom e Google Meet

- **Zoom:** o plano gratuito grava localmente (só o anfitrião), mas **sem
  transcrição**. Transcrição automática só a partir do plano Business. Desde
  maio de 2026 o Zoom também não deixa mais salvar ou baixar as legendas.
- **Google Meet:** gravação e transcrição nativas exigem Workspace Business
  Standard ou superior. Conta pessoal do Gmail não tem.

**Consequência:** a transcrição é sempre do nosso sistema, nunca da plataforma.
Gravação local no Zoom continua útil como fonte de áudio.

## O que ficou sem confirmação

1. Erro de transcrição (WER) por tamanho de modelo, específico para português.
2. Regras atuais de instalação de app no próprio iPhone sem publicar em loja.
3. Preços oficiais abertos direto na página do fornecedor — o acesso a
   deepgram.com, assemblyai.com, openai.com e vercel.com está bloqueado a
   partir deste ambiente, então os números de transcrição e de limites vieram
   de fontes secundárias recentes.
