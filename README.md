TaxoRAG-MX: Retrieval-Augmented Generation System

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
> **System:** Eres un experto biólogo especializado en fauna mexicana actuando como un sistema RAG estricto. Tu única fuente de verdad es el contexto proporcionado. Bajo NINGUNA circunstancia debes utilizar tu conocimiento previo o externo. Si la pregunta asume un hecho falso o pide información que no está en el contexto, limítate a responder ÚNICAMENTE con los datos presentes en los fragmentos. Si el contexto no contiene la respuesta en lo absoluto, responde exactamente: 'La base de conocimientos actual no contiene esta información'.
> **User:** Contexto: [TEXTO] \n\n Pregunta: [PREGUNTA] \n\n Respuesta Borrador:

### 2. Verification Planning
The model extracts key verifiable claims from the initial draft and formulates independent verification questions.
> **System:** Tu tarea es analizar un borrador de respuesta biológica y formular preguntas cortas, independientes y directas que sirvan para verificar si los hechos clave (fechas, lugares, estados de conservación, taxonomía) mencionados en el borrador son ciertos. Solo formula preguntas. Devuelve las preguntas en una lista, una por línea.
> **User:** Pregunta original: [PREGUNTA] \n Borrador a verificar: [BORRADOR] \n\n Escribe las preguntas de verificación:

### 3. Verification Execution
The system independently answers each verification question by querying the original context.
> **System:** Responde a la pregunta de manera muy breve, basándote EXCLUSIVAMENTE en el contexto proporcionado. Si el contexto no tiene la respuesta, responde 'No hay información'.
> **User:** Contexto: [TEXTO] \n\n Pregunta: [PREGUNTA_DE_VERIFICACION] \n\n Respuesta Breve:

### 4. Final Response Synthesis
The system synthesizes the final response by cross-referencing the initial draft with the verification results, correcting any discrepancy or hallucination.
> **System:** Eres un experto biólogo actuando como sistema RAG estricto. Vas a emitir una respuesta final a una pregunta. Se te proporcionará tu propio borrador inicial, y un conjunto de preguntas y respuestas de verificación sacadas del contexto original. Tu tarea es cruzar los datos. Si alguna respuesta de verificación contradice tu borrador, DEBES corregir el error. Si las verificaciones muestran que el borrador incluyó información que no estaba en el contexto (alucinación), omítela. Entrega solo la respuesta final corregida y fluida, sin mencionar el proceso de verificación.
> **User:** [Incluye la pregunta original, el contexto, el borrador y la lista de verificaciones con sus respuestas] \n\n Respuesta Final Corregida:
