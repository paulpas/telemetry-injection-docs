# Documentation Status & Obsolescence Report

**Last Updated**: 2025-11-02
**Purpose**: Track which documentation is current, obsolete, or superseded

---

## Current & Recommended Documentation (2025)

These documents are accurate and maintained:

### Primary References (START HERE)
1. **INDEX.md** ✅ - Navigation guide, quick summary
2. **ARCHITECTURE_REFACTORED.md** ✅ - Complete architecture reference (NEW)
3. **AI_USAGE_DETAILED.md** ✅ - Detailed AI/LLM usage guide (NEW)
4. **RUNBOOK.md** ✅ - Operations & troubleshooting guide (NEW)
5. **QUICK_REFERENCE.md** ✅ - Fast lookup guide

### Implementation Details
6. **SCRIPT_BASED_ARCHITECTURE.md** ✅ - Script-based injection deep dive
7. **TREE_SITTER_IMPLEMENTATION.md** ✅ - Tree-sitter analyzer details
8. **METADATA_TRACKING.md** ✅ - Metadata & provenance tracking

### Getting Started
9. **QUICKSTART.md** ✅ - Installation & first run
10. **runbooks/OLLAMA_SETUP.md** ✅ - Ollama configuration (superseded by RUNBOOK.md but still useful)

---

## Obsolete or Partially Obsolete Documentation

### ⚠️ ARCHITECTURE.md (Partially Obsolete)

**Status**: Superseded by **ARCHITECTURE_REFACTORED.md**

**Issues**:
- Contains mermaid diagrams that reference old components
- Discusses "Parallel Executor" and "Judge Ensemble" which are not in current pipeline
- Mentions GPU scheduling which is not actively used
- Missing script-based architecture

**Recommendation**: Archive or update to reference new architecture

**What's Still Useful**:
- Some conceptual diagrams
- Historical context

**Action**: Rename to `ARCHITECTURE_LEGACY.md` and add warning banner

---

### ⚠️ Multiple runbooks/ files (Consolidate)

**Current Runbooks**:
- `runbooks/OLLAMA_SETUP.md` - Still useful but redundant with RUNBOOK.md
- `runbooks/CLOUD_METADATA_GUIDE.md` - May be obsolete
- `runbooks/TELEMETRY_V2_REFERENCE.md` - Check relevance
- `runbooks/TELEMETRY_V2_USAGE.md` - Check relevance

**Recommendation**:
- Consolidate into RUNBOOK.md (done)
- Keep Ollama setup as standalone reference
- Archive or remove others if redundant

---

### ⚠️ architecture/ subdirectory (Mixed Status)

**Files**:
- `architecture/FUNCTION_BY_FUNCTION_ARCHITECTURE.md` - May be obsolete
- `architecture/LLM_GUIDANCE.md` - Check if still relevant
- `architecture/SEQUENCE_DIAGRAMS_ADDED.md` - Likely obsolete (diagrams in ARCHITECTURE.md)
- `architecture/IMPLEMENTATION_SUMMARY.md` - Check if current
- `architecture/LANGUAGE_INSTRUCTION_ARCHITECTURE.md` - Check relevance
- `architecture/MIGRATION_TO_NEW_PROMPTS.md` - Historical, may archive
- `architecture/CLI_TOOLING_DOCUMENTATION.md` - May be redundant with RUNBOOK.md
- `architecture/LEARNING_SYSTEM_DOCUMENTATION.md` - Still relevant

**Recommendation**: Review each file, consolidate into primary docs, or archive

---

### ⚠️ Diagrams & Examples (Review Needed)

**Files**:
- `diagrams/ARCHITECTURE_DIAGRAMS.md` - Check if diagrams match current state
- `VISUAL_FLOW_DIAGRAM.md` - May be superseded by ARCHITECTURE_REFACTORED.md
- `MERMAID_VISUALIZER.md` - If about telemetry visualizer, OK; if about architecture, obsolete

