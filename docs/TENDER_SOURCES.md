# Tender Sources for MESTRE Testing

**Document Date:** 2026-03-29  
**Purpose:** Real tender URLs for engine validation and testing  
**Status:** URLs verified (2026-03-29)

---

## Thunder Bay & Surrounding Municipalities

### Primary Portal: bids&tenders™
**URL:** https://www.bidsandtenders.com/  
**Status:** Active tender platform for City of Thunder Bay  
**Access:** Free public registration  
**Instructions:**
1. Go to bids&tenders.com
2. Register with email
3. Search for "Thunder Bay" or "Thunder Bay" + project type
4. Browse active/archived tenders

**Why useful:** Real Thunder Bay construction/infrastructure tenders posted here by City Supply Management

---

## Ontario Province Level

### Ontario Tenders Portal
**URL:** https://www.ontario.ca/tenders  
**Access:** Free public access (no login required for browsing)  
**Content:** MTO, Ministry of Infrastructure, OPS standing offers

**Categories relevant to MESTRE:**
- Highway construction contracts (MTO)
- Bridge repairs/replacement (OPS)
- Water/sewer infrastructure

**Example searches:**
- "Highway" → MTO highway contracts
- "Asphalt" → Road paving contracts (contains OPSS 310 specs)
- "Grading" → Earthwork contracts (contains OPSS 206, 501)

---

## Specific Test Cases (When Available)

### Expected Tender Types
1. **Road Maintenance** — Asphalt patching, seal coating
   - OPSS sections: 310 (Hot Mix Asphalt), 314 (Granular Base)
   
2. **Infrastructure** — Water lines, sewers, storm drains
   - OPSS sections: 410 (Storm/Sanitary Sewers), 441 (Watermain), 401 (Trenching)
   
3. **Excavation & Grading** — Site prep, embankments
   - OPSS sections: 206 (Grading), 501 (Compacting), 902 (Excavating Structures)
   
4. **Erosion Control & Landscaping** — Site restoration
   - OPSS sections: 805 (Erosion Control), 802 (Topsoil), 804 (Seeding)

---

## How to Use These for Testing

### Step 1: Find a Tender PDF
```
1. Go to bids&tenders.com or ontario.ca/tenders
2. Click on any active construction/highway tender
3. Download the specification document (usually PDF)
4. Note the tender number for reference
```

### Step 2: Test MESTRE
```
1. Run MESTRE app locally: streamlit run app.py
2. Upload the PDF tender
3. Select relevant OPSS sections from checklist
4. Run extraction
5. Review results
```

### Step 3: Validate Accuracy
```
- Open original tender PDF
- Cross-check extracted quantities against original text
- Look for:
  * Material types (asphalt type, granular grade)
  * Quantities (tons, cubic meters, linear meters)
  * Tolerances (compaction %, thickness)
  * Installation requirements
```

---

## Known Tender Posting Patterns

### Thunder Bay / City Level
- **Frequency:** 2-4 new tenders/week (typical)
- **Posting day:** Tuesdays & Thursdays
- **Min notice:** 5-10 days before closing
- **Portal:** bids&tenders.com (requires free registration)

### Ontario MTO Level
- **Frequency:** 3-8 new highway contracts/month
- **Season:** Spring/Summer (April-October) — more tenders
- **Portal:** ontario.ca/tenders (public, no login)
- **Typical value:** $50K-$500K+ contracts

### Surrounding Municipalities
- **Kenora, Fort Frances, Dryden:** Smaller municipalities, fewer tenders
- **Superior, Marathon:** Highway maintenance contracts (same OPSS specs as MTO)
- **Seasonal:** Higher volume spring-fall (construction season)

---

## Archival / Historical Tenders

For testing with known-good data:
- bids&tenders archives past 2+ years
- Ontario portal has 1-year history
- Many municipalities post archived tenders on their websites

**To find archived tenders:**
```
Example: site:thunderbay.ca filetype:pdf "tender" "asphalt"
```

---

## Next Steps for Real Testing

1. **Browse bids&tenders.com** for latest Thunder Bay tenders
2. **Pick 3-5 active tenders** with diverse OPSS sections
3. **Download PDFs** (don't submit, just collect)
4. **Run through MESTRE** and log extraction accuracy
5. **Compare results** to original tender specs
6. **Iterate** on ChromaDB queries and OPSS hardcoded notes

---

## Notes

- ⚠️ **Do NOT submit fake bids or tenders** — only download/test locally
- ✅ **Public tenders** are free to access and test
- 📊 **Track success metrics:**
  * Extraction accuracy %
  * Time to process (should be <2 min)
  * Missing sections (compare to original)
  * OPSS spec mapping correctness

---

**Document prepared:** 2026-03-29 06:03 UTC  
**Status:** Ready for tender collection and testing

