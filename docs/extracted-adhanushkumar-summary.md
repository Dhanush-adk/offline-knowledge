# Backend extraction result: adhanushkumar.com

Ran the extraction pipeline (WebView → chunk → store) for **https://adhanushkumar.com/** and captured everything the app backend extracted.

---

## 1. Extracted title

**A Dhanush Kumar Portfolio**

---

## 2. Extracted raw body (full text)

The backend stored this as the single “raw body” for the page (visible text from the site after JS render):

- **Header/nav:** ADK's Portfolio, About, Projects, Experience, Publications, Resume, Contact  
- **Hero:** Data Engineer • Data Analyst • Software Engineer, Hello I'm Dhanush Kumar, I'm a Data Analyst  
- **About:** Data Engineer, Data Analyst, and Software Engineer based in the United States, currently working at **ChekOut.AI** building AI-driven systems and backend infrastructure. Over three years of experience; data automation, real-time analytics, scalable systems at **Juspay Technologies** and **King Machine**. Expertise: Python, SQL, Golang, GCP, AWS; workflow automation, distributed systems, AI-powered solutions.  
- **More About Me:** Software Engineer, backend systems, data engineering, AI-driven automation. Engineering Intern at King Machine (automated file management, Git-based version control). At Juspay: real-time data analysis, 98% processing time reduction, 80% infrastructure cost reduction. ML Engineer experience; research on abnormal activity detection (deep learning) published in Springer Nature LNNS, presented at WORLD S4 2021.  
- **Projects:** Multi-Processor Cache Coherence Simulation (C++); Vehicle Speed Estimation (RNNs/Transformers, 94.25% accuracy); Adaptive Decision-Making (Monte Carlo, DQN); Abnormal Activity Detection (Spatio-Temporal Autoencoders, C3D, 78%); Age Group Prediction from Voice; Self-Driving Car (Raspberry Pi, computer vision); Job Aggregator (Selenium, Flask, Heroku); Menu Scraper (Swiggy, Zomato); Spam Detection (NLP/sentiment); Line-Following Robot (Arduino).  
- **Work experience:**  
  - Founding Engineer, ChekOut.AI (Jul 2025–Present): one-click agent deployment, RAG, Agent Deployment Engine, Cloud Run.  
  - Engineering Intern, King Machine (Jan–Dec 2025): file management, Git, automation.  
  - Engineering Intern, UNC Charlotte (Oct 2023–Jan 2025): R Shiny, MongoDB, facilities equipment analysis, 50% cost reduction.  
  - Senior Data Analyst, Juspay (Jan 2022–Jun 2023): R, BigQuery, ClickHouse, Golang, Docker, Jenkins, 80% infra cost reduction.  
  - Machine Learning Engineer, Deeploop (Jul–Dec 2021): Business Genie (81%), Django/AWS Lambda, skin cancer detection (83%).  
- **Education:** Master's Computer Engineering, UNC Charlotte (Aug 2023–May 2025); Bachelor's ECE, JNTU Hyderabad (Jul 2017–Aug 2021).  
- **Publication:** Abnormal Activity Detection Using Deep Learning — Springer Nature LNNS, World S4 2021.  
- **Skills:** R, Python, Golang, JavaScript, HTML, PostgreSQL, BigQuery, ClickHouse; Agentic AI, LLM (OpenAI, Claude, Gemini), RAG, Vector DBs (Pinecone, Chroma); Data Analysis, ML, Deep Learning (PyTorch, TensorFlow), Selenium, SQL, Docker, Jenkins, Agile; Langflow, N8N, GCP, Tableau, etc.  
- **Footer:** Contact form, “A Dhanush Kumar”, © 2026.

So the backend **is** extracting the full visible content from adhanushkumar.com (no browser used for this run; pipeline only).

---

## 3. Chunks (indexed for search)

- **Total chunks:** **500** (pipeline cap: `maxChunksPerItem = 500`).  
- Each chunk is a substring of the raw body (max ~600 chars, overlap 100), used for embedding and semantic search.  
- Chunk 1 starts with: “ADK's Portfolio, About, Projects, Experience, Publications, Resume, Contact, Resume, Data Engineer • Data Analyst • Software Engineer, Hello, I'm Dhanush Kumar, I'm a Data Analyst”.  
- Later chunks cover the rest of the page (About paragraph, projects, experience, education, skills, etc.).

---

## 4. How this was produced

- Test: `SaveAndAskIntegrationTests.testExtractAdhanushkumarAndPrintAllExtracted`.  
- It runs only the backend: WebView load of adhanushkumar.com → extract title + body → chunk → store in a temp DB (with `MockEmbeddingService`).  
- The test then printed: `EXTRACTED_TITLE`, `EXTRACTED_RAW_BODY`, `EXTRACTED_CHUNKS_COUNT`, and all chunk texts.  
- This summary is a condensed view of that output.

---

**Conclusion:** The app backend successfully extracts the full visible content from adhanushkumar.com and stores it as one raw body plus 500 chunks for indexing and search.
