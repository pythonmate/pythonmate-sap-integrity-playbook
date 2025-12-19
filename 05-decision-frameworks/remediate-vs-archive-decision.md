# The Delete vs. Archive vs. Migrate Decision Framework: Pre-Migration Strategic Assessment

## Pre-Migration Decision Context: Three Destinies for SAP Content

Before committing to S/4HANA migration, organizations must decide the fate of accumulated SAP content. This decision directly impacts HANA licensing costs, migration complexity, and post-migration operational requirements. Not all content deserves the same treatment - this framework establishes decision logic for each attachment category.

Every attachment in SOFFCONT1 must be assigned one of three outcomes before migration:

**1. MIGRATE** (Keep in S/4HANA post-migration)
- Operational necessity: Active business process requires access
- Regulatory mandate: Legal hold or retention period not expired
- Cost justified: Value of retention > cost of HANA storage

**2. ARCHIVE** (Move to external storage before migration)
- Historical value: May be needed for audit, but not daily operations
- Compliance required: Retention period not expired, but infrequent access acceptable
- Cost optimized: Cheap storage (object storage, tape) vs. HANA RAM

**3. DELETE** (Permanent removal before migration)
- No legal hold: Retention period expired
- No business value: Obsolete process, decommissioned systems
- Risk accepted: Legal/Compliance approved permanent deletion

---

### Decision Matrix by Document Type - Pre-Migration Planning

| Document Type | Example | Pre-Migration Default Destiny | Retention Driver | Typical Retention |
|---------------|---------|------------------------------|------------------|-------------------|
| **Financial Records** | Invoice PDFs | Archive (after 7 years) | Tax law (IRS, HMRC, etc.) | 7-10 years |
| **Contracts** | Signed agreements | Migrate (if active), Archive (if expired) | Contract term + 7 years | Duration + 7 years |
| **Quality Records** | CoA, test reports | Archive (after product lifecycle) | ISO 9001, FDA | 10-15 years post-product |
| **Email Correspondence** | User-attached emails | Delete (after 3 years) | Business policy | 3-5 years |
| **Technical Drawings** | CAD exports, schematics | Migrate (if current), Archive (if superseded) | Product lifecycle | Life of product + 5 years |
| **HR Documents** | Performance reviews | Delete (after separation + 7 years) | Employment law | 7 years post-termination |
| **Temporary Work Docs** | Draft documents, notes | Delete immediately | None | 0 years (should not exist) |
| **Photos/Images** | Equipment photos, incident documentation | Archive (after case closed) | Safety/liability | 10 years |
| **Scanned Correspondence** | Vendor letters, customer complaints | Archive (after 7 years) | Business records law | 7 years |

---

### The Legal Hold Exception - Migration Blocking Condition

**If ANY of these conditions apply, content CANNOT be deleted before or during migration**:

- ✋ **Active Litigation**: Case filed, discovery pending
- ✋ **Regulatory Investigation**: Government inquiry open
- ✋ **Audit Period**: External audit in progress
- ✋ **Contractual Obligation**: Customer/supplier requires retention
- ✋ **Insurance Claim**: Open claim referencing documents

**Legal holds are document-specific**, not system-wide. Use SRGBTBREL links to identify affected attachments before migration.

---

### Pre-Migration Cost-Driven Decision Logic

For documents where retention is **discretionary** (no legal mandate) before migration:

**Calculate Pre-Migration Cost Per Document**:
```
Annual HANA Cost = (Document Size in GB) × $3.50/GB/month × 12 months

Example:
5 MB invoice PDF = 0.005 GB × $3.50 × 12 = $0.21/year

If 100,000 such PDFs exist:
  HANA Cost = $21,000/year
  Archive Cost (S3) = $1,200/year
  Savings = $19,800/year
```

**Pre-Migration Decision Rule**:
- If (HANA Cost - Archive Cost) > Archive Migration Cost / 3 years → Archive
- Else → Migrate (not worth effort)

**Pre-Migration Threshold**: Typically, archival justified when saving > $10K/year.

---

### Pre-Migration Risk-Adjusted Decision for Grey Areas

Some documents fall into regulatory grey areas before migration:

**Example**: Email attachments from 8 years ago, no clear retention requirement.

**Pre-Migration Risk Matrix**:

| Risk Level | Description | Decision |
|------------|-------------|----------|
| **High** | Financial, contract, safety-critical | Archive (defensible retention) |
| **Medium** | Operational correspondence, vendor communications | Archive if volume > 10 GB, else Migrate |
| **Low** | Internal notes, drafts, duplicate copies | Delete (with approval) |

