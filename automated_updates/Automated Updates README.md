## Approach Overview
The approach relies on feeding images and prompts into the multi-modal OpenAI API (gpt-4o-mini) for automated parsing. This end-to-end approach *by-far* outperforms OCR libraries and other fragmented LLM-based workflows. It is also way easier to implement and continue to develop, and will continue getting better and cheaper as these LLM's improve.

It is set up so that most users will not have to run the automation pipeline. `all_source_data` and `final_data` folders are small enough to commit to GitHub, so ideally one person will run this code periodically, and everyone else can just easily access these artifacts for their own use (unless you are contributing to development of the automation)

## Setup

To set up the full automation pipeline, follow these steps:

1. **Create a Python Virtual Environment**  
   - Use Python version 3.11.6 or similar.
   - Run: `python -m venv your-env-name`
   - Activate the environment, then install dependencies:  
     ```bash
     pip install -r requirements.txt
     ```

2. **Set Environment Variables**  
   - Create a `.env` file in `automated_updates/.env`
   - **Congress API Key**: Get an API key from Congress.gov and add it to `.env` as follows:  
     ```plaintext
     CONGRESS_GOV_API_KEY='your_api_key'
     ```
   - **ChromeDriver**:  
     - Download ChromeDriver matching your Chrome version and OS from [Chrome for Testing](https://googlechromelabs.github.io/chrome-for-testing/).
     - Add the path to `.env` like this:  
       ```plaintext
       CHROME_DRIVER_PATH='/path/to/chromedriver'
       ```
   - **OpenAI API Key**: Get a secret key from OpenAI, ensuring sufficient funds to use the API, and add it to `.env` as:  
     ```plaintext
     OPENAI_API_KEY='your_openai_api_key'
     ```

**Note on pymupdf**  
   If you encounter the error: `ModuleNotFoundError: No module named 'frontend'`, fix it by reinstalling the pymupdf package:
     ```bash
     pip uninstall pymupdf
     pip install pymupdf
     ```
   This seems to be an issue with the package.

#### Example .env File
```plaintext
CONGRESS_GOV_API_KEY='rv92...'
CHROME_DRIVER_PATH='/path/to/chromedriver'
OPENAI_API_KEY='sk-kiKX...'
```

## Using a Test Dataset

To quickly set up and ensure all code paths are functioning, a pre-defined test dataset is available. This dataset includes pre-selected congress members to hit all relevant code paths.

1. **Modify Folder Paths in `config.py`**: Update `config.py` with new test dataset directories to ignore the existing files. Example modifications:
   ```
   source_data_dir = './all_source_data_test/'  
   intermediate_files_dir = './intermediate_files_test/'  
   processed_data_dir = './all_processed_data_test/'  
   ```
   Alternatively, just delete the existing source files.
   
2. **Run the Gather Script with the Test Flag**: Execute the `gather_source_data.py` script with the `--test-set` flag to load and process the test dataset:

   ```python gather_source_data.py --test-set```


## Run the Pipeline in Three Parts

**gather_source_data.py**  
* Retrieves congress member data from the Congress API: https://api.congress.gov/v3/member/congress  
* Scrapes data from the House and Senate financial disclosure sites:
    - House Financial Disclosures: https://disclosures-clerk.house.gov/
    - Senate Financial Disclosures: https://efdsearch.senate.gov/
* Organizes data for processing and converts PDF/GIF files to JPEGs.

*Note: This step can be skipped if you already have recent source data, which should be up to date and committed to the repo. For example, if you are working on the processing of the source files and and don't want to re-run the gather module.*

**parse_asset_names_llm.py**  
* Sends images to OpenAI's API with specific prompts, extracting asset names and saving them to `./all_processed_data`.

**summarize_results.py**  
* Summarizes files in `./all_processed_data` into `final_datasets/final_asset_data.csv` and `final_datasets/final_summary_data.csv`.
* Implements bitcoin/crypto term classification based on a predefined list.


## Ideas for Improvement
It currently works sufficiently well to detect the bitcoin/crypto terms. The following ideas could improve the asset list beyond just bitcoin/crypto related assets.

* Better method for image rotation. Assumptions can be made for most files, but sometimes an image will be rotated incorrectly, which hurts the LLM performance.
* Use gpt-4o instead of gpt-4o-mini (current choice to save $).
    * I have experimented with this on select files and it does seem to be markedly better.
    * Using gpt-4o-mini for clean house files (92%), gpt-4o for messy house files (~7%) and senate gif files (~1%) might be best bang for buck.
* Refine prompts.
* Crop images intelligently to focus on relevant sections.
* Currently asking the LLM when to skip a file. This may be counter-productive. Consider just one-shot prompt each image.
* Directly ask the LLM, does this image mention bitcoin or cryptocurrency? not sure if this will work, might encourage hallucination. worth testing.
* links to all sources are available in the data scraping process, need to write them to file for easy inclusion in the main readMe