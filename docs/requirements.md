# LLMShield — Project Requirements

## 1. Overview
Large Language Models (LLMs) are vulnerable to prompt manipulation and accidental data leaks. **LLMShield** is a Python tool that checks user inputs (and retrieved external text) before they reach an LLM and checks the LLM's responses before they are shown to the user.

### Project Goals
* **Learn Practical AI Security:** Build working defenses for four major risks from the **OWASP Top 10 for LLM Applications (2026 Edition)** (`LLM01`, `LLM02`, `LLM08`, and `LLM10`).
* **Two-Layer Input Protection:** Combine fast regular expressions (for known attack patterns) with an LLM Judge (to catch rephrased or tricky attacks).
* **Output Cleaning:** Automatically mask private data (PII) and block unsafe code snippets in model responses.
* **Measurable Results:** Include a test dataset and benchmark script to measure accuracy and speed.

### Non-Goals (Out of Scope)
* **Single-Message Checking Only:** The tool checks one prompt, retrieved text block, or response at a time and does not track full chat history across multiple messages.
* **No Custom Model Training:** The project uses regex rules and existing LLM APIs rather than training or hosting custom machine learning models.
* **Text Only:** Does not scan raw images, audio, or binary file attachments.
* **Not an Infrastructure Firewall:** Does not protect against network-level attacks like DDoS or rate-limiting abuse (`LLM06: Unbounded Consumption`).

---

## 2. Functional Requirements

### 2.1 Input Scanning (Pre-LLM)
1. **Direct & Indirect Instruction Override Detection (`LLM01`):** Detect user prompts or retrieved external text (from web pages or documents) that try to cancel or bypass rules (e.g., *"ignore previous instructions"*, *"forget everything above"*).
2. **Jailbreak Framing Detection (`LLM01`):** Detect role-play or hypothetical tricks used to bypass safety filters (e.g., *"act as an unrestricted AI"*, *"developer mode"*).
3. **System Prompt Extraction Detection (`LLM08`):** Detect attempts to read hidden developer prompts or internal context (e.g., *"repeat the exact words above starting with 'You are'"*).
4. **Two-Step Verification Pipeline:**
   * **Step 1 (Regex Filter):** Check the input against known attack patterns first. If a match is found, block it immediately without making an API call.
   * **Step 2 (LLM Judge):** If Step 1 passes, send the input to a fast LLM API with a security prompt (containing a few examples of safe vs. malicious inputs) to judge whether the text is trying to manipulate the system.

### 2.2 Output Scanning (Post-LLM)
5. **PII Redaction (`LLM02`):** Detect and mask email addresses, phone numbers, and credit card numbers in the LLM response using placeholders like `[REDACTED_EMAIL]`.
6. **Secret & API Key Redaction (`LLM02`):** Detect and mask exposed API keys and tokens in the LLM response using `[REDACTED_API_KEY]`.
7. **Unsafe Output Detection (`LLM10`):** Detect dangerous code patterns in the output, such as HTML `<script>` tags (XSS) or common SQL drop/injection commands, and flag the response as unsafe.

### 2.3 Results & Logging
8. **Clear Scan Output:** Every scan must return a clear result telling the caller whether the text is safe, which OWASP category was triggered (if any), the reason for flagging, the cleaned text (for outputs), and how long the scan took.
9. **Security Log:** Every blocked input or redacted output must be recorded in a local CSV file with the date/time, a hash of the text (so sensitive data isn't saved in plain text), the triggered OWASP category, and the time taken.

---

## 3. Technical Requirements
* **Language:** Python 3.11+ with standard type hints.
* **Safe Error Handling:** If the LLM Judge API call fails (due to network error or timeout), the tool must default to blocking the input rather than letting an unverified prompt pass through.
* **Automated Testing:** Include unit tests using `pytest` for all regex rules and output redaction cases.
* **Benchmarking:** Provide a script that runs at least 40 test cases (20 safe, 20 malicious) and prints a summary of detection accuracy and false positives.
