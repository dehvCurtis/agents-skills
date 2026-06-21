# Plan: Restrict PDF/Report Exports to Paying Tiers

**Date:** February 1, 2026
**Task:** Ensure only paying tiers (team, growth, enterprise) can export PDF reports
**Status:** Ready for Implementation

---

## Summary

Restrict PDF report export functionality to paying tiers only. The free `developer` tier should see an upgrade prompt instead of export functionality.

---

## Current State Analysis

### Source of Truth Conflict

There is currently a **conflict** between the source of truth and the documentation/feature tests:

| Source | Developer Tier Export Setting |
|--------|-------------------------------|
| `blocksecops-shared/tier-config/tiers.json` | `exportEnabled: true` ❌ (wrong) |
| `docs/standards/tier-standards.md` (Feature table) | "Yes" for all tiers ❌ (wrong) |
| `docs/feature-tests/02-quota-system.md` (Section 10) | Export Disabled for Developer ✅ (correct) |
| Migration 030 (database trigger) | Uses old 5-tier model, free=false ⚠️ (outdated) |

### Current Tier Model (4-Tier)

| Tier | Hierarchy | Pricing | Should Have Export? |
|------|-----------|---------|---------------------|
| developer | 0 | $0/month | **No** (free tier) |
| team | 1 | $299/month | Yes |
| growth | 2 | $699/month | Yes |
| enterprise | 3 | $1,999+/month | Yes |

### Backend Check Already Exists

The backend endpoint at `blocksecops-api-service/src/presentation/api/v1/endpoints/scans.py:1800-1809` already checks `user_quota.export_enabled` and returns 403 if false. The issue is:
1. The `tiers.json` source of truth says developer has export enabled
2. The database trigger/migration uses outdated tier names
3. The frontend has no UI gating

---

## Implementation Steps

### Step 1: Update Source of Truth (tiers.json)

**File:** `blocksecops-shared/tier-config/tiers.json`

Change developer tier export setting:
```json
"developer": {
  "features": {
    "exportEnabled": false  // Change from true to false
  }
}
```

### Step 2: Update Tier Standards Documentation

**File:** `docs/standards/tier-standards.md`

Update the Feature Availability table (~line 196):
```markdown
| Feature | Developer | Team | Growth | Enterprise |
|---------|-----------|------|--------|------------|
| Export Reports | **No** | Yes | Yes | Yes |
```

Update the database values section (~line 482):
```sql
-- Developer tier (Free)
export_enabled = false,  -- Change from true
```

### Step 3: Create Database Backup

**File:** `docs/database/backups/solidity_security_pre_export_restriction_YYYYMMDD_HHMMSS.sql`

```bash
# Create backup before migration
kubectl exec -n postgresql-local postgresql-0 -- pg_dump -U postgres -d solidity_security \
  -f /tmp/solidity_security_pre_export_restriction_$(date +%Y%m%d_%H%M%S).sql

# Copy backup to docs/database/backups/
kubectl cp postgresql-local/postgresql-0:/tmp/solidity_security_pre_export_restriction_*.sql \
  /home/pwner/Git/docs/database/backups/
```

### Step 4: Create Database Migration

**File:** `blocksecops-api-service/alembic/versions/20260201_HHMM-061_restrict_developer_export.py`

```python
"""restrict_developer_export

Revision ID: 061_restrict_developer_export
Revises: 060_cleanup_invalid_scanner_ids
Create Date: 2026-02-01

Restrict export feature to paying tiers only.
Developer tier (free) users will no longer have export access.

Source of Truth: blocksecops-shared/tier-config/tiers.json
"""
from alembic import op

revision = '061_restrict_developer_export'
down_revision = '060_cleanup_invalid_scanner_ids'
branch_labels = None
depends_on = None

def upgrade() -> None:
    # Update existing developer tier users to have export disabled
    op.execute("""
    UPDATE user_quotas
    SET export_enabled = false
    WHERE tier = 'developer';
    """)

    # Update trigger function for new user creation
    # (Update the CASE statement for export_enabled)
    op.execute("""
    -- Update create_user_quota trigger to set export_enabled=false for developer tier
    -- This is handled by updating the trigger function
    """)

def downgrade() -> None:
    # Re-enable export for developer tier
    op.execute("""
    UPDATE user_quotas
    SET export_enabled = true
    WHERE tier = 'developer';
    """)
```

**Note:** Also need to update the `create_user_quota()` trigger function to reflect the new export policy for developer tier.

### Step 5: Update Frontend - ScanResults Page

**File:** `blocksecops-dashboard/src/pages/ScanResults.tsx`

Add TierGate import and wrap export dropdown:
```tsx
import { TierGate } from '@/components/common/TierGate';

{/* Export Dropdown - lines ~537-557 */}
<TierGate requiredTier="team" featureName="Report Export" mode="preview">
  <div className="relative inline-block">
    <select ...>
      <option value="pdf">Export PDF</option>
      ...
    </select>
  </div>
</TierGate>
```

### Step 6: Update Frontend - DashboardAnalytics Page

**File:** `blocksecops-dashboard/src/pages/DashboardAnalytics.tsx`

Wrap export buttons with TierGate:
```tsx
import { TierGate } from '@/components/common/TierGate';

{/* Export Buttons - lines ~274-336 */}
<TierGate requiredTier="team" featureName="Analytics Export" mode="preview">
  <div className="flex items-center gap-2">
    <button onClick={handleExportPDF}>Export PDF</button>
    <button onClick={handleExportJSON}>Export JSON</button>
    <button onClick={handleExportCSV}>Export CSV</button>
  </div>
</TierGate>
```

### Step 7: Update Backend Export Endpoint

