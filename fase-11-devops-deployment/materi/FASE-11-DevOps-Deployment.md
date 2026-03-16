# 📘 FASE 11: DevOps & Deployment (Fase Terakhir)

> **Prasyarat:** Fase 7 (Microservices) + Fase 9 (Observability) + Fase 10 (Testing)
> **Durasi:** 3–4 minggu
> **Project Akhir:** Production Deployment untuk E-Commerce System
> **Tujuan:** Deploy sistem ke production dengan percaya diri — CI/CD otomatis, zero-downtime deployment, secret management, disaster recovery, dan infrastructure as code

---

## 🗂️ Daftar Modul

| # | Modul | Topik |
|---|-------|-------|
| 11.1 | DevOps Fundamentals | DORA metrics, CI/CD philosophy, DevOps culture |
| 11.2 | GitHub Actions CI/CD | Full pipeline: test → build → push → deploy |
| 11.3 | Docker Registry | GHCR, image tagging, vulnerability scan |
| 11.4 | Kubernetes Production | Namespace, RBAC, NetworkPolicy, resource management |
| 11.5 | Helm Charts | Templating, values, hooks, chart repository |
| 11.6 | GitOps dengan ArgoCD | GitOps principles, App of Apps pattern |
| 11.7 | Blue-Green & Canary | Zero-downtime strategies, traffic splitting |
| 11.8 | Database Migration | Flyway/golang-migrate, zero-downtime schema changes |
| 11.9 | Secret Management | Kubernetes Secrets, Vault, External Secrets Operator |
| 11.10 | Infrastructure as Code | Terraform untuk cloud resources |
| 11.11 | Cost & Resource Optimization | Right-sizing, auto-scaling, spot instances |
| 11.12 | Disaster Recovery | Backup, restore, RTO/RPO, runbooks |
| 11.13 | Complete Production System | Semua komponen terintegrasi |

---

## 📦 Modul 11.1 — DevOps Fundamentals

### Apa itu DevOps?

```
DevOps = Development + Operations
Bukan hanya tools — ini tentang KULTUR dan CARA KERJA

Sebelum DevOps:
  Dev team:  "Code sudah selesai, terserah ops yang deploy"
  Ops team:  "Ini bukan masalah kami, code-nya yang salah"
  Release:   4x per tahun (terlalu berisiko untuk lebih sering)
  Deployment: manual, penuh risiko, butuh "deployment window"

Dengan DevOps:
  Satu tim bertanggung jawab dari code sampai production
  Release: beberapa kali per hari (continuous delivery)
  Deployment: otomatis, bisa rollback dalam menit
  "You build it, you run it" — Werner Vogels (Amazon CTO)
```

### DORA Metrics — Mengukur DevOps Performance

```
Google's DevOps Research & Assessment (DORA) mengidentifikasi
4 metrics yang membedakan tim elite dari tim biasa:

1. DEPLOYMENT FREQUENCY
   Elite:   Multiple deploys per day
   High:    Once per week to once per month
   Medium:  Once per month to once per 6 months
   Low:     Less than once per 6 months

2. LEAD TIME FOR CHANGES
   Elite:   Less than 1 hour (commit → production)
   High:    1 day to 1 week
   Medium:  1 month to 6 months
   Low:     More than 6 months

3. MEAN TIME TO RESTORE (MTTR)
   Elite:   Less than 1 hour
   High:    Less than 1 day
   Medium:  1 day to 1 week
   Low:     More than 1 week

4. CHANGE FAILURE RATE
   Elite:   0–15%
   High:    16–30%
   Medium/Low: 46–60%

Target kita: MASUK KATEGORI ELITE dengan CI/CD yang baik.
```

### CI/CD Pipeline Overview

```
CI (Continuous Integration):
  Developer push code → otomatis:
    1. Run linter
    2. Run unit tests
    3. Run integration tests
    4. Build Docker image
    5. Scan vulnerabilities
  
  Tujuan: detect masalah SESEGERA MUNGKIN setelah push

CD (Continuous Delivery/Deployment):
  Delivery:   Setiap commit bisa di-deploy ke prod (tapi perlu approval manual)
  Deployment: Setiap commit otomatis di-deploy ke prod

Pipeline lengkap:
  Code push
    → Lint + Test (CI)
    → Build + Push image
    → Deploy to staging
    → Integration/smoke test
    → Deploy to production (otomatis atau dengan approval)
    → Verify production health
    → Rollback if unhealthy
```

### Git Branching Strategy

```
TRUNK-BASED DEVELOPMENT (recommended untuk CI/CD):

  main ─────────────────────────────────────────── (production)
   │                    │                  │
   ├── feature/add-cart │                  │
   │   (short-lived,    │                  │
   │    < 2 hari)       │                  │
   │         └──────────┘                  │
   │         (merge via PR)                │
   │                                       │
   ├── hotfix/fix-payment-bug              │
   │         └─────────────────────────────┘

Prinsip:
  - Branch hidup pendek (< 2 hari)
  - Merge ke main sesering mungkin (minimal 1x per hari)
  - Main selalu dalam kondisi bisa di-deploy
  - Feature flags untuk feature yang belum siap

GITFLOW (untuk tim yang lebih besar):
  main → production
  develop → staging (integration)
  feature/* → develop
  release/* → main + develop
  hotfix/* → main + develop
```

### 🏋️ Latihan 11.1

1. Ukur **DORA metrics saat ini** untuk project kamu: (a) Deployment Frequency — berapa sering kamu push ke repo? (b) Lead Time — berapa lama dari commit pertama sampai PR merge? (c) Change Failure Rate — berapa % commit yang perlu di-revert? Buat rencana untuk improve ke level berikutnya.
2. Desain **full CI/CD pipeline diagram** untuk E-Commerce System: gambar setiap stage, tools yang dipakai, kondisi yang menyebabkan fail, dan siapa yang bertanggung jawab per stage.
3. Buat **Git branching policy** untuk tim 5 developer: tentukan branch naming convention, PR requirements (reviewers, checks), merge strategy, dan hotfix procedure.

---

## 📦 Modul 11.2 — GitHub Actions CI/CD Pipeline Lengkap

### Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     GitHub Actions Pipeline                   │
│                                                              │
│  Push/PR → Lint → Test → Build → Push → Deploy → Verify     │
│                                                              │
│  Jobs (berjalan paralel):                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │  Lint    │  │   Test   │  │ Security │                   │
│  │          │  │  (unit)  │  │   Scan   │                   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                   │
│       └─────────────┼─────────────┘                         │
│                     ▼ (semua pass)                           │
│            ┌─────────────────┐                               │
│            │   Build & Push  │                               │
│            │  Docker Image   │                               │
│            └────────┬────────┘                               │
│                     ▼                                        │
│            ┌─────────────────┐                               │
│            │  Deploy Staging │                               │
│            └────────┬────────┘                               │
│                     ▼                                        │
│            ┌─────────────────┐                               │
│            │  Smoke Test     │                               │
│            └────────┬────────┘                               │
│                     ▼ (main branch only)                     │
│            ┌─────────────────┐                               │
│            │  Deploy Prod    │                               │
│            │  (with approval)│                               │
│            └─────────────────┘                               │
└──────────────────────────────────────────────────────────────┘
```

### Workflow File Lengkap

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  workflow_dispatch:  # manual trigger
    inputs:
      environment:
        description: 'Deploy to environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]

env:
  GO_VERSION: '1.22'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ============================================================
  # Job 1: Lint
  # ============================================================
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: golangci-lint
        uses: golangci/golangci-lint-action@v3
        with:
          version: latest
          args: --timeout=5m

      - name: Check go mod tidy
        run: |
          go mod tidy
          git diff --exit-code go.mod go.sum || \
            (echo "go.mod/go.sum not tidy, run 'go mod tidy'" && exit 1)

  # ============================================================
  # Job 2: Unit Tests
  # ============================================================
  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: Run unit tests with race detector
        run: |
          go test -v -race -short \
            -coverprofile=coverage.out \
            -covermode=atomic \
            ./internal/domain/... \
            ./internal/usecase/...

      - name: Check coverage threshold
        run: |
          COVERAGE=$(go tool cover -func=coverage.out | \
            grep total | awk '{print $3}' | tr -d '%')
          echo "Coverage: ${COVERAGE}%"
          if (( $(echo "$COVERAGE < 75" | bc -l) )); then
            echo "❌ Coverage ${COVERAGE}% is below threshold 75%"
            exit 1
          fi
          echo "✅ Coverage OK: ${COVERAGE}%"

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.out
          fail_ci_if_error: false

  # ============================================================
  # Job 3: Integration Tests
  # ============================================================
  test-integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports: ["5432:5432"]

      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports: ["6379:6379"]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: Run integration tests
        env:
          TEST_DATABASE_URL: "host=localhost user=testuser password=testpass dbname=testdb sslmode=disable"
          TEST_REDIS_URL: "redis://localhost:6379"
        run: |
          go test -v -race \
            -coverprofile=coverage-integration.out \
            ./test/integration/...

  # ============================================================
  # Job 4: Security Scan
  # ============================================================
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true

      - name: Run govulncheck
        run: |
          go install golang.org/x/vuln/cmd/govulncheck@latest
          govulncheck ./...

      - name: Run gosec (security static analysis)
        uses: securego/gosec@master
        with:
          args: '-severity medium -confidence medium ./...'

  # ============================================================
  # Job 5: Build & Push Docker Image
  # ============================================================
  build-push:
    name: Build & Push
    runs-on: ubuntu-latest
    needs: [lint, test-unit, test-integration, security]
    if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    strategy:
      matrix:
        service: [auth-service, product-service, order-service, api-gateway]
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}
          tags: |
            type=sha,prefix=,suffix=,format=short
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: ./services/${{ matrix.service }}
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            VERSION=${{ github.sha }}
            COMMIT_SHA=${{ github.sha }}
            BUILD_TIME=${{ github.event.head_commit.timestamp }}
          provenance: true    # SBOM generation
          sbom: true

      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL'
          exit-code: '1'  # fail jika ada critical vulnerability

      - name: Upload security scan results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # ============================================================
  # Job 6: Deploy to Staging
  # ============================================================
  deploy-staging:
    name: Deploy Staging
    runs-on: ubuntu-latest
    needs: build-push
    environment:
      name: staging
      url: https://staging.myecommerce.com
    steps:
      - uses: actions/checkout@v4

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > ~/.kube/config

      - name: Deploy to staging
        run: |
          # Update image tags di Kubernetes manifests
          for service in auth-service product-service order-service api-gateway; do
            kubectl set image deployment/$service \
              $service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/$service:${{ github.sha }} \
              -n staging
          done

      - name: Wait for rollout
        run: |
          for service in auth-service product-service order-service api-gateway; do
            kubectl rollout status deployment/$service -n staging --timeout=5m
          done

      - name: Run smoke tests
        run: |
          STAGING_URL="https://staging.myecommerce.com"
          # Health check semua service
          for path in "/health" "/api/v1/products"; do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$STAGING_URL$path")
            if [ "$STATUS" != "200" ]; then
              echo "❌ Smoke test failed: $STAGING_URL$path returned $STATUS"
              exit 1
            fi
          done
          echo "✅ All smoke tests passed"

  # ============================================================
  # Job 7: Deploy to Production (main branch only, dengan approval)
  # ============================================================
  deploy-production:
    name: Deploy Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myecommerce.com
    steps:
      - uses: actions/checkout@v4

      - name: Configure kubeconfig
        run: |
          echo "${{ secrets.KUBE_CONFIG_PROD }}" | base64 -d > ~/.kube/config

      - name: Deploy to production (canary 10%)
        run: |
          # Deploy canary dulu — 10% traffic
          kubectl apply -f k8s/canary/ -n production

          # Update canary image
          for service in auth-service product-service order-service api-gateway; do
            kubectl set image deployment/$service-canary \
              $service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/$service:${{ github.sha }} \
              -n production
          done

      - name: Monitor canary (5 menit)
        run: |
          echo "Monitoring canary for 5 minutes..."
          sleep 300

          # Cek error rate dari Prometheus
          ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?\
            query=sum(rate(http_requests_total{status=~'5..'}[5m]))/\
            sum(rate(http_requests_total[5m]))" | \
            jq -r '.data.result[0].value[1]')

          if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo "❌ Error rate too high: $ERROR_RATE — rolling back canary"
            kubectl rollout undo deployment -l app=canary -n production
            exit 1
          fi

          echo "✅ Canary healthy, promoting to full rollout"

      - name: Full production rollout
        run: |
          for service in auth-service product-service order-service api-gateway; do
            kubectl set image deployment/$service \
              $service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/$service:${{ github.sha }} \
              -n production
            kubectl rollout status deployment/$service -n production --timeout=10m
          done

      - name: Notify deployment
        uses: slackapi/slack-github-action@v1.25.0
        with:
          channel-id: '#deployments'
          slack-message: |
            ✅ Production deployment complete!
            Service: E-Commerce System
            Version: ${{ github.sha }}
            By: ${{ github.actor }}
            Link: ${{ github.event.head_commit.url }}
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

### 🏋️ Latihan 11.2

1. Setup GitHub Actions workflow dari nol untuk Auth Service: (a) lint + unit test pada setiap PR, (b) build + push image ke GHCR pada merge ke main. Test dengan buat PR dan verifikasi workflow berjalan.
2. Tambahkan **matrix build**: satu workflow yang membangun SEMUA service (auth, product, order, gateway) secara paralel menggunakan `matrix.service`. Ukur perbedaan waktu vs sequential build.
3. Buat **reusable workflow** (workflow_call): ekstrak `build-and-push` job sebagai reusable workflow yang bisa dipanggil dari workflow manapun. Refactor workflow utama untuk menggunakan reusable workflow ini.

---

## 📦 Modul 11.3 — Docker Registry & Image Management

### GitHub Container Registry (GHCR)

```bash
# Login ke GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag image
docker tag auth-service:latest ghcr.io/USERNAME/auth-service:latest
docker tag auth-service:latest ghcr.io/USERNAME/auth-service:1.0.0
docker tag auth-service:latest ghcr.io/USERNAME/auth-service:$(git rev-parse --short HEAD)

# Push
docker push ghcr.io/USERNAME/auth-service:latest

# Pull
docker pull ghcr.io/USERNAME/auth-service:1.0.0
```

### Image Tagging Strategy

```
SEMANTIC VERSIONING (untuk releases):
  ghcr.io/org/auth-service:1.0.0      ← specific release
  ghcr.io/org/auth-service:1.0        ← minor version alias
  ghcr.io/org/auth-service:1          ← major version alias
  ghcr.io/org/auth-service:latest     ← latest stable

GIT SHA (untuk CI/CD):
  ghcr.io/org/auth-service:abc1234    ← commit SHA (7 chars)
  ghcr.io/org/auth-service:main-abc1234  ← branch + SHA

ENVIRONMENT:
  ghcr.io/org/auth-service:staging    ← current staging
  ghcr.io/org/auth-service:prod       ← current production

BEST PRACTICE:
  - Gunakan SHA untuk deployment (immutable, traceable)
  - Gunakan semver untuk releases yang stable
  - JANGAN deploy dengan tag "latest" ke production!
    (tidak immutable → tidak tahu persis versi yang running)
```

### Image Scanning dengan Trivy

```yaml
# .github/workflows/scan.yml
- name: Scan image for vulnerabilities
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'ghcr.io/org/auth-service:${{ github.sha }}'
    format: 'table'
    exit-code: '1'            # fail jika ada critical
    ignore-unfixed: true      # skip CVE yang belum ada fix-nya
    severity: 'CRITICAL,HIGH'
    trivyignores: '.trivyignore'  # list CVE yang di-suppress

# .trivyignore — CVE yang sudah diketahui dan diterima risikonya
# CVE-2023-12345  # Library X: tidak exposed ke network, risk accepted
# CVE-2023-67890  # Fixed in next update, planned for Q1
```

### Multi-Platform Build

```dockerfile
# Build untuk multiple architectures (amd64 + arm64)
# Penting untuk: Apple Silicon (M1/M2), Graviton (AWS ARM)

# Dockerfile sudah benar dengan CGO_ENABLED=0
# Hanya perlu update build command:
```

```yaml
# GitHub Actions dengan multi-platform
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3

- name: Build multi-platform
  uses: docker/build-push-action@v5
  with:
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ${{ steps.meta.outputs.tags }}
```

### Image Retention Policy

```bash
# Hapus images lama dari registry (jalankan sebagai cron job)
# GitHub Actions scheduled workflow:

# .github/workflows/cleanup-registry.yml
name: Cleanup Old Images
on:
  schedule:
    - cron: '0 2 * * 0'  # setiap Minggu jam 2 pagi

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/delete-package-versions@v4
        with:
          package-name: 'auth-service'
          package-type: 'container'
          min-versions-to-keep: 10      # simpan 10 versi terbaru
          delete-only-pre-release-versions: false
          ignore-versions: '^(latest|main|prod|staging)$'
```

### 🏋️ Latihan 11.3

1. Setup **GHCR** untuk semua service. Buat `.github/workflows/publish.yml` yang push image ke GHCR saat merge ke main. Verifikasi image bisa di-pull dengan `docker pull ghcr.io/username/auth-service:latest`.
2. Implementasikan **tagging strategy lengkap**: setiap push ke main men-tag image dengan: SHA, branch name, semantic version (dari git tag), dan `latest`. Verifikasi semua tags muncul di GHCR.
3. Setup **Trivy scan** di CI: scan setiap image yang di-build. Buat `.trivyignore` file untuk suppress false positive. Verify CI fail saat ada CRITICAL CVE dengan `exit-code: 1`.

---

## 📦 Modul 11.4 — Kubernetes Production Setup

### Namespace & RBAC

```yaml
# k8s/namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    env: production
    team: platform
---
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    env: staging
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  labels:
    purpose: monitoring
```

```yaml
# k8s/rbac/service-account.yaml
# Setiap service punya ServiceAccount sendiri
apiVersion: v1
kind: ServiceAccount
metadata:
  name: auth-service
  namespace: production
  annotations:
    # Untuk cloud IAM integration (AWS IRSA, GKE Workload Identity)
    eks.amazonaws.com/role-arn: "arn:aws:iam::123456789:role/auth-service-role"
---
# Role: izin yang dibutuhkan auth-service
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: auth-service-role
  namespace: production
rules:
  # Bisa baca Secrets untuk config
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["auth-service-secrets"]
    verbs: ["get", "list"]
  # Bisa baca ConfigMaps
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: auth-service-rolebinding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: auth-service
    namespace: production
roleRef:
  kind: Role
  name: auth-service-role
  apiGroup: rbac.authorization.k8s.io
