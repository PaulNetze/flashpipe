# Documentation Cleanup Plan

## Executive Summary

Current state: **5,172 lines** across 19 markdown files with significant redundancy
Target state: **~2,500 lines** with consolidated, focused documentation

## Problems Identified

1. **Redundant Example Files**: Separate "config examples" files that duplicate main docs
2. **Multiple GitHub Actions Files**: 5 separate files for different GitHub Actions scenarios
3. **Verbose Documentation**: Some files have 700+ lines with too many examples
4. **Unclear Structure**: Mix of reference docs, tutorials, and examples
5. **Development Artifacts**: Files like DOCUMENTATION_CLEANUP.md in production docs

## Files Analysis

### Keep As-Is (Core Reference)
- `index.md` (28 lines) - Documentation index ✅
- `oauth_client.md` (94 lines) - Authentication setup ✅
- `flashpipe-cli.md` (606 lines) - Complete CLI reference ✅
- `release-notes.md` (342 lines) - Version history ✅
- `_config.yml` - Jekyll config ✅

### Consolidate & Reduce

#### 1. Orchestrator Documentation (1,492 lines → ~600 lines)
**Current:**
- `orchestrator.md` (735 lines) - Too verbose
- `orchestrator-quickstart.md` (226 lines) - Good, keep this
- `orchestrator-migration.md` (531 lines) - Niche use case

**Action:**
- ✂️ Reduce `orchestrator.md` to ~400 lines (remove redundant examples)
- ✅ Keep `orchestrator-quickstart.md` as-is
- 🗑️ Delete `orchestrator-migration.md` (move key info to orchestrator.md)

#### 2. Partner Directory (1,098 lines → ~450 lines)
**Current:**
- `partner-directory.md` (715 lines) - Too verbose
- `partner-directory-config-examples.md` (383 lines) - Redundant

**Action:**
- ✂️ Consolidate both into single `partner-directory.md` (~450 lines)
- 🗑️ Delete `partner-directory-config-examples.md`

#### 3. GitHub Actions (560 lines → ~300 lines)
**Current:**
- `github-actions-upload.md` (62 lines)
- `github-actions-snapshot.md` (90 lines)
- `github-actions-sync-to-git.md` (96 lines)
- `github-actions-sync-to-tenant.md` (40 lines)
- `github-actions-sync-apim.md` (98 lines)
- `azure-pipelines-upload.md` (174 lines)
- `documentation.md` (65 lines) - Main integration doc

**Action:**
- ✂️ Consolidate into 2 files:
  - `github-actions.md` (~250 lines) - All GitHub Actions scenarios
  - `azure-pipelines.md` (~150 lines) - Keep Azure separate
- 🗑️ Delete individual scenario files
- 🗑️ Delete vague `documentation.md`

#### 4. Other Commands
- `configure.md` (417 lines) ✅ Already cleaned, keep
- `config-generate.md` (342 lines) ✂️ Reduce to ~200 lines

### Delete
- `DOCUMENTATION_CLEANUP.md` (128 lines) - Development artifact 🗑️

## New Documentation Structure

```
docs/
├── index.md                          # Documentation home (updated)
│
├── Getting Started/
│   ├── oauth_client.md              # Auth setup (keep)
│   └── quickstart.md                # NEW: Overall quickstart (100 lines)
│
├── Commands/
│   ├── orchestrator.md              # Consolidated (400 lines)
│   ├── orchestrator-quickstart.md   # Keep (226 lines)
│   ├── configure.md                 # Keep (417 lines)
│   ├── config-generate.md           # Reduced (200 lines)
│   └── partner-directory.md         # Consolidated (450 lines)
│
├── CI/CD Integration/
│   ├── github-actions.md            # Consolidated (250 lines)
│   └── azure-pipelines.md           # Reduced (150 lines)
│
├── Reference/
│   ├── flashpipe-cli.md             # Keep (606 lines)
│   └── release-notes.md             # Keep (342 lines)
│
└── examples/                         # Keep example YAMLs
    ├── flashpipe-config-with-orchestrator.yml
    ├── flashpipe-cpars-example.yml
    └── orchestrator-config-example.yml
```

## Detailed Actions

