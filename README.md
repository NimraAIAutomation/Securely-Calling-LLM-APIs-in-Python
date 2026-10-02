# Securely Calling LLM APIs in Python

A beginner-friendly Python project demonstrating how to securely connect
to an LLM API using **Groq**, load API credentials from environment
variables, process model responses, and handle common API errors.

## Project Overview

This project uses the OpenAI Python SDK with Groq's OpenAI-compatible
API endpoint. It includes:

-   Secure API-key loading with `python-dotenv`
-   A reusable `ask_llm()` function
-   LLM response handling
-   Rate-limit handling with a limited retry loop
-   Authentication and general error handling
-   Healthcare-focused example prompts

## Tech Stack

-   Python
-   Jupyter Notebook
-   Groq API
-   OpenAI Python SDK
-   `python-dotenv`

## Project Structure

``` text
llm-api-project/
├── .env
├── .gitignore
├── requirements.txt
├── main.ipynb
└── README.md
```

## Setup

### 1. Clone the repository

``` bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
cd YOUR-REPOSITORY
```

Replace the URL with your repository's URL.

### 2. Create and activate a virtual environment

``` bash
python -m venv venv
```

On Windows:

``` bash
venv\Scripts\activate
```

On macOS or Linux:

``` bash
source venv/bin/activate
```

### 3. Install dependencies

``` bash
pip install -r requirements.txt
```

### 4. Configure your API key

Create a `.env` file in the project directory:

``` env
GROQ_API_KEY=your_groq_api_key
```

Get an API key from the Groq developer console. Never commit your actual
API key to GitHub.

Make sure `.gitignore` contains:

``` gitignore
.env
venv/
__pycache__/
.ipynb_checkpoints/
```

### 5. Run the notebook

Start Jupyter:

``` bash
jupyter notebook
```

Open `main.ipynb` and run the cells in order.

## How It Works

### Load the API key

``` python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv("GROQ_API_KEY")
```

### Create the client

``` python
from openai import OpenAI

client = OpenAI(
    api_key=api_key,
    base_url="https://api.groq.com/openai/v1"
)
```

### Make an API request

``` python
response = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {
            "role": "user",
            "content": "Explain hypertension in simple terms for a patient."
        }
    ]
)

print(response.choices[0].message.content)
```

The model name above is the one used during this project. Model
availability can vary by account and over time.

### Reusable function with error handling

``` python
import time
from openai import RateLimitError, AuthenticationError

def ask_llm(prompt, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="openai/gpt-oss-20b",
                messages=[
                    {
                        "role": "user",
                        "content": prompt
                    }
                ]
            )

            return response.choices[0].message.content

        except RateLimitError:
            print(f"Rate limit reached. Attempt {attempt + 1}/{max_retries}")

            if attempt < max_retries - 1:
                time.sleep(5)
            else:
                print("Maximum retries reached.")
                return None

        except AuthenticationError:
            print("Authentication failed. Check your API key.")
            return None

        except Exception as e:
            print("API request failed:", e)
            return None
```

Example:

``` python
result = ask_llm(
    "Explain why regular blood pressure monitoring is important."
)
print(result)
```

The retry logic is demonstrated in code, but a rate-limit response was
not intentionally triggered during testing.

## What I Learned

-   How to keep API credentials out of source code
-   How environment variables work in Python
-   How to configure an OpenAI-compatible client for Groq
-   How to send prompts and extract generated text
-   How to inspect available API models
-   How to handle authentication, rate-limit, and other request errors
-   How to organize repeated API logic into a reusable function

## Security Notes

-   Keep `.env` out of version control.
-   Never print, publish, or share your API key.
-   If a key is accidentally exposed, revoke it and create a new one.
-   Use a finite retry limit to avoid repeated requests.
-   Do not treat generated healthcare information as a diagnosis or a
    substitute for professional medical advice.

## Future Improvements

-   Add exponential backoff and respect retry timing information
    returned by the API.
-   Handle connection errors and timeouts separately.
-   Validate empty prompts and missing API keys.
-   Add logging instead of relying only on `print()`.
-   Move API configuration into a separate module for larger projects.

------------------------------------------------------------------------

**Learning project:** Securely Calling LLM APIs in Python\
**Provider:** Groq\
**Interface:** OpenAI-compatible Python SDK
