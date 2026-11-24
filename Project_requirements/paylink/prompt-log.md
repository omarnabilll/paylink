1. Prompt1 
Date:
 2026-11-22
Author:
 Islam-Heikal
Stage:
 Requirements
Tool and Model:
 Gemini / Fast
Prompt:
 List of stakeholders
Summary of answer:
 Stakeholders with a detailed brief description
Verdict:
 Rejected , Because is A detailed so there is conflict in info and Not concentrated
------------------------------------------------------
2. Prompt2 
Date:
 2026-11-22
Author:
 Islam-Heikal
Stage:
 Requirements
Tool and Model:
 Gemini / Fast
Prompt:
 For every stackholder and each brief description in one simple statement
Summary of answer:
 Stakeholders with a summarized NOT detailed brief description
Verdict:
 Fixed , As the info of each stakeholder is concentrated and Detected
-------------------------------------------------------
3. Prompt3
Date:
 2026-11-21
Author:
 Islam-Heikal
Stage:
 Requirements
Tool and Model:
 Gemini / Fast
Prompt:
 Write a user story for each requirement
Summary of answer:
 A user story with an Acceptance criteria
Verdict:
 Accepted , As written for each Non functional requirement its user story followed by Acceptance criteria With the Given format
-------------------------------------------------------
4. Prompt4
Date:
 2026-11-22
Author:
 Islam-Heikal
Stage:
 Requirements
Tool and Model:
 Gemini / Fast
Prompt:
 Extract non functional requirements 
Summary of answer:
   A Mentioned Non functional requirements without its target
Verdict:
 Rejected , As Detecting Non functional requirements is not enough it must get Each non functional requirement with its target
-------------------------------------------------------
5. Prompt5
Date:
 2026-11-22
Author:
 Islam-Heikal
Stage:
 Requirements
Tool and Model:
 Gemini / Fast
Prompt:
 Extract non functional requirements With its target for everyone
Summary of answer:
  A Mentioned Non functional requirements with its target
Verdict:
 Accepted , As it detect each requirement with its target
------------------------------------------------------
6. Prompt 6
Date:
2026-11-24
Author:
Omar
Stage:
Quality and DevOps
Tool and Model:
OpenAI Codex
Prompt:
Review the Phase 1 setup for our NestJS and PostgreSQL project. Check whether Docker Compose starts the app and database, and whether GitHub Actions runs the build, lint, and tests. Identify any remaining gaps.
Summary of answer:
Docker Compose successfully built and started the NestJS app and PostgreSQL; PostgreSQL reported healthy. Both GitHub Actions runs passed. The app has not been connected to PostgreSQL yet, so confirm whether the Phase 1 brief requires a database connection or only requires both services to start.
Verdict:
Accepted with a follow-up check. The Docker startup and CI checks passed; the team should verify the database-connection requirement against the brief.
