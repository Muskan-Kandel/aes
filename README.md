# Automated Essay Scoring with Feedback Generation (AES)

## Project Overview
This project focuses on transitioning Automated Essay Scoring (AES) from a black-box numerical grading system into a transparent, formative tool. While traditional systems provide scores without explanation, our framework integrates Chain-of-Thought (CoT) reasoning to provide qualitative feedback alongside quantitative scores. By using a Joint Task Model, we aim to improve both grading accuracy and the educational utility of the feedback.

## Core Objectives
- **Interpretability:** Addressing the transparency gap by forcing the model to provide a rationale before assigning a grade.
- **Integrated Assessment:** Moving beyond standalone numerical regression to capture pedagogical nuances.
- **Formative Support:** Shifting focus from purely summative scoring to instructional feedback that helps students improve.

## Workflow

### 1. Preprocessing
- **Text Normalization:** Removing HTML tags, artifacts, and noise from the ASAP++ dataset.
- **Sub-word Tokenization:** Utilizing algorithms like WordPiece or Byte-Pair Encoding (BPE) to handle rare words and prevent out-of-vocabulary errors.
- **Rubric Integration:** Appending specific educational criteria to the input to ensure feedback is rubric-grounded.

### 2. Reasoning Phase
- **Chain-of-Thought (CoT) Prompting:** The model is prompted to analyze the essay based on rubric traits and generate a qualitative critique first.
- **Grounded Scoring:** The final grade is mathematically grounded in the linguistic evidence identified during the initial critique phase.

### 3. Optimization
- **Multi-Task Learning (MTL):** Simultaneous training on multiple writing dimensions, including Content, Language, Prompt Adherence, and Narrativity, to enhance internal quality representation.
- **Weighted Loss Function:** Combining Generative Loss (Cross-Entropy) for feedback quality and Regression Loss (Mean Squared Error) for score accuracy.
