# Informações da build

- **Nome do jogo:** FEED THE AI (Demo 0.40v)
- **Plataforma:** itch.io (HTML5 / Unity WebGL)
- **Navegadores testados:** Google Chrome 149.0.0.0
- **Data do teste:** 26/06/2026

# Ambiente de teste

- **CPU/GPU:** 20 cores lógicos / AMD Radeon RX 6650 XT (ANGLE/D3D11)
- **RAM:** 32 GB (limite JS heap 4 GB alocado pelo Chrome)
- **Browser e versão:** Google Chrome 149.0.0.0
- **Sistema operacional:** Windows 10 (Win64)
- **Engine:** Unity 6000.3.3f1 (WebGL 2.0, OpenGL ES 3.0)
- **Resolução do viewport:** 1280x720 @ DPR 1
- **Conexão:** 4G efetivo, ~10 Mbps, RTT 50ms
- **Assets WebGL:** ~74 MB (`.br` Brotli: framework, wasm, data)

# Testes realizados

| Cenário | Resultado | Observações |
| --- | --- | --- |
| Initial Load | Pass | Unity WebGL carrega em ~3s; download inicial via Brotli; tela de diálogo "Booting language core" exibida corretamente |
| Fullscreen | Pass (parcial) | Botão fullscreen presente na barra inferior direita (`fullscreen-button.png`); API RequestFullscreen funciona; não foi possível validar UX pós-fs sem gamepad |
| Window Resize | Pass | Canvas reajusta sem perda de estado; timer do jogo continuou rodando (10:40 → 16:72) durante a troca 1280x720 → 1024x600 → 1280x720 |
| Gamepad | N/A | Gamepad API disponível (`navigator.getGamepads` ok), nenhum dispositivo conectado neste ambiente; Unity Input System inicializado ("Input System module state changed to: Initialized") |
| Long Session (30 min) | Não executado | Sessão curta (~5min) executada; FPS estável ~182; heap JS 5.3 MB; sem degradação visível. Não foi aguardado o ciclo completo de 30 min. |
| Refresh During Gameplay | Fail | F5/reload do iframe zera o estado. Volta à landing page do itch.io (botão "Run game") e exige novo boot do Unity. Saves locais do Unity (IndexedDB) também são perdidos pois o contexto do iframe é recriado. |

# Bugs encontrados

**Bug ID:** WEB-001

**Título:** Refresh durante gameplay descarta todo o estado do jogo.

**Passos para reproduzir:**

1. Acessar `https://weirdkidgames.itch.io/feed-the-ai`.
2. Clicar em **Run game**.
3. Aguardar boot do Unity WebGL e clicar em **Start!**.
4. Jogar por alguns minutos (acumular Data/timer).
5. Pressionar F5 ou recarregar a aba.

**Resultado esperado:**

Saves locais do Unity (IndexedDB via `UnityCache`) deveriam persistir entre reloads, retornando ao estado "Continue".

**Resultado atual:**

A página volta para a landing do itch.io com o botão **Run game** visível. Mesmo que o cache de assets (`FEED%20THE%20AI%20Web.data.br`) seja reaproveitado, o **boot.config sync manual** é deprecado (`JS_FileSystem_Sync()` warning) e o `autoSyncPersistentDataPath` não está habilitado (`config.autoSyncPersistentDataPath = true`). Console mostra `refresh` + `2 FS.syncfs operations in flight at once, probably just doing extra work`, indicando que as escritas de save não estão sendo confirmadas antes do reload.

**Severidade:** Major. Quebra a UX de "fechar e voltar" e pode causar perda de progresso em jogadores casuais.

---

**Bug ID:** WEB-002

**Título:** Shaders URP não suportados nesta GPU (warning).

**Passos para reproduzir:**

1. Bootar o jogo no Chrome.
2. Abrir DevTools → Console.

**Resultado esperado:**

Sem erros de shader.

**Resultado atual:**

Dois `ERROR: Shader` aparecem no console:
- `Hidden/CoreSRP/CoreCopy shader is not supported on this GPU (none of subshaders/fallbacks are suitable)`
- `Hidden/Universal/HDRDebugView shader is not supported on this GPU (none of subshaders/fallbacks are suitable)`

