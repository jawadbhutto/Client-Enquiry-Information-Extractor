# Client Enquiry Information Extractor

## Project Purpose

This project extracts structured information from free-text client enquiries (e.g. emails or chat messages) using Google's Gemini API. Instead of manually reading through enquiries to identify the client's name, company, industry, requirements, and urgency, this tool parses raw text and returns clean, validated JSON.

It's designed for scenarios like sales/support pipelines where enquiries arrive as unstructured text and need to be triaged, logged, or routed automatically. It also flags which expected fields were missing from the enquiry, so a human can follow up for the missing details.

**Extracted fields:**

* `client\_name`
* `company`
* `industry`
* `requirement` (list)
* `urgency`
* `missing\_information` (list of fields the client didn't provide)

## Installation Steps

1. **Clone the repository**

```bash
   git clone https://github.com/jawadbhutto/Client-Enquiry-Information-Extractor.git

&#x20;  cd Client-Enquiry-Information-Extractor
   ```

2. **Create a virtual environment (recommended)**

```bash
   python -m venv venv
   source venv/bin/activate      # On Windows: venv\\Scripts\\activate
   ```

3. **Install dependencies**

```bash
   pip install -r requirements.txt
   ```

4. **Set up your environment variables**

   * Copy `.env.example` to `.env`
   * Add your actual Gemini API key (see below)

```bash
   cp .env.example .env
   ```

## Required Environment Variables

|Variable|Description|
|-|-|
|`GEMINI\_API\_KEY`|Your API key from [Google AI Studio](https://aistudio.google.com/). Required to authenticate all requests to the Gemini API.|

Your `.env` file should look like:

```
GEMINI\_API\_KEY=your\_actual\_key\_here
```

⚠️ **Never commit your real `.env` file to GitHub.** Only `.env.example` (with a placeholder) should be tracked in version control. Add `.env` to your `.gitignore`.

## How to Run the Application

1. Make sure `test\_inputs.json` is in the project root (a sample enquiry set is included — see below).
2. Open `main.ipynb` in Jupyter:

```bash
   jupyter notebook main.ipynb
   ```

3. Run all cells in order. The notebook will:

   * Load your API key from `.env`
   * Define the extraction schema and prompt logic
   * Run a single test enquiry through `prompt\_passing()`
   * Batch-process every enquiry in `test\_inputs.json` and print the extracted result (or error) for each

To process a single custom enquiry, call the function directly:

```python
result = prompt\_passing("Hi, I'm Jason from GreenTech Energy. We need an AI chatbot.")
print(result)
```

## Example Input and Output

**Input:**

```
Hello, my name is Sarah Johnson, and I work at BrightMart Retail. We are looking for an AI chatbot that can answer customer queries on our website and integrate with Shopify. We'd like to launch it within the next two weeks for our marketing campaign.
```

**Output:**

```json
{
  "client\_name": "Sarah Johnson",
  "company": "BrightMart Retail",
  "industry": "Retail",
  "requirement": \[
    "AI chatbot for customer queries",
    "Shopify integration"
  ],
  "urgency": "Within the next two weeks",
  "missing\_information": \[]
}
```

**Input (incomplete enquiry):**

```
Looking for AI to predict sales.
```

**Output:**

```json
{
  "client\_name": "",
  "company": "",
  "industry": "",
  "requirement": \["AI to predict sales"],
  "urgency": "",
  "missing\_information": \["client\_name", "company", "industry", "urgency"]
}
```

Notice the model never guesses missing values — it leaves them empty and lists them under `missing\_information` instead.

## Sample Test Inputs

`test\_inputs.json` contains a set of sample enquiries covering different scenarios, including:

|ID|Category|Example|
|-|-|-|
|1|Complete Enquiry|Full name, company, requirement, and urgency all stated|
|2|Complete Enquiry|Requirement stated but industry omitted|
|3|Incomplete Enquiry|Name given, but company and industry missing|
|4|Incomplete Enquiry|Company stated, pricing/timeline requested instead of given|
|5|Very Short Enquiry|Single-sentence request with almost no context|
|6|Very Short Enquiry|Single-sentence request, no client/company info|
|7|Missing Name|Company and requirement given, but no personal name|
|8|Missing Company|Name and industry given, but no company name|
|9|Multiple Requirements|Enquiry listing 3 separate requirements|
|10|Multiple Requirements|Enquiry listing 3 separate requirements, different domain|

This mix is intentional — it stress-tests the extractor against complete enquiries, sparse one-liners, and enquiries with multiple bundled requirements, to confirm the schema and "never guess" rule hold up consistently.

## Known Limitations

* **Free-tier rate limits:** The Gemini free tier allows a limited number of requests per day (e.g. 20 requests/day on some models). Batch-processing many enquiries in a short time can trigger `429 RESOURCE\_EXHAUSTED` errors. The code catches this and returns an `api\_failure` error object rather than crashing, but processing will need to be spread out or upgraded to a paid tier for larger batches.
* **No automatic retry/backoff:** When a rate limit or transient API error occurs, the current implementation does not automatically retry after the suggested delay — it simply reports the error. Adding retry logic with exponential backoff would improve reliability for larger batches.
* **Single-language assumption:** The extraction prompt has been tested primarily on English-language enquiries. Behavior on other languages is untested.
* **Ambiguous phrasing:** Enquiries with vague or implicit urgency/industry cues (e.g. "soon," "when possible") may be extracted inconsistently, since the model has to interpret natural language rather than match exact keywords.
* **No persistence layer:** Results are printed to console/notebook output only; there's no built-in step to save results back to a file or database. This would need to be added for production use.
* **Single requirement schema:** The schema is fixed to one enquiry format. Enquiries needing additional fields (e.g. budget, contact email) would require extending the `ClientEnquiry` schema and system prompt.

## Tech Stack

* [google-genai](https://pypi.org/project/google-genai/) — Gemini API SDK
* [Pydantic](https://docs.pydantic.dev/) — schema definition and response validation
* [python-dotenv](https://pypi.org/project/python-dotenv/) — environment variable management
* Model used: `gemini-3-flash-preview`

