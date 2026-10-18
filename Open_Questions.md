# Data Engineering Case Study — Open Questions

**Current solution:** Databricks, PySpark and Delta Lake  
**Scope:** Improvements for production use and a possible implementation in dbt Core

## Context

The submitted solution uses three layers:

- **Bronze:** The two CSV files are loaded into Delta tables, with all source columns stored as strings.
- **Silver:** Data types are corrected, JSON fields are extracted, and event parameters are combined so that one row represents one event.
- **Gold:** The `top_item` datamart shows the number of views, yearly ranking and most-used platform for each item and year.

The answers below describe how I would approach further development. The dbt Core option is a proposed alternative, **not an implementation that I have completed for this assignment**.

---

## 1. What steps are missing to industrialize the solution further?

The current solution works for the assignment, but it still requires manual file uploads and notebook execution. To make it suitable for regular use, I would focus on the following improvements:

1. **Automate the pipeline.** Instead of uploading files and running the notebook manually, I would automate data ingestion and run the Bronze, Silver and Gold steps in the correct order.

2. **Make reruns reliable.** The pipeline should be able to run again after a failure without creating duplicate data. If new data arrives regularly, I would also consider processing only new or changed records instead of rebuilding every table each time.

3. **Add data-quality checks.** I would validate that required columns are present, dates and numbers are converted correctly, event IDs are unique in Silver, and the total number of views in Gold matches the underlying events. If a check fails, the issue should be visible rather than silently ignored.

4. **Monitor execution.** I would track whether each run succeeds, how long it takes and how many records are processed. Failures should be reported so they can be investigated and the pipeline can be rerun.

5. **Improve maintenance and security.** I would keep the code in version control, separate configuration from business logic, and ensure that only authorized users or processes can modify the data.

I would start with **automation, safe reruns and data-quality checks**, as they address the biggest gaps in the current notebook. More advanced performance improvements could follow if data volumes or usage increased.

---

## 2. How would the architecture change with dbt Core? Would additional cloud resources be needed?

I would keep **Databricks and the existing Bronze, Silver and Gold structure**. The main difference would be how the transformations are developed and managed.

Currently, we use PySpark code in a notebook to transform the Bronze data into Silver and Gold tables. With dbt Core, I would move most of that transformation logic into **SQL models**. For example, separate models could handle parsing event data, combining event parameters, calculating item views and creating the final `top_item` datamart.

The flow would look like this:

```text
CSV files
    |
    v
Bronze tables in Databricks  <-- ingestion remains a separate step
    |
    v
Silver tables               <-- transformations managed by dbt Core
    |
    v
Gold: top_item              <-- final SQL model in dbt Core
```

I would still need a way to load the CSV files into Bronze, because dbt Core is mainly intended to transform data that is already available in the data platform.

**Additional cloud resources are not necessarily required.** We could continue using Databricks for storage and processing. We would need to configure dbt Core to connect to Databricks and provide compute to execute its SQL. dbt could be run locally during development and, later, scheduled using Databricks Jobs. A separate cloud platform or a paid dbt service would not be required for this basic setup.

---

## 3. What would dbt Core bring to the project? What are the upsides and downsides?

I see the main benefit of dbt Core as **making SQL transformations easier to organize, test and maintain**, especially as the project grows.

### Upsides

- **Clearer organization:** Instead of having all transformation steps in one notebook, each step can have its own SQL model.
- **Dependencies:** dbt can track which models depend on other models and run them in the correct order.
- **Testing:** We could add checks for missing values, duplicate event IDs and unique `(item_id, year)` combinations in the final datamart.
- **Documentation:** Model and column descriptions can be kept alongside the SQL code.
- **Collaboration:** Separate SQL files can be easier to review and maintain in Git when multiple people work on the project.

### Downsides

- **Additional tool to learn and maintain:** The project would need dbt configuration and a reliable way to run it.
- **Data ingestion is still needed:** dbt would not replace the step that reads the original CSV files into Bronze.
- **Some transformations may be less straightforward:** For example, combining multiple event-parameter rows into one event may require more SQL than the equivalent PySpark code.
- **More complexity for a small project:** With only two source files and one datamart, the current notebook may be simpler.

**My view:** I would consider dbt Core if this pipeline were expected to grow or be maintained by a team. For a small one-time assignment, PySpark is a reasonable choice. Moving to dbt would mainly improve how the project is managed; it would not automatically improve performance or solve data-quality problems.

---

## 4. How much effort would I estimate for implementing the solution in dbt Core?

For a developer who is already comfortable with SQL, Databricks and the basics of dbt Core, **I would estimate around 8–10 working days** to convert and validate the current solution.

| Work | Estimated effort |
|---|---:|
| Set up the dbt project and connection to Databricks | 1 day |
| Rewrite the Silver transformations as SQL models, including JSON parsing and combining event parameters | 2–3 days |
| Create the Gold `top_item` model | 1 day |
| Add tests and document the models | 1–2 days |
| Compare dbt results with the current PySpark results | 1–2 days |
| Set up and check a repeatable run | 1 day |
| **Overall estimate** | **7–10 working days, allowing roughly 8–10 for planning** |

This estimate assumes that the Bronze tables already exist, that the Databricks environment is available, and that the business requirements stay the same. It covers **converting the existing transformations**, not building a fully production-ready platform.

The main area of uncertainty would be rewriting and validating the event transformation, because multiple source rows must be combined correctly into one logical event. I would reserve time to compare the final view totals and rankings with those produced by the existing notebook.

If automatic ingestion, incremental processing, monitoring and deployment were also required, I would estimate that additional work separately after confirming those requirements. An engineer learning dbt Core for the first time would also need more time.

---

## Conclusion

The current PySpark solution is appropriate for the assignment. The next practical improvements would be automation, safe reruns and stronger validation. dbt Core could be useful later if the project grows, because it provides a more structured way to manage SQL transformations, tests and documentation. I would keep Databricks as the data platform and assess whether the benefits of dbt justify the additional setup for a project of this size.

## References

- [Databricks — Run dbt Core in jobs](https://docs.databricks.com/aws/en/jobs/how-to/use-dbt-in-workflows)
- [Databricks — dbt task for jobs](https://docs.databricks.com/aws/en/jobs/tasks/dbt)
- [dbt — Quickstart for dbt and Databricks](https://docs.getdbt.com/guides/databricks)
