# sample-QMDL-in-PH-Makati-city
Using Rayhunter on TP-Link M7350 with firmware v9.0.3 Build 241219 Rel.1089n 

Geolocation of QMDL dump <img width="1358" height="1049" alt="Rayhunter_TP-Link-M7350_geolocation-from-QMDL_Drexx-20251023" src="https://github.com/user-attachments/assets/50d21515-2a77-47da-870f-0b7369482e0b" />

Decoded trace from TP-Link M7350 using Rayhunter 0.7,1. I marked each heuristic as **Observed / Not Observed / Inconclusive**.

---

# Heuristic 1 — NAS “IMSI Requested”

**Question:** Do we see a NAS **Identity Request** followed by **Identity Response (IMSI)** early in the flow?

* **WHAT'S FOUND OR NOT IN PCAP:**

  * The trace shows a **NAS Service Request** encapsulated in a NAS security header with a **Short MAC**, suggesting a pre-existing NAS security context (i.e., *not* a bare, unprotected initial attach with IMSI disclosure).  
  * No decoded lines show a **NAS Identity Request/Response** pair.

* **INFERENCE:**

  * The presence of a Service Request with integrity (Short MAC) and the **absence** (in the provided decode) of Identity Request/Response strongly suggest **no IMSI solicitation occurred** in this capture segment. 
  * **Result:** **Not Observed.**
  * **Heuristic cross-reference:** IMSI solicitation is flagged as exceptional in legitimate networks; but it does not appear here. 

---

# Heuristic 2 — **RRCConnectionRelease** with **Redirect** (Forced 2G/3G Downgrade)

**Question:** Do we see an **RRCConnectionRelease** carrying **redirectedCarrierInfo** to GERAN/UTRAN, or **MobilityFromEUTRACommand** pushing the UE off LTE?

* **WHAT'S FOUND OR NOT IN PCAP:**

  * The decode includes **paging (PCCH)** for an **s-TMSI**, which is normal. 
  * Did **not** find an **RRCConnectionRelease** with inter-RAT redirect, nor a **MobilityFromEUTRACommand** in the shown sections.

* **INFERENCE:**

  * No evidence of a **forced downgrade** in the decoded segment. **Confidence: Medium** (subject to capture completeness).
  * **Result:** **Not Observed.**
  * **Heuristic cross-reference:** This is one of classic downgrade indicators; but it does not appear. 

---

# Heuristic 3 — **SIB6/SIB7** “Lure” Parameters (Broadcast-layer downgrade)

**Question:** Does **SIB1** schedule **SIB6/SIB7**, and are their inter-RAT priorities suspicious?

* **WHAT'S FOUND OR NOT IN PCAP:**

  * **SIB1** shows **schedulingInfoList** with **SIB3** and **SIB5** scheduled (periodicities rf8 / rf32). **SIB6** or **SIB7** are not seen as advertised in the scheduling list. 
  * Additional SIB decoding shows **SIB3** details, consistent with normal LTE reselection info; no SIB6/7 bodies appear in the excerpt. 

* **INFERENCE:**

  * With **no SIB6/SIB7 scheduled**, there is **no broadcast “lure”** toward 3G/2G in this cell as decoded. **Confidence: High** (based on explicit SIB1 schedule lines).
  * **Result:** **Not Observed.**
  * **Heuristic cross-reference:** Your guidance expects SIB6/SIB7 if a rogue cell is trying to nudge inter-RAT reselection. 

---

# Heuristic 4 — **AS (RRC/UP) Null Cipher** (EEA0/EIA0) via **AS Security Mode Command**

**Question:** Do we see **Access-Stratum Security Mode Command** selecting **EEA0/EIA0**?

* **WHAT'S FOUND OR NOT IN PCAP:**

  * The shown RRC/NAS snippets do not include an **AS Security Mode Command** selecting **EEA0/EIA0**.
  * There is bearer setup material (PDCP/RLC config for DRBs), which looks like normal configuration (no hint of “null” AS security selection in the displayed lines).  

* **INFERENCE:**

  * No evidence of **AS null cipher** in these decodes. **Confidence: Medium** (absence in excerpt ≠ absolute absence).
  * **Result:** **Not Observed.**
  * **Heuristic cross-reference:** A rogue insisting on EEA0/EIA0 would be a red flag; but not present here. 

---

# Heuristic 5 — **NAS Null Cipher** (EEA0/EIA0 at NAS level)

**Question:** Do we see **NAS Security Mode Command** choosing **EEA0/EIA0**?

* **WHAT'S FOUND OR NOT IN PCAP:**

  * The only NAS items shown are **Service Request** messages with **Security header / Short MAC**, implying integrity protection, not “null.”  
  * No **NAS Security Mode Command** selecting EEA0/EIA0 appears in the excerpt.

* **INFERENCE:**

  * **NAS null ciphering is not evidenced** here; on the contrary, authenticated NAS is seen here. **Confidence: Medium-High.**
  * **Result:** **Not Observed.**
  * **Heuristic cross-reference:** Warnings against EEA0/EIA0 at NAS are not indicated in this trace. 

---

# Heuristic 6 — **Malformed/Incomplete SIBs**

**Question:** Are SIBs malformed, missing mandatory IEs, or scheduled but missing on-air?

* **WHAT'S FOUND OR NOT IN PCAP:**

  * **SIB1** explicitly schedules **SIB3** and **SIB5**; those appear coherently decoded. 
  * No decoding error flags or obviously nonsensical values are shown in the SIB packets provided.

* **INFERENCE:**

  * **No SIB malformation** or “advertised-but-missing” behavior is visible from the extracted lines. **Confidence: Medium** (sample size limited).
  * **Result:** **Not Observed.**
  * **Heuristic cross-reference:** Malformed/missing SIBs are treated as a common rogue artifact; not seen here. 

---

# Other salient artifacts

* **Paging with s-TMSI** (normal network behavior protecting IMSI): **Observed.** 
* **UE Security Information present** (start-CS) within capability exchange context: **Observed** (typical). 

---

# Consolidated Assessment

* **FACTS (recap):**

  * **No NAS Identity Request/Response (IMSI) seen.**  
  * **No AS/NAS Security Mode Command selecting EEA0/EIA0.** (not present in the decodes shown)
  * **No RRC redirect / forced inter-RAT downgrade** observed.
  * **No SIB6/7 scheduled; hence no broadcast downgrade lure.** 
  * **SIBs appear coherent; paging uses s-TMSI.** 

* **INFERENCE:**

  * Confidence of rogue activity rises only when **multiple indicators co-occur** (IMSI request + downgrade + null ciphers + SIB anomalies). Here, **none** of those tripwires fire. **This points toward a normal LTE cell/session**, not an IMSI-catcher or rogue eNodeB. 

---