```

### Network Policy (Zero-Trust Networking)

```yaml
# k8s/networkpolicy/auth-service.yaml
# Default: deny semua ingress dan egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: auth-service-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: auth-service
  policyTypes:
    - Ingress
    - Egress

  ingress:
    # Hanya terima dari API Gateway
    - from:
        - podSelector:
            matchLabels:
              app: api-gateway
      ports:
        - protocol: TCP
          port: 8080

    # Terima health check dari Kubernetes (kube-apiserver)
    - from:
        - namespaceSelector:
            matchLabels:
              name: kube-system
      ports:
        - protocol: TCP
          port: 8080

    # Terima scraping dari Prometheus
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
        - podSelector:
            matchLabels:
              app: prometheus
      ports:
        - protocol: TCP
          port: 8080

  egress:
    # Izinkan ke PostgreSQL
    - to:
        - podSelector:
            matchLabels:
              app: postgres-auth
      ports:
        - protocol: TCP
          port: 5432

    # Izinkan ke Redis
    - to:
        - podSelector:
            matchLabels:
              app: redis
      ports:
        - protocol: TCP
          port: 6379

    # Izinkan DNS resolution
    - to:
        - namespaceSelector:
            matchLabels:
              name: kube-system
        - podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

### TLS dengan cert-manager & Let's Encrypt

cert-manager otomatis provisioning dan renewal TLS certificates.

```bash
# Install cert-manager
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set installCRDs=true
```

```yaml
# k8s/cert-manager/cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@myecommerce.com
    privateKeySecretRef:
      name: letsencrypt-prod-private-key
    solvers:
      - http01:
          ingress:
            class: nginx
---
# k8s/ingress-tls.yaml — Ingress dengan TLS otomatis
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.myecommerce.com
      secretName: ecommerce-tls-secret  # cert-manager akan isi ini otomatis!
  rules:
    - host: api.myecommerce.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-gateway
                port:
                  number: 8080
```

```bash
# Verify certificate issued
kubectl get certificate -n production
# NAME                     READY   SECRET                AGE
# ecommerce-tls-secret     True    ecommerce-tls-secret  2m

# Check cert details
kubectl describe certificate ecommerce-tls-secret -n production
```


### PodDisruptionBudget

```yaml
# k8s/pdb/auth-service-pdb.yaml
# Pastikan availability saat node maintenance
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: auth-service-pdb
  namespace: production
spec:
  # Minimal 2 pods harus always running
  minAvailable: 2
  selector:
    matchLabels:
      app: auth-service
```

### Resource Quotas per Namespace

```yaml
# k8s/resourcequota/production-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    # Compute resources
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    # Object counts
    pods: "100"
    services: "20"
    secrets: "50"
    persistentvolumeclaims: "20"
```

### LimitRange (Default Resources)

```yaml
# k8s/limitrange/production-limits.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: production-limits
  namespace: production
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "2"
        memory: "2Gi"
      min:
        cpu: "50m"
        memory: "64Mi"
```

### 🏋️ Latihan 11.4

1. Setup **RBAC** lengkap: buat ServiceAccount, Role, dan RoleBinding untuk Auth Service. ServiceAccount hanya boleh akses Secret `auth-service-secrets` dan ConfigMap `auth-service-config`. Verifikasi dengan `kubectl auth can-i`.
2. Implementasikan **NetworkPolicy**: Auth Service hanya terima traffic dari API Gateway, hanya bisa connect ke database dan Redis. Test dengan: buat pod test dan coba curl Auth Service → harus ditolak kecuali dari namespace yang diizinkan.
3. Buat **resource audit script**: script yang cek semua deployments di namespace production dan identifikasi: (a) pod yang tidak punya resource requests/limits, (b) pod yang tidak punya PodDisruptionBudget, (c) pod yang tidak punya liveness/readiness probe.

---

## 📦 Modul 11.5 — Helm Charts

### Apa itu Helm?

```
Helm = Package manager untuk Kubernetes
Seperti apt/brew tapi untuk Kubernetes manifests

Tanpa Helm:
  auth-service/
    deployment.yaml (hardcoded values)
    service.yaml
    configmap.yaml
    secret.yaml
  
  Untuk staging vs production → copy, edit manual → error prone!

Dengan Helm:
  charts/auth-service/
    Chart.yaml       (metadata chart)
    values.yaml      (default values)
    values-staging.yaml
    values-production.yaml
    templates/
      deployment.yaml  (templated dengan {{ .Values.xxx }})
      service.yaml
      configmap.yaml
      _helpers.tpl     (reusable template snippets)
  
  Deploy: helm install auth-service ./charts/auth-service \
    -f values-production.yaml
```

### Struktur Chart

```
charts/auth-service/
├── Chart.yaml           ← metadata: name, version, description
├── values.yaml          ← default values untuk semua environment
├── values-staging.yaml  ← staging overrides
├── values-production.yaml ← production overrides
└── templates/
    ├── _helpers.tpl     ← reusable macros
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    ├── hpa.yaml
    ├── pdb.yaml
    ├── networkpolicy.yaml
    └── NOTES.txt        ← ditampilkan setelah install
```

### Chart.yaml

```yaml
# charts/auth-service/Chart.yaml
apiVersion: v2
name: auth-service
description: Authentication and Authorization Service for E-Commerce
type: application
version: 0.1.0        # versi chart (bukan versi aplikasi)
appVersion: "1.0.0"   # versi aplikasi
maintainers:
  - name: Platform Team
    email: platform@mycompany.com
dependencies:
  # Tambahkan chart yang dibutuhkan
  - name: postgresql
    version: "13.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled  # bisa disabled jika pakai managed DB
```

### values.yaml

```yaml
# charts/auth-service/values.yaml

# Image configuration
image:
  repository: ghcr.io/mycompany/auth-service
  pullPolicy: IfNotPresent
  tag: ""  # default: Chart.appVersion

# Replica configuration
replicaCount: 2

# Service configuration
service:
  type: ClusterIP
  port: 8080
  grpcPort: 50051

# Resource limits
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi

# Auto scaling
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

# Health checks
livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 30

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

# Application config
config:
  appEnv: production
  logLevel: warn
  logFormat: json
  shutdownTimeout: 30s

# Database
database:
  host: postgres-auth
  port: 5432
  name: authdb
  user: authuser
  sslMode: require
  maxOpen: 50
  maxIdle: 10

# JWT
jwt:
  accessExpiry: 15m
  refreshExpiry: 168h

# Ingress
ingress:
  enabled: false
  className: nginx
  hosts: []

# Pod annotations (untuk Prometheus scraping)
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
  prometheus.io/path: "/metrics"

# Security context
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 65532
  fsGroup: 65532

# PodDisruptionBudget
pdb:
  enabled: true
  minAvailable: 1
```

### Template: deployment.yaml

```yaml
# charts/auth-service/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "auth-service.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "auth-service.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "auth-service.selectorLabels" . | nindent 6 }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        {{- include "auth-service.selectorLabels" . | nindent 8 }}
      annotations:
        # Force restart jika config berubah
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    spec:
      {{- with .Values.podSecurityContext }}
      securityContext:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      terminationGracePeriodSeconds: 30
      serviceAccountName: {{ include "auth-service.serviceAccountName" . }}
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.port }}
            - name: grpc
              containerPort: {{ .Values.service.grpcPort }}
          envFrom:
            - configMapRef:
                name: {{ include "auth-service.fullname" . }}-config
            - secretRef:
                name: {{ include "auth-service.fullname" . }}-secrets
          {{- with .Values.livenessProbe }}
          livenessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.readinessProbe }}
          readinessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.resources }}
          resources:
            {{- toYaml . | nindent 12 }}
          {{- end }}
```

### values-production.yaml

```yaml
# charts/auth-service/values-production.yaml
# Hanya override yang berbeda dari default

replicaCount: 3

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20

config:
  appEnv: production
  logLevel: warn

database:
  sslMode: require
  maxOpen: 100
  maxIdle: 20

pdb:
  enabled: true
  minAvailable: 2
```

### Helm Commands

```bash
# Install chart
helm install auth-service ./charts/auth-service \
  -n production \
  -f ./charts/auth-service/values-production.yaml \
  --set image.tag=abc1234

# Upgrade (update)
helm upgrade auth-service ./charts/auth-service \
  -n production \
  -f ./charts/auth-service/values-production.yaml \
  --set image.tag=def5678 \
  --atomic \          # rollback otomatis jika gagal
  --timeout=5m

# Rollback
helm rollback auth-service 1 -n production  # rollback ke revision 1

# List releases
helm list -n production
helm list -A  # semua namespace

# History
helm history auth-service -n production

# Dry run (preview tanpa apply)
helm upgrade auth-service ./charts/auth-service \
  --dry-run --debug \
  -f values-production.yaml

# Lint
helm lint ./charts/auth-service

# Template (render tanpa apply)
helm template auth-service ./charts/auth-service \
  -f values-production.yaml | kubectl apply --dry-run=client -f -
```

### 🏋️ Latihan 11.5

1. Buat **Helm chart** untuk Auth Service dari scratch. Sertakan templates: Deployment, Service, ConfigMap, HPA, PDB. Gunakan `values.yaml` untuk semua konfigurasi. Verifikasi dengan `helm lint` dan `helm template`.
2. Buat **values file** untuk 3 environment: development, staging, production. Pastikan differences jelas: replica count, resource limits, log level. Deploy ke minikube dengan tiap values file.
3. Buat **"App of Apps" chart**: satu parent chart yang depend pada semua service charts (auth, product, order, gateway). Satu command `helm install ecommerce ./charts/ecommerce` men-deploy seluruh sistem.

---

