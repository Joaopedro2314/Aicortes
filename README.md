# AI Cutter — MVP

App Android (Kotlin) que detecta automaticamente os **melhores momentos** de um vídeo e gera um "reel" de highlights — **tudo processado no próprio celular**, sem enviar nada para a nuvem.

## Como funciona a "IA"

Este MVP usa uma abordagem **heurística e 100% local** (não depende de internet nem de modelos pesados de deep learning, então roda em qualquer aparelho):

1. **Áudio** — decodifica a trilha de áudio e mede a energia (RMS) em janelas de 0,5s. Picos de volume costumam indicar risadas, gritos, música alta, impactos, fala enfática etc.
2. **Vídeo** — amostra 1 frame por segundo e mede a diferença de luminância entre frames consecutivos, capturando cortes de cena e movimento intenso.
3. **Combinação** — as duas curvas são normalizadas, suavizadas e combinadas (60% áudio + 40% movimento) numa curva única de "pontuação de destaque".
4. **Seleção** — os picos dessa curva viram os segmentos finais (por padrão: 5 trechos de 6 segundos), evitando sobreposição.
5. **Exportação** — os trechos escolhidos são cortados e concatenados num único vídeo usando o **Media3 Transformer** do Android (não precisa de FFmpeg externo).

> Evolução futura: trocar/complementar a heurística por um modelo TensorFlow Lite (ex: detecção de rostos sorrindo, reconhecimento de ação) mantendo a mesma interface `HighlightDetector`.

## Estrutura do projeto

```
AICutter/
├── app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/aicutter/mvp/
│       │   ├── MainActivity.kt        # UI e orquestração
│       │   ├── HighlightDetector.kt   # "IA" — detecção dos melhores momentos
│       │   └── VideoExporter.kt       # corte e exportação do vídeo final
│       └── res/                       # layout, temas, ícones
├── build.gradle.kts
└── settings.gradle.kts
```

## Como abrir e rodar

1. Abra a pasta `AICutter/` no **Android Studio** (versão Koala/2024.1 ou mais recente).
2. Deixe o Android Studio baixar o Gradle Wrapper automaticamente na primeira sincronização (o `gradle-wrapper.jar` não está incluso neste pacote; o Android Studio o gera sozinho — ou rode `gradle wrapper` manualmente se preferir usar linha de comando).
3. Conecte um dispositivo Android (ou emulador) com **API 24+**.
4. Rode o app (▶️).
5. No app:
   - Toque em **"Selecionar vídeo"** e escolha um vídeo da galeria.
   - Toque em **"Detectar melhores momentos"** — o app analisa áudio e movimento (leva alguns segundos, dependendo da duração do vídeo).
   - Revise os trechos encontrados (com timestamps e pontuação).
   - Toque em **"Gerar vídeo com os melhores momentos"** — o resultado é salvo em:
     `Android/data/com.aicutter.mvp/files/highlights/`

## Limitações do MVP (propositais, para manter simples)

- Quantidade de trechos (5) e duração de cada um (6s) são fixas no código — dá para expor isso na UI depois.
- A "IA" é heurística (áudio + movimento), não um modelo de ML treinado — funciona bem para a maioria dos vídeos, mas não "entende" o conteúdo (não sabe o que é um gol, uma piada, etc.).
- Vídeos muito longos (>15-20 min) podem demorar mais na etapa de análise de movimento, pois ela lê frames via `MediaMetadataRetriever`.
- O vídeo exportado fica na pasta interna do app; para compartilhar facilmente, seria bom adicionar um botão de "Compartilhar" (`Intent.ACTION_SEND`) ou salvar via `MediaStore` para aparecer na galeria — próximo passo natural.

## Próximos passos sugeridos

- Botão de compartilhar / salvar na galeria (`MediaStore`).
- Preview do vídeo com `ExoPlayer` antes de exportar.
- Permitir ajustar número e duração dos trechos na UI.
- Trocar a heurística por um modelo TFLite leve (ex: classificação de áudio "risada/aplausos" ou detecção de rosto).
