# AI-Powered Tools for Transkribus Medieval Legal Documents

Developed by Joshua David Isom

This repository contains a suite of Python scripts and a recommended workflow designed to process medieval latin legal documents (CP40 and KB27) within [Transkribus](https://transkribus.ai/). The tools leverage a state-of-the-art Handwritten Text Recognition (HTR) model and Anthropic's Claude 3.7 Sonnet to move from raw manuscript image to fully analyzed, structured data.

This guide outlines a complete, three-step workflow:
1.  **Initial Transcription:** Use a specialized, public HTR model in Transkribus to generate a highly accurate baseline transcription.
2.  **Transcription Refinement:** Use the `latin_courthand_correction.ipynb` script to perform a final, AI-powered polishing pass on the HTR output.
3.  **Case Extraction & Analysis:** Use the `latin_case_extractor.ipynb` script on the polished text to segment it into legal cases, extract metadata, and generate translations.

All scripts are designed for easy use in **Google Colab**.

## ✨ Features

### 1. HTR Correction Script (`latin_courthand_correction.ipynb`)
*   **AI-Powered Correction:** Leverages the advanced multimodal capabilities of Anthropic's Claude models to read and correct transcriptions from the document image.
*   **Two-Call Verification:** Calls the API twice for each text segment and compares the results to measure the model's confidence and flag inconsistencies.
*   **Intelligent Image Handling:** Automatically crops, rotates, and annotates image chunks to provide the best possible context to the model.
*   **Direct Transkribus Integration:** Writes the final, corrected transcription and metadata directly back to your Transkribus document, creating a new version.

### 2. Case Extraction & Analysis Script (`latin_case_extractor.ipynb`)
*   **AI-Powered Case Segmentation:** Intelligently identifies the boundaries of individual legal cases within a continuous text transcription.
*   **Structured Metadata Extraction:** For each case, it extracts key details like plaintiffs, defendants, plea type, county, and monetary values.
*   **Latin Expansion:** Automatically expands common medieval Latin abbreviations into their full forms (e.g., `p'd'co` -> `predicto`).
*   **English Translation:** Provides a clear, modern English translation of the expanded Latin text.
*   **Flexible Output:** Formats the final, structured data into your choice of **JSON**, **Wiki Markup**, or **Markdown**.

## ⚙️ The Recommended Workflow: From Image to Analysis

For the best results, follow this three-step process.

### Step 1: Generate a High-Quality Baseline with the HTR Model

Before using the Colab scripts, the recommended first step is to run your documents through the specialized public HTR model. This provides a fast, stable, and highly accurate baseline transcription directly within Transkribus.

*   **Model Name:** `Latin Court Hand - KB27/795 (1460)`
*   **How to Use:** In your Transkribus project, select the pages you want to transcribe and run the "Recognize" tool. Search for the model by name in the public models list.
*   **Link to Model:** **[https://app.transkribus.org/models/public/text/latin-court-hand-kb27795-1460](https://app.transkribus.org/models/public/text/latin-court-hand-kb27795-1460)**

This step gives you a strong foundation and is more reproducible and cost-effective for large-scale transcription than using a live LLM for the initial pass.

### Step 2: Refine the Transcription with LLM Correction (`latin_courthand_correction.ipynb`)

Once you have the baseline transcription from the HTR model, use this script for a final polishing pass to achieve near-perfect accuracy.

*   **Purpose:** To correct any residual errors from the HTR model and produce a publication-quality text.
*   **Process:** The script takes the HTR output, compares it against the manuscript image chunk-by-chunk using a live LLM, and uploads the corrected version back to Transkribus.

### Step 3: Extract and Analyze Cases (`latin_case_extractor.ipynb`)

With a fully corrected and polished transcription, this final script allows you to perform high-level analysis.

*   **Purpose:** To transform the flat text into structured, analyzable data.
*   **Process:** The script reads the corrected text, uses an LLM to identify case boundaries, and then extracts metadata, expands Latin, and translates each case. The results are printed in your chosen format.

## 🚀 Getting Started

### Prerequisites

*   A **Transkribus account** with access to the collection and document you want to process.
*   An **Anthropic API key**.
*   A **Google account** to use Google Colab.

### Setup and Usage

1.  **Initial Transcription (in Transkribus):**
    *   Open your project in Transkribus.
    *   Select your pages and run the **`Latin Court Hand - KB27/795 (1460)`** HTR model on them.

2.  **Set Up Colab Secrets:**
    *   In a new Colab notebook, click the **🔑 key icon** in the left sidebar.
    *   Add the following three secrets:
        *   `TRANKRIBUS_USER` (Value: your Transkribus email)
        *   `TRANKRIBUS_PASSWORD` (Value: your Transkribus password)
        *   `ANTHROPIC_API_KEY` (Value: your Anthropic API key)

3.  **Run the Correction Script (Optional but Recommended):**
    *   Copy the code from `latin_courthand_correction.ipynb` into a Colab cell.
    *   Configure the `COLLECTION_ID` and `DOCUMENT_ID`.
    *   Run the cell to polish the HTR transcription.

4.  **Run the Extraction Script:**
    *   Copy the code from `latin_case_extractor.ipynb` into a new Colab cell (or the same notebook).
    *   Configure the `COLLECTION_ID`, `DOCUMENT_ID`, and desired `OUTPUT_FORMAT`.
    *   Run the cell to analyze the final text and get your structured output.

## 🔧 Configuration

All user-configurable variables are located at the top of each script in the "USER CONFIGURATION" section. Key variables include `COLLECTION_ID`, `DOCUMENT_ID`, `ANTHROPIC_MODEL_NAME`, and `OUTPUT_FORMAT` (for the extractor).

## 🧠 The System Prompts

The quality of the AI's output is heavily dependent on the `SYSTEM_PROMPT` variables in the scripts. These prompts contain detailed instructions on paleography, transcription rules, data extraction formats, and more. Feel free to modify these prompts to better suit the specific characteristics of your documents.

## ⚠️ Troubleshooting

*   **`ModuleNotFoundError`**: This means the initial `!pip install` command failed. Check the error log in Colab.
*   **`FATAL: Could not get page number...` or `FATAL: Could not get page details...`**: This is the most common error.
    1.  **Check your `COLLECTION_ID` and `DOCUMENT_ID`** in the script. Make sure they are correct.
    2.  **Verify Permissions:** Open a web browser and go to `https://transkribus.eu/r/collections/YOUR_COLLECTION_ID/YOUR_DOCUMENT_ID`. If you can't see the document, your user account doesn't have permission.
    3.  **Check for Empty Documents:** The document might exist in Transkribus but have no pages uploaded. The scripts require at least one page to process.
*   **`FATAL: Anthropic API Error...`**:
    *   Check that your `ANTHROPIC_API_KEY` secret in Colab is correct and doesn't have extra spaces.
    *   Ensure your Anthropic account has credits available.
*   **`JSONDecodeError`**: This means Claude's response was not valid JSON. This can happen occasionally. The scripts are designed to handle minor errors, but if it persists, it might indicate a problem with the prompt or the model's current state.
