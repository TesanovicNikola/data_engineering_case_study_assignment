# Data Engineering Case Study

Submission-ready Databricks/IPython notebook implementing Bronze, Silver, and Gold Delta tables.

## Contents
- `Data_Engineering_Case_Study.ipynb` — all PySpark/SQL DDL, ETL and validations, with Markdown documentation.

## Run in Databricks Free Edition
1. Download the public `event.csv` and `item.csv` files from the S3 URLs stated in the notebook.
2. Upload both CSV files to a Unity Catalog volume.
3. In Databricks Workspace, import `Data_Engineering_Case_Study.ipynb`.
4. Open the configuration cell and update `CATALOG`, `EVENT_PATH` and `ITEM_PATH` if necessary.
5. Attach Serverless compute and run all notebook cells in sequence.
6. Inspect `workspace.gold.top_item` in the Catalog, and check the final data-quality section.

**Important:** This notebook uses `INSERT OVERWRITE` to rebuild tables on each run.

## Reference outcome for supplied data
- 853,640 raw event rows, 2,198 item rows.
- 428,364 logical events; 262,786 item view events.
- 13,188 final item-year rows, 95 with zero views.
- `DENSE_RANK` preserves first-place ties in 2013, 2014 and 2015.

## Notes
- Real-time S3 access may be blocked by Databricks Free Edition; upload CSVs to a volume.
- The volume path and catalog can be changed in the notebook configuration cell.
