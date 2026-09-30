# TaxoRAG-MX: Retrieval-Augmented Generation System 

This repository contains the official implementation of the TaxoRAG-MX system, linked to the corresponding research article. It presents a modular Retrieval-Augmented Generation (RAG) architecture specifically designed for the biological domain, with a special emphasis on Mexican fauna. The system is designed to operate locally, allowing for intelligent indexing and querying of private document corpora using vector databases and open-source large language models (LLMs).

## Architecture and Workflow

The RAG system processes document corpora in text or Markdown format to ground the language model's generated responses strictly in the provided documents. The general workflow consists of three main stages:

1. **Indexing:** Ingests local files, and performs processing and precise text segmentation (chunking). Subsequently, it computes semantic vectors (embeddings) and stores them in a local vector database.
2. **Retrieval:** Upon receiving a user query, the system retrieves the most relevant text chunks based on semantic similarity. This process incorporates lexical and taxonomic enhancements characteristic of the TaxoRAG-MX methodology.
3. **Generation with Chain of Verification (CoVe):** Constructs the final response employing advanced verification techniques to mitigate hallucinations. The retrieved information is systematically cross-referenced to guarantee biological accuracy and adherence to the original source.

## Core Technologies

The system integrates the following technologies and frameworks:

* **ChromaDB:** Local vector database used for the storage and retrieval of embeddings.
* **LM Studio:** Local server providing an OpenAI-compatible API to host embedding models (e.g., `embeddinggemma-300m`) and language models (e.g., `qwen2.5-7b-instruct`) without dependency on cloud infrastructure.
* **Tiktoken:** Employed for precise, token-based document segmentation.
* **RAGAS:** Integrated into the evaluation scripts to measure system performance using metrics such as *faithfulness*, *answer relevancy*, and *answer correctness*.
* **LangChain:** Utilized in auxiliary and evaluation scripts for workflow orchestration.

## Usage Guide

1. **Prerequisites:** Ensure Python (3.10+) is installed. LM Studio must be running locally with the server enabled (default port: `1234`).
2. **Installation:**
   ```bash
   pip install -r requirements.txt
   ```
3. **Configuration:** Modify the `config.yaml` file to set up the vector database parameters and link the models loaded in LM Studio.
4. **Pipeline Execution:** To process and index the document corpus:
   ```bash
   python run_pipeline.py
   ```
5. **Interactive Mode (CLI):** To query the indexed documents via the command line:
   ```bash
   python interactive.py
   ```
6. **Evaluation:** To run the test suite and ablation studies comparing the baseline RAG with TaxoRAG-MX using CoVe:
   ```bash
   python run_generation_ablations.py
   ```

## Chain of Verification (CoVe) Methodology

To improve the factual accuracy of the generated responses, the system implements a 4-phase Chain of Verification (CoVe) process. The core system prompts (located in `src/generator.py`) are described below:

### 1. Initial Draft Generation
The model generates an initial response strictly constrained by the retrieved context.
> **System:** You are an expert biologist specializing in Mexican wildlife, acting as a strict RAG system. Your only source of truth is the provided context. Under NO circumstances should you use your prior or external knowledge. If the question assumes a false fact or asks for information not found in the context, limit your response to ONLY the data present in the passages. If the context does not contain the answer at all, respond exactly as follows: “The current knowledge base does not contain this information.”

> **User:** Context: [TEXT] \n\n Question: [QUESTION] \n\n Draft Answer:

### 2. Verification Planning
The model extracts key verifiable claims from the initial draft and formulates independent verification questions.
> **System:** Your task is to analyze a draft biological response and formulate short, independent, and direct questions that will help verify whether the key facts (dates, locations, conservation status, taxonomy) mentioned in the draft are accurate. Formulate questions only. Submit the questions as a list, one per line.

> **User:** Original question: [QUESTION] \n Draft to be reviewed: [DRAFT] \n\n Enter the review questions:

### 3. Verification Execution
The system independently answers each verification question by querying the original context.
> **System:** Answer the question very briefly, based EXCLUSIVELY on the context provided. If the context does not contain the answer, respond with “No information available.”

> **User:** Context: [TEXT] \n\n Question: [VERIFICATION_QUESTION] \n\n Short Answer:

### 4. Final Response Synthesis
The system synthesizes the final response by cross-referencing the initial draft with the verification results, correcting any discrepancy or hallucination.
> **System:** You are an expert biologist acting as a strict RAG system. You will provide a final answer to a question. You will be given your own initial draft, as well as a set of verification questions and answers taken out of the original context. Your task is to cross-check the data. If any of the verification answers contradict your draft, you MUST correct the error. If the verifications show that the draft included information that was not in the context (hallucination), omit it. Submit only the final, corrected, and coherent answer, without mentioning the verification process.

> **User:** [Includes the original question, the context, the draft, and the checklist with its answers] \n\n Final Corrected Answer:
