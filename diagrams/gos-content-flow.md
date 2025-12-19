# GOS Content Flow Diagram Explanation

## Purpose
This textual diagram explains the flow of content through SAP's Generic Object Services (GOS) architecture, specifically the relationship between business objects and content storage via the KPRO framework.

## Components
- **Business Object Layer**: Standard SAP objects (BUS2012 for PO, BUS1006 for Business Partner)
- **Relationship Layer**: Links between business objects and content objects (SRGBTBREL)
- **Content Repository Layer**: Physical storage of binary data (SOFFCONT1 with SOFFPHIO metadata)

## Flow Description
1. **User Action**: Attaches document to business object (ME23N for PO)
2. **GOS Processing**: Creates SRGBTBREL relationship, SOFFPHIO metadata, SOFFCONT1 binary data
3. **KPRO Abstraction**: Manages storage destination (internal DB vs. external content server)
4. **Access Pattern**: User → Business Object → SRGBTBREL → SOFFPHIO → SOFFCONT1

## Risk Points
- Archiving removes SRGBTBREL but leaves SOFFPHIO/SOFFCONT1 (creates orphans)
- Migration tools process all three tables but may break referential integrity
- Backup/recovery must maintain all three tables in sync

## Visual Structure
```
[User] → [Business Object] → [SRGBTBREL] → [SOFFPHIO] → [SOFFCONT1]
                    ↓              ↓           ↓           ↓
              (BUS2012)    (Relationship) (Metadata) (Binary Content)
```

This architecture creates the foundation for content risk when relationships are severed during archiving or migration.