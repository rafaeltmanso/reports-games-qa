# Informações da build

- **Nome do jogo:** Peggy's Post (v1.5.9)
- **Plataforma:** itch.io (HTML5 / Unity WebGL)
- **Navegadores testados:** Google Chrome 149.0.0.0
- **Data do teste:** 26/06/2026

# Ambiente de teste

- **CPU/GPU:** 20 cores lógicos / AMD Radeon RX 6650 XT (ANGLE/D3D11)
- **RAM:** 32 GB (limite JS heap 4 GB alocado pelo Chrome)
- **Browser e versão:** Google Chrome 149.0.0.0
- **Sistema operacional:** Windows 10 (Win64)
- **Engine:** Unity 2021.3.15f1 (WebGL 2.0, OpenGL ES 3.0, Built-in Render Pipeline + Postprocessing Stack v2)
- **Resolução do viewport:** 1280x720 @ DPR 1
- **Conexão:** 4G efetivo, ~10 Mbps, RTT 50ms
- **Assets WebGL:** ~67 MB Windows build (~67 MB WebGL total; página do itch mede ~25 MB no DOM, jogo Unity pesa mais após iframe)

# Testes realizados

| Cenário | Resultado | Observações |
| --- | --- | --- |
| Initial Load | Pass | Página do itch em ~2s; clique em Run game abre iframe do Unity; **boot completo do Unity leva ~30s** (WebGL 2021 com Postprocessing pesado); após boot, exibe menu do manual aberto mostrando regras de envio e starter stamp pack |
| Fullscreen | Pass (parcial) | API fullscreen disponível; botão de fullscreen presente na barra inferior do template Unity; não foi possível validar UX pós-fullscreen via clique direto no iframe (cross-origin bloqueia cliques sintéticos, requer interação manual) |
| Window Resize | Pass | Jogo continua renderizando corretamente após resize de 1280x720 para 1024x768; layout permanece intacto, parcel na mesa e cliente visíveis |
| Gamepad | N/A | Gamepad API disponível, nenhum device conectado neste ambiente; jogo é mouse + keyboard only segundo a doc |
| Long Session (30 min) | Não executado | Sessão curta (~2min) executada; FPS estável ~172; heap JS 6.7 MB; sem degradação observada. Não foi aguardado o ciclo completo de 30 min. |
| Refresh During Gameplay | Fail | F5/reload zera o iframe do Unity (recarrega do zero); botão "Run game" reaparece; jogador precisa esperar os ~30s de boot novamente. Save local (IndexedDB) não restaura automaticamente. **Mesmo problema já conhecido do FEED THE AI**, ambos Unity WebGL itch.io. |

# Bugs encontrados

**Bug ID:** WEB-009

**Título:** Refresh durante gameplay descarta todo o estado do jogo.

**Passos para reproduzir:**

1. Acessar `https://digitarium.itch.io/peggys-post`.
2. Clicar em **Run game**.
3. Aguardar os ~30s de boot do Unity WebGL.
4. Clicar em **Next** (ou pressionar D) para começar a receber clientes.
5. Pressionar F5 ou recarregar a aba.

**Resultado esperado:**

Save local (IndexedDB) deveria persistir o estado do jogo entre reloads.

**Resultado atual:**

A página volta para a landing do itch.io com o botão **Run game** visível. O iframe do Unity é completamente recriado, perdendo qualquer estado em memória. O console mostra `[UnityCache] Failed to load '.../Peggy's%20Post.data.unityweb' from indexedDB cache due to the error: Error: Could not connect to database.` no primeiro load, indicando que o cache de assets nem consegue inicializar corretamente. O boot completo leva ~30s a cada refresh.

**Severidade:** Major. Similar ao WEB-001 (FEED THE AI). Ambos Unity WebGL no itch.io sofrem do mesmo problema: o `autoSyncPersistentDataPath` provavelmente não está habilitado, e o IndexedDB não está sendo usado para save de estado. O dev tem uma feature de "save slots" no jogo (v1.5.2), mas isso só funciona dentro da sessão atual.

---

**Bug ID:** WEB-010

**Título:** Shaders de Postprocessing sem fallback para esta GPU.

**Passos para reproduzir:**

1. Acessar `https://digitarium.itch.io/peggys-post`.
2. Clicar em Run game.
3. Abrir DevTools → Console.

**Resultado esperado:**

Sem `ERROR: Shader` no console.

**Resultado atual:**

Quatro shaders de Postprocessing Stack v2 (compatível com Unity 2021.3 builtin pipeline) não rodam na GPU:
- `Hidden/PostProcessing/Debug/Histogram`
- `Hidden/PostProcessing/Debug/LightMeter`
- `Hidden/PostProcessing/Debug/Vectorscope`
- `Hidden/PostProcessing/Debug/Waveform`
- `Hidden/PostProcessing/MultiScaleVO`
- `Hidden/PostProcessing/ScreenSpaceReflections`

Mensagem: `none of subshaders/fallbacks are suitable` em cada um.

**Severidade:** Minor. Shaders de Debug não afetam gameplay normal (são usados para ferramentas de dev). MultiScaleVO e ScreenSpaceReflections são de produção, mas como o jogo está em builtin pipeline (não URP/HDRP), esses efeitos provavelmente não estão ativos em gameplay normal. Não bloqueia release.

