# #70 reply-router — Deployment Plan (codeboost-tr)

## Ön Koşullar

- [ ] `api.runx.ai` hosted harness endpoint çalışıyor (şu an 404)
- [ ] codeboost-tr GitHub hesabı aktif
- [ ] codeboost-tr GitHub'da runxhq/runx yıldızlanmış
- [ ] runx CLI 0.6.14+ kurulu

## Adım 1: GitHub Repo Oluştur

```bash
# codeboost-tr GitHub'da "runx-reply-router" repo oluştur (public)
git init
git add .
git commit -m "feat: add reply-router skill"
git remote add origin https://github.com/codeboost-tr/runx-reply-router.git
git push -u origin main
```

## Adım 2: runx Registry'e Publish Et

```bash
runx login --provider github --for publish
runx registry publish ./skills/reply-router/SKILL.md --registry https://api.runx.ai
# => <codeboost-tr>/reply-router@<version>
```

## Adım 3: PR Aç (runxhq/runx)

```bash
# Fork runxhq/runx
# Branch: skill/reply-router
# Commit: skills/reply-router klasörünü ekle
git clone https://github.com/codeboost-tr/runx.git
cp -r <repo>/skills/reply-router runx/skills/
git checkout -b skill/reply-router
git add skills/reply-router/
git commit -m "feat(skills): add reply-router skill"
git push origin skill/reply-router
# GitHub'da PR aç → https://github.com/runxhq/runx/pull/<number>
```

## Adım 4: Local Harness Test

```bash
export RUNX_RECEIPT_SIGN_KID=codeboost-tr-key
export RUNX_RECEIPT_SIGN_ED25519_SEED_BASE64=$(node -e "console.log(require('crypto').randomBytes(32).toString('base64'))")
export RUNX_RECEIPT_SIGN_ISSUER_TYPE=ci
runx harness ./skills/reply-router --json
# Beklenen: status=passed, 2 cases, 0 errors
```

## Adım 5: Dogfood Test

```bash
runx skill codeboost-tr/reply-router@<version> --registry https://api.runx.ai --input-json inbound_reply=<fixture> --receipt-dir receipts --json
runx verify --receipt receipts/<receipt-id>.json --json
```

## Adım 6: evidence.json'u Doldur

`skills/reply-router/evidence/evidence.json` dosyasındaki placeholder'ları gerçek değerlerle değiştir:
- `<your-github-username>` → `codeboost-tr`
- `<version>` → publish edilen version (örn: `sha-a1b2c3d`)
- `<commit-sha>` → PR head commit SHA
- `<sha256-receipt-id>` → dogfood receipt ID'si
- `<sha256-idempotency-key>` → suppression event key'i
- `<sha256-digest>` → verification digest

## Adım 7: Preflight + Deliver

```bash
# Preflight
curl -sS https://gofrantic.com/v1/deliveries/preflight \
  -H 'content-type: application/json' \
  -d '{
    "bounty": 70,
    "artifact_refs": [
      "public_url=https://runx.ai/x/codeboost-tr/reply-router@<version>",
      "source_url=https://github.com/codeboost-tr/runx-reply-router/tree/<commit>/skills/reply-router",
      "pr_url=https://github.com/runxhq/runx/pull/<number>",
      "x_yaml=https://raw.githubusercontent.com/runxhq/runx/<pr-head-commit>/skills/reply-router/X.yaml",
      "skill_md=https://raw.githubusercontent.com/runxhq/runx/<pr-head-commit>/skills/reply-router/SKILL.md",
      "evidence_json=https://raw.githubusercontent.com/codeboost-tr/runx-reply-router/<commit>/skills/reply-router/evidence/evidence.json",
      "verification_json=https://raw.githubusercontent.com/codeboost-tr/runx-reply-router/<commit>/skills/reply-router/evidence/verification.json",
      "receipt_ref=runx:receipt:<sha256>",
      "report=https://raw.githubusercontent.com/codeboost-tr/runx-reply-router/<commit>/skills/reply-router/evidence/report.md"
    ]
  }'

python manage_frantic.py deliver 70 \
  --public-url "https://runx.ai/x/codeboost-tr/reply-router@<version>" \
  --source-url "https://github.com/codeboost-tr/runx-reply-router/tree/<commit>/skills/reply-router" \
  --pr-url "https://github.com/runxhq/runx/pull/<number>" \
  --x-yaml "https://raw.githubusercontent.com/runxhq/runx/<pr-head-commit>/skills/reply-router/X.yaml" \
  --skill-md "https://raw.githubusercontent.com/runxhq/runx/<pr-head-commit>/skills/reply-router/SKILL.md" \
  --evidence-json "https://raw.githubusercontent.com/codeboost-tr/runx-reply-router/<commit>/skills/reply-router/evidence/evidence.json" \
  --verification-json "https://raw.githubusercontent.com/codeboost-tr/runx-reply-router/<commit>/skills/reply-router/evidence/verification.json" \
  --receipt-ref "runx:receipt:<sha256>" \
  --report "https://raw.githubusercontent.com/codeboost-tr/runx-reply-router/<commit>/skills/reply-router/evidence/report.md"
```

## Kritik Uyarılar

1. **SHA-based URL kullan** (main branch değil) — raw.githubusercontent.com CDN cache sorunu
2. **evidence.json'daki 6 zorunlu alan** (claim_type, public_url, runx_link_found, summary, audience, why_allowed) JSON üst seviyede olmalı
3. **observations** array formatı: her item `{id, status, summary}` şeklinde
4. **dogfood.harness_cases** her case için `{name, status}` objesi olmalı, string değil
5. Tüm artifact'ler aynı commit SHA'sını referans almalı
