# Game QA Report Template

Use este template como ponto de partida para novos relatórios de QA de jogos web.

```markdown
# Informações da build

- **Nome do jogo:** [Nome + versão]
- **Plataforma:** [itch.io URL]
- **Navegadores testados:** [Chrome 149, Firefox, Edge, etc]
- **Data do teste:** [YYYY-MM-DD]

# Ambiente de teste

- **CPU/GPU:** [ex: 20 cores / AMD Radeon RX 6650 XT]
- **RAM:** [ex: 32 GB]
- **Browser e versão:** [ex: Google Chrome 149.0.0.0]
- **Sistema operacional:** [ex: Windows 10 (Win64)]
- **Engine:** [ex: Unity 6000.3.3f1 WebGL 2.0 / HTML5+JS vanilla / etc]
- **Resolução do viewport:** [ex: 1280x720 @ DPR 1]
- **Conexão:** [ex: 4G efetivo, ~10 Mbps, RTT 50ms]
- **Assets:** [ex: ~74 MB total (.br Brotli)]

# Testes realizados

| Cenário | Resultado | Observações |
| --- | --- | --- |
| Initial Load | Pass | [Detalhes] |
| Fullscreen | Pass (parcial) | [Detalhes] |
| Window Resize | Pass | [Detalhes] |
| Gamepad | N/A | [Sem device conectado] |
| Long Session (30 min) | Não executado | [Sessão curta ~Xmin] |
| Refresh During Gameplay | Fail | [Detalhes] |

# Bugs encontrados

**Bug ID:** WEB-XXX

**Título:** [Descrição curta]

**Passos para reproduzir:**

1. [Passo 1]
2. [Passo 2]

**Resultado esperado:**

[O que deveria acontecer]

**Resultado atual:**

[O que acontece]

**Severidade:** [Major / Minor / Blocker]

---

# Performance Analysis

- **Average FPS:** ~XXX FPS
- **Peak Memory Usage:** ~XX MB
- **Console Errors:** [número e tipos]
- **Network:** [X requests, ~XX MB]
- **Render path:** [WebGL 2.0 + URP / DOM-based / etc]

# Recommendation

**[Approved for Release / Needs Fixes Before Release / Blocked]**

**Reason:**

[Resumo dos achados principais]
```

## Convenções

- Use `Bug ID: WEB-XXX` para numeração sequencial
- Severidade: Blocker (impede uso) > Major (perda significativa de funcionalidade) > Minor (polimento)
- Não use em-dashes (`—`) - substitua por ponto final ou estrutura diferente
- Sempre incluir screenshot no diretório do jogo para cada cenário testado
- Performance trace deve ser salvo em `perf-baseline.json` quando coletado
```
