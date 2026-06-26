# Web Game QA Reports

Reports de QA para jogos web rodando no itch.io, usando Chrome DevTools MCP para inspeção automatizada.

## Setup

- **MCP Server:** `chrome-devtools-mcp` configurado em `opencode.json`
- **Browser:** Google Chrome 149.0.0.0
- **SO:** Windows 10 (Win64)
- **Hardware:** 20 cores, AMD Radeon RX 6650 XT, 32 GB RAM
- **Viewport de teste:** 1280x720 @ DPR 1

## Como executar

1. Abrir o opencode neste diretório
2. Fornecer a URL de um jogo no itch.io
3. Os 6 cenários padrão são testados:
   - Initial Load
   - Fullscreen
   - Window Resize
   - Gamepad
   - Long Session (30 min) - simulado em janela curta
   - Refresh During Gameplay

Cada jogo recebe seu próprio subdiretório com screenshots, traces (quando aplicável) e `REPORT.md`.

## Resumo comparativo dos jogos testados

| Jogo | Engine | Build | FPS | Console Issues | Veredicto |
| --- | --- | --- | --- | --- | --- |
| [FEED THE AI](feed-the-ai/REPORT.md) (Demo 0.40v) | Unity 6000.3.3f1 WebGL | ~74 MB | ~182 | 1 erro, 4 shader errors, 7 warnings | Needs Fixes Before Release |
| [Serenitrove](serenitrove/REPORT.md) (v1.2.0) | HTML5/JS vanilla | ~25 MB | ~182 | 1 erro (410), 3 warnings | Approved for Release |
| [Peggy's Post](peggys-post/REPORT.md) (v1.5.9) | Unity 2021.3.15f1 WebGL | ~67 MB | ~172 | 1 erro (410), 6 shader errors, 10945 warnings de camera | Needs Fixes Before Release |

## Padrões de bugs cross-game

### WEB-001 / WEB-009 (Major): Refresh zera save em Unity WebGL no itch.io
Ambos os jogos Unity WebGL (FEED THE AI e Peggy's Post) sofrem do mesmo problema:
- Save local não persiste entre reloads (F5 ou refresh)
- Cache IndexedDB falha com `Could not connect to database` no boot
- Console mostra warning `JS_FileSystem_Sync()` deprecated e `autoSyncPersistentDataPath` não habilitado
- Boot completo leva de 3s a 30s a cada reload (dependendo do tamanho do build)

**Recomendação geral:** devs de Unity WebGL devem habilitar `config.autoSyncPersistentDataPath = true;` na `createUnityInstance()` para sincronizar saves automaticamente.

### WEB-006 / WEB-013 (Minor): Erro HTTP 410 em endpoint de tracking do itch
Ambos os jogos (Serenitrove e Peggy's Post) recebem erro `410 Gone` em `GET /rh/...` (endpoint de tracking do itch). Provavelmente deprecated do lado do itch.io. Não bloqueia gameplay.

### Shaders URP/Postprocessing sem fallback (Minor)
Jogos Unity com URP (FEED THE AI) e Postprocessing Stack v2 (Peggy's Post) mostram `ERROR: Shader` no console por falta de subshaders compatíveis com a GPU testada (AMD RX 6650 XT / ANGLE). Funcional mas deve ser validado em hardware representativo do público.

## Bugs por severidade

### Major (bloqueante)
- **WEB-001 (FEED THE AI):** Refresh zera estado, autoSync desabilitado
- **WEB-009 (Peggy's Post):** Refresh zera estado, IndexedDB cache falha

### Minor (polimento)
- **WEB-002 (FEED THE AI):** Shaders URP sem fallback (CoreCopy, HDRDebugView)
- **WEB-003 (FEED THE AI):** Audio warnings no boot (14 ocorrências)
- **WEB-004 (FEED THE AI):** Deprecation warning de JS_FileSystem_Sync
- **WEB-005 (Serenitrove):** Refresh não auto-restaura autosave
- **WEB-006 (Serenitrove):** Erro 410 do itch
- **WEB-007 (Serenitrove):** Iframe fixo 800x600 não escala
- **WEB-008 (Serenitrove):** Cross-origin impede inspeção programática
- **WEB-010 (Peggy's Post):** Shaders Postprocessing sem fallback
- **WEB-011 (Peggy's Post):** 10945 warnings repetidos de Camera viewport
- **WEB-012 (Peggy's Post):** Iframe 1300x800 não cabe em viewport comum
- **WEB-013 (Peggy's Post):** Erro 410 do itch

## Estatísticas

- **Jogos testados:** 3
- **Bugs encontrados:** 13 (2 Major, 11 Minor)
- **Screenshots capturados:** 19
- **Performance traces:** 1 (FEED THE AI)
- **Veredictos:**
  - 1 Approved for Release (Serenitrove)
  - 2 Needs Fixes Before Release (FEED THE AI, Peggy's Post)

## Observações metodológicas

- **Long Session (30 min)** foi simulado em janela curta (~2-5min) com medições de FPS, heap e console ao longo da sessão. Não foi aguardado o ciclo completo de 30min por restrições de tempo do QA.
- **Gamepad** não foi testado em nenhum dos jogos por ausência de hardware; apenas a presença da Gamepad API foi verificada.
- **Refresh During Gameplay** é o teste mais revelador para Unity WebGL: ambos os jogos Unity (FEED THE AI e Peggy's Post) falham, enquanto o jogo vanilla JS (Serenitrove) passa porque documenta explicitamente o comportamento e provém Export manual.
- **Fullscreen** não foi 100% validado por limitação do Chrome DevTools MCP (cross-origin iframe bloqueia cliques sintéticos em coordenadas); apenas a disponibilidade da API foi confirmada.