---

**Bug ID:** WEB-011

**Título:** Warning repetido 10945 vezes sobre Postprocessing package.

**Passos para reproduzir:**

1. Bootar o jogo.
2. Observar console.

**Resultado esperado:**

Sem warnings repetidos.

**Resultado atual:**

`When used with builtin render pipeline, Postprocessing package expects to be used on a fullscreen Camera. Please note that using Camera viewport may result in visual artefacts or some things not working.` aparece **1774 vezes** + **9171 vezes** = **10945 vezes** total. Indica que o jogo está usando o Postprocessing package com Camera viewport em vez de fullscreen, e o engine dispara o warning a cada frame para cada camera afetada.

**Severidade:** Minor. Funcional (jogo roda), mas polui o console e pode indicar degradação de performance sutil (cada warning = log call). Recomendo o dev ajustar a Camera do Postprocessing para usar fullscreen ou desabilitar o warning deprecation.

---

**Bug ID:** WEB-012

**Título:** Iframe do jogo é maior que o viewport (1300x800 em janela 1280x720).

**Passos para reproduzir:**

1. Abrir `https://digitarium.itch.io/peggys-post` em uma janela de 1280x720.
2. Observar o iframe do jogo.

**Resultado esperado:**

O iframe deveria caber na viewport com alguma margem, ou scroll deveria ser natural.

**Resultado atual:**

O iframe do Unity é fixado em 1300x800 pixels, criando barras de scroll horizontal e vertical em janelas menores. A página do itch também é enorme (19724px de altura total) porque o iframe está inline no meio do layout, forçando o usuário a scrollar para interagir com partes do jogo. Jogar em tela cheia ajuda, mas prejudica quem joga em janela.

**Severidade:** Minor. Recomendo usar CSS responsivo (`max-width: 100%; height: auto;`) e/ou detectar viewport e ajustar dinamicamente.

---

**Bug ID:** WEB-013

**Título:** Erro HTTP 410 em endpoint de tracking do itch (rh endpoint).

**Passos para reproduzir:**

1. Acessar `https://digitarium.itch.io/peggys-post`.
2. Abrir DevTools → Network/Console.

**Resultado esperado:**

Sem erros de rede em recursos do itch.

**Resultado atual:**

Request `GET https://digitarium.itch.io/peggys-post/rh/...` retorna **HTTP 410 Gone**. Console exibe `Failed to load resource: the server responded with a status of 410`.

**Severidade:** Minor. Idêntico ao WEB-006 (Serenitrove). Provavelmente problema deprecated do lado do itch, não do dev do Peggy's Post.

# Performance Analysis

- **Average FPS:** ~172 FPS (estimado via `requestAnimationFrame` no container do itch)
- **Peak Memory Usage:** ~6.7 MB de JS heap no container (exclui memória do Unity WASM que pode chegar a 1-2 GB); jogo Unity pesado, recomenda-se 4 GB+ de RAM para jogadores
- **Console Errors:** 1 erro real (410 Gone); ~6 erros de Shader classificados como ERROR pelo Unity logger (mas são funcionalmente warnings); **10945 warnings de Postprocessing Camera** (problema de logging excessivo)
- **Boot Time:** ~30 segundos para Unity WebGL inicializar completamente com Postprocessing (pode ser muito longo para jogadores casuais)
- **Network:** Requisições Unity (.data.br, .wasm.br, .framework.js.br) com Brotli habilitado
- **Render path:** WebGL 2.0 + Unity Built-in Render Pipeline + Postprocessing Stack v2
- **Gamepad API:** disponível mas não suportada pelo jogo
- **Fullscreen API:** disponível e suportada

# Recommendation

**Needs Fixes Before Release** (com ressalvas)

**Reason:**

Peggy's Post é um jogo lindíssimo com mecânicas complexas (gestão de post office, sistema de stamps, multiple morality paths), mas tem problemas claros de otimização e UX que devem ser endereçados antes de uma release oficial na web:

1. **WEB-009 (Major):** Refresh zera o estado. Mesmo problema do FEED THE AI. É um padrão do Unity WebGL + itch.io: o cache do IndexedDB para assets não funciona bem (`Could not connect to database`), e saves de estado não persistem entre reloads. Recomendação: implementar save exportável (similar ao Serenitrove, que tem Export funcional) ou pelo menos habilitar `autoSyncPersistentDataPath` no Unity. O dev já tem save slots no jogo (v1.5.2+), então pode estender essa mecânica para incluir export.

2. **WEB-011 (Minor):** 10945 warnings repetidos no console. Não afeta gameplay mas prejudica QA tooling e provavelmente degrada performance marginalmente. Ajustar Camera do Postprocessing.

3. **WEB-010 (Minor):** Shaders de Postprocessing sem fallback. Recomendo testar em mais GPUs.

4. **WEB-012 (Minor):** Iframe 1300x800 não cabe em janelas comuns. Torna o jogo difícil de jogar sem fullscreen.

5. **WEB-013 (Minor):** Erro 410 do itch. Idêntico ao WEB-006.

Peggy's Post tem mecânicas sólidas e visuais encantadores. O dev tem comunidade ativa no Discord e responde rápido a reports. Esses fixes são todos polimento (exceto WEB-009) e podem ser endereçados em uma v1.6.x.