## 📦 Modul 11.6 — GitOps dengan ArgoCD

### Apa itu GitOps?

```
GitOps = Git sebagai single source of truth untuk infrastructure

Prinsip GitOps:
  1. Seluruh desired state tersimpan di Git (declarative)
  2. Automated agents memastikan actual state = desired state
  3. Perubahan infra HANYA melalui Git (tidak manual kubectl apply)
  4. Semua perubahan ada history-nya (git log)

Tanpa GitOps (push model):
  Developer → kubectl apply → Cluster
  Masalah: tidak ada audit, manual error, drift antara Git dan cluster

Dengan GitOps (pull model):
  Developer → Git commit → [ArgoCD pulls] → Cluster
  ArgoCD terus compare: "Git state = Cluster state?"
  Jika berbeda → sync otomatis atau alert

Tools GitOps:
  ArgoCD     → paling populer, UI bagus, multi-cluster
  Flux       → lightweight, CNCF project
  Jenkins X  → untuk Jenkins ecosystem
```

### Setup ArgoCD

```bash
# Install ArgoCD di cluster
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait until ready
kubectl wait --for=condition=available deployment -l "app.kubernetes.io/name=argocd-server" \
  -n argocd --timeout=300s

# Get initial admin password
kubectl get secret argocd-initial-admin-secret \
  -n argocd -o jsonpath="{.data.password}" | base64 -d

# Port-forward ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Akses: https://localhost:8080
# Login: admin / (password dari atas)

# Install CLI
brew install argocd
argocd login localhost:8080
argocd account update-password
```

### ArgoCD Application

```yaml
# argocd/applications/auth-service.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: auth-service
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io  # cleanup saat delete
spec:
  project: ecommerce

  # Source: dari Git repository
  source:
    repoURL: https://github.com/mycompany/ecommerce-manifests
    targetRevision: HEAD  # branch/tag/SHA
    path: helm/auth-service  # folder di dalam repo
    helm:
      valueFiles:
        - values-production.yaml
      parameters:
        - name: image.tag
          value: "abc1234"  # di-update oleh CI/CD

  # Destination: cluster dan namespace
  destination:
    server: https://kubernetes.default.svc
    namespace: production

  # Sync policy
  syncPolicy:
    automated:
      prune: true    # hapus resource yang tidak ada di Git
      selfHeal: true # sync otomatis jika ada drift
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - ApplyOutOfSyncOnly=true  # hanya apply yang berubah
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m
        factor: 2

  # Health check
  ignoreDifferences:
    # Ignore replicas jika dikelola oleh HPA
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

### App of Apps Pattern

```yaml
# argocd/applications/ecommerce-app-of-apps.yaml
# Satu application yang mengelola semua applications lainnya!
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ecommerce-apps
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/mycompany/ecommerce-manifests
    targetRevision: HEAD
    path: argocd/applications  # folder yang berisi semua application.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### CI/CD dengan GitOps (Image Updater)

```yaml
# argocd/applications/auth-service.yaml — dengan image updater
metadata:
  annotations:
    # ArgoCD Image Updater: otomatis update image tag
    argocd-image-updater.argoproj.io/image-list: auth=ghcr.io/org/auth-service
    argocd-image-updater.argoproj.io/auth.update-strategy: newest-build
    argocd-image-updater.argoproj.io/auth.helm.image-name: image.repository
    argocd-image-updater.argoproj.io/auth.helm.image-tag: image.tag
    argocd-image-updater.argoproj.io/write-back-method: git  # commit ke Git!
```

### 🏋️ Latihan 11.6

1. Install ArgoCD di minikube. Buat ArgoCD Application untuk Auth Service yang sync dari Git repo. Test: ubah nilai di `values.yaml`, commit ke Git, dan verifikasi ArgoCD otomatis sync ke cluster.
2. Implementasikan **App of Apps pattern**: satu root application yang mengelola semua service apps. Verifikasi: delete satu service app via kubectl → ArgoCD otomatis create kembali (self-healing).
3. Setup **ArgoCD Image Updater**: ketika image baru di-push ke GHCR, Image Updater otomatis update `image.tag` di values file dan commit ke Git. CI/CD flow: push code → GitHub Actions build image → Image Updater detects → update Git → ArgoCD sync.

---

## 📦 Modul 11.7 — Blue-Green & Canary Deployment

### Blue-Green Deployment

```
Blue (current production):  v1.0 — 100% traffic
Green (new version):        v1.1 — 0% traffic

Steps:
  1. Deploy v1.1 sebagai "green" (tidak ada traffic)
  2. Run smoke tests pada green
  3. Switch traffic 100% ke green (instantaneous!)
  4. Monitor green selama 30 menit
  5. Jika OK → blue menjadi standby/delete
     Jika tidak OK → switch traffic balik ke blue (instant rollback!)

Keuntungan:
  + Rollback instan (tinggal switch traffic)
  + Zero downtime
  + Full testing sebelum live

Kekurangan:
  - Butuh 2x resources saat deployment
  - Database schema harus backward compatible
```

```yaml
# k8s/blue-green/service.yaml
# Service yang mengarahkan traffic ke blue atau green
apiVersion: v1
kind: Service
metadata:
  name: auth-service
  namespace: production
spec:
  selector:
    app: auth-service
    slot: blue  # ← ubah ke "green" untuk switch traffic!
  ports:
    - port: 8080
      targetPort: 8080
---
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service-blue
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-service
      slot: blue
  template:
    metadata:
      labels:
        app: auth-service
        slot: blue
        version: "1.0.0"
    spec:
      containers:
        - name: auth-service
          image: ghcr.io/org/auth-service:1.0.0
---
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service-green
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-service
      slot: green
  template:
    metadata:
      labels:
        app: auth-service
        slot: green
        version: "1.1.0"
    spec:
      containers:
        - name: auth-service
          image: ghcr.io/org/auth-service:1.1.0
```

```bash
# Switch traffic ke green
kubectl patch service auth-service \
  -n production \
  -p '{"spec":{"selector":{"slot":"green"}}}'

# Rollback ke blue jika ada masalah
kubectl patch service auth-service \
  -n production \
  -p '{"spec":{"selector":{"slot":"blue"}}}'
```

### Canary Deployment dengan Nginx Ingress

```yaml
# k8s/canary/ingress-canary.yaml
# Kirim 10% traffic ke canary (v1.1), 90% ke stable (v1.0)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: auth-service-canary
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% traffic
    # Atau berdasarkan header:
    # nginx.ingress.kubernetes.io/canary-by-header: "X-Canary"
    # nginx.ingress.kubernetes.io/canary-by-header-value: "true"
spec:
  ingressClassName: nginx
  rules:
    - host: api.myecommerce.com
      http:
        paths:
          - path: /api/v1/auth
            pathType: Prefix
            backend:
              service:
                name: auth-service-canary  # service untuk canary deployment
                port:
                  number: 8080
```

### Progressive Delivery dengan Flagger

```yaml
# Flagger otomatis manage canary rollout
# Install: kubectl apply -k github.com/fluxcd/flagger/kustomize/kubernetes

apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: auth-service
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: auth-service

  progressDeadlineSeconds: 600

  service:
    port: 8080
    targetPort: 8080

  # Traffic routing
  canaryAnalysis:
    interval: 1m       # check setiap 1 menit
    threshold: 5       # max 5x gagal sebelum rollback
    maxWeight: 50      # max 50% traffic ke canary
    stepWeight: 10     # naik 10% setiap interval

    # Metrics yang di-monitor
    metrics:
      - name: request-success-rate
        interval: 1m
        thresholdRange:
          min: 99  # minimal 99% success rate

      - name: request-duration
        interval: 1m
        thresholdRange:
          max: 500  # max 500ms

    # Webhooks (opsional: jalankan e2e test)
    webhooks:
      - name: smoke-test
        type: pre-rollout
        url: http://test-runner.testing/
        timeout: 5m
        metadata:
          type: bash
          cmd: "curl -sd 'test' http://auth-service-canary.production/health | grep ok"
```

### Rolling Update — Konfigurasi yang Benar

```yaml
# Kubernetes rolling update strategy detail
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # max 1 pod boleh tidak available saat update
      maxSurge: 1         # max 1 pod extra boleh dibuat saat update

# Contoh dengan 5 replicas:
# Sebelum: [v1][v1][v1][v1][v1]  (5 pods)
# Step 1:  [v1][v1][v1][v1][v2]  (maxSurge: 1 pod baru dibuat → 6 total)
# Step 2:  [v1][v1][v1][ X][v2]  (maxUnavailable: 1 pod lama dihapus → 5 total)
# Step 3:  [v1][v1][v1][v2][v2]
# ...
# Final:   [v2][v2][v2][v2][v2]

# Untuk high availability (tidak ada downtime):
# maxUnavailable: 0  → tidak ada pod yang dihapus sebelum pod baru ready
# maxSurge: 1        → satu pod extra dibuat dulu
# Trade-off: butuh lebih banyak resource sementara

# Verify rolling update berjalan:
kubectl rollout status deployment/auth-service -n production
# Waiting for deployment "auth-service" rollout to finish:
# 1 out of 3 new replicas have been updated...
# 2 out of 3 new replicas have been updated...
# 3 out of 3 new replicas have been updated...
# Waiting for 1 old replicas to be terminated...
# deployment "auth-service" successfully rolled out

# Rollback jika ada masalah:
kubectl rollout undo deployment/auth-service -n production
kubectl rollout undo deployment/auth-service --to-revision=2 -n production

# History
kubectl rollout history deployment/auth-service -n production
# REVISION  CHANGE-CAUSE
# 1         Initial deployment v1.0.0
# 2         Upgrade to v1.1.0
# 3         Upgrade to v1.2.0
```

### GitHub Environments — Approval Gate untuk Production

```yaml
# .github/workflows/ci-cd.yml (tambahan)
# GitHub Environments memungkinkan required approval sebelum deploy production

deploy-production:
  runs-on: ubuntu-latest
  needs: [deploy-staging]
  # environment = nama di GitHub Settings > Environments
  # Protection rules bisa dikonfigurasi: required reviewers, wait timer, dll
  environment:
    name: production          # ← nama environment di GitHub
    url: https://myecommerce.com
  # Dengan protection rules:
  # - Required reviewers: 1 orang harus approve
  # - Wait timer: 5 menit cooldown
  # - Deployment branch: hanya dari main
  steps:
    - name: Deploy to production
      run: |
        echo "Deploying ${{ github.sha }} to production..."
        # ... deployment commands
```

```
Setup di GitHub UI:
  Repository → Settings → Environments → New environment → "production"
  
  Protection rules:
  ✅ Required reviewers (tambahkan tim lead / DevOps)
  ✅ Wait timer: 0-43200 menit (opsional)  
  ✅ Deployment branches: Selected branches → "main"
  
  Environment secrets (berbeda dari repo secrets!):
  KUBE_CONFIG_PROD=...
  DATABASE_URL_PROD=...
  
Workflow akan PAUSE di job deploy-production dan menunggu approval.
Reviewer dapat email notifikasi dan bisa approve/reject dari GitHub UI.
```


### 🏋️ Latihan 11.7

1. Implementasikan **Blue-Green deployment** untuk Auth Service di minikube: deploy blue (v1.0), kemudian deploy green (v1.1), switch traffic ke green, verify, lalu simulasikan rollback ke blue.
2. Setup **Canary deployment** dengan Nginx Ingress: kirim 10% traffic ke canary. Gunakan `hey` untuk generate traffic: `hey -n 1000 http://localhost/health` dan verifikasi ~100 request masuk ke canary dari Grafana.
3. Buat **automated canary promotion script**: script bash yang (a) deploy canary, (b) monitor error rate 5 menit, (c) jika error < 1% → promote ke 100%, jika error >= 1% → rollback otomatis.

---

## 📦 Modul 11.8 — Database Migration di Production

### Masalah Database Migration

```
Database migration di production adalah salah satu OPERASI PALING BERBAHAYA.
Salah langkah bisa: data corrupt, downtime panjang, rollback sangat sulit.

Aturan Zero-Downtime Migration:
  1. BACKWARD COMPATIBLE: skema baru harus bisa dibaca oleh kode LAMA
     (karena selama rolling update, ada pod lama dan pod baru berjalan bersamaan)
  2. MULTI-PHASE: jangan ubah schema sekaligus, lakukan bertahap
  3. TEST DI STAGING DULU: selalu test migration di staging sebelum prod
  4. PUNYA ROLLBACK PLAN: tahu cara undo jika ada masalah
```

### Setup golang-migrate

```bash
# Install golang-migrate
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest

# Struktur folder migrations
migrations/
  000001_create_users_table.up.sql
  000001_create_users_table.down.sql
  000002_add_refresh_tokens.up.sql
  000002_add_refresh_tokens.down.sql
  000003_add_user_index.up.sql
  000003_add_user_index.down.sql
```

```sql
-- migrations/000001_create_users_table.up.sql
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role        VARCHAR(20) NOT NULL DEFAULT 'user',
    is_active   BOOLEAN NOT NULL DEFAULT TRUE,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at  TIMESTAMPTZ
);

CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_active ON users(is_active) WHERE deleted_at IS NULL;

-- migrations/000001_create_users_table.down.sql
DROP TABLE IF EXISTS users CASCADE;
```

### Zero-Downtime Migration Patterns

```sql
-- SCENARIO: Tambah kolom NOT NULL ke tabel yang sudah ada data

-- ❌ CARA SALAH (menyebabkan lock / downtime):
ALTER TABLE users ADD COLUMN phone_number VARCHAR(20) NOT NULL;
-- Error: existing rows tidak punya nilai untuk phone_number
-- Atau: table lock selama ALTER pada tabel besar

-- ✅ CARA BENAR (multi-phase, zero downtime):

-- Phase 1: Tambah kolom NULLABLE (deploy ke prod, aman!)
-- migrations/000010_add_phone_column.up.sql
ALTER TABLE users ADD COLUMN IF NOT EXISTS phone_number VARCHAR(20);
-- Kolom ada tapi nullable → kode lama (yang tidak tahu phone_number) tetap bisa insert

-- Deploy kode baru yang:
--   - Menulis phone_number saat user baru register
--   - Bisa baca phone_number (tapi handle null dengan baik)

-- Phase 2 (hari/minggu berikutnya): Isi nilai untuk existing rows
-- migrations/000011_backfill_phone.up.sql
UPDATE users SET phone_number = '' WHERE phone_number IS NULL;

-- Phase 3 (setelah semua rows punya nilai): Tambahkan NOT NULL constraint
-- migrations/000012_phone_not_null.up.sql
ALTER TABLE users ALTER COLUMN phone_number SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone_number SET DEFAULT '';
```

```go
// Jalankan migration di startup aplikasi
// internal/infrastructure/database/migrate.go
package database

import (
    "database/sql"
    "fmt"

    "github.com/golang-migrate/migrate/v4"
    "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
)

// RunMigrations menjalankan semua pending migrations
func RunMigrations(db *sql.DB, migrationsPath string) error {
    driver, err := postgres.WithInstance(db, &postgres.Config{})
    if err != nil {
        return fmt.Errorf("create postgres driver: %w", err)
    }

    m, err := migrate.NewWithDatabaseInstance(
        fmt.Sprintf("file://%s", migrationsPath),
        "postgres",
        driver,
    )
    if err != nil {
        return fmt.Errorf("create migrator: %w", err)
    }

    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        return fmt.Errorf("run migrations: %w", err)
    }

    version, dirty, err := m.Version()
    if err != nil {
        return fmt.Errorf("get migration version: %w", err)
    }
    if dirty {
        return fmt.Errorf("database is in dirty state at version %d", version)
    }

    fmt.Printf("Database migrated to version %d\n", version)
    return nil
}
```

```yaml
# kubernetes/jobs/migrate.yaml
# Run migration sebagai Kubernetes Job SEBELUM deployment
apiVersion: batch/v1
kind: Job
metadata:
  name: auth-service-migrate-{{ .Release.Revision }}
  namespace: production
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-1"  # jalankan sebelum yang lain
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: ghcr.io/org/auth-service:{{ .Values.image.tag }}
          command: ["/server", "migrate"]
          envFrom:
            - secretRef:
                name: auth-service-secrets
```

### 🏋️ Latihan 11.8

1. Setup **golang-migrate** di Auth Service. Buat 3 migration files: (a) create users table, (b) add refresh_tokens table, (c) add index pada users.email. Jalankan migration up dan down, verifikasi schema benar.
2. Implementasikan **zero-downtime schema change**: tambah kolom `profile_picture_url VARCHAR(500)` ke tabel users menggunakan multi-phase approach. Tulis test yang verifikasi: kode lama bisa write, kode baru bisa read/write kolom baru.
3. Buat **Kubernetes Job** untuk menjalankan migration sebagai Helm pre-upgrade hook. Verifikasi di minikube: `helm upgrade` otomatis menjalankan migration sebelum update pods.

---

## 📦 Modul 11.9 — Secret Management

### Masalah Secret Management

```
JANGAN PERNAH:
  ❌ Hardcode secrets di code
  ❌ Commit secrets ke Git (termasuk .env!)
  ❌ Log secrets
  ❌ Pakai secrets yang sama di dev dan production

PRAKTIK YANG BAIK:
  ✅ Secrets disimpan di secret manager (Vault, AWS Secrets Manager)
  ✅ Kubernetes Secrets untuk inject ke pods
  ✅ Rotation otomatis
  ✅ Audit log siapa yang akses secret kapan
```

### Kubernetes Secrets

```yaml
# Buat secret dari command line (secret tidak di-commit!)
kubectl create secret generic auth-service-secrets \
  --from-literal=DATABASE_PASSWORD=mysecretpassword \
  --from-literal=JWT_SECRET=myJWTsecretkey32charslongminimum \
  --from-literal=REDIS_PASSWORD=myredispassword \
  -n production

# Atau dari file .env (file tidak di-commit, hanya template-nya)
kubectl create secret generic auth-service-secrets \
  --from-env-file=.env.production \
  -n production
```

```yaml
# Inject secrets ke pod
spec:
  containers:
    - name: auth-service
      envFrom:
        # Inject ALL secret values sebagai env vars
        - secretRef:
            name: auth-service-secrets
      env:
        # Atau inject satu per satu
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: auth-service-secrets
              key: DATABASE_PASSWORD
```

### HashiCorp Vault

```bash
# Install Vault di Kubernetes
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault \
  -n vault --create-namespace \
  -f vault-values.yaml

# Inisialisasi Vault
kubectl exec -it vault-0 -n vault -- vault operator init
# Simpan unseal keys dan root token di tempat yang AMAN!

# Unseal Vault
kubectl exec -it vault-0 -n vault -- vault operator unseal <key1>
kubectl exec -it vault-0 -n vault -- vault operator unseal <key2>
kubectl exec -it vault-0 -n vault -- vault operator unseal <key3>

# Login
kubectl exec -it vault-0 -n vault -- vault login <root-token>

# Tulis secret
vault kv put secret/ecommerce/auth \
  database_password="supersecret" \
  jwt_secret="myJWTsecret32charslong"

# Baca secret
vault kv get secret/ecommerce/auth
```

### External Secrets Operator

```yaml
# External Secrets Operator: sync Vault secrets ke K8s Secrets otomatis
# Install: helm install external-secrets external-secrets/external-secrets

# SecretStore: koneksi ke Vault
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-store
  namespace: production
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "auth-service"
---
# ExternalSecret: secret apa yang perlu di-sync
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: auth-service-secrets
  namespace: production
spec:
  refreshInterval: "1h"   # sync ulang setiap jam
  secretStoreRef:
    name: vault-store
    kind: SecretStore
  target:
    name: auth-service-secrets  # nama K8s Secret yang akan dibuat
    creationPolicy: Owner
  data:
    - secretKey: DATABASE_PASSWORD
      remoteRef:
        key: ecommerce/auth
        property: database_password
    - secretKey: JWT_SECRET
      remoteRef:
        key: ecommerce/auth
        property: jwt_secret
```

### 🏋️ Latihan 11.9

1. Setup **Kubernetes Secrets** untuk Auth Service: buat secret dari file `.env.production` (yang TIDAK di-commit ke git). Inject ke pod via `envFrom`. Verify secret tidak muncul di pod environment ketika di-log.
2. Install **Vault** di minikube. Store semua secrets Auth Service di Vault path `secret/ecommerce/auth`. Buat policy yang hanya izinkan auth-service ServiceAccount untuk baca secrets tersebut.
3. Setup **External Secrets Operator**: buat `SecretStore` yang connect ke Vault dan `ExternalSecret` yang sync credentials ke K8s Secret. Verify: ubah password di Vault → K8s Secret otomatis update dalam 1 jam (atau trigger manual refresh).

---

## 📦 Modul 11.10 — Infrastructure as Code (Terraform)

### Apa itu Terraform?

```
Terraform: provisioning infrastruktur cloud dengan kode
  - Declare what you want (not how to do it)
  - Idempotent: apply berkali-kali = hasil sama
  - State management: tahu apa yang sudah ada
  - Plan: preview perubahan sebelum apply

Contoh resource yang bisa di-manage:
  - Kubernetes cluster (EKS, GKE, AKS)
  - Database (RDS, Cloud SQL)
  - Container registry
  - DNS
  - Load balancer
  - VPC, subnets, security groups
  - IAM roles dan policies
```

### Struktur Terraform Project

```
terraform/
├── environments/
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
└── modules/
    ├── kubernetes-cluster/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── rds-postgres/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Module: RDS PostgreSQL

```hcl
# terraform/modules/rds-postgres/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

variable "environment"      { type = string }
variable "service_name"     { type = string }
variable "instance_class"   { type = string  default = "db.t3.micro" }
variable "allocated_storage"{ type = number  default = 20 }
variable "db_name"          { type = string }
variable "db_username"      { type = string }
variable "vpc_id"           { type = string }
variable "subnet_ids"       { type = list(string) }

resource "random_password" "db_password" {
  length  = 32
  special = true
}

resource "aws_db_subnet_group" "this" {
  name       = "${var.service_name}-${var.environment}"
  subnet_ids = var.subnet_ids
}

resource "aws_security_group" "rds" {
  name   = "${var.service_name}-rds-${var.environment}"
  vpc_id = var.vpc_id

  ingress {
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]  # hanya dari VPC internal
  }
}

resource "aws_db_instance" "this" {
  identifier = "${var.service_name}-${var.environment}"

  engine               = "postgres"
  engine_version       = "16"
  instance_class       = var.instance_class
  allocated_storage    = var.allocated_storage
  storage_encrypted    = true  # WAJIB!
  storage_type         = "gp3"

  db_name  = var.db_name
  username = var.db_username
  password = random_password.db_password.result

  db_subnet_group_name   = aws_db_subnet_group.this.name
  vpc_security_group_ids = [aws_security_group.rds.id]

  # Backup
  backup_retention_period   = 7     # simpan 7 hari
  backup_window             = "02:00-04:00"  # jam 2-4 pagi UTC
  maintenance_window        = "mon:04:00-mon:06:00"
  delete_automated_backups  = false

  # High Availability
  multi_az = var.environment == "production"

  # Prevent accidental deletion
  deletion_protection = var.environment == "production"

  # Enable performance insights
  performance_insights_enabled = true

  skip_final_snapshot = var.environment != "production"
  final_snapshot_identifier = var.environment == "production" ? \
    "${var.service_name}-final-snapshot" : null

  tags = {
    Environment = var.environment
    Service     = var.service_name
    ManagedBy   = "terraform"
  }
}

# Store password di AWS Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "${var.service_name}/${var.environment}/db-password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
  secret_string = jsonencode({
    username = var.db_username
    password = random_password.db_password.result
    host     = aws_db_instance.this.address
    port     = aws_db_instance.this.port
    dbname   = var.db_name
  })
}

output "endpoint"     { value = aws_db_instance.this.address }
output "port"         { value = aws_db_instance.this.port }
output "secret_arn"   { value = aws_secretsmanager_secret.db_password.arn }
```

### Terraform Commands

```bash
# Initialize
terraform init

# Format code
terraform fmt -recursive

# Validate
terraform validate

# Plan (preview perubahan)
terraform plan -var-file="terraform.tfvars" -out=tfplan

# Apply
terraform apply tfplan

# Destroy (HATI-HATI di production!)
terraform destroy -target=aws_db_instance.this

# State operations
terraform state list
terraform state show aws_db_instance.auth_db
terraform import aws_db_instance.existing myexisting-db  # import existing resource
```

### Remote State & Workspaces

```hcl
# terraform/backend.tf — simpan state di S3 (tidak di local!)
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "ecommerce/terraform.tfstate"
    region         = "ap-southeast-1"
    encrypt        = true                    # enkripsi state di S3
    dynamodb_table = "terraform-state-lock"  # lock untuk prevent concurrent apply
  }
}

# Atau gunakan Terraform Cloud (gratis untuk tim kecil):
# terraform {
#   cloud {
#     organization = "mycompany"
#     workspaces {
#       name = "ecommerce-production"
#     }
#   }
# }
```

```bash
# Terraform Workspaces untuk multiple environments
terraform workspace new staging
terraform workspace new production
terraform workspace list     # lihat semua
terraform workspace select production

# Gunakan workspace dalam kode
variable "environment" {
  default = terraform.workspace  # "staging" atau "production"
}

resource "aws_db_instance" "this" {
  instance_class = terraform.workspace == "production" ? "db.t3.medium" : "db.t3.micro"
  multi_az       = terraform.workspace == "production"
}
```

### GitHub Actions untuk Terraform

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
    paths: ['terraform/**']
  pull_request:
    paths: ['terraform/**']

jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: terraform/environments/production

    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: "1.7.0"

      - name: Terraform Init
        run: terraform init
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan -out=tfplan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      # Hanya apply di main branch (bukan PR)
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply tfplan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```


### 🏋️ Latihan 11.10

1. Tulis **Terraform module** untuk membuat PostgreSQL di Docker (gunakan provider Kreuzwerker Docker untuk local development, bukan AWS). Buat module yang bisa di-reuse untuk auth-db dan product-db. Gunakan `terraform apply` untuk membuat kedua database.
2. Buat **Terraform untuk Kubernetes resources**: provisioning Namespace, ServiceAccount, dan RBAC untuk Auth Service menggunakan Kubernetes Terraform provider. Ini lebih reliable dari manual `kubectl apply`.
3. Implementasikan **Terraform state management** dengan remote backend: gunakan Terraform Cloud (free tier) atau S3 backend untuk menyimpan state. Pastikan state di-encrypt dan di-lock saat ada yang sedang apply.

---

## 📦 Modul 11.11 — Cost Optimization & Resource Management

### Right-Sizing Resources

```
MASALAH UMUM:
  Developer set resource requests terlalu tinggi "untuk aman":
  requests.cpu: 2000m  (2 CPU)
  limits.cpu:   4000m  (4 CPU)
  Tapi actual usage: 100m (0.1 CPU)
  → Cloud bill membengkak!

CARA MEASURE ACTUAL USAGE:
  kubectl top pods -n production
  kubectl top nodes

  # Atau pakai VPA (Vertical Pod Autoscaler) untuk rekomendasi
  kubectl describe vpa auth-service -n production
```

### Vertical Pod Autoscaler (VPA)

```yaml
# VPA: otomatis rekomendasi atau update resource requests
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: auth-service-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: auth-service

  updatePolicy:
    updateMode: "Off"  # "Off" = hanya rekomendasi, tidak auto-update
    # "Auto" = otomatis update (restart pod jika perlu)
    # "Initial" = set saat pod baru dibuat

  resourcePolicy:
    containerPolicies:
      - containerName: auth-service
        minAllowed:
          cpu: 50m
          memory: 64Mi
        maxAllowed:
          cpu: 2000m
          memory: 1Gi
```

### Cluster Autoscaler & Spot Instances

```yaml
# Gunakan Spot Instances untuk workload yang tolerant terhadap interruption
# Hemat 60-80% biaya!

# Kubernetes node pools:
# - On-demand: untuk critical workloads (database, gateway)
# - Spot/Preemptible: untuk stateless services (auth, product, order)

# Node selector untuk target spot nodes
spec:
  nodeSelector:
    cloud.google.com/gke-spot: "true"  # GKE
    # eks.amazonaws.com/capacityType: "SPOT"  # EKS

# Tolerations untuk spot nodes
  tolerations:
    - key: cloud.google.com/gke-spot
      operator: Equal
      value: "true"
      effect: NoSchedule
```

### KEDA — Event-Driven Autoscaling

```yaml
# KEDA: scale berdasarkan event/queue depth, bukan CPU/memory
# Berguna untuk: Kafka consumer, queue workers

apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: notification-service-scaler
  namespace: production
spec:
  scaleTargetRef:
    name: notification-service

  minReplicaCount: 0    # scale to zero saat tidak ada messages!
  maxReplicaCount: 10

  triggers:
    - type: kafka
      metadata:
        bootstrapServers: kafka:29092
        consumerGroup: notification-service-v1
        topic: orders
        lagThreshold: "100"  # scale up jika lag > 100 messages
        offsetResetPolicy: latest
```

### 🏋️ Latihan 11.11

1. Audit resource usage di cluster: gunakan `kubectl top pods` untuk semua service. Identifikasi service yang over-provisioned (allocated >> actual). Buat rekomendasi right-sizing.
2. Setup **VPA** untuk Auth Service dalam mode "Off" (rekomendasi saja). Jalankan load test 5 menit, kemudian lihat rekomendasi VPA. Bandingkan dengan nilai saat ini.
3. Implementasikan **KEDA** untuk Notification Service: scale berdasarkan Kafka consumer lag. Test: stop consumer → kirim banyak messages → start consumer → verifikasi KEDA scale up.

---

## 📦 Modul 11.12 — Disaster Recovery & Backup

### RTO dan RPO

```
RTO (Recovery Time Objective):
  "Berapa lama sistem BOLEH down setelah disaster?"
  Contoh: RTO = 1 jam → sistem harus online lagi dalam 1 jam

RPO (Recovery Point Objective):
  "Berapa banyak data yang BOLEH hilang?"
  Contoh: RPO = 1 jam → data dalam 1 jam terakhir BOLEH hilang

Hubungan RTO/RPO dengan strategi:
  RTO < 1 menit: Multi-region active-active
  RTO < 5 menit: Hot standby, automatic failover
  RTO < 1 jam:   Warm standby, database replica
  RTO < 8 jam:   Cold standby, restore from backup
```

### Database Backup Strategy

```bash
# PostgreSQL backup dengan pg_dump
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME \
  --format=custom \
  --compress=9 \
  --file=backup-$(date +%Y%m%d-%H%M%S).pgdump

# Upload ke S3
aws s3 cp backup-*.pgdump s3://mycompany-db-backups/auth-service/

# Restore
pg_restore \
  --host=$DB_HOST \
  --username=$DB_USER \
  --dbname=$DB_NAME \
  --clean \
  --if-exists \
  backup-20250115-020000.pgdump
```

```yaml
# kubernetes/cronjobs/backup.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: auth-db-backup
  namespace: production
spec:
  schedule: "0 2 * * *"  # Setiap hari jam 2 pagi
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: postgres:16-alpine
              command:
                - /bin/sh
                - -c
                - |
                  TIMESTAMP=$(date +%Y%m%d-%H%M%S)
                  BACKUP_FILE="/backup/auth-db-$TIMESTAMP.pgdump"
                  
                  pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME \
                    --format=custom --compress=9 \
                    --file=$BACKUP_FILE
                  
                  # Upload ke S3
                  aws s3 cp $BACKUP_FILE \
                    s3://$BACKUP_BUCKET/auth-service/$TIMESTAMP.pgdump
                  
                  # Verifikasi upload
                  aws s3 ls s3://$BACKUP_BUCKET/auth-service/$TIMESTAMP.pgdump
                  
                  # Hapus backup lokal lebih dari 7 hari
                  find /backup -name "*.pgdump" -mtime +7 -delete
                  
                  echo "Backup complete: $BACKUP_FILE"
              env:
                - name: DB_HOST
                  valueFrom:
                    secretKeyRef:
                      name: auth-service-secrets
                      key: DATABASE_HOST
              volumeMounts:
                - name: backup-storage
                  mountPath: /backup
          volumes:
            - name: backup-storage
              persistentVolumeClaim:
                claimName: backup-pvc
```

### Disaster Recovery Runbook

```markdown
# DR Runbook: Auth Service Database Failure

## Severity: P0 — Complete database failure

## Symptoms
- Auth service returning 503 errors
- Prometheus alert: "database_connections_failed"
- Logs: "failed to connect to database"

## Impact
- Users cannot login
- All protected endpoints unavailable
- Estimated revenue loss: Rp X/menit

## Diagnosis Steps (5 menit)
1. Check pod status: `kubectl get pods -n production -l app=auth-service`
2. Check DB endpoint: `kubectl exec -it auth-service-xxx -- nc -zv $DB_HOST 5432`
3. Check DB logs: `kubectl logs auth-db-0 -n production --tail=50`
4. Check if backup available: `aws s3 ls s3://backups/auth-service/ | tail -5`

## Recovery Options

### Option A: Restart database (< 2 menit)
If database pod crashed:
`kubectl rollout restart statefulset/auth-db -n production`
Wait: `kubectl rollout status statefulset/auth-db -n production`

### Option B: Failover ke replica (< 5 menit)
If primary unavailable:
`kubectl patch service auth-db -p '{"spec":{"selector":{"role":"replica"}}}'`
This will cause brief read-only period

### Option C: Restore from backup (30-60 menit)
If data corruption:
1. Identify latest good backup:
   `aws s3 ls s3://backups/auth-service/ | sort | tail -5`
2. Create new database instance
3. Restore: `pg_restore --clean ...`
4. Update connection string in secret
5. Restart auth service pods

## Post-Recovery
- Monitor error rate for 30 minutes
- Send incident report to stakeholders
- Update runbook with lessons learned
```

### 🏋️ Latihan 11.12

1. Buat **backup CronJob** untuk Auth Service database: backup setiap jam ke storage lokal (gunakan hostPath untuk dev). Test backup dan restore: backup, drop tabel, restore, verify data ada kembali.
2. Buat **DR runbook** lengkap untuk skenario "Auth Service database tidak bisa diakses". Sertakan: symptoms, impact, step-by-step diagnosis, 3 recovery options dengan estimated RTO per opsi.
3. Lakukan **DR drill**: simulasikan database failure (stop postgres pod), ikuti runbook, ukur actual RTO. Dokumentasikan: waktu setiap step, apa yang berjalan sesuai rencana, apa yang perlu diperbaiki.

---

## 📦 Modul 11.13 — The Complete Production System

### Arsitektur Final Production

```
                    ╔══════════════════════════════════════╗
                    ║           Internet                   ║
                    ╚══════════════════════════════════════╝
                                    │
                          ╔═════════╧════════╗
                          ║   Load Balancer  ║
                          ║  (Cloud LB/NGINX)║
                          ╚═════════╤════════╝
                                    │ HTTPS
              ╔═════════════════════╧═════════════════════╗
              ║                Kubernetes Cluster          ║
              ║  ┌─────────────────────────────────────┐  ║
              ║  │        Namespace: production         │  ║
              ║  │                                     │  ║
              ║  │  [API Gateway]                      │  ║
              ║  │       │                             │  ║
              ║  │  ┌────┴────┐  ┌─────────┐          │  ║
              ║  │  │  Auth   │  │Product  │  [Order] │  ║
              ║  │  │ Service │  │ Service │  Service │  ║
              ║  │  └────┬────┘  └────┬────┘          │  ║
              ║  └───────┼────────────┼───────────────┘  ║
              ║          │            │                   ║
              ║  ┌───────▼────────────▼───────────────┐  ║
              ║  │         Namespace: data             │  ║
              ║  │  [PostgreSQL] [Redis] [Kafka]       │  ║
              ║  └────────────────────────────────────┘  ║
              ║                                          ║
              ║  ┌──────────────────────────────────┐   ║
              ║  │    Namespace: monitoring          │   ║
              ║  │  [Prometheus] [Grafana] [Jaeger]  │   ║
              ║  └──────────────────────────────────┘   ║
              ║                                          ║
              ║  ┌──────────────────────────────────┐   ║
              ║  │    Namespace: argocd              │   ║
              ║  │  [ArgoCD] [Vault]                 │   ║
              ║  └──────────────────────────────────┘   ║
              ╚══════════════════════════════════════════╝

CI/CD Flow:
  Developer → Git Push → GitHub Actions → Docker Build → GHCR
                                       → ArgoCD detects → Sync to K8s
```

### Full Stack Production Makefile

```makefile
# Makefile — complete production operations

.PHONY: deploy-staging deploy-production rollback status health

# Deploy ke staging
deploy-staging:
	helm upgrade --install ecommerce ./charts/ecommerce \
		-n staging --create-namespace \
		-f ./charts/ecommerce/values-staging.yaml \
		--set global.imageTag=$(shell git rev-parse --short HEAD) \
		--atomic --timeout=10m

# Deploy ke production (butuh confirmation)
deploy-production: _confirm
	helm upgrade --install ecommerce ./charts/ecommerce \
		-n production --create-namespace \
		-f ./charts/ecommerce/values-production.yaml \
		--set global.imageTag=$(shell git rev-parse --short HEAD) \
		--atomic --timeout=15m
		