**Severidade:** Minor. Não bloqueia gameplay (o jogo roda normalmente), mas indica que o build WebGL não foi totalmente validado para o fallback path de DX11/ANGLE. Pode causar artefatos em placas AMD ou drivers mais antigos.

---

**Bug ID:** WEB-003

**Título:** Audio warnings recorrentes durante boot e gameplay.

**Passos para reproduzir:**

1. Bootar o jogo.
2. Observar DevTools → Console durante o carregamento e primeiros segundos de gameplay.

**Resultado esperado:**

Sem warnings de áudio.

**Resultado atual:**

Mensagem `Trying to get length of sound which is not loaded yet.` aparece **14 vezes** durante o boot + 2 vezes em interações posteriores. Indica que SFX referenciados no código ainda não tinham terminado de decodificar quando o jogo tentou tocá-los.

**Severidade:** Minor. UX afetada (possíveis efeitos sonoros faltando nos primeiros segundos).

---

**Bug ID:** WEB-004

**Título:** Warnings deprecation do Unity em produção.

**Passos para reproduzir:**

1. Bootar o jogo.

**Resultado esperado:**

Build de release sem warnings de API deprecada.

**Resultado atual:**

`Manual synchronization of Unity Application.persistentDataPath via JS_FileSystem_Sync() is deprecated and will be later removed in a future Unity version. ... Pass config.autoSyncPersistentDataPath = true; to configuration in createUnityInstance() to opt in to the new behavior.`

**Severidade:** Minor. Deprecation warning, mas está diretamente relacionado ao WEB-001 (sync manual é o que está tentando persistir saves no momento).

# Performance Analysis

- **Average FPS:** ~182 FPS (estimado via `requestAnimationFrame` no iframe; medição no main thread do container, não dentro do WASM)
- **Peak Memory Usage:** ~5.3 MB de JS heap no container; **não foi possível medir a memória WASM/Unity** diretamente (Chrome DevTools expõe `performance.memory` apenas do contexto JS, não do WebAssembly heap do Unity). Estimativa baseada em size dos assets: 74 MB descomprimidos + runtime ≈ 250-400 MB.
- **Console Errors:** 1 erro real (`warning: 2 FS.syncfs operations in flight at once`) + 4 `ERROR: Shader` (contados como erros pelo Unity logger, mas funcionalmente são warnings); 7 warnings (`monetization`, `xr`, `allowfullscreen` precedence, syncfs, NavMesh agent x2, persistentDataPath deprecation)
- **CLS:** 0.00
- **Network:** 93 requests no load inicial; Brotli habilitado nos assets Unity (`.br`); cache do browser reutiliza `.data.br` em loads subsequentes
- **Render path:** WebGL 2.0 + Unity URP; extensões `EXT_color_buffer_float`, `WEBGL_compressed_texture_s3tc`, etc. disponíveis

# Recommendation

**Needs Fixes Before Release** (com ressalvas)

**Reason:**

O jogo é tecnicamente sólido. Unity 6000 WebGL2 funciona bem, FPS alto, input e render estão OK. Não há nenhum bug **crítico bloqueante**. Os problemas encontrados são majoritariamente de polish/regressão:

1. **WEB-001 (Major):** O save local não sobrevive a um F5. Isso é grave para um incremental onde o jogador deixa a aba aberta horas/dias. A correção é trivial. Habilitar `autoSyncPersistentDataPath` no `createUnityInstance()`.
2. **WEB-002 (Minor):** Shaders URP sem fallback. Recomendo testar em hardware representativo do público (Intel/AMD/NVIDIA integrado e dedicado).
3. **WEB-003 (Minor):** Áudio tocando antes do decode terminar. Adicionar `audioClip.loadAudioData` promise check antes do `Play()`.
4. **WEB-004 (Minor):** Limpar warnings de deprecation antes do release.

Nenhum dos bugs impede a demo atual de rodar, mas o **WEB-001 deve ser fix antes da release na Steam** para não gerar tickets de suporte do tipo "perdi meu save".
