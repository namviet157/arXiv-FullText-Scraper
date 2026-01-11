# arXiv Full-Text Scraper

A Python-based tool for downloading arXiv papers (LaTeX sources) and extracting their references from Semantic Scholar. The project processes papers in parallel, downloads all versions, extracts metadata, and fetches citation information.

## Features

- **Parallel Processing**: Process multiple papers simultaneously with configurable parallelism
- **Multi-version Support**: Downloads all versions of each arXiv paper
- **Reference Extraction**: Fetches references from Semantic Scholar API (only arXiv papers)
- **Resource Monitoring**: Tracks RAM and disk usage during processing
- **Automatic Cleanup**: Removes non-LaTeX files, keeping only `.tex` and `.bib` files
- **Progress Tracking**: Real-time statistics and progress reports
- **Resume Support**: Automatically skips already processed papers

## Project Structure

```
arXiv-FullText-Scraper/
├── main.ipynb              # Main Jupyter notebook (run this)
├── requirements.txt        # Python dependencies
├── README.md               # This file
├── .gitignore             # Git ignore rules
└── ArXivPapers/           # Output directory (created after running)
    └── YYMM-NNNNN/        # Paper folders (format: YYMM-NNNNN)
        ├── metadata.json  # Paper metadata
        ├── references.json # References (only arXiv papers)
        └── tex/           # LaTeX source files
            ├── YYMM-NNNNNv1/  # Version 1 folder
            │   ├── *.tex
            │   ├── *.bib
            │   └── <subfolders>/  # Preserves original structure
            ├── YYMM-NNNNNv2/  # Version 2 folder (if exists)
            └── ...
```

**Note**: The notebook generates three Python scripts (`arxiv_crawler.py`, `reference_extractor.py`, `main.py`) when executed. These are created using `%%writefile` and can be run standalone if needed.

## Installation

### Prerequisites

- **Python 3.7+** (Python 3.8 or higher recommended)
- Jupyter Notebook or JupyterLab (for running the notebook)

### Install Dependencies

Install the required packages using `requirements.txt`:

```bash
pip install -r requirements.txt
```

Or install packages individually:

```bash
pip install arxiv requests psutil memory_profiler
```

**Dependencies:**
- `arxiv>=2.1.0` - arXiv API client
- `requests>=2.31.0` - HTTP library for Semantic Scholar API
- `psutil>=5.9.0` - System and process utilities for resource monitoring
- `memory_profiler>=0.61.0` - Memory profiling for IPython extension

## Usage

### Step 1: Open the Notebook

Open `main.ipynb` in:
- Jupyter Notebook
- JupyterLab
- VS Code (with Jupyter extension)
- Google Colab
- Kaggle Notebooks

### Step 2: Configure Parameters

Edit the configuration in the `main()` function (Cell 3) or modify the variables directly:

```python
START_MONTH = "2023-04"      # Start month in format "YYYY-MM"
START_ID = 14607             # Starting arXiv ID number
END_MONTH = "2023-05"        # End month in format "YYYY-MM"
END_ID = 14596               # Ending arXiv ID number
MAX_PARALLELS = 2            # Number of parallel threads
SAVE_DIR = "./ArXivPapers"   # Output directory
```

#### Parameter Descriptions

- **START_MONTH** / **END_MONTH**: Date range in `"YYYY-MM"` format
  - Single month: Set both to the same month (e.g., `"2023-04"` to `"2023-04"`)
  - Multi-month: Set different months (e.g., `"2023-04"` to `"2023-05"`)

- **START_ID** / **END_ID**: arXiv ID number range (5-digit format)
  - Example: `14607` corresponds to `2304.14607` (if month is April 2023)
  - The system automatically finds the last valid ID in the start month if processing multiple months

- **MAX_PARALLELS**: Number of parallel threads
  - **Recommended**: 2-3 threads
  - Higher values = faster processing but more resource usage
  - Consider your network bandwidth and system resources

- **SAVE_DIR**: Output directory path (default: `"./ArXivPapers"`)

### Step 3: Run the Notebook

Run all cells sequentially. The notebook will:
1. Install dependencies (Cell 0)
2. Generate `arxiv_crawler.py` (Cell 1)
3. Generate `reference_extractor.py` (Cell 2)
4. Generate `main.py` (Cell 3)
5. Run the main processing (Cell 4 - optional, for memory profiling)
6. Display resource usage (Cell 5 - optional)

### Step 4: Monitor Progress

The script provides real-time progress updates:
- Progress reports every 10 papers
- Resource monitoring (RAM and disk usage)
- Final statistics (success rates and failure counts)
- Status indicators per paper

## Output Format

### Directory Structure

Each processed paper creates a folder:

```
ArXivPapers/
└── 2304-14609/              # Paper folder (format: YYMM-NNNNN)
    ├── metadata.json        # Paper metadata
    ├── references.json      # References (only arXiv papers)
    └── tex/                 # LaTeX source files
        ├── 2304-14609v1/    # Version 1
        │   ├── *.tex
        │   ├── *.bib
        │   └── <subfolders>/  # Preserves original structure
        ├── 2304-14609v2/    # Version 2 (if exists)
        └── 2304-14609v3/    # Version 3 (if exists)
```

### metadata.json Structure

