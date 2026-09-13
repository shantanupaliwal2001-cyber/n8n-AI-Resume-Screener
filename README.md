# AI-Powered Resume Screening Pipeline

An automated, defensive data pipeline built in n8n that screens batch resumes, catches "keyword stuffers," and operates entirely on local hardware without crashing.

## The Business Problem
Recruiters spend countless hours manually scanning resumes, often falling victim to candidates who stuff their resumes with keywords but lack actual experience. Furthermore, using AI to solve this often results in exorbitant API costs, or if run locally, severe hardware bottlenecks. 

This production-grade pipeline solves both issues. It standardizes the grading process to save recruiter time, and uses a highly optimized architecture to run locally on a 7B LLM without frying the CPU.

## Architecture & Technical Highlights

*   **Compute Optimization:** The Job Description (JD) rubric is extracted outside the main loop just once, acting as a static baseline. This saves roughly 80% on token generation costs and keeps the local model breathing easily.
*   **Defensive Routing (Check PDF Readability):** Candidates often upload unreadable image-based PDFs. A pre-flight sanity check detects these and routes them straight to human review, preventing wasted AI compute.
*   **Map-Reduce AI Flow:** Inside the loop, cognitive load is split across a dual-LLM architecture:
    *   *AI #1 (Extract Resume JSON):* Acts as a data structurer, turning messy PDF text into strict JSON.
    *   *AI #2 (The Grader LLM):* Acts as the hiring manager, ruthlessly cross-verifying a candidate's claimed skills against the evidence in their actual work experience.
*   **Data Normalization (LLM Output Parser):** Local models occasionally hallucinate markdown tags (like ````json````) or conversational text. I built custom JavaScript try...catch blocks to act as a "Blast Shield," actively stripping this out so the pipeline never crashes mid-batch.

## The Pipeline Flow
1. **Ingestion:** Read Candidate Resumes & Map Resume Text.
2. **Validation:** Unreadable files are logged; valid files pass to the Combine JD & Resumes merge node.
3. **Sequential Processing:** The Process Resumes One-by-One loop feeds data to the local LLMs sequentially to prevent memory overloads.
4. **Output:** The parser translates numerical scores into deterministic verdicts (Strong/Potential/Weak Fit) and pushes the final payload to Google Sheets and Telegram.

## Tech Stack
*   **Orchestration:** n8n (Node.js)
*   **AI/LLM:** Ollama (Local 7B Model)
*   **Integrations:** Google Workspace, Telegram API, Local File System Parsing

## How to Run
1. Clone this repository and download Resume_Parser.json.
2. Open your n8n instance and click "Import from File".
3. Configure your local Google Sheets and Telegram credentials.
4. Drop your candidate PDFs into the designated local folder and hit Execute!
