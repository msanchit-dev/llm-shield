# LLMShield: Security Filter for LLM Applications

![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![OWASP LLM Top 10](https://img.shields.io/badge/OWASP-LLM%20Top%2010%20(2026)-red.svg)

**LLMShield** is a pluggable Python security filter built to protect Large Language Model (LLM) applications from common attacks and accidental data leaks.

When building apps with LLMs, users can trick the model into ignoring its rules, or the model might accidentally leak private data. LLMShield sits between the user and the LLM to screen both incoming prompts and outgoing responses based on the **OWASP Top 10 for LLM Applications (2026 Edition)**:

* **LLM01 (Prompt Injection):** Blocks attempts to override system rules or jailbreak the model.
* **LLM02 (Sensitive Information Disclosure):** Finds and masks private data (emails, phone numbers, API keys, credit cards) in both incoming prompts and model outputs.
* **LLM08 (Hidden Context Exposure / System Prompt Leakage):** Blocks prompts that try to extract hidden developer instructions and prevents the model from leaking them in responses.
* **LLM10 (Improper Output Handling):** Detects dangerous code patterns (like XSS `<script>` tags or SQL injection commands) in the model's output before they reach downstream systems.