**Pre-Migration Approval Chain**:
- High Risk Deletion: General Counsel
- Medium Risk Deletion: Compliance Officer
- Low Risk Deletion: IT Director

---

### The "90-Day Safe Harbor" Rule - Pre-Migration Assessment

For organizations without formal retention policies before migration:

**Assume**:
- All financial documents: 7 years (safe harbor for most jurisdictions)
- All contracts: Term + 7 years
- All other: 3 years minimum

**Then apply pre-migration**:
```
IF document_age > safe_harbor_period THEN
  IF legal_hold = FALSE THEN
    eligible_for_archive_or_delete = TRUE
  END IF
END IF
```

This approach minimizes legal risk while maximizing pre-migration cleanup opportunity.

---

### Pre-Migration Special Case: Duplicate Content

SOFFCONT1 often contains identical files attached to multiple business objects before migration.

**Example**:
- Same invoice PDF attached to:
  - Purchase Order (ME23N)
  - Goods Receipt (MIGO)
  - Invoice Document (FB03)

**Pre-Migration Decision**:
- **Keep ONE copy** (linked to primary object, typically Invoice Document)
- **Delete duplicates** (after verifying identical content via checksum)
- **Maintain SRGBTBREL link** (redirect secondary links to primary content)

**Pre-Migration Savings**: Duplicate rates of 30-40% are common → Immediate 30-40% SOFFCONT1 reduction without legal risk.

---

### Pre-Migration Validation Checkpoint Before Deletion

**Before permanently deleting ANY content before migration**:

1. ✅ **Legal/Compliance Sign-Off**: Document retention policy applied correctly
2. ✅ **Checksum Backup**: SHA-256 hash recorded (proof of prior existence)
3. ✅ **Metadata Preservation**: SRGBTBREL entry updated (deletion date, approver)
4. ✅ **Audit Trail**: Deletion logged in change management system
5. ✅ **Test Restoration**: If archived, verify retrieval from archive works

**If all 5 boxes not checked before migration → DO NOT DELETE**.

---

**Pre-Migration Guiding Principle**: When in doubt, archive. Archival is reversible; deletion is not. The cost of over-retention (few thousand dollars) is less than the cost of under-retention (regulatory penalty, lost litigation).

---

# Remediate vs Archive Decision Framework - Pre-Migration Strategic Assessment

## Executive Summary: Pre-Migration Content Risk Strategy

Organizations facing SAP content risk must choose between two strategic approaches before making irreversible S/4HANA migration decisions: remediation (active cleanup of SOFFCONT1) or archival (migration to external content management). This document provides a decision framework to evaluate which approach aligns with organizational capabilities, risk tolerance, and business objectives before migration commitments are made.

The timing of this decision is critical - it should occur during pre-migration planning when there is still flexibility to adjust timelines and budgets based on the chosen approach. The decision directly impacts HANA licensing costs, migration complexity, and post-migration operational requirements.

---

## Pre-Migration Decision Matrix Overview

The remediate vs. archive decision depends on multiple factors that should be evaluated before irreversible migration commitments:

| Factor | Remediate (Clean Up Before Migration) | Archive (Migrate Out Before Migration) |
|--------|--------------------------------------|----------------------------------------|
| **Risk Profile** | Immediate risk reduction pre-migration | Risk transfer to content server pre-migration |
| **Cost Structure** | Upfront project cost during planning | Ongoing operational cost post-migration |
| **Technical Complexity** | High (database manipulation pre-migration) | Medium (integration configuration pre-migration) |
| **Business Impact** | Minimal ongoing impact post-migration | Process change management pre-migration |
| **Compliance Readiness** | Improves data governance pre-migration | Depends on archive solution implemented pre-migration |
| **Migration Timing** | Allows optimized migration post-remediation | May extend migration timeline pre-migration |

---

## Remediation: Active SOFFCONT1 Cleanup Strategy - Pre-Migration Focus

### Definition
Direct deletion of orphaned and unneeded content from SOFFCONT1, SOFFPHIO, and related tables before S/4HANA migration.

### When to Choose Pre-Migration Remediation

