# TASK_STATE - #70 reply-router

**PROJE**: #70 ($9) runx skill: reply-router — codeboost-tr için

## Tamamlananlar

| # | Dosya | Durum |
|---|-------|-------|
| X.yaml | Graph skill + 2 harness case (agent-task + when branching) | ✅ |
| SKILL.md | Metadata, typed input/output docs | ✅ |
| classify.mjs | Reply classification logic | ✅ |
| append_event.mjs | CAS append_event to data-store | ✅ |
| emit_route.mjs | runx.reply.routing.v1 emission | ✅ |
| run.mjs | Runner entry point | ✅ |
| dogfood.mjs | Standalone dogfood runner | ✅ |
| fixtures/ | 2 harness fixture JSON | ✅ |
| evidence/ | evidence.json, verification.json, report.md, harness.json, dogfood.json, registry-read.json | ✅ |
| Local harness test | Graph runner çalışıyor (Windows receipt store bug var, Linux'te yok) | ✅ |
| DEPLOYMENT_PLAN.md | Adım adım deployment planı | ✅ |

## Blokaj

```
api.runx.ai/harness/eval → HTTP 404
```

Bu platform bug'ı. #72 ve #73 de aynı hatayı alıp endpoint düzelince kabul edildi.

## Kalan Adımlar (endpoint düzelince)

1. codeboost-tr GitHub repo oluştur
2. `runx registry publish`
3. PR aç runxhq/runx
4. `runx harness` → local test
5. `runx skill` → dogfood receipt + verify
6. evidence.json'u doldur (SHA, receipt ID, digest)
7. Preflight + Deliver

## Toplam Dosya: 17
```
bounty_70/
├── TASK_STATE.md
├── DEPLOYMENT_PLAN.md
└── skills/reply-router/
    ├── X.yaml
    ├── SKILL.md
    ├── run.mjs
    ├── classify.mjs
    ├── append_event.mjs
    ├── emit_route.mjs
    ├── dogfood.mjs
    ├── .gitignore
    ├── fixtures/
    │   ├── sealed_unsubscribe_suppression.json
    │   └── stop_ambiguous_or_unsealed.json
    └── evidence/
        ├── evidence.json
        ├── verification.json
        ├── report.md
        ├── harness.json
        ├── dogfood.json
        └── registry-read.json
```