**Recommendation**: Update diagrams to match current architecture

---

### ⚠️ Feature Documentation (Check Status)

**Files**:
- `FEATURES_AND_BUSINESS_VALUE.md` - Review for accuracy
- `TELEMETRY_COMPARISON.md` - Review for accuracy
- `SELF_IMPROVING_TEMPLATES.md` - Check implementation status
- `LOG_REPLAY.md` - Check relevance
- `PARALLEL_PROCESSING.md` - May be redundant with ARCHITECTURE_REFACTORED.md

**Recommendation**: Update or consolidate

---

### ✅ Changelog & Historical Docs (Keep)

**Files**:
- `changelog/` - Keep all (historical record)
- `lessons/` - Keep all (used by script generator)
- `LLM_REFUSAL_FIX.md` - Historical, keep
- `UNKNOWN_EVENTS_FIX.md` - Historical, keep

**Status**: Archive-worthy, keep for historical context

---

## Documentation Hierarchy (New Structure)

```
docs/
├── INDEX.md                           ← START HERE
│
├── Core References (Read These First)
│   ├── ARCHITECTURE_REFACTORED.md     ← Complete architecture
│   ├── AI_USAGE_DETAILED.md           ← When AI is used
│   ├── RUNBOOK.md                     ← Operations guide
│   └── QUICK_REFERENCE.md             ← Fast lookup
│
├── Getting Started
│   ├── QUICKSTART.md                  ← Installation
│   └── EXAMPLES.md                    ← Usage examples
│
├── Implementation Details
│   ├── SCRIPT_BASED_ARCHITECTURE.md   ← Caching deep dive
│   ├── TREE_SITTER_IMPLEMENTATION.md  ← Fast analysis
│   ├── METADATA_TRACKING.md           ← Provenance tracking
│   └── PARALLEL_PROCESSING.md         ← Async processing
│
├── Legacy / Archive (For Reference Only)
│   ├── ARCHITECTURE.md                ← Old architecture (superseded)
│   ├── architecture/                  ← Historical implementation docs
│   └── changelog/                     ← Historical changes
│
└── Support Resources
    ├── lessons/                       ← Language-specific patterns (USED BY CODE)
    ├── runbooks/                      ← Specialized runbooks
    └── diagrams/                      ← Visual references
```

---

## Recommended Actions

### Immediate (High Priority)

1. ✅ **Create**: `ARCHITECTURE_REFACTORED.md` - Done
2. ✅ **Create**: `AI_USAGE_DETAILED.md` - Done
3. ✅ **Create**: `RUNBOOK.md` - Done
4. ✅ **Create**: `DOCUMENTATION_STATUS.md` (this file) - Done

### Short-Term (This Week)

5. **Rename**: `ARCHITECTURE.md` → `ARCHITECTURE_LEGACY.md`
6. **Add Banner** to legacy files:
   ```markdown
   # ⚠️ LEGACY DOCUMENTATION

   **This document is superseded by**: `ARCHITECTURE_REFACTORED.md`

   **Last Updated**: (old date)
   **Status**: Historical reference only

   For current documentation, see: [INDEX.md](INDEX.md)

   ---
   ```

7. **Update**: `INDEX.md` to point to new primary docs

### Medium-Term (This Month)

8. **Review**: All `architecture/` subdirectory files
9. **Consolidate**: Redundant runbooks into `RUNBOOK.md`
10. **Update**: Diagrams to match current architecture
11. **Audit**: `FEATURES_AND_BUSINESS_VALUE.md`, `TELEMETRY_COMPARISON.md`

### Long-Term (Ongoing)

12. **Maintain**: Primary docs with each major feature change
13. **Archive**: Historical docs to `docs/archive/` directory
14. **Auto-generate**: Code documentation from inline comments (future)

---

## Documentation Maintenance Schedule

### After Each Feature
- Update relevant sections in primary docs
- Add entries to changelog
- Update `INDEX.md` if structure changes

