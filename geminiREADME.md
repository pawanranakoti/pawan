# 🛡️ AI-Powered Semantic Anomaly Detector

**Submission for:** [Code with Gemini Hackathon](https://code-with-gemini.devpost.com/)  
**Developed by:** Pawan Ranakoti  
**Tech Stack:** Python, Google Gemini 2.5 Flash, Google GenAI SDK

## 🚀 Overview
Standard data validators only check for data types (e.g., is "Age" an integer?). This tool goes deeper by using **LLM-based Semantic Reasoning** to find logical contradictions that are technically valid but practically impossible.

### **Key Features**
- **Semantic Audit:** Detects gender-diagnosis mismatches, lethal drug combinations, and temporal anomalies.
- **Fault-Tolerant:** Implements **Exponential Backoff** to handle 503 (Server Busy) and 429 (Rate Limit) errors.
- **Batch Processing:** Handles multi-line CSV/Text datasets in a single audit pass.
- **Domain Agnostic:** Easily adapts to Healthcare, Automotive, or Finance data.

## 🛠️ Setup Instructions
1. **API Key:** Generate a key from [Google AI Studio](https://aistudio.google.com/).
2. **Environment:**  `GEMINI_API_KEY`.
3. **Installation:**
   ```bash
   pip install -U google-genai
   ---

### **2. The Final Python Code **


```python
# 1. Installation
!pip install -q -U google-genai --no-deps

import os
import time
import random
from google import genai
from google.genai import types
from kaggle_secrets import UserSecretsClient

# 2. Secure Client Setup
try:
    user_secrets = UserSecretsClient()
    api_key = user_secrets.get_secret("GEMINI_API_KEY")
    client = genai.Client(api_key=api_key)
    print("✅ System Online: AI Semantic Auditor Active")
    print("="*50)
except Exception as e:
    print(f"❌ Connection Failed: {e}")

# 3. System Instructions
sys_instruct = """
You are a Professional Data Auditor. 
Perform 'Semantic Reasoning' on the provided dataset to find logical impossibilities.
- Check for medical contradictions (e.g., Male Pregnancy, lethal drug doses).
- Check for automotive/logistics errors (e.g., Diesel Electric cars, Future years).
Format the report as a clean Markdown Table. If clean, say: '✅ No anomalies detected.'
"""

# 4. Model Setup
MODEL_ID = 'gemini-1.5-flash' 
chat = client.chats.create(
    model=MODEL_ID,
    config=types.GenerateContentConfig(
        system_instruction=sys_instruct,
        temperature=0.0
    )
)

# 5. Smart Execution with Retry Logic (Exponential Backoff)
def execute_with_retry(data, max_retries=5):
    for i in range(max_retries):
        try:
            return chat.send_message(data)
        except Exception as e:
            if "503" in str(e) or "429" in str(e):
                wait = (2 ** i) + random.random()
                print(f"⚠️ Server busy. Retrying in {round(wait, 2)}s...")
                time.sleep(wait)
            else:
                raise e
    return None

# 6. Interactive Input
print("👨‍⚕️ Welcome to the AI powered semantic anomaly detector.")
print("👉 Paste your dataset below. Type 'DONE' on a new line to start.")
print("-" * 50)

dataset_lines = []
while True:
    line = input()
    if line.strip().upper() == 'DONE':
        break
    dataset_lines.append(line)

batch_data = "\n".join(dataset_lines)

if batch_data.strip():
    print(f"\n🤖 Auditing {len(dataset_lines)} rows... Please wait...")
    try:
        response = execute_with_retry(batch_data)
        if response:
            print("\n🚨 AI BATCH AUDIT REPORT 🚨\n")
            print(response.text)
        else:
            print("❌ Audit failed: Server Unavailable.")
    except Exception as e:
        print(f"❌ Error: {e}")
else:
    print("⚠️ No data provided.") 
Patient_ID, Age, Gender, Diagnosis, Prescribed_Medicine, Heart_Rate_BPM
P-001, 45, Male, Type 2 Diabetes, Metformin, 72
P-002, 12, Female, Routine Checkup, Vitamins, 80
P-003, 35, Male, Pregnancy Complications, Folic Acid, 75
P-004, 145, Female, Asthma, Albuterol Inhaler, 65
P-005, 50, Male, Severe Bleeding, Blood Thinners (Aspirin), 90
P-006, 28, Male, Broken Leg, Throat Lozenges, 60
DONE