```json
{
    "arxiv_id": "2304-14609",
    "paper_title": "On the temperature dependence...",
    "abstract": "We report neutron-scattering measurements...",
    "authors": [
        "Sha Jin",
        "Xue Fan",
        ...
    ],
    "submission_date": "2023-04-28",
    "revised_dates": [
        "2024-02-07",
        "2024-03-24"
    ],
    "publication_venue": "Scientific Reports volume 14...",
    "latest_version": 3,
    "categories": [
        "cond-mat.soft",
        "cond-mat.dis-nn"
    ],
    "pdf_url": "https://arxiv.org/pdf/2304.14609v3"
}
```

### references.json Structure

Contains only references that are arXiv papers (filtered by Semantic Scholar):

```json
{
    "2305-02524": {
        "paper_title": "A fresh look at the vibrational...",
        "authors": [
            "Hai-Long Xu",
            "Matteo Baggioli",
            "T. Keyes"
        ],
        "submission_date": "2023-05-04",
        "semantic_scholar_id": "4b17685939a939cb0e6b56e9786951e4d5c9fdb3"
    },
    ...
}
```

## Rate Limiting

### arXiv Rate Limiting

- **Delay between downloads**: 0.3 seconds (configurable in `arxiv_crawler.py`)
- The code includes delays between version downloads to avoid overwhelming the server
- arXiv recommends being respectful with requests

### Semantic Scholar Rate Limiting

- **Retry delay**: 3 seconds (configurable in `reference_extractor.py`)
- **Rate limits**:
  - Without API key: 1 request per second, 100 requests per 5-minute window
  - With API key: Higher limits (varies by tier)
  - Get a free API key at: https://www.semanticscholar.org/product/api

**To use API key**: Set environment variable:
```python
import os
os.environ["SEMANTIC_SCHOLAR_API_KEY"] = "your_api_key_here"
```

The code automatically handles rate limit errors (HTTP 429) and retries with exponential backoff.

### Adjusting Scraping Rate

**arXiv delay** (in `arxiv_crawler.py`):
```python
time.sleep(0.3)  # Change this value (in seconds)
```

**Semantic Scholar delay** (in `reference_extractor.py`):
```python
def get_paper_references(arxiv_id, delay=3):  # Change default delay
```

## Parallelism Configuration

### Recommended Settings

- **Low (1 thread)**: 
  - Suitable for slow networks
  - Lower resource usage
  - More reliable, less likely to hit rate limits

- **Medium (2-3 threads)**: ⭐ **Recommended**
  - Good balance between speed and reliability
  - Default: 2 threads

- **High (4+ threads)**:
  - Faster processing but higher risk of rate limiting
  - Requires good network bandwidth
  - May need to increase delays between requests

## Performance Metrics

Based on test runs processing **1500 papers** (2305.8001 to 2305.9500):

- **Total Run Time**: ~2 hours 17 minutes
- **Average processing time**: ~5.48 seconds per paper
- **Success rate**: 100% (both phases combined)
- **Peak RAM usage**: ~122 MB (memory_profiler) / ~8.2 GB (psutil)
- **Disk usage**: ~2.4 GB (~1.5-2.4 MB per paper)

> **Note**: Performance may vary based on network speed, system resources, and API rate limits.

## Troubleshooting

### Common Issues

1. **Rate Limiting Errors**
   - Reduce `MAX_PARALLELS`
   - Increase delays in the code
   - Use Semantic Scholar API key

2. **Disk Space Issues**
   - Monitor disk usage in progress reports
   - Clean up failed paper folders if needed
   - Ensure sufficient disk space (papers can be large)

3. **Memory Issues**
   - Reduce `MAX_PARALLELS`
   - Process papers in smaller batches
   - Monitor RAM usage in reports

4. **"file" command not found (Windows)**
   - The code will still work but with reduced file type detection
   - Uses fallback method based on file extensions
   - Consider using WSL or Git Bash for better compatibility

5. **Semantic Scholar 404 Errors**
   - Some papers may not be in Semantic Scholar database
   - This is expected and will be logged
   - Empty `references.json` files will be created for these papers

6. **Import Errors**
   - Make sure all dependencies are installed: `pip install -r requirements.txt`
   - Restart the kernel after installing packages

## Example Usage

### Example 1: Single Month Range

```python
START_MONTH = "2023-05"
START_ID = 2001
END_MONTH = "2023-05"
END_ID = 2010
MAX_PARALLELS = 2
```

This processes papers `2305.02001` through `2305.02010`.

### Example 2: Multi-Month Range

```python
START_MONTH = "2023-04"
START_ID = 14607
END_MONTH = "2023-05"
END_ID = 14596
MAX_PARALLELS = 2
```

This processes:
- Papers from April 2023 starting at ID 14607 until the last valid ID
- Papers from May 2023 from ID 1 to 14596

## Saving Outputs on Google Colab

If running on Google Colab and want to persist outputs to Google Drive:

1. Mount Google Drive (uncomment and run Cell 7)
2. Copy the output folder:

```bash
# Option 1: Copy directly
!cp -r /content/ArXivPapers /content/drive/MyDrive/

# Option 2: Create zip first (faster for many files)
!zip -r ArXivPapers.zip /content/ArXivPapers
!cp ArXivPapers.zip /content/drive/MyDrive/
```

**Note**: Existing files with the same names may be overwritten.