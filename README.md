# agent-desktop 速查表 | agent-desktop Cheatsheet | Guía rápida de agent-desktop

> 一页看懂 agent-desktop（Vercel Labs 出品）：给 AI agent 装上可靠的「操作电脑」能力——它不看截图像素猜坐标，而是读操作系统的无障碍树（accessibility tree），树上每个可操作元素都带一个稳定 ref，agent 点 ref 做动作、再拍一张新快照复查，动作可以安全重试。让 agent 用电脑，别再让它对着截图猜按钮在哪。

- 上游项目：[vercel-labs/agent-desktop](https://github.com/vercel-labs/agent-desktop)（2026-11-02 实测 7,564 ★ / 540 forks，47 个 release，Apache-2.0）
- 平台：macOS / Windows / Linux 三平台 CLI（另附同进程调用的 C 库 libagent_desktop_ffi）
- 本页结构：中文主文 → English → Español → Português

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 它是什么、不是什么

| 是 | 不是 |
|---|---|
| 给 agent 的桌面「眼睛 + 手」：快照返回 JSON，无障碍树上每个元素带 ref | 不是截图识别方案：全程不做像素匹配、不猜坐标 |
| 一个 CLI（+ C 库）：循环逻辑留在你自己的 agent 里，工具只负责看与做 | 不是 agent 本体：它不做规划、不调模型，派活的还是你的 agent |
| 默认 headless：动作走无障碍 API，不抢你的鼠标、不夺焦点 | 不是远程桌面：它就在本机跑，操作的是当前用户的桌面会话 |
| 每条命令输出一个 JSON 信封（`ok` + `data` 或 `error`），错误带 code 与可否重试的 disposition | 不保证界面不变：界面一变，旧 ref 就作废，必须重拍快照（见三步循环的三个坑） |

## 安装一行版

```bash
npm install -g agent-desktop
agent-desktop version   # 验证装好
```

- Bun、Yarn、pnpm 装的是同一个包；包内含 x64 与 ARM64 的原生二进制与平台助手。
- 不想用 JS 启动器：去 GitHub Releases 下对应平台的原生压缩包，解压后把 helper 文件和可执行文件放在一起。
- **Homebrew 与 Scoop 的官方包还在提交中，默认目录里还没有**，上游也不再维护自有 tap/bucket——看到「brew install agent-desktop」类说法先打个问号（2026-11-02 上游 README 口径）。

### 三平台权限卡（装完先过这一关）

| 平台 | 要做什么 |
|---|---|
| macOS | 授予 Accessibility（无障碍）权限，跟着向导走：`agent-desktop permissions --guide` |
| Windows | 与终端同一完整性级别的应用免授权，直接用 |
| Linux | 需要用户的图形会话 + `at-spi2-core`；没有图形会话跑不起来 |

## 核心三步循环卡

任何桌面任务都是同一个循环：**snapshot 看 → 按 ref 动作 → 再 snapshot 验**。

```bash
agent-desktop snapshot --app Finder -i      # 看：列出可交互元素，每个带 ref
agent-desktop click @s8f3k2p9:e8            # 做：点快照里的某个 ref
agent-desktop snapshot --app Finder -i      # 验：再拍一张，确认结果
```

三个必须记住的坑：

1. **ref 只属于它那张快照**：ref 必须写全形 `@快照id:e序号`；只写 `@e8` 会被 `INVALID_ARGS` 拒绝。
2. **界面变了就报 STALE_REF**：动作返回 `STALE_REF`（或 `AMBIGUOUS_TARGET`）不是失败，是提醒你重拍快照、换新 ref 再来。多个窗口同名时也是 `AMBIGUOUS_TARGET`，快照会在 error.details 里列候选窗口，用 `--window-id` 指定。
3. **PowerShell 里 ref 必须加引号**：`'@s8f3k2p9:e8'`——裸写的 `@` 是 PowerShell 的 splatting 运算符，不加引号必翻车。

其他实用招：界面太密的大应用先瘦身看骨架 `snapshot --skeleton --app Slack -i --compact`，再用 `snapshot --root @快照id:e3 -i --compact` 钻进一个区域细看；默认 headless 不动你的指针，需要物理键鼠时才加全局 `--headed`。

**给 agent 的内置文档**（版本随二进制走，不会和手头版本对不上）：

```bash
agent-desktop skills get desktop    # 循环、ref、验证与重试规则
agent-desktop skills get platform   # 当前系统的平台差异全表
agent-desktop <command> --help      # 单条命令的参数
npx skills add https://github.com/vercel-labs/agent-desktop --skill agent-desktop   # 把 skill 装进 agent 框架
```

## 命令速查（60 个命令名，56 个可运行）

上游口径：共 60 个命令名，其中 56 个是可运行的操作命令；`key-down`、`key-up`、`mouse-down`、`mouse-up` 四个在能安全释放按键的守护机制就绪前**拒绝运行**，别在脚本里依赖它们。

| 组 | 命令 |
|---|---|
| 观察 Observe | `snapshot`、`find`、`get`、`is`、`screenshot`、`list-surfaces` |
| 按 ref 动作 Act on a ref | `click`、`double-click`、`triple-click`、`right-click`、`type`、`set-value`、`clear`、`focus`、`select`、`toggle`、`check`、`uncheck`、`expand`、`collapse`、`scroll`、`scroll-to` |
| 键鼠 Keyboard & mouse | `press`、`hover`、`drag`、`mouse-move`、`mouse-click`、`mouse-wheel`、`key-down`、`key-up`、`mouse-down`、`mouse-up` |
| 应用与窗口 Apps & windows | `launch`、`close-app`、`list-apps`、`list-windows`、`list-displays`、`focus-window`、`resize-window`、`move-window`、`minimize`、`maximize`、`restore`、`open-system-surface` |
| 通知 Notifications | `list-notifications`、`dismiss-notification`、`dismiss-all-notifications`、`notification-action` |
| 剪贴板 Clipboard | `clipboard-get`、`clipboard-set`、`clipboard-clear` |
| 等待 Wait | `wait` |
| 会话 Sessions | `session`、`trace`、`cursor-overlay` |
| 系统 System | `status`、`permissions`、`version`、`batch`、`skills` |

重试纪律：错误信封里的 `error.disposition.retry` 为 `safe` 时才重试；其余情况先按 `suggestion` 处理或重拍快照，不要无脑循环重试。

## 三平台支持矩阵（上游 README 原表口径）

| 能力 | macOS | Windows | Linux |
|---|---|---|---|
| 无障碍树 | Yes | Yes | Yes（AT-SPI） |
| 点击/输入/键盘 | Yes | Yes | AT-SPI；X11 / GNOME 按键 |
| 鼠标输入 | Yes | Yes | X11，需 `--headed` |
| 截图 | Yes | Yes | X11 窗口；Wayland 走 portal |
| 剪贴板 | Yes | Yes | Yes；Wayland 需 `wl-clipboard` |
| 应用与窗口管理 | Yes | Yes | 启动/列举；X11 窗口控制 |
| 通知 | Yes | Yes | GNOME Shell |
| 系统界面（open-system-surface） | Planned | Yes | GNOME 托盘、Action Center |

一句话读法：macOS 与 Windows 是全功能；Linux 能读能点，但鼠标、窗口控制、截图都和 X11/Wayland 的会话形态强相关，先跑 `agent-desktop skills get platform` 看本机实际能力表再设计任务。

## 诚实标注区

- **省 token 数字是厂商自报**：「快照比截图省 78–96% token」是 Vercel 方面的自报口径，西语圈新闻也注明未经独立评测；方向合理（结构化文本天然比图像便宜），但具体数字请按证据 B 看待，别当实测引用。
- **桌面隐私会进快照**：上游明确警告——快照、剪贴板读取、截图、命令输出与调试产物都可能包含桌面隐私内容，不要往公开日志、公开仓库里传；可选的 JEV helper 会把屏幕描述发给你配置的模型服务商，它和原生 CLI 是两回事，启用前先想清楚。
- **macOS 的小脾气**：每次快照命名空间有一个私有命令宿主进程，闲置 15 分钟自动退出；`TMPDIR` 必须属于当前用户且权限 0700（macOS 默认的每用户临时目录满足）；headless 调用禁止前台激活与合成输入，这是设计约束不是 bug。
- **四个命令暂时是摆设**：`key-down / key-up / mouse-down / mouse-up` 会拒绝运行（等守护机制保证能释放按键），需要「按住不放」类动作的流程现在设计不了。
- **项目很新、迭代很快**：2026-02 才建仓、已有 47 个 release；命令与行为口径以你装的那一版 `skills get` 输出为准，本页对应 2026-11-02 的上游 README。

---

## English (summary)

agent-desktop (by vercel-labs, ~7.6k stars as of 2026-11-02, Apache-2.0) gives AI agents computer use through the OS accessibility tree instead of screenshots: a snapshot returns JSON where every actionable element carries a stable ref, the agent acts on a ref, then snapshots again to verify — no pixel matching, and the loop stays in your agent. It runs on macOS, Windows and Linux, headless by default (actions go through the accessibility API without moving the pointer or stealing focus; `--headed` opts into physical input).

Install with `npm install -g agent-desktop` (Bun/Yarn/pnpm work too; official Homebrew and Scoop packages are still being submitted). Permissions: macOS needs the Accessibility grant (`agent-desktop permissions --guide`), Windows needs no grant at your terminal's integrity level, Linux needs a graphical session with `at-spi2-core`. Refs belong to their snapshot — a bare `@e8` is rejected, a changed UI returns `STALE_REF` and you simply re-snapshot; in PowerShell, always quote refs. It ships 60 command names (56 operational; the four key/mouse hold commands refuse to run for now) across observe, act-on-ref, keyboard/mouse, apps/windows, notifications, clipboard, wait, sessions and system groups, and the binary carries its own version-matched docs via `agent-desktop skills get desktop` / `skills get platform`.

Honest notes: the "78–96% fewer tokens than screenshots" figure is the vendor's own claim, not independently benchmarked; snapshots, screenshots and clipboard reads can contain private desktop content, so keep them out of public logs; and Linux support is real but leans on AT-SPI/X11, with several capabilities varying by session type.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

agent-desktop (de vercel-labs, ~7,6 mil estrellas a noviembre de 2026, Apache-2.0) da a los agentes de IA la capacidad de usar la computadora a través del árbol de accesibilidad del sistema operativo, sin capturas de pantalla: una instantánea (*snapshot*) devuelve JSON donde cada elemento accionable lleva una referencia estable (*ref*); el agente actúa sobre la ref y vuelve a capturar para verificar. Sin correspondencia de píxeles, y el bucle vive en tu propio agente. Funciona en macOS, Windows y Linux, en modo *headless* por defecto: las acciones van por la API de accesibilidad sin mover el puntero ni robar el foco (con `--headed` se permite la entrada física).

**Instalación**: `npm install -g agent-desktop` (también con Bun, Yarn o pnpm; los paquetes oficiales de Homebrew y Scoop aún están en trámite). **Permisos**: en macOS hay que conceder Accesibilidad (`agent-desktop permissions --guide`); en Windows no hace falta ningún permiso para apps de tu mismo nivel de integridad; en Linux se necesita la sesión gráfica del usuario con `at-spi2-core`.

**El bucle de tres pasos**: `snapshot --app <app> -i` para ver, actuar sobre una ref (p. ej. `click @id:e8`), y otro `snapshot` para verificar. Tres trampas conocidas: la ref pertenece a su instantánea (un `@e8` a secas se rechaza); si la interfaz cambió, la acción devuelve `STALE_REF` y hay que capturar de nuevo; y en PowerShell las refs van siempre entre comillas. Trae 60 nombres de comando (56 operativos; los cuatro de «mantener pulsado» se niegan a correr por ahora) en grupos de observación, acciones por ref, teclado/ratón, apps/ventanas, notificaciones, portapapeles, espera, sesiones y sistema; y el propio binario trae su documentación con `agent-desktop skills get desktop` y `skills get platform`.

**Notas honestas**: la cifra de «78–96 % menos tokens que con capturas» es del propio fabricante, sin evaluación independiente; las instantáneas, capturas y lecturas del portapapeles pueden contener contenido privado de tu escritorio, así que no deben ir a registros públicos; y en Linux varias capacidades dependen de X11/Wayland y del tipo de sesión.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

O agent-desktop (da vercel-labs, ~7,6 mil estrelas em novembro de 2026, Apache-2.0) dá aos agentes de IA a capacidade de usar o computador pela árvore de acessibilidade do sistema operacional, sem capturas de tela: um *snapshot* devolve JSON onde cada elemento acionável carrega uma referência estável (*ref*); o agente age sobre a ref e captura de novo para verificar. Sem correspondência de pixels, e o loop fica no seu próprio agente. Roda em macOS, Windows e Linux, em modo *headless* por padrão: as ações vão pela API de acessibilidade sem mover o ponteiro nem roubar o foco (com `--headed`, a entrada física é permitida).

**Instalação**: `npm install -g agent-desktop` (também com Bun, Yarn ou pnpm; os pacotes oficiais de Homebrew e Scoop ainda estão em submissão). **Permissões**: no macOS é preciso conceder Acessibilidade (`agent-desktop permissions --guide`); no Windows nenhum permiso é necessário para apps do mesmo nível de integridade do terminal; no Linux é preciso a sessão gráfica do usuário com `at-spi2-core`.

**O loop de três passos**: `snapshot --app <app> -i` para ver, agir sobre uma ref (p. ex. `click @id:e8`), e outro `snapshot` para verificar. Três armadilhas conhecidas: a ref pertence ao seu snapshot (um `@e8` sozinho é rejeitado); se a interface mudou, a ação devolve `STALE_REF` e é preciso capturar de novo; e no PowerShell as refs vão sempre entre aspas. São 60 nomes de comando (56 operacionais; os quatro de «manter pressionado» se recusam a rodar por enquanto) em grupos de observação, ações por ref, teclado/mouse, apps/janelas, notificações, área de transferência, espera, sessões e sistema; e o próprio binário traz sua documentação com `agent-desktop skills get desktop` e `skills get platform`.

**Notas honestas**: o número de «78–96% menos tokens que capturas» é do próprio fabricante, sem avaliação independente; snapshots, capturas e leituras da área de transferência podem conter conteúdo privado da sua área de trabalho, então nada de registros públicos; e no Linux várias capacidades dependem de X11/Wayland e do tipo de sessão.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [vercel-labs/agent-desktop](https://github.com/vercel-labs/agent-desktop)
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文；命令名单、平台矩阵与权限要求以 2026-11-02 的上游 README 为准，上游迭代很快，请以官方文档与 `agent-desktop skills get` 的版本内文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao
