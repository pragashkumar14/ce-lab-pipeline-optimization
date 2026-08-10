# Lab M5.09 - Pipeline Optimization

## Pipeline Performance Comparison

| Metric | Baseline (Slow) | Optimized |
|--------|-----------------|-----------|
| Total Duration | 0 min 19 sec | 0 min 46 sec (first run, cold cache) |
| `terraform init` | ~5 sec | ~5 sec (cache populated on this run; faster on subsequent runs) |
| Job Structure | 1 sequential job | 3 parallel jobs (lint + validate in parallel, plan waits on both) |
| Path Filtering | None (runs on all changes) | `terraform/**` only |
| Version Testing | Single version (1.6.0) | Matrix (1.6.0, 1.7.0, 1.8.0) |

**Note:** The Optimized Pipeline's first run took longer in wall-clock time than the Baseline because each parallel job spins up its own GitHub Actions runner (setup overhead), and the dependency cache was empty on this first run. The real benefit of caching shows up on subsequent runs, where `terraform init` skips provider downloads entirely on a cache hit. The structural benefit (parallel lint/validate instead of sequential) and the path filter (skipping runs for non-Terraform changes) provide consistent savings regardless of cache state.

## Matrix Testing Results

| Terraform Version | Result | Notes |
|--------------------|--------|-------|
| 1.6.0 | ❌ Failed | `openpgp: key expired` — HashiCorp provider signing key issue affecting older pinned CLI versions |
| 1.7.0 | ✅ Passed | |
| 1.8.0 | ✅ Passed | |

This result demonstrates the risk of the "single Terraform version only" anti-pattern from the baseline: pinning CI to one fixed, aging version leaves the pipeline exposed to upstream issues (like an expired signing key) that newer versions have already resolved.

## Optimizations Applied

1. **Dependency Caching** — `actions/cache@v4` stores `.terraform/` providers
2. **Job Parallelization** — lint and validate run simultaneously
3. **Path Filters** — skip pipeline for non-Terraform changes
4. **Matrix Testing** — validate across multiple Terraform versions

## Repository Structure
```
├── .github/workflows/
│   ├── baseline-slow.yml.disabled
│   ├── optimized.yml
│   └── matrix-test.yml
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── .gitignore
└── README.md
```
