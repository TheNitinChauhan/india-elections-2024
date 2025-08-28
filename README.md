# INDIA GENERAL ELECTION RESULTS ANALYSIS — 2024

**Author:** Nitin Chauhan  
**Contact:** linkedin.com/in/nitin-chauhan-9857a4255 / https://github.com/TheNitinChauhan

---

## Project summary
This repository contains a reproducible end-to-end **SQL + Power BI** project analyzing the **India General Elections 2024** using publicly available Kaggle data. The project integrates SQL-based data modeling and transformation with an interactive Power BI report designed for exploration and storytelling.

Key highlights:
- **Dynamic Landing Page** for navigation and quick KPIs.  
- **Drill-through functionality** to move seamlessly from state → constituency → candidate details.  
- **Multi-page report** covering: Landing, Overview, State Demographics, Political Landscape, Constituency Analysis, and Details Grid.  
- **Alliance and party-level insights**: NDA, I.N.D.I.A, and Others with seat counts, margins, and vote share.  
- **Detailed candidate view**: EVM vs Postal vote split and winner–runner-up analysis.  
- **Data source**: curated Kaggle election dataset, cleaned and modeled into a relational schema for SQL + Power BI integration.  
- **Reproducibility**: SQL scripts for data preparation and transformations ensure that results in Power BI align with backend queries.


---
## 📱 Quick Access
Scan the QR code below to open the interactive Power BI dashboard on your phone:

![QR Code to Dashboard](assets/qr/qr_dashboard.png)

## Problem statement
Build an end-to-end analysis that:
- Aggregates constituency-level results into state and alliance summaries.
- Visualizes seat distribution, party performance, and vote breakdowns (EVM vs Postal).
- Enables drill-downs from state → constituency → candidate.
- Produces an interactive Power BI report with navigation and exportable detail grids.


---

## Repository layout (recommended)
```
/india-elections-2024/
├─ data/
│  ├─ constituencywise_results.csv
│  ├─ constituencywise_details.csv
│  ├─ partywise_results.csv
│  ├─ statewise_results.csv
│  └─ states.csv
├─ sql/
│  ├─ create_tables.sql
│  ├─ load_data.sql
│  └─ analysis_queries.sql  
├─ powerbi/
│  └─ India_Elections_2024.pbix
├─ docs/
│  ├─ Problem Statement PBi.pdf
│  └─ Data Modeling PPT.pptx
├─ assets/
│  └─ party_logos/, screenshots/
└─ README.md
```

---

## Data model (high level)
Key tables expected (refer to `docs/Data Modeling PPT.pptx` for ERD):
- `states` — master list of states (State_ID, State_Name, etc.)  
- `statewise_results` — state-level aggregates and metadata  
- `constituencywise_results` — one row per constituency (Constituency_ID, Parliament_Constituency, Constituency_Name, Winning_Party_ID, Total_Votes, Margin, etc.)  
- `constituencywise_details` — candidate-level rows per constituency (Candidate, Party, EVM_Votes, Postal_Votes, Total_Votes)  
- `partywise_results` — party-level summary (Party_ID, Party_Name, Seats_Won, etc.)

---

## Important SQL snippets (copy-paste ready)

### 1. Total seats in the dataset
```sql
SELECT COUNT(DISTINCT Parliament_Constituency) AS Total_Seats
FROM constituencywise_results;
```

### 2. Seats available per state
```sql
SELECT s.State AS State_Name,
       COUNT(cr.Constituency_ID) AS Total_Seats_Available
FROM constituencywise_results cr
JOIN statewise_results sr ON cr.Parliament_Constituency = sr.Parliament_Constituency
JOIN states s ON sr.State_ID = s.State_ID
GROUP BY s.State
ORDER BY s.State;
```

