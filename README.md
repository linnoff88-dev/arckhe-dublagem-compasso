# dublagem-compasso

Dubla um vêÂ­deo de **personagem falando** (gerado no Kling, Veo, Seedanceê¢â¬Â¦) com uma **voz clonada local** (chatterbox), alinhando cada pedaêÂ§o da fala nas **batidas da boca** ê¢â¬â o "compasso" ê¢â¬â em vez de sêÂ³ colar uma narraêÂ§êÂ£o por cima.

## O problema que resolve

Geradores de vêÂ­deo com IA falam idiomas nêÂ£o-ingleses com pronêÂºncia ruim (PT-BR sai com sotaque e palavras trocadas), e **nêÂ£o aceitam êÂ¡udio de entrada** ê¢â¬â nêÂ£o dêÂ¡ pra mandar sua voz. Trocar o êÂ¡udio por uma narraêÂ§êÂ£o TTS comum quebra o sincronismo: boca abrindo sem som, fala atravessando pausas.

## O mêÂ©todo (validado em produêÂ§êÂ£o, 2026-07)

1. **O êÂ¡udio original êÂ© a rêÂ©gua de tempo.** Gere o vêÂ­deo com o prompt inteiro no idioma alvo + a fala entre aspas + êÂ¡udio nativo ligado ê¢â¬â mesmo ruim, ele faz a boca articular as frases certas nos tempos certos.
2. **TranscriêÂ§êÂ£o com timestamps de palavra** (faster-whisper) ê¢â â blocos de fala separados por silêÂªncio = o compasso da boca.
3. **Alinhamento do texto correto aos blocos** (difflib ê¢â¬â mesma lêÂ­ngua e mesma fala garantem alta correspondêÂªncia).
4. **TTS multi-tomada por bloco** (chatterbox + voz clonada): gera N tomadas e escolhe a que cabe **natural** na janela (ajuste fino ê¢â°Â¤ êÂ±20%; nêÂ£o coube = outra tomada, nunca esticêÂ£o).
5. **Montagem nos offsets exatos** + troca do êÂ¡udio (vêÂ­deo intacto, `-c:v copy`).
6. **VerificaêÂ§êÂ£o automêÂ¡tica**: energia de fala presente do inêÂ­cio ao fim de cada janela ê¢â¬â *boca aberta sem som* êÂ© o defeito nêÂº 1 e o script falha alto se detectar.

## Uso

```bash
conda run -n chatterbox python3 dublar.py \
  --video clip.mp4 \
  --texto "NêÂ£o adianta ficar sêÂ³ assistindo. VocêÂª precisa praticar." \
  --ref ~/vozes/minha-voz.wav \
  --out clip-dublado.mp4 \
  --cauda 2          # opcional: congela o êÂºltimo frame por 2s (evita final duro)
```

OpêÂ§êÂµes êÂºteis:
- `--takes 6` ê¢â¬â tomadas por bloco (mais tomadas = melhor encaixe, mais GPU)
- `--gap 0.30` ê¢â¬â silêÂªncio (s) que separa blocos de fala
- `--lang pt` ê¢â¬â idioma da transcriêÂ§êÂ£o e do TTS
- `--texto-file roteiro.txt` ê¢â¬â texto vindo de arquivo

## Dicas de calibragem

- Se um bloco da boca êÂ© **lento** demais pro texto (fala pausada no original), escreva o texto com **reticêÂªncias** ("ler... testar... errar...") ê¢â¬â o TTS reproduz o ritmo pausado.
- A referêÂªncia de voz: WAV 24kHz mono, ~10s de fala limpa (o chatterbox usa sêÂ³ os primeiros 10s).
- Close extremo de boca exige lip-sync por fonema (ex.: HeyGen); este mêÂ©todo alinha por batida de frase/pedaêÂ§o ê¢â¬â excelente pra plano mêÂ©dio, aceitêÂ¡vel em close.

## Requisitos

- env conda `chatterbox` com `chatterbox-tts` e `faster-whisper`
- `ffmpeg` no PATH
- GPU ajuda (TTS); a transcriêÂ§êÂ£o roda em CPU

## Origem

MêÂ©todo desenvolvido e validado no projeto [klingaimcp](https://github.com/linnoff88-dev/klingaimcp) (avatar falante do Kling dublado em PT-BR com voz clonada).

---

Projeto adaptado da comunidade ARCKHê TDS - ARCKHÉe