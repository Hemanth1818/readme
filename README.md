<div align="center">

# AI Data Extraction Dashboard

<p align="center">
  <img src="/api/placeholder/1200/300" alt="AI Data Extraction Dashboard Banner"/>
</p>

[![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=Streamlit&logoColor=white)](https://streamlit.io/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Google Sheets](https://img.shields.io/badge/Google%20Sheets-34A853?style=for-the-badge&logo=google-sheets&logoColor=white)](https://www.google.com/sheets/about/)
[![OpenAI](https://img.shields.io/badge/Groq-412991?style=for-the-badge&logo=openai&logoColor=white)](https://groq.com/)

Automated data extraction powered by AI - From spreadsheets to intelligent insights in seconds.

[View Demo](https://your-demo-link.com) · [Report Bug](issues/new) · [Request Feature](issues/new)

</div>

## 🚀 Features

- **Multi-Source Data Input**: CSV & Google Sheets integration
- **AI-Powered Search**: Automated web scraping with SerpAPI
- **Smart Processing**: LLM-based data extraction via Groq
- **Export Ready**: Download results or sync with Google Sheets

## ⚡ Quick Start

```bash
# Clone repo
git clone https://github.com/yourusername/ai-data-extraction-dashboard.git

# Install dependencies
pip install -r requirements.txt

# Run dashboard
streamlit run app.py
```

## 🛠️ Tech Stack

| Category | Technologies |
|----------|-------------|
| Frontend | ![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=Streamlit&logoColor=white) |
| Backend | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) |
| Data Processing | ![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white) |
| APIs | ![Google Sheets](https://img.shields.io/badge/Google%20Sheets-34A853?style=flat-square&logo=google-sheets&logoColor=white) ![Groq](https://img.shields.io/badge/Groq-412991?style=flat-square&logo=openai&logoColor=white) ![SerpAPI](https://img.shields.io/badge/SerpAPI-000000?style=flat-square&logo=serpapi&logoColor=white) |

## 💻 Code Examples

### 1. Data Loading and Validation

```python
# Example of loading and validating data with error handling
def load_data(source: str, file_path: str) -> pd.DataFrame:
    try:
        if source == "csv":
            df = pd.read_csv(file_path)
        else:
            df = load_from_gsheet(file_path)
            
        # Validate required columns
        required_cols = ["entity_name", "category"]
        missing_cols = [col for col in required_cols if col not in df.columns]
        
        if missing_cols:
            raise ValueError(f"Missing required columns: {missing_cols}")
            
        return df
    except Exception as e:
        st.error(f"Error loading data: {str(e)}")
        return None

# Usage
df = load_data("csv", "companies.csv")
```

### 2. Smart Web Search Implementation

```python
# Rate-limited web search with caching
class SmartWebSearcher:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.cache = {}
        self.rate_limit = 1.0  # seconds
        self.last_request = 0

    def _respect_rate_limit(self):
        elapsed = time.time() - self.last_request
        if elapsed < self.rate_limit:
            time.sleep(self.rate_limit - elapsed)
        self.last_request = time.time()

    async def search_batch(self, queries: List[str]) -> Dict[str, Any]:
        results = {}
        for query in queries:
            if query in self.cache:
                results[query] = self.cache[query]
                continue
                
            self._respect_rate_limit()
            response = await self._make_request(query)
            self.cache[query] = response
            results[query] = response
            
        return results

# Usage
searcher = SmartWebSearcher(SERPAPI_KEY)
results = await searcher.search_batch(["company A", "company B"])
```

### 3. AI Processing with Context Management

```python
# Advanced LLM processing with context window management
class AdvancedGroqProcessor:
    def __init__(self, api_key: str, max_tokens: int = 8192):
        self.api_key = api_key
        self.max_tokens = max_tokens

    def _chunk_context(self, context: str, chunk_size: int = 1000) -> List[str]:
        words = context.split()
        chunks = []
        current_chunk = []
        current_length = 0
        
        for word in words:
            if current_length + len(word) > chunk_size:
                chunks.append(" ".join(current_chunk))
                current_chunk = [word]
                current_length = len(word)
            else:
                current_chunk.append(word)
                current_length += len(word)
                
        if current_chunk:
            chunks.append(" ".join(current_chunk))
        return chunks

    async def process_with_context(self, query: str, context: str) -> str:
        chunks = self._chunk_context(context)
        
        messages = [
            {"role": "system", "content": "Extract relevant information from the context."},
            {"role": "user", "content": f"Query: {query}\nContext: {chunks[0]}"}
        ]
        
        try:
            response = await self._call_groq_api(messages)
            return response.get("choices", [{}])[0].get("message", {}).get("content", "")
        except Exception as e:
            return f"Error processing query: {str(e)}"

# Usage
processor = AdvancedGroqProcessor(GROQ_API_KEY)
result = await processor.process_with_context(
    "What is the company's main product?",
    context_text
)
```

### 4. Streamlit UI Components

```python
# Interactive dashboard components
def create_search_interface():
    st.sidebar.header("Search Configuration")
    
    # Dynamic query builder
    base_query = st.text_input("Base Query Template:", 
                              "Find {entity}'s {attribute}")
    attributes = st.multiselect("Select Attributes:",
                              ["email", "location", "founded_date"])
    
    # Advanced options
    with st.expander("Advanced Options"):
        search_depth = st.slider("Search Depth", 1, 10, 5)
        min_confidence = st.slider("Min Confidence", 0.0, 1.0, 0.7)
        
    return {
        "query": base_query,
        "attributes": attributes,
        "depth": search_depth,
        "confidence": min_confidence
    }

# Progress tracking
def process_with_progress(entities: List[str], config: dict):
    progress_bar = st.progress(0)
    status_text = st.empty()
    
    results = []
    for i, entity in enumerate(entities):
        status_text.text(f"Processing {entity}...")
        result = process_entity(entity, config)
        results.append(result)
        progress_bar.progress((i + 1) / len(entities))
        
    return results
```

### 5. Export and Integration

```python
# Data export with multiple formats
class DataExporter:
    @staticmethod
    def to_csv(data: pd.DataFrame, filename: str):
        try:
            data.to_csv(filename, index=False)
            return f"Successfully exported to {filename}"
        except Exception as e:
            return f"Export failed: {str(e)}"

    @staticmethod
    async def to_gsheet(data: pd.DataFrame, sheet_id: str):
        try:
            # Update Google Sheet
            worksheet = await connect_to_sheet(sheet_id)
            await worksheet.update([data.columns.values.tolist()] + 
                                data.values.tolist())
            return "Successfully updated Google Sheet"
        except Exception as e:
            return f"Update failed: {str(e)}"

# Usage
exporter = DataExporter()
csv_status = exporter.to_csv(results_df, "extracted_data.csv")
sheet_status = await exporter.to_gsheet(results_df, SHEET_ID)
```

## 📁 Project Structure

```
app/
├── dashboard.py      # Main UI components
├── data_manager.py   # Data handling & validation
├── web_searcher.py   # Search implementation
├── groq_processor.py # AI processing logic
├── exporters.py      # Data export utilities
└── utils/
    ├── cache.py     # Caching mechanisms
    ├── rate_limit.py # Rate limiting
    └── validators.py # Input validation
```

[Rest of the README remains the same...]
