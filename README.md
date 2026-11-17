# dublagem-compasso

Dubla um vÃ­deo de **personagem falando** (gerado no Kling, Veo, Seedanceâ€¦) com uma **voz clonada local** (chatterbox), alinhando cada pedaÃ§o da fala nas **batidas da boca** â€” o "compasso" â€” em vez de sÃ³ colar uma narraÃ§Ã£o por cima.

## O problema que resolve

Geradores de vÃ­deo com IA falam idiomas nÃ£o-ingleses com pronÃºncia ruim (PT-BR sai com sotaque e palavras trocadas), e **nÃ£o aceitam Ã¡udio de entrada** â€” nÃ£o dÃ¡ pra mandar sua voz. Trocar o Ã¡udio por uma narraÃ§Ã£o TTS comum quebra o sincronismo: boca abrindo sem som, fala atravessando pausas.

## O mÃ©todo (validado em produÃ§Ã£o, 2026-07)

1. **O Ã¡udio original Ã© a rÃ©gua de tempo.** Gere o vÃ­deo com o prompt inteiro no idioma alvo + a fala entre aspas + Ã¡udio nativo ligado â€” mesmo ruim, ele faz a boca articular as frases certas nos tempos certos.
2. **TranscriÃ§Ã£o com timestamps de palavra** (faster-whisper) â†’ blocos de fala separados por silÃªncio = o compasso da boca.
3. **Alinhamento do texto correto aos blocos** (difflib â€” mesma lÃ­ngua e mesma fala garantem alta correspondÃªncia).
4. **TTS multi-tomada por bloco** (chatterbox + voz clonada): gera N tomadas e escolhe a que cabe **natural** na janela (ajuste fino â‰¤ Â±20%; nÃ£o coube = outra tomada, nunca esticÃ£o).
5. **Montagem nos offsets exatos** + troca do Ã¡udio (vÃ­deo intacto, `-c:v copy`).
6. **VerificaÃ§Ã£o automÃ¡tica**: energia de fala presente do inÃ­cio ao fim de cada janela â€” *boca aberta sem som* Ã© o defeito nÂº 1 e o script falha alto se detectar.

## Uso

```bash
conda run -n chatterbox python3 dublar.py \
  --video clip.mp4 \
  --texto "NÃ£o adianta ficar sÃ³ assistindo. VocÃª precisa praticar." \
  --ref ~/vozes/minha-voz.wav \
  --out clip-dublado.mp4 \
  --cauda 2          # opcional: congela o Ãºltimo frame por 2s (evita final duro)
```

OpÃ§Ãµes Ãºteis:
- `--takes 6` â€” tomadas por bloco (mais tomadas = melhor encaixe, mais GPU)
- `--gap 0.30` â€” silÃªncio (s) que separa blocos de fala
- `--lang pt` â€” idioma da transcriÃ§Ã£o e do TTS
- `--texto-file roteiro.txt` â€” texto vindo de arquivo

## Dicas de calibragem

- Se um bloco da boca Ã© **lento** demais pro texto (fala pausada no original), escreva o texto com **reticÃªncias** ("ler... testar... errar...") â€” o TTS reproduz o ritmo pausado.
- A referÃªncia de voz: WAV 24kHz mono, ~10s de fala limpa (o chatterbox usa sÃ³ os primeiros 10s).
- Close extremo de boca exige lip-sync por fonema (ex.: HeyGen); este mÃ©todo alinha por batida de frase/pedaÃ§o â€” excelente pra plano mÃ©dio, aceitÃ¡vel em close.

## Requisitos

- env conda `chatterbox` com `chatterbox-tts` e `faster-whisper`
- `ffmpeg` no PATH
- GPU ajuda (TTS); a transcriÃ§Ã£o roda em CPU

## Origem

MÃ©todo desenvolvido e validado no projeto [klingaimcp](https://github.com/inematds/klingaimcp) (avatar falante do Kling dublado em PT-BR com voz clonada).

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe