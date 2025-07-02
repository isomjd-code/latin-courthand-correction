# AI-Powered Tools for Transkribus Medieval Legal Documents

Developed by Joshua David Isom

This repository contains a suite of Python scripts designed to automate the processing of medieval legal documents from [Transkribus](https://transkribus.ai/). The tools leverage Anthropic's Claude 3.7 Sonnet model to perform advanced text correction and data extraction tasks.

The repository includes two primary tools, each designed for a specific purpose and intended for use in **Google Colab**:

1.  **`latin_courthand_correction.ipynb`**: A script for correcting Handwritten Text Recognition (HTR) results. It reviews a document line-by-line, corrects the transcription, and uploads the improved version back to Transkribus.
2.  **`latin_case_extractor.ipynb`**: A script for analyzing a corrected transcription. It segments the text into individual legal cases and, for each case, extracts structured metadata, expands abbreviated Latin, and provides an English translation.

## ✨ Features

### 1. HTR Correction (`latin_courthand_correction.ipynb`)

*   **AI-Powered Correction:** Leverages the advanced multimodal capabilities of Anthropic's Claude models to read and transcribe from the document image.
*   **Chunk-Based Analysis:** Breaks the document image into smaller, manageable chunks for more accurate, focused analysis by the LLM.
*   **Two-Call Verification:** Calls the API twice for each chunk and compares the results to measure the model's confidence and flag inconsistencies.
*   **Intelligent Image Handling:** Automatically crops, rotates, and annotates image chunks with line numbers to provide the best possible context to the model.
*   **Uncertainty Flagging:** Lines where the two AI transcriptions differ significantly are automatically tagged as `unclear` in the output PAGE XML.
*   **Direct Transkribus Integration:** Writes the final, corrected transcription and metadata directly back to your Transkribus document, creating a new version.

### 2. Case Extraction & Analysis (`latin_case_extractor.ipynb`)

*   **AI-Powered Case Segmentation:** Intelligently identifies the boundaries of individual legal cases within a continuous text transcription.
*   **Structured Metadata Extraction:** For each case, it extracts key details like plaintiffs, defendants, plea type, county, and monetary values.
*   **Latin Expansion:** Automatically expands common medieval Latin abbreviations into their full forms (e.g., `p'd'co` -> `predicto`).
*   **English Translation:** Provides a clear, modern English translation of the expanded Latin text.
*   **Flexible Output:** Formats the final, structured data into your choice of **JSON**, **Wiki Markup**, or **Markdown**.

## ⚙️ How It Works

### Workflow 1: HTR Correction

The `latin_courthand_correction.ipynb` script follows a linear, step-by-step workflow:

1.  **Authentication:** Securely logs into Transkribus and initializes the Anthropic API client.
2.  **Data Fetching:** Downloads the page image and the latest PAGE XML (containing HTR text and line coordinates) for the specified document.
3.  **Chunk Processing Loop:** Iterates through the document's text lines in chunks. For each chunk, it:
    a. **Prepares the Image:** Crops, annotates with line numbers, and rotates the image chunk for optimal AI analysis.
    b. **Calls Claude (x2):** Sends the prepared image chunk and HTR text to the Claude API twice to get two independent transcriptions.
4.  **Analysis & Consolidation:**
    *   Compares the two full transcriptions (Run A and Run B) to identify and flag uncertain lines.
    *   Selects the "best" version for each line by comparing both AI outputs to the original HTR text.
5.  **XML Modification & Upload:** Updates the PAGE XML with the corrected text and `unclear` tags, then uploads it back to Transkribus as a new version.

### Workflow 2: Case Extraction

The `latin_case_extractor.ipynb` script is designed to be run *after* you have a clean, corrected transcription (ideally from the first script).

1.  **Authentication & Data Fetching:** Logs into Transkribus and downloads the latest PAGE XML for the specified document.
2.  **Case Segmentation:** Sends the entire page's text to Claude with a prompt asking it to identify the start and end lines of each legal case.
3.  **Case Processing Loop:** For each identified case, it:
    a. **Collates Text:** Gathers the text lines belonging to that case.
    b. **Calls Claude:** Sends the case text to the Claude API with a prompt asking for metadata extraction, Latin expansion, and English translation.
4.  **Formatting Output:** Gathers the structured data from all cases and formats it into the user-specified output format (JSON, Wiki, or Markdown) before printing it to the console.

## 🚀 Getting Started

Both scripts are optimized for Google Colab. Follow these steps to run them.

### Prerequisites

*   A **Transkribus account** with access to the collection and document you want to process.
*   An **Anthropic API key**.
*   A **Google account** to use Google Colab.

### Setup and Usage

1.  **Open Google Colab:** Go to [colab.research.google.com](https://colab.research.google.com) and create a new notebook.

2.  **Set Up Secrets (Highly Recommended):** For security, do not paste your credentials directly into the code.
    *   In your Colab notebook, click the **🔑 key icon** in the left sidebar.
    *   Add the following three secrets:
        *   `TRANKRIBUS_USER` (Value: your Transkribus email)
        *   `TRANKRIBUS_PASSWORD` (Value: your Transkribus password)
        *   `ANTHROPIC_API_KEY` (Value: your Anthropic API key, starting with `sk-ant-`)

3.  **Copy the Code:**
    *   For HTR correction, copy the entire content of `latin_courthand_correction.ipynb`.
    *   For case extraction, copy the entire content of `latin_case_extractor.ipynb`.
    *   Paste the chosen script's code into a single cell in your notebook.

4.  **Configure the Script:** In the script, locate the **"USER CONFIGURATION"** section and set the required values.
    *   **For both scripts:**
        ```python
        # --- Document to Process ---
        COLLECTION_ID = 2093054  # <<< --- YOUR COLLECTION ID --- >>>
        DOCUMENT_ID = 9343917    # <<< --- THE SPECIFIC DOCUMENT ID YOU WANT TO PROCESS --- >>>
        ```
    *   **For `latin_case_extractor.ipynb` only:**
        ```python
        # --- Output Configuration ---
        OUTPUT_FORMAT = 'wiki'  # Choose 'json', 'wiki', or 'markdown'
        ```

5.  **Run the Script:** Execute the cell by clicking the play button or pressing `Shift+Enter`.
    *   The first time you run it, it will automatically install the required Python libraries.
    *   You will see verbose output logging every step of the process.

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