### Phase 1: Delete Redundant Files (Immediate)
```bash
# Delete development artifacts
rm docs/DOCUMENTATION_CLEANUP.md

# Delete redundant example files
rm docs/partner-directory-config-examples.md

# Delete migration guide (merge critical info first)
rm docs/orchestrator-migration.md

# Delete individual GitHub Actions files
rm docs/github-actions-upload.md
rm docs/github-actions-snapshot.md
rm docs/github-actions-sync-to-git.md
rm docs/github-actions-sync-to-tenant.md
rm docs/github-actions-sync-apim.md
rm docs/documentation.md
```

### Phase 2: Consolidate Files

#### A. Partner Directory
**New `partner-directory.md` structure:**
1. Overview (50 lines)
2. Commands Reference (150 lines)
   - pd-snapshot
   - pd-deploy
3. Configuration (100 lines)
   - File format
   - All options
4. Examples (100 lines)
   - 3 focused examples only
5. Troubleshooting (50 lines)

**Total: ~450 lines** (down from 1,098)

#### B. GitHub Actions
**New `github-actions.md` structure:**
1. Overview (30 lines)
2. Setup (50 lines)
3. Workflows (150 lines)
   - Upload/Deploy
   - Snapshot
   - Sync to Git
   - Sync to Tenant
   - APIM Sync
4. Examples (20 lines per scenario = 100 lines)

**Total: ~330 lines** (down from 486)

#### C. Orchestrator
**Streamline `orchestrator.md`:**
1. Overview (40 lines)
2. Quick Start (60 lines)
3. Configuration (120 lines)
4. Operation Modes (80 lines)
5. Examples (100 lines) - 5 focused examples only
6. Troubleshooting (50 lines)

**Total: ~450 lines** (down from 735)

#### D. Config Generate
**Streamline `config-generate.md`:**
1. Overview (30 lines)
2. Usage (60 lines)
3. Options (60 lines)
4. Examples (50 lines) - 3 examples only

**Total: ~200 lines** (down from 342)

### Phase 3: Create New Files

#### quickstart.md (NEW - 100 lines)
**Purpose:** Get users started in 5 minutes
**Content:**
1. Install FlashPipe (20 lines)
2. Setup Authentication (30 lines)
3. First Deployment (30 lines)
4. Next Steps (20 lines)

### Phase 4: Update Index

Update `index.md` to reflect new structure with clear sections:
- Getting Started
- Commands
- CI/CD Integration
- Reference

## Guidelines for Consolidation

### DO:
✅ Keep essential configuration options
✅ Show 2-3 focused examples per feature
✅ Include troubleshooting sections
✅ Maintain command reference tables
✅ Keep quick starts separate from detailed docs

### DON'T:
❌ Show 10+ variations of same example
❌ Repeat authentication setup in every file
❌ Include development/implementation notes
❌ Maintain separate files for minor variations
❌ Keep migration guides for old versions

## Success Metrics

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Total Lines | 5,172 | ~2,800 | -46% |
| Total Files | 19 | 12 | -37% |
| Avg File Size | 272 lines | 233 lines | -14% |
| Redundant Content | High | Low | ✅ |
| User Navigation | Confusing | Clear | ✅ |

## Implementation Order

1. ✅ Configure docs (already done)
2. 🔲 Delete redundant files
3. 🔲 Consolidate Partner Directory
4. 🔲 Consolidate GitHub Actions
5. 🔲 Streamline Orchestrator
6. 🔲 Streamline Config Generate
7. 🔲 Create quickstart.md
8. 🔲 Update index.md
9. 🔲 Update README.md links
10. 🔲 Test all documentation links

## Timeline

- **Phase 1 (Delete):** 15 minutes
- **Phase 2 (Consolidate):** 2 hours
- **Phase 3 (Create New):** 30 minutes
- **Phase 4 (Update Links):** 30 minutes
- **Testing:** 30 minutes

**Total Estimated Time:** 4 hours

## Rollback Plan

Before starting:
```bash
git checkout -b docs-cleanup
git add docs/
git commit -m "Backup: Pre-cleanup documentation state"
```

If issues arise:
```bash
git checkout main
git branch -D docs-cleanup
```

## Post-Cleanup Maintenance

1. **One Command = One File** rule
2. **Max 500 lines per doc** (except CLI reference)
3. **Max 3 examples per feature**
4. **No separate "examples" documentation files**
5. **Quarterly review** for redundancy

## Notes

- Keep all example YAML files in `docs/examples/`
- Keep all images in `docs/images/`
- Archive old docs in separate branch if needed
- Update any external links (blog posts, videos) to new structure