### Monthly Review
- Check accuracy of runbooks
- Update performance benchmarks
- Review and archive obsolete docs

### Quarterly Audit
- Full documentation review
- Update cost estimates
- Verify all examples still work
- Check for broken links

---

## File-by-File Status Report

| File | Status | Action Needed | Priority |
|------|--------|---------------|----------|
| **INDEX.md** | ✅ Current | None | - |
| **ARCHITECTURE_REFACTORED.md** | ✅ Current (NEW) | None | - |
| **AI_USAGE_DETAILED.md** | ✅ Current (NEW) | None | - |
| **RUNBOOK.md** | ✅ Current (NEW) | None | - |
| **QUICK_REFERENCE.md** | ✅ Current | None | - |
| **ARCHITECTURE.md** | ⚠️ Obsolete | Rename to LEGACY, add banner | High |
| **SCRIPT_BASED_ARCHITECTURE.md** | ✅ Current | None | - |
| **TREE_SITTER_IMPLEMENTATION.md** | ✅ Current | None | - |
| **METADATA_TRACKING.md** | ✅ Current | None | - |
| **QUICKSTART.md** | ✅ Current | Review for accuracy | Low |
| **EXAMPLES.md** | ⚠️ Unknown | Review | Medium |
| **GO_QUICK_REFERENCE.md** | ✅ Current | None (Go-specific) | - |
| **PROVIDER_SWITCHING_GUIDE.md** | ⚠️ Check | May be redundant with RUNBOOK.md | Medium |
| **GPU_DETECTION.md** | ⚠️ Check | May be obsolete (not used?) | Medium |
| **PARALLEL_PROCESSING.md** | ⚠️ Check | May be redundant | Medium |
| **FEATURES_AND_BUSINESS_VALUE.md** | ⚠️ Check | Review accuracy | Medium |
| **TELEMETRY_COMPARISON.md** | ⚠️ Check | Review accuracy | Medium |
| **architecture/*.md** | ⚠️ Mixed | Review each file | Low |
| **runbooks/OLLAMA_SETUP.md** | ✅ Current | Keep (useful standalone) | - |
| **runbooks/CLOUD_METADATA_GUIDE.md** | ⚠️ Check | May archive | Low |
| **runbooks/TELEMETRY_V2_*.md** | ⚠️ Check | Review relevance | Low |
| **changelog/*.md** | ✅ Archive | Keep (historical) | - |
| **lessons/**/*.md** | ✅ Current | Keep (used by code) | - |

---

## How to Use This Report

### For New Contributors
1. Read **INDEX.md**
2. Read **ARCHITECTURE_REFACTORED.md** for complete understanding
3. Use **RUNBOOK.md** for operations
4. Refer to **AI_USAGE_DETAILED.md** for LLM details

### For Maintainers
1. Use this report to identify obsolete docs
2. Follow recommended actions (above)
3. Update this file after each doc change
4. Review quarterly

### For Users
1. Trust docs marked ✅ Current
2. Be cautious with docs marked ⚠️
3. Ignore docs in `archive/` directory

---

## Summary

**New Documentation Created** (2025-11-02):
- ✅ ARCHITECTURE_REFACTORED.md (23KB, comprehensive)
- ✅ AI_USAGE_DETAILED.md (27KB, detailed AI usage)
- ✅ RUNBOOK.md (18KB, operations guide)
- ✅ DOCUMENTATION_STATUS.md (this file)

**Obsolete/Superseded**:
- ⚠️ ARCHITECTURE.md → Rename to LEGACY
- ⚠️ Various architecture/ files → Review/consolidate

**Action Items**:
- 4 files to rename/flag as legacy
- ~10 files to review
- ~5 files to potentially archive

**Result**: Clear documentation hierarchy, accurate references, obsolete docs flagged

---

**Last Updated**: 2025-11-02
**Next Review**: 2025-12-01
**Maintained By**: Documentation Team