#### High-Confidence Pre-Migration Scenarios
1. **Orphan Rate > 30%**: Clear majority of content has no active business object links - significant cost savings opportunity before HANA licensing commitment
2. **Historical Content**: >5 years old documents with no ongoing business relevance - low business risk for deletion pre-migration
3. **Large File Concentration**: SOFFCONT1 contains many >100MB files that are orphaned - substantial HANA memory cost reduction opportunity
4. **Budget Constraints**: Limited ongoing operational budget but available project funds during planning phase
5. **Regulatory Compliance**: Need to demonstrate active data governance and minimization before migration audit scrutiny

#### Pre-Migration Organizational Readiness Requirements
- **Skilled SAP Basis Team**: Experience with cluster table maintenance and pre-migration assessment
- **Risk Management Capability**: Ability to plan and execute changes with appropriate backup and rollback procedures
- **Business Stakeholder Support**: Clear ownership for content validation decisions before migration timeline pressure
- **Change Management Maturity**: Established protocols for technical changes during planning phase

### Pre-Migration Remediation Advantages
- **Immediate HANA Memory Reduction**: Direct impact on licensing costs before procurement decisions
- **Simplified Architecture**: Single system, no external dependencies for post-migration operations
- **Migration Performance Improvement**: Smaller database results in shorter migration downtime windows
- **Compliance Benefits**: Demonstrate data minimization principle pre-migration
- **Reduced Migration Complexity**: Lower risk of SYSTEM_NO_ROLL and other migration failures
- **Cost Predictability**: Eliminate ongoing costs for orphaned content in HANA environment

### Pre-Migration Remediation Disadvantages
- **Timeline Extension Risk**: Remediation project may extend overall migration timeline before commitment
- **Technical Risk**: Complex database operations with potential side effects during planning phase
- **Extensive Testing Requirements**: Must validate all business processes before migration
- **Limited Automation**: Manual validation required for many content types before deletion
- **Business Disruption Risk**: Potential impact during cleanup execution before go-live
- **Irreversible Operations**: Deleted content cannot be recovered easily after migration

---

## Archival: External Content Server Migration Strategy - Pre-Migration Planning

### Definition
Migration of content from SOFFCONT1 to external content management systems (ArchiveLink, HTTP Content Server, Cloud Storage) before or during S/4HANA migration.

### When to Choose Pre-Migration Archival

#### High-Confidence Pre-Migration Scenarios
1. **Low Orphan Rate (< 15%)**: Most content has active business value - archival preserves access while reducing HANA load
2. **Compliance Requirements**: Need long-term retention with audit trail that extends beyond migration
3. **Growing Content Volume**: Ongoing content accumulation requires scalable solution before HANA commitment
4. **Existing Content Management**: Organization has established ECM strategy and infrastructure
5. **Budget Profile**: Preference for operational expense over capital expense for content management

#### Pre-Migration Organizational Readiness Requirements
- **ECM Strategy**: Document management governance in place before migration
- **IT Infrastructure**: Capability to support external content servers during migration planning
- **User Training Capability**: Can manage process change for content access before go-live
- **Integration Expertise**: Experience with ArchiveLink or similar integrations during planning phase

### Pre-Migration Archival Advantages
- **Risk Distribution**: Content stored on less expensive media before HANA licensing commitment
- **Scalability**: External systems scale independently of SAP for future growth
- **Compliance Features**: Advanced retention and legal hold capabilities maintained through migration
- **SAP Performance**: Reduced database size without content loss
- **Business Continuity**: Content remains accessible through same UI post-migration
- **Migration Flexibility**: Content volume reduction achieved without deletion risk

### Pre-Migration Archival Disadvantages
- **Timeline Complexity**: May extend migration timeline for content server setup and migration
- **Integration Complexity**: Additional system dependencies introduced before migration
- **Performance Impact**: Potential latency for content access post-migration
- **Change Management**: Users may experience different access patterns before go-live
- **Backup Complexity**: Multiple systems require coordinated backup strategy for migration
- **Ongoing Costs**: Content server licensing and maintenance post-migration

---

## Pre-Migration Hybrid Approach: Phased Strategy

### Scenario 1: Remediate Then Archive (Recommended for High-Risk Content)
1. **Phase 1**: Remediation of clearly orphaned content before migration
2. **Phase 2**: Archival of remaining active content before or during migration
3. **Result**: Maximum risk reduction with long-term content management sustainability

### Scenario 2: Archive Then Remediate (Conservative Approach)
1. **Phase 1**: Archival of all content to external system before migration
2. **Phase 2**: Remediation of orphaned content in external system post-migration
3. **Result**: Content preservation safety net with eventual cleanup opportunity

