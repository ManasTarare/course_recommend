# Course Recommend

A data-driven course structure recommendation and generation prototype. It collects course information from Coursera and Microsoft Learn, trains machine learning models to recommend a course structure, and can use retrieval-augmented generation (RAG) to draft course content from uploaded source materials.

## Notebooks

- **`scrapper.ipynb`** collects and prepares course data. It fetches Microsoft Learn catalog data, discovers Coursera courses matching configured search terms, and produces CSV and JSON outputs.
- **`main.ipynb`** trains and evaluates course recommendation models, provides a Gradio interface for structure recommendations, and includes an optional RAG workflow for course content generation.

## Workflow

```text
scrapper.ipynb
  ├─ Microsoft Learn catalog → CSV + course structure JSON
  ├─ Coursera search results → CSV + course structure JSON
  ├─ Merge course CSVs → merged.csv
  └─ Normalize features → normalized_output.csv + mapping/metadata JSONs
                         │
                         ▼
main.ipynb
  ├─ Validate and align CSV and course structure records
  ├─ Train and evaluate regression and structure-tag classifiers
  ├─ Recommend a course outline from user inputs
  └─ Optional: retrieve source material and generate course content with Gemini
```

## Data outputs

The scraper and preparation notebook creates these files for the recommendation notebook:

- `courses_dataset_coursera.csv`
- `mslearn_courses_dataset.csv`
- `coursera_course_structures.json`
- `mslearn_course_structures.json`
- `merged.csv`
- `normalized_output.csv`
- `normalized_output_provider_map.json`
- `normalized_output_category_map.json`
- `normalized_output_meta.json`

`main.ipynb` expects the training inputs under `/content/course_recommendation/DATA` in Google Colab, or at the paths configured in the notebook. It validates that the CSV and JSON rows refer to the same courses before training.

## Course structure recommendation

The recommendation interface accepts course details such as provider, category, depth level, chapter count, and duration. Models predict content counts (such as videos, readings, assignments, and quizzes) and course structure tags. The notebook evaluates multiple algorithms and selects models based on held-out data results. It uses the predicted structure to build a readable outline based on course templates in the training data.

The notebook compares regression models including KNN, Random Forest, Extra Trees, Gradient Boosting, and XGBoost (or HistGradientBoosting when XGBoost is unavailable). It also evaluates multi-label classifiers including Logistic Regression, Linear SVC, Random Forest, and Extra Trees.

## Optional RAG course generation

The RAG section of `main.ipynb` accepts TXT, PDF, and DOCX source files. It chunks the extracted text, generates embeddings with `sentence-transformers/all-MiniLM-L6-v2`, and stores the vectors in an OpenSearch k-NN index. It retrieves relevant passages for the recommended outline and sends the context and course metadata to Gemini to draft a course in Markdown.

Generated files include `generated_course.md` and `generated_course.json`. The recommendation interface also saves the recommended structure and metadata as files for the RAG stage.

## Running in Google Colab

1. Upload or clone the repository into Colab.
2. Run `scrapper.ipynb` to fetch the course data and produce the CSV/JSON outputs.
3. Merge and normalize the course CSVs in the scraper notebook.
4. Make sure all required CSV and JSON files are in the `DATA` directory expected by `main.ipynb`.
5. Run `main.ipynb` to train and evaluate models, then use the Gradio interface to request a course structure.
6. For RAG generation, configure OpenSearch and Gemini credentials, upload source material, and run the RAG cells.

## Technology

Python, Google Colab, pandas, NumPy, scikit-learn, XGBoost, DuckDB, Gradio, Sentence Transformers, OpenSearch, Gemini, PyPDF, and python-docx.

## Configuration and data handling

- Coursera discovery is limited to courses matching the search terms configured in `scrapper.ipynb`.
- Microsoft Learn collection requests the full modules and units catalogs, unless a limit is configured.
- OpenSearch credentials and the Gemini API key are requested by notebook cells. Keep credentials in secure Colab inputs or secrets; do not commit them to the repository.
- Course source files, scraped datasets, trained model bundles, and generated course content are runtime artifacts and should only be committed when intended.

## Scope

This is a notebook-based prototype. The notebooks cover course data collection, feature preparation, model training and evaluation, interactive structure recommendation, and optional RAG-based course drafting. They are run as separate stages and exchange data through generated files.