### 3. Add alliance mapping and count seats by alliance
```sql
ALTER TABLE partywise_results
ADD party_alliance VARCHAR(50);

-- Mark parties (example)
UPDATE partywise_results
Party Alliance = 
IF(
    partywise_results[Party] = "Bharatiya Janata Party - BJP" ||
    partywise_results[Party] = "Telugu Desam - TDP" ||
    partywise_results[Party] = "Janata Dal  (United) - JD(U)" ||
    partywise_results[Party] = "Shiv Sena - SHS" ||
    partywise_results[Party] = "AJSU Party - AJSUP" ||
   partywise_results[Party] = "Apna Dal (Soneylal) - ADAL" ||
    partywise_results[Party] = "Asom Gana Parishad - AGP" ||
    partywise_results[Party] = "Hindustani Awam Morcha (Secular) - HAMS" ||
    partywise_results[Party] = "Janasena Party - JnP" ||
    partywise_results[Party] = "Janata Dal  (Secular) - JD(S)" ||
    partywise_results[Party] = "Lok Janshakti Party(Ram Vilas) - LJPRV" ||
    partywise_results[Party] = "Nationalist Congress Party - NCP" ||
    partywise_results[Party]= "Rashtriya Lok Dal - RLD" ||
    partywise_results[Party] = "Sikkim Krantikari Morcha - SKM",
    "NDA",
    IF(
        partywise_results[Party] = "Indian National Congress - INC" ||
        partywise_results[Party] = "Aam Aadmi Party - AAAP" ||
        partywise_results[Party] = "All India Trinamool Congress - AITC" ||
        partywise_results[Party] = "Bharat Adivasi Party - BHRTADVSIP" ||
        partywise_results[Party]= "Communist Party of India  (Marxist) - CPI(M)" ||
        partywise_results[Party] = "Communist Party of India  (Marxist-Leninist)  (Liberation) - CPI(ML)(L)" ||
        partywise_results[Party] = "Communist Party of India - CPI" ||
        partywise_results[Party] = "Dravida Munnetra Kazhagam - DMK" ||
        partywise_results[Party] = "Indian Union Muslim League - IUML" ||
        partywise_results[Party] = "Jammu & Kashmir National Conference - JKN" ||
        partywise_results[Party] = "Jharkhand Mukti Morcha - JMM" ||
        partywise_results[Party] = "Kerala Congress - KEC" ||
        partywise_results[Party] = "Marumalarchi Dravida Munnetra Kazhagam - MDMK" ||
        partywise_results[Party] = "Nationalist Congress Party Sharadchandra Pawar - NCPSP" ||
        partywise_results[Party] = "Rashtriya Janata Dal - RJD" ||
        partywise_results[Party] = "Rashtriya Loktantrik Party - RLTP" ||
        partywise_results[Party] = "Revolutionary Socialist Party - RSP" ||
        partywise_results[Party] = "Samajwadi Party - SP" ||
        partywise_results[Party] = "Shiv Sena (Uddhav Balasaheb Thackrey) - SHSUBT" ||
        partywise_results[Party] = "Viduthalai Chiruthaigal Katchi - VCK",
        "I.N.D.I.A.",
        "OTHER"
    )
)


SELECT p.party_alliance,
       COUNT(cr.Constituency_ID) AS Seats_Won
FROM constituencywise_results cr
JOIN partywise_results p ON cr.Party_ID = p.Party_ID
GROUP BY p.party_alliance
ORDER BY Seats_Won DESC;
```

### 4. Winner & runner-up per constituency (CTE + window)
```sql
WITH RankedCandidates AS (
  SELECT cd.Constituency_ID,
         cd.Candidate,
         cd.Party,
         cd.EVM_Votes + cd.Postal_Votes AS Total_Votes,
         ROW_NUMBER() OVER (PARTITION BY cd.Constituency_ID ORDER BY cd.EVM_Votes + cd.Postal_Votes DESC) AS VoteRank
  FROM constituencywise_details cd
)
SELECT rc.Constituency_ID,
       MAX(CASE WHEN rc.VoteRank = 1 THEN rc.Candidate END) AS Winning_Candidate,
       MAX(CASE WHEN rc.VoteRank = 2 THEN rc.Candidate END) AS Runnerup_Candidate
FROM RankedCandidates rc
GROUP BY rc.Constituency_ID;
```

