# MESTRE Demo Script — Contractor Meetings

## Setup (before meeting)
1. Open Terminal: `cd /Users/vanshsmac/MESTRE && streamlit run app.py`
2. Open browser: `localhost:8501`
3. Have Essex-Windsor PDF ready OR ask contractor to bring their own tender

## Demo Flow (5 minutes)
1. "How long does your takeoff usually take?" (let them answer — 4-8 hours)
2. Fill in their company profile in sidebar
3. Upload their tender (or Essex-Windsor)
4. While it processes (~90 seconds): "MESTRE reads every page, extracts every line item, checks OPSS compliance, flags risks, and builds your bid checklist"
5. Show results: "82 items extracted. Let me show you the 6 sheets"
   - Sheet 1: Takeoff — every item, quantity, unit, spec reference
   - Sheet 4: Strategy & Risks — 72 cost risk flags
   - Sheet 5: Bid Checklist — 22 submission requirements, 19 critical
   - Sheet 6: Timeline — every deadline including mandatory site meeting
6. Show Q&A: type "What are the biggest cost risks?" — live AI answer
7. "This replaces 4-8 hours of work. It costs $29. How many tenders do you bid on per month?"

## Objection Handling
- "How accurate is it?" → "100% on quantities we've tested. Every item has a confidence score. Always verify critical items — this is a first-pass tool that saves hours."
- "Is my data safe?" → "Your tender is processed and deleted within 24 hours. We never store or train on your data."
- "Can I try it?" → "Upload any tender you have right now. First preview is free."
- "What if it misses something?" → "The cross-verification catches gaps. Plus the Q&A lets you ask specific questions about your tender."