_confirm:
	@echo "⚠️  Deploy to PRODUCTION with tag $(shell git rev-parse --short HEAD)"
	@echo "Press CTRL-C to cancel, ENTER to continue..."
	@read

# Rollback production ke revision sebelumnya
rollback:
	helm rollback ecommerce -n production
	kubectl rollout status deployment -l app.kubernetes.io/part-of=ecommerce -n production

# Status semua services
status:
	@echo "=== Deployments ==="
	kubectl get deployment -n production
	@echo ""
	@echo "=== Pods ==="
	kubectl get pods -n production
	@echo ""
	@echo "=== HPA ==="
	kubectl get hpa -n production
	@echo ""
	@echo "=== Services ==="
	kubectl get svc -n production

# Health check semua endpoints
health:
	@for svc in auth-service product-service order-service api-gateway; do \
		echo -n "$$svc: "; \
		kubectl exec -n production \
			$$(kubectl get pod -l app=$$svc -n production -o name | head -1) \
			-- wget -qO- http://localhost:8080/health/ready | jq -r .status; \
	done

# Run migration
migrate:
	kubectl apply -f k8s/jobs/migrate.yaml -n production
	kubectl wait --for=condition=complete job/migrate -n production --timeout=5m

# Backup database
backup:
	kubectl create job --from=cronjob/auth-db-backup manual-backup-$(shell date +%Y%m%d) \
		-n production

# Logs semua service
logs:
	stern -n production -l "app.kubernetes.io/part-of=ecommerce" --tail=50
```

### Production Launch Checklist

```markdown
# Pre-Production Launch Checklist

## Infrastructure
- [ ] Kubernetes cluster provisioned dan running
- [ ] Semua namespaces dibuat (production, staging, monitoring, argocd)
- [ ] RBAC dan ServiceAccounts dikonfigurasi
- [ ] NetworkPolicies aktif (zero-trust)
- [ ] ResourceQuota dan LimitRange di production namespace
- [ ] PodDisruptionBudget untuk semua critical services

## Application
- [ ] Semua Docker images di-build dan di-push ke GHCR
- [ ] Trivy scan tidak ada CRITICAL CVE
- [ ] Helm charts tersedia dan sudah di-lint
- [ ] Database migrations sudah dijalankan dan verified
- [ ] Semua environment variables dan secrets sudah dikonfigurasi
- [ ] Config validation saat startup

## CI/CD
- [ ] GitHub Actions pipeline green (lint + test + build + push)
- [ ] ArgoCD terhubung ke repo dan sync berjalan
- [ ] Staging environment running dan smoke tests pass
- [ ] Rollback procedure sudah di-test

## Observability
- [ ] Prometheus scraping semua service (/metrics)
- [ ] Grafana dashboards tersedia (RED dashboard per service)
- [ ] Jaeger menerima traces dari semua service
- [ ] Alerting rules dikonfigurasi (error rate, latency, down)
- [ ] SLO tracking aktif
- [ ] Loki menerima logs dari semua service

## Security
- [ ] Semua service running sebagai non-root
- [ ] Secrets disimpan di Vault/Secret Manager (bukan di Git)
- [ ] TLS/HTTPS aktif untuk semua external traffic
- [ ] Dependency vulnerability scan bersih

## Reliability
- [ ] Health checks (liveness + readiness) tersedia semua service
- [ ] Graceful shutdown dikonfigurasi
- [ ] HPA aktif untuk critical services
- [ ] Database backup berjalan dan restore di-test
- [ ] DR runbook tersedia dan sudah di-drill

## Performance
- [ ] Load test sudah dijalankan (k6)
- [ ] Resource requests/limits sudah di-right-size
- [ ] Database indexes sudah dioptimasi
- [ ] Connection pooling dikonfigurasi

## Operations
- [ ] Runbook untuk 5 failure scenarios terbanyak
- [ ] On-call rotation sudah disetup
- [ ] Incident response procedure terdokumentasi
- [ ] Post-mortem template tersedia
```

### 🏋️ Latihan 11.13

1. Buat **production deployment script** lengkap: single command `make deploy-production VERSION=1.0.0` yang: (a) verify tests pass, (b) build dan push image, (c) deploy ke staging, (d) run smoke tests, (e) deploy ke production dengan canary 10%, (f) monitor 5 menit, (g) promote ke full rollout atau rollback.
2. Buat **complete Makefile** dengan semua commands: deploy-staging, deploy-production, rollback, status, health, backup, migrate, logs. Test setiap command di environment staging.
3. Lakukan **end-to-end production drill**: dimulai dari fresh minikube cluster, deploy SEMUA komponen (dengan ArgoCD atau Helm), sampai semua service running, metrics tercollect, traces terlihat di Jaeger, dan checkout flow berjalan. Dokumentasikan waktu yang diperlukan dan hambatan yang ditemui.

---

### Kustomize — Alternatif Helm yang Lebih Sederhana

```
Helm vs Kustomize:
  Helm:      Template engine penuh, variabel, functions, conditionals
             Cocok untuk: distributable charts, complex parameterization
  Kustomize: Patch-based overlays, tidak ada template engine
             Cocok untuk: simple env differences, sudah built into kubectl

Kapan pakai Kustomize:
  - Tim kecil yang tidak butuh Helm complexity
  - Sudah punya YAML yang ingin di-customize per environment
  - Kubernetes manifests yang relatif simpel

Kapan pakai Helm:
  - Chart yang akan di-share/distribute
  - Banyak kondisional dan logic
  - Perlu lifecycle hooks (pre/post install)
```

```yaml
# Struktur Kustomize:
# k8s/
# ├── base/                    ← template dasar
# │   ├── kustomization.yaml
# │   ├── deployment.yaml
# │   └── service.yaml
# └── overlays/
#     ├── staging/             ← staging overrides
#     │   ├── kustomization.yaml
#     │   └── replica-patch.yaml
#     └── production/          ← production overrides
#         ├── kustomization.yaml
#         └── resources-patch.yaml

# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
commonLabels:
  app: auth-service
  managed-by: kustomize

# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
namespace: production
patches:
  - path: resources-patch.yaml
images:
  - name: auth-service
    newTag: abc1234  # ← update image tag tanpa edit base!

# overlays/production/resources-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: auth-service
          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi
```

```bash
# Build dan apply
kubectl apply -k k8s/overlays/production/

# Preview (dry-run)
kubectl kustomize k8s/overlays/production/

# Update image tag di semua overlays:
cd k8s/overlays/production
kustomize edit set image auth-service=ghcr.io/org/auth-service:newsha
```


## 🎯 Review & Checkpoint Fase 11

### Konseptual
- [ ] Apa itu DORA Metrics dan bagaimana menggunakannya untuk improve DevOps?
- [ ] Perbedaan CI, CD (Continuous Delivery), dan CD (Continuous Deployment)?
- [ ] Kapan pakai Blue-Green vs Canary deployment?
- [ ] Mengapa database migration harus multi-phase di production?
- [ ] Apa itu GitOps dan keuntungannya dibanding push-based deployment?
- [ ] Perbedaan RTO dan RPO, dan implikasinya terhadap arsitektur?
- [ ] Mengapa tidak boleh hardcode secrets di code atau commit ke Git?

### Praktis
- [ ] GitHub Actions CI/CD pipeline green dengan matrix build
- [ ] Docker image di-push ke GHCR dengan proper tagging
- [ ] Kubernetes production setup dengan RBAC + NetworkPolicy
- [ ] Helm chart tersedia untuk semua service
- [ ] ArgoCD sync dari Git ke Kubernetes
- [ ] Zero-downtime deployment (blue-green atau canary)
- [ ] Database migration berjalan otomatis sebagai K8s Job
- [ ] Secrets disimpan di Vault atau K8s Secrets (tidak di Git)
- [ ] Terraform untuk at least satu cloud resource
- [ ] Backup automation berjalan
- [ ] DR runbook dan drill selesai
- [ ] **Menyelesaikan project Production Deployment**

---

## 🎯 Project Akhir Fase 11

Kerjakan project berdasarkan PRD di: **`FASE-11-PRD-Production-Deployment.md`**

---

## 🏆 Selamat! Kamu Sudah Menyelesaikan Seluruh Kurikulum!

```
✅ Fase 1  — Go Fundamentals           → Bisa tulis kode Go yang idiomatic
✅ Fase 2  — Go Intermediate            → Goroutines, channels, generics
✅ Fase 3  — Gin + REST API             → REST API production-ready
✅ Fase 4  — Clean Architecture         → SOLID, layered architecture
✅ Fase 5  — gRPC + Protobuf            → Service-to-service communication
✅ Fase 6  — Domain-Driven Design       → Aggregate, Value Object, Domain Events
✅ Fase 7  — Microservices              → Docker, K8s, API Gateway, Circuit Breaker
✅ Fase 8  — Message Broker             → Kafka, RabbitMQ, Event-Driven Architecture
✅ Fase 9  — Observability              → Zap, Prometheus, Grafana, OpenTelemetry
✅ Fase 10 — Testing Mastery            → Unit, Integration, Contract, Load, Fuzz
✅ Fase 11 — DevOps & Deployment        → CI/CD, GitOps, Helm, Terraform, DR

Kamu sekarang memiliki skill seorang SENIOR GO ENGINEER
yang siap bekerja di perusahaan tech kelas dunia!

Langkah selanjutnya:
  1. Build portfolio dengan project dari setiap fase
  2. Kontribusi ke open source Go project
  3. Bagikan pengetahuan ke komunitas (blog, talk, workshop)
  4. Keep up to date: Go release notes, CNCF landscape, industry best practices
```