### Scenario 3: Selective Remediation (Risk-Based Approach)
1. **Phase 1**: Immediate remediation of highest-risk content (large orphaned files) before migration
2. **Phase 2**: Archival of remaining content with complex relationships before migration
3. **Result**: Immediate risk reduction with controlled approach for complex content

---

## Pre-Migration Financial Analysis Framework

### Pre-Migration Remediation Cost Model
```
Total Remediation Cost = Setup + Execution + Validation + Risk Reserve
Pre-Migration Variables:
- Setup: SAP Basis configuration during planning phase, testing environment
- Execution: Actual cleanup operations before migration, typically $X per GB
- Validation: Testing and verification during planning, ~20% of execution cost
- Risk Reserve: Potential rework contingency during planning, ~25% of total
- Timeline Impact: Additional months in planning phase for remediation
```

### Pre-Migration Archival Cost Model
```
Total Archival Cost = Setup + Licensing + Operations + Training
Pre-Migration Variables:
- Setup: Content server deployment during planning, ArchiveLink configuration
- Licensing: Initial content server licensing investment before migration
- Operations: Planning for ongoing hosting and maintenance post-migration
- Training: User and admin training during preparation phase
- Integration: Network and security configuration during planning
```

### Pre-Migration Break-Even Analysis
- **Remediation**: Upfront cost with immediate HANA licensing savings post-migration
- **Archival**: Ongoing cost with gradual HANA licensing reduction post-migration
- **Decision Point**: At what time horizon does pre-migration investment pay back?

### Pre-Migration Example Calculation
For 500GB SOFFCONT1 with 40% orphans requiring pre-migration action:

**Remediation Option**:
- Cost: $75,000 (pre-migration cleanup project)
- Annual HANA Savings: $100,000 (post-migration memory reduction)
- Pre-Migration Timeline Impact: 3 months additional planning
- Payback: 9 months post-migration

**Archival Option**:
- Initial Investment: $50,000 (content server setup pre-migration)
- Annual Operational Cost: $15,000 (post-migration content server licensing)
- Annual HANA Savings: $70,000 (post-migration memory reduction)
- Net Annual Benefit: $55,000 post-migration
- Pre-Migration Timeline Impact: 4 months additional planning

---

## Pre-Migration Risk Assessment Matrix

### Technical Risk Factors During Planning Phase
| Factor | Remediation Risk During Planning | Archival Risk During Planning |
|--------|----------------------------------|-------------------------------|
| **Database Operations** | High (direct table manipulation pre-migration) | Low (SAP-integrated content server setup) |
| **Migration Complexity** | Reduced (smaller migration post-remediation) | Standard (content migration adds complexity) |
| **Performance Impact** | Medium (cleanup during planning phase) | Low (external system during planning) |
| **Backup/Recovery** | Simplified post-remediation | Complex coordination required pre-migration |

### Business Risk Factors During Pre-Migration Phase
| Factor | Remediation Risk During Planning | Archival Risk During Planning |
|--------|----------------------------------|-------------------------------|
| **Timeline Impact** | Moderate (cleanup timeline integration) | Higher (content server deployment timeline) |
| **Compliance During Migration** | Improved governance post-remediation | Maintained through archived content |
| **User Impact** | Minimal (during planning phase) | Training required pre-migration |
| **Business Continuity** | Improved post-migration simplicity | New dependency introduced pre-migration |

---

## Pre-Migration Decision Tree

```
S/4HANA Migration Planning Phase:
Content Volume Assessment?
├── < 100GB → Minor issue, monitor only during migration
├── 100GB - 500GB →
│   ├── Orphan Rate > 30%? → Remediation likely optimal during planning
│   ├── Orphan Rate 10-30%? → Hybrid approach during planning
│   └── Orphan Rate < 10%? → Archival likely optimal during planning
└── > 500GB →
    ├── Organizational readiness for complex cleanup during planning? → Remediation option viable
    ├── Existing ECM strategy during planning? → Archival optimal
    └── Compliance requirements strict during planning? → Archival with retention policies
```

---

## Pre-Migration Organizational Capability Assessment

### Remediation Capability Assessment During Planning Phase
- [ ] SAP Basis team experienced with cluster tables and pre-migration cleanup
- [ ] Database backup and recovery procedures tested during planning phase
- [ ] Business stakeholder ownership for content validation before migration timeline pressure
- [ ] Testing environment available during planning phase
- [ ] Change management process for technical changes during preparation
- [ ] Risk tolerance for database-level operations before go-live
- [ ] Timeline flexibility for remediation activities during planning