**File:** `blocksecops-api-service/src/presentation/api/v1/endpoints/scans.py`

Add `require_tier("team")` dependency for defense-in-depth:
```python
from src.infrastructure.auth.middleware import require_tier

@router.get(
    "/{scan_id}/export",
    summary="Export scan report",
    dependencies=[Depends(require_tier("team"))],  # ADD THIS
)
async def export_scan_report(...):
```

Update error message (~line 1805):
```python
"message": "Export feature is not available on your current tier. Upgrade to Team or higher to export reports."
```

---

## Documentation Updates Required

### General Documentation (`docs/*`)

| File | Update Required |
|------|-----------------|
| `docs/standards/tier-standards.md` | Update Feature Availability table, Database Values section |
| `docs/database/MIGRATIONS.md` | Add Migration 061 entry |
| `docs/database/backups/` | Add pre-migration backup file |

### Task Documentation (`TaskDocs-BlockSecOps/*`)

| File | Update Required |
|------|-----------------|
| `DOCUMENTATION-UPDATE-2026-02-01-EXPORT-RESTRICTION.md` | Create new documentation update file |

### Feature Tests (`docs/feature-tests/*`)

| File | Update Required |
|------|-----------------|
| `docs/feature-tests/02-quota-system.md` | Section 10 already correct, verify test cases |

### Playbooks (`docs/playbooks/*`)

| File | Update Required |
|------|-----------------|
| `docs/playbooks/adjust-pricing.md` | Add note about export feature tier gating |

### Website Support Docs (`blocksecops-docs/*`)

| File | Update Required |
|------|-----------------|
| `blocksecops-docs/features/export.md` | Create or update export feature documentation |
| `blocksecops-docs/pricing/tier-comparison.md` | Update feature matrix if exists |

---

## Files to Modify (Summary)

| Action | File | Description |
|--------|------|-------------|
| **EDIT** | `blocksecops-shared/tier-config/tiers.json` | Set developer.features.exportEnabled = false |
| **EDIT** | `docs/standards/tier-standards.md` | Update Feature Availability table |
| **CREATE** | `docs/database/backups/solidity_security_pre_export_restriction_*.sql` | Pre-migration backup |
| **CREATE** | `blocksecops-api-service/alembic/versions/...-061_restrict_developer_export.py` | Database migration |
| **EDIT** | `blocksecops-api-service/src/presentation/api/v1/endpoints/scans.py` | Add require_tier, fix message |
| **EDIT** | `blocksecops-dashboard/src/pages/ScanResults.tsx` | Add TierGate to export dropdown |
| **EDIT** | `blocksecops-dashboard/src/pages/DashboardAnalytics.tsx` | Add TierGate to export buttons |
| **EDIT** | `docs/database/MIGRATIONS.md` | Document migration 061 |
| **CREATE** | `TaskDocs-BlockSecOps/DOCUMENTATION-UPDATE-2026-02-01-EXPORT-RESTRICTION.md` | Implementation summary |

---

## Verification Checklist

### Backend Verification
```bash
# As free tier user (developer) - should return 403
curl -H "Authorization: Bearer $FREE_TIER_TOKEN" \
  http://127.0.0.1:8000/api/v1/scans/{scan_id}/export?format=pdf
# Expected: 403 Forbidden with upgrade message

# As team tier user - should return PDF
curl -H "Authorization: Bearer $TEAM_TIER_TOKEN" \
  http://127.0.0.1:8000/api/v1/scans/{scan_id}/export?format=pdf
# Expected: 200 OK with PDF content
```

### Frontend Verification
1. **Free tier user (developer):**
   - Visit Scan Results page - export buttons should show grayed out with upgrade overlay
   - Visit Analytics Dashboard - export buttons should show grayed out with upgrade overlay
   - Clicking upgrade badge should navigate to /pricing

2. **Paying tier user (team/growth/enterprise):**
   - All export functionality works as before
   - PDF downloads successfully
   - No upgrade prompts shown

### Database Verification
```sql
-- Verify developer tier has export disabled
SELECT tier, export_enabled FROM user_quotas WHERE tier = 'developer';
-- Expected: all rows show export_enabled = false

-- Verify paying tiers have export enabled
SELECT tier, export_enabled FROM user_quotas WHERE tier IN ('team', 'growth', 'enterprise');
-- Expected: all rows show export_enabled = true
```

---

## Rollback Plan

If issues arise:

1. **Revert tiers.json:**
   ```bash
   git checkout HEAD~1 -- blocksecops-shared/tier-config/tiers.json
   ```

2. **Run database downgrade:**
   ```bash
   kubectl exec -n api-service-local deployment/api-service -- alembic downgrade 060_cleanup_invalid_scanner_ids
   ```

3. **Revert frontend changes:**
   ```bash
   git checkout HEAD~1 -- blocksecops-dashboard/src/pages/ScanResults.tsx
   git checkout HEAD~1 -- blocksecops-dashboard/src/pages/DashboardAnalytics.tsx
   ```

4. **Restore from backup if needed:**
   ```bash
   kubectl exec -n postgresql-local postgresql-0 -- psql -U postgres -d solidity_security \
     -f /tmp/solidity_security_pre_export_restriction_*.sql
   ```

---

## Benefits

1. **Revenue Protection:** Premium feature encourages upgrades to paid tiers
2. **Consistent UX:** Uses existing TierGate pattern with upgrade prompts
3. **Defense in Depth:** Both frontend and backend enforce restrictions
4. **Clear Upgrade Path:** Users see what they're missing and how to get it
5. **Source of Truth Alignment:** All sources now consistent

---

## Dependencies

- Migration 060 must be applied first (current head)
- TierGate component already exists and works correctly
- require_tier middleware already exists in auth module
