# Session 07 — Tự động hóa quy trình CI/CD với GitHub Actions

Rikkei Education slide deck scaffolded with [hssf-slides](https://www.npmjs.com/package/hssf-slides) via `create-hssf`.

**Runtime mode:** `cdn` (hssf-slides@0.5.0)

**Topic:** CI/CD automation with GitHub Actions — workflows, runners, job orchestration, Gradle build, cache, and debug logs.

## Serve (required)

Do **not** open `index.html` as `file://` for reliable keyboard/fullscreen.

```bash
npx serve .
# open the printed local URL
```

Keyboard: `←` `→` `Space` · `Home` `End` · `F` fullscreen · hash `#2`

## Slide map (20 slides)

| # | Label | Focus |
|---|-------|--------|
| 1 | Title | Tiêu đề Session 07 |
| 2 | Agenda | 5 phần trọng tâm |
| 3 | Sec-01 | Tổng quan GitHub Actions & Runner |
| 4 | Pain-Manual | Hạn chế build/test thủ công |
| 5 | Concept-CICD | CI / CD / Deployment / Tool |
| 6 | Flow-Architecture | Kiến trúc Server–Runner |
| 7 | Runner-Types | Hosted vs Self-hosted |
| 8 | Sec-02 | Cấu trúc Workflow |
| 9 | Workflow-Syntax | YAML keys + sample |
| 10 | Workflow-Demo | `ci.yml` QuickBite |
| 11 | Sec-03 | Điều phối Jobs |
| 12 | Job-Dependency | `needs` + parallel jobs |
| 13 | Matrix-Strategy | Multi JDK / OS |
| 14 | Sec-04 | Build Spring Boot Gradle |
| 15 | Gradle-Build | `bootJar` workflow |
| 16 | Cache-Gradle | Cache dependency |
| 17 | Sec-05 | Debug & log |
| 18 | Debug-Log | 4 bước chẩn đoán |
| 19 | Summary | Tổng kết |
| 20 | End | Brand end |

## Files

| Path | Role |
|------|------|
| `index.html` | Deck markup |
| `styles/deck.css` | Local overrides (`--hssf-stage-font-size`) |
| `assets/images/` | Images |
| `AGENTS.md` | Rules + copy-paste for AI agents |

## CDN mode

CSS/JS load from jsDelivr pin `hssf@0.5.0`. Needs network.

## Authoring

See **AGENTS.md**. Brand: white + red, Montserrat, footer Rikkei Academy.