### 5. EVM vs Postal votes for a constituency
```sql
SELECT cd.Candidate, cd.Party, cd.EVM_Votes, cd.Postal_Votes, (cd.EVM_Votes + cd.Postal_Votes) AS Total_Votes
FROM constituencywise_details cd
JOIN constituencywise_results cr ON cd.Constituency_ID = cr.Constituency_ID
WHERE cr.Constituency_Name = 'MATHURA'   -- change as needed
ORDER BY Total_Votes DESC;
```

> Save these queries in `sql/analysis_queries.sql` and adapt syntax to your SQL engine (T-SQL / PostgreSQL / MySQL) as needed.

---

## Power BI – what to build & publish
**Pages to include (one .pbix file, multiple pages):**
- Landing Page (navigation + summary KPIs)  
- Overview (seat & alliance summaries, top parties)  
- State Demographics (map / choropleth + state KPIs)  
- Political Landscape by State (donuts, stacked bars)  
- Constituency Analysis (candidate leaderboards, EVM vs Postal breakdown)  
- Details Grid (exportable table with all columns)

**Tips while building**
- Use consistent page size and theme (Format → Page size / Theme).  
- Add a Landing/Home page with navigation buttons (Insert → Buttons → Action → Page navigation).  
- Use `View → Sync slicers` to keep common filters synchronized across pages.  
- Use Bookmarks + Selection Pane for toggles (e.g., show/hide map layers).  
- Save as a single `.pbix` (File → Save). That file holds all report pages.

**Publish & share (Power BI Service)**
1. Home → Publish → choose Workspace.  
2. In Service, pin key visuals from multiple pages to a single Dashboard (pin icon on visuals).  
3. Optionally create an App from the workspace (Create → App) and include the report + dashboard for distribution.  
4. Configure dataset refresh + gateway (Workspace → Datasets → Settings) if your data is not cloud-based.

---

## How to reproduce (step-by-step)
1. Create database `india_elections_2024` (SQL Server recommended for T-SQL, or adapt to other DB engines).  
2. Run `sql/create_tables.sql` to create schemas.  
3. Load CSVs from `/data/` into tables (BULK INSERT / COPY / Python ETL).  
4. Run `sql/load_data.sql` and `sql/analysis_queries.sql` to populate derived fields (party_alliance, views).  
5. Open `powerbi/India_Elections_2024.pbix` or create a new report and connect to the database.  
6. Recreate relationships per the ERD, add measures, create the visuals listed above.  
7. Validate KPIs by cross-checking SQL outputs vs Power BI measures.

---

## Key insights (examples from screenshots)
> These are sample values visible in the provided screenshots — run the SQL to reproduce exact numbers from your data.

- **NDA Alliance** — example: 292 seats (~53.8%)  
- **I.N.D.I.A Alliance** — example: 234 seats (~43.1%)  
- **Other / Independents** — example: 17 seats (~3.1%)

---

## Suggested improvements & roadmap
1. **Automate ingestion**: use Azure Data Factory / Python scripts to fetch and transform CSVs.  
2. **Create master tables** for parties and states to standardize names and avoid mismatches.  
3. **Add historical time series**: include past election data for trend analysis.  
4. **Predictive modeling**: build models to forecast margins or seat swings.  
5. **Publish to Power BI Service App** and enable scheduled refresh + row-level security (if needed).  
6. **Mobile-optimized pages** and narrative bookmarks for easy storytelling in interviews/demos.

---

## Notes & assumptions
- Alliance mapping (`party_alliance`) must be updated if parties change alignment.  
- Validate joins and keys (duplicate counting often comes from wrong join cardinality).  
- The SQL examples assume columns named as described—adjust column names to match your CSVs/tables.

---

## How to present this project (short script)
1. Start with the Landing Page — show Top KPIs and the “big picture” alliance seat counts.  
2. Drill into Overview — show party-level performance and top winners.  
3. Move to State Demographics — highlight regional patterns using the map.  
4. Open Constituency Analysis — show a candidate-level example (EVM vs Postal votes, margin).  
5. End with the Details Grid and reproducibility steps (SQL + data model).

Use bookmarks for a guided tour and keep a 3-slide PDF summary for interviews.

---

## Contact
**Author:** Nitin Chauhan  
linkedin.com/in/nitin-chauhan-9857a4255 / https://github.com/TheNitinChauhan
---
