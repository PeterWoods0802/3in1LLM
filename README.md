# 3in1LLM
This is a template to help users get responses from multiple LLMS by asking the question one time.

# LLM Response Combiner

Combine answers from ChatGPT, Perplexity, and DeepSeek in a single response!  
Ask your question, and get a more well-rounded answer from multiple Large Language Models.

## Features

- Queries ChatGPT (OpenAI), Perplexity, and DeepSeek at once
- Combines the responses into a single, easy-to-read answer
- Command-line tool for quick use
- Extendable for more LLM APIs in the future

## Requirements

- Python 3.8+
- API keys for:
  - [OpenAI (ChatGPT)](https://platform.openai.com/account/api-keys)
  - [Perplexity](https://docs.perplexity.ai/docs/api-overview)
  - [DeepSeek](https://platform.deepseek.com/api-keys)

## Installation

Clone the repo and install dependencies:

```bash
git clone https://github.com/PeterWoods0802/3in1LLM.git
cd 3in1LLM.git
pip install -r requirements.txt
````

## Setup

Set your API keys as environment variables (recommended):

```bash
export OPENAI_API_KEY=your-openai-key
export PERPLEXITY_API_KEY=your-perplexity-key
export DEEPSEEK_API_KEY=your-deepseek-key
```

Or, enter them when prompted the first time you run the script.

## Usage

```bash
python llm_combiner.py "Your question here"
```

Example:

```bash
python llm_combiner.py "How does a transformer neural network work?"
```

## Output

You'll get a single combined answer, with responses from each LLM separated and labeled.

---

### Notes

* Make sure your API keys are valid.
* For heavy use, mind your API rate limits and usage costs.

## License

MIT License

## Credits

* [OpenAI](https://openai.com/)
* [Perplexity](https://www.perplexity.ai/)
* [DeepSeek](https://www.deepseek.com/)

````

---

## llm_combiner.py

```python
import os
import sys
import requests
import json
from typing import Dict

def get_api_keys():
    # Try to get from environment, otherwise ask for input
    openai = os.environ.get("OPENAI_API_KEY") or input("Enter OpenAI API Key: ")
    perplexity = os.environ.get("PERPLEXITY_API_KEY") or input("Enter Perplexity API Key: ")
    deepseek = os.environ.get("DEEPSEEK_API_KEY") or input("Enter DeepSeek API Key: ")
    return openai.strip(), perplexity.strip(), deepseek.strip()

def query_chatgpt(api_key: str, prompt: str) -> str:
    url = "https://api.openai.com/v1/chat/completions"
    headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}
    payload = {
        "model": "gpt-3.5-turbo",
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 600,
        "temperature": 0.7,
    }
    resp = requests.post(url, headers=headers, json=payload)
    resp.raise_for_status()
    return resp.json()['choices'][0]['message']['content'].strip()

def query_perplexity(api_key: str, prompt: str) -> str:
    url = "https://api.perplexity.ai/chat/completions"
    headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}
    payload = {
        "model": "pplx-70b-online",
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 600,
        "temperature": 0.7,
    }
    resp = requests.post(url, headers=headers, json=payload)
    resp.raise_for_status()
    return resp.json()['choices'][0]['message']['content'].strip()

def query_deepseek(api_key: str, prompt: str) -> str:
    url = "https://api.deepseek.com/v1/chat/completions"
    headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}
    payload = {
        "model": "deepseek-chat",
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 600,
        "temperature": 0.7,
    }
    resp = requests.post(url, headers=headers, json=payload)
    resp.raise_for_status()
    return resp.json()['choices'][0]['message']['content'].strip()

def main():
    if len(sys.argv) < 2:
        print("Usage: python llm_combiner.py \"Your question here\"")
        sys.exit(1)

    prompt = sys.argv[1]
    openai_key, perplexity_key, deepseek_key = get_api_keys()

    results: Dict[str, str] = {}
    print("Querying ChatGPT...")
    try:
        results['ChatGPT'] = query_chatgpt(openai_key, prompt)
    except Exception as e:
        results['ChatGPT'] = f"Error: {e}"

    print("Querying Perplexity...")
    try:
        results['Perplexity'] = query_perplexity(perplexity_key, prompt)
    except Exception as e:
        results['Perplexity'] = f"Error: {e}"

    print("Querying DeepSeek...")
    try:
        results['DeepSeek'] = query_deepseek(deepseek_key, prompt)
    except Exception as e:
        results['DeepSeek'] = f"Error: {e}"

    print("\n" + "="*50)
    print("Combined LLM Response")
    print("="*50)
    for name, answer in results.items():
        print(f"\n{name}:\n{'-'*len(name)}\n{answer}\n")
    print("="*50)

if __name__ == "__main__":
    main()