### Archival Capability Assessment During Planning Phase
- [ ] IT infrastructure readiness for content server during planning
- [ ] Network architecture for content server connectivity during planning
- [ ] User change management capability for access pattern changes
- [ ] Integration testing expertise during planning phase
- [ ] Ongoing content server administration skills post-migration
- [ ] Budget for initial setup and operational expenses during planning
- [ ] Timeline availability for content server deployment during planning

---

## Pre-Migration Implementation Timeline Comparison

### Pre-Migration Remediation Timeline (Integration into Planning Phase)
- **Week 1-2**: Content risk assessment and orphan identification during planning
- **Week 3-4**: Business stakeholder validation and approval during planning
- **Week 5-8**: Production cleanup execution during preparation phase
- **Week 9-10**: Post-remediation validation and migration preparation
- **Total Planning Impact**: 2.5 - 3 months added to preparation phase

### Pre-Migration Archival Timeline (Integration into Planning Phase)
- **Week 1-3**: Content server selection and procurement during planning
- **Week 4-6**: Server deployment and ArchiveLink configuration during planning
- **Week 7-12**: Content migration execution during preparation phase
- **Week 13-14**: Post-migration validation and optimization
- **Total Planning Impact**: 3 - 4 months added to preparation phase

---

## Pre-Migration Success Metrics and Planning Validation

### For Pre-Migration Remediation Planning
- **Orphan Reduction Target**: % of identified orphans planned for removal during preparation
- **Database Size Reduction Goal**: Projected GB reduction in SOFFCONT1 before migration
- **HANA Licensing Impact**: Quantified cost reduction for post-migration budgeting
- **Timeline Integration**: Remediation activities planned within migration preparation

### For Pre-Migration Archival Planning
- **Content Server Performance**: Planned access time requirements for post-migration operations
- **Integration Stability**: ArchiveLink functionality requirements validated during planning
- **User Adoption Plan**: Change management strategy for pre-migration implementation
- **Compliance Requirements**: Retention and audit requirements met during planning

---

## Pre-Migration Warning Signs for Each Approach

### Remediation Pre-Migration Red Flags
- Migration timeline too tight to accommodate remediation planning activities
- Business stakeholders unwilling to validate content for deletion before go-live
- SAP Basis team expresses concerns about complex cleanup during preparation
- Regulatory environment requires long-term retention of all content without deletion

### Archival Pre-Migration Red Flags
- No existing ECM strategy or governance during planning phase
- IT team lacks experience with external content systems during planning
- Network architecture cannot support content server integration during planning
- Budget model cannot accommodate setup and ongoing operational costs during planning

---

## Pre-Migration Executive Decision Criteria

For CIOs and Finance Leads evaluating options before irreversible migration commitments:

**Choose Pre-Migration Remediation When**:
- HANA cost reduction is primary objective before licensing commitment
- Organization has strong technical capabilities for pre-migration cleanup
- Risk tolerance supports database operations during preparation phase
- Budget profile favors capital investment during planning over operational expenses
- Timeline flexibility exists for extended preparation phase

**Choose Pre-Migration Archival When**:
- Long-term content governance is strategic priority during planning
- Organization has or wants to develop ECM capabilities during planning
- Risk tolerance prefers SAP-integrated solutions during migration
- Budget model supports infrastructure investment and operational expense distribution
- Compliance requirements mandate content preservation over deletion

**Choose Pre-Migration Hybrid When**:
- Mixed content profile (some clear orphans, some valuable content) during planning
- Organizational capability spans both approaches during planning
- Risk management prefers phased approach during preparation
- Timeline allows for multi-phase content management during planning

---

## Pre-Migration Decision Timing and Commitment Framework

The remediation vs. archival decision must be made during pre-migration planning when there is still flexibility to adjust timelines and budgets. The decision directly impacts:

- **Infrastructure Procurement**: HANA sizing and licensing requirements
- **Migration Timeline**: Complexity and risk profile of migration activities
- **Budget Allocation**: Capital vs. operational expense structure
- **Resource Planning**: Technical team requirements during preparation phase
- **Stakeholder Communication**: Risk posture communicated before go-live commitments

Organizations should evaluate their specific risk tolerance, technical capabilities, and strategic objectives during the pre-migration planning phase when making this critical choice that impacts both migration success and ongoing operational costs.

---

**Next**: See `boutique-si-differentiation.md` for service provider strategy considerations.