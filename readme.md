# Claude-Based Transkribus Correction Script (Single Document Version)

This repository contains a Python script designed to automate the correction of Handwritten Text Recognition (HTR) results in [Transkribus](https://transkribus.ai/). The script processes a single specified document, using Anthropic's Claude 3 Sonnet model to review and correct the transcription line by line.

The core of this script is a "two-call" verification method. For each chunk of text, it calls the Claude API twice to generate two independent transcriptions. It then compares these versions to identify and flag uncertain lines, ensuring a higher degree of confidence in the final output.

This version is specifically designed for ease of use in **Google Colab**.

## ✨ Features

*   **Single Document Processing:** Focuses on correcting one document at a time for simplicity and control.
*   **AI-Powered Correction:** Leverages the advanced multimodal capabilities of Anthropic's Claude 3 models.
*   **Chunk-Based Analysis:** Breaks the document image into smaller, manageable chunks for more accurate, focused analysis by the LLM.
*   **Two-Call Verification:** Calls the API twice for each chunk and compares the results to measure the model's confidence and flag inconsistencies.
*   **Intelligent Image Handling:** Automatically crops, rotates, and annotates image chunks with line numbers to provide the best possible context to the model.
*   **Uncertainty Flagging:** Lines where the two AI transcriptions differ significantly are automatically tagged as `unclear` in the output PAGE XML.
*   **Direct Transkribus Integration:** Writes the final, corrected transcription and metadata directly back to your Transkribus document, creating a new version.
*   **Colab-Ready:** Designed to run out-of-the-box in a Google Colab notebook with minimal setup.

## ⚙️ How It Works

The script follows a linear, step-by-step workflow:

1.  **Authentication:** Securely logs into Transkribus and initializes the Anthropic API client using your credentials.
2.  **Data Fetching:**
    *   Retrieves the page details (page number, image URL) for the specified document.
    *   Downloads the full-resolution page image.
    *   Downloads the latest version of the document's PAGE XML, which contains the HTR text and line coordinates.
3.  **Chunk Processing Loop:** The script iterates through the document's text lines in chunks (e.g., 10 lines at a time). For each chunk, it:
    a. **Crops the Image:** Calculates a bounding box around the lines in the chunk and crops the main image.
    b. **Annotates the Image:** Draws the original line numbers and baselines onto the cropped image chunk.
    c. **Rotates the Image:** Calculates the average text angle and rotates the chunk so the text is horizontal.
    d. **Calls Claude (x2):** Sends the prepared image chunk and the original HTR text to the Claude API twice.
    e. **Parses Results:** Extracts the transcribed text from both of Claude's JSON responses.
4.  **Analysis & Consolidation:**
    *   After processing all chunks, it compares the two full transcriptions (Run A and Run B).
    *   It flags any line where the Character Error Rate (CER) between the two versions is above a set threshold.
    *   For each line, it selects the "best" version by comparing both AI outputs to the original HTR text, choosing the one with the lower CER.
5.  **XML Modification:**
    *   It creates a new in-memory version of the PAGE XML.
    *   It updates each text line with the selected "best" transcription.
    *   It adds the `unclear` tag to lines that were flagged for high CER.
    *   It injects metadata to indicate that the document was processed by this script.
6.  **Final Upload:** It uploads the modified PAGE XML back to Transkribus, creating a new, corrected transcript version.

## 🚀 Getting Started

This script is optimized for Google Colab. Follow these steps to run it.

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
        *   `anthropic_key` (Value: your Anthropic API key, starting with `sk-ant-`)

3.  **Copy the Code:** Copy the entire content of the `correct_single_document.py` script and paste it into a single cell in your notebook.

4.  **Configure the Script:** In the script, locate the **"USER CONFIGURATION"** section and set the following values:
    ```python
    # --- Document to Process ---
    COLLECTION_ID = 1957043  # <<< --- YOUR COLLECTION ID --- >>>
    DOCUMENT_ID = 9310076  # <<< --- THE SPECIFIC DOCUMENT ID YOU WANT TO PROCESS --- >>>
    ```
    You can also adjust other settings like `ANTHROPIC_MODEL_NAME` or `CHUNK_SIZE` if needed.

5.  **Run the Script:** Execute the cell by clicking the play button or pressing `Shift+Enter`.
    *   The first time you run it, it will automatically install the required Python libraries.
    *   After the installation, the main process will start, and you will see verbose output logging every step of the process.

## 🔧 Configuration

All user-configurable variables are located at the top of the script in the "USER CONFIGURATION" section.

| Variable                | Description                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| `TRANKRIBUS_USERNAME`   | Your Transkribus email (handled by Colab Secrets).                                                      |
| `TRANKRIBUS_PASSWORD`   | Your Transkribus password (handled by Colab Secrets).                                                   |
| `ANTHROPIC_API_KEY`     | Your Anthropic API key (handled by Colab Secrets).                                                      |
| `COLLECTION_ID`         | The numerical ID of the Transkribus collection containing your document.                                |
| `DOCUMENT_ID`           | The numerical ID of the specific Transkribus document you want to process.                              |
| `ANTHROPIC_MODEL_NAME`  | The Claude model to use (e.g., `claude-3-sonnet-20240229`).                                              |
| `CHUNK_SIZE`            | The number of lines to process in each API call. `10` is a good default.                                |
| `BBOX_PADDING`          | Pixels of padding to add around text lines when cropping the image.                                     |
| `CER_THRESHOLD`         | The Character Error Rate threshold for flagging a line as `unclear`.                                    |

## 🧠 The System Prompt

The quality of the AI's output is heavily dependent on the `SYSTEM_PROMPT_BASE` variable in the script. This prompt contains detailed instructions on paleography, transcription rules, abbreviation handling, and the required output format.

Feel free to modify this prompt to better suit the specific characteristics of your documents.

## ⚠️ Troubleshooting

*   **`ModuleNotFoundError`**: This means the initial `!pip install` command failed. Check the error log in Colab. It's often due to a typo in a package name.
*   **`FATAL: Could not get page details...`**: This is the most common error.
    1.  **Check your `COLLECTION_ID` and `DOCUMENT_ID`** in the script. Make sure they are correct.
    2.  **Verify Permissions:** Open a web browser and go to `https://transkribus.eu/r/collections/YOUR_COLLECTION_ID/YOUR_DOCUMENT_ID`. If you can't see the document, your user account doesn't have permission.
    3.  **Check for Empty Documents:** The document might exist in Transkribus but have no pages uploaded. The script requires at least one page to process.
*   **`FATAL: Anthropic API Error...`**:
    *   Check that your `anthropic_key` secret in Colab is correct and doesn't have extra spaces.
    *   Ensure your Anthropic account has credits available.
*   **`JSONDecodeError`**: This means Claude's response was not valid JSON. This can happen occasionally. The script is designed to handle minor errors, but if it persists, it might indicate a problem with the prompt or the model's current state.