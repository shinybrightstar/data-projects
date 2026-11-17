# Data Projects

A collection of data projects spanning analysis, dashboarding, and engineering — built with SQL, Python, Excel, and Power BI, with data engineering projects being added as they're built. Each project folder includes the queries/code used, as well as a write-up of the approach and findings.

## 📁 Projects

| Project | Tools | Description |
|---|---|---|
| [Retail Sales & Customer Analysis](./sql/classic-models-analysis) | SQL | Analysed a retail database to identify restocking priorities, segment VIP vs. least-engaged customers by profit, and calculate customer lifetime value to guide acquisition spend. |
| Hacker News Pipeline | Data Engineering | The goal was to find the top 100 keywords of Hacker News posts in 2014. Because Hacker News is the most popular technology social media site, this gave us an understanding of the most talked-about tech topics in 2014! |

---

### How we completed the Hacker News Pipeline
1. Imported the Pipeline class from the pipeline module.
2. Instantiated an instance of the Pipeline class and assign it to the variable pipeline
3. Created a pipeline.task() function that takes in no arguments.
4. Call the function file_to_json(), where the function does the following:
  Loads the hn_stories_2014.json file into a Python dict.
  Returns the list of stories
5. Created a pipeline.task() function that depends on the file_to_json() function.
6. Called the new function filter_stories(), which filters popular stories that have more than 50 points, more than 1 comment, and do not begin with Ask HN.
7. filter_stories() returns a generator of these filtered stories.
8. Created a pipeline.task() function that depends on the filter_stories() function.
9. Called the new function json_to_csv(), which writes the filtered JSON stories to a CSV file:
  Imported build_csv from the pipeline module and io. The build_csv() function has the same API as the one you wrote in the second and third lesson.
  Created a CSV file with the headers 'objectID', 'created_at', 'url', 'points', and 'title'.
  Parsed the created_at column using datetime.datetime.
10. json_to_csv() returns the value from build_csv() using the above header, lines, and the io.StringIO() file.
11. Created a pipeline.task() function that depends on the json_to_csv() function.
12. Called the new function extract_titles(), that returns a generator of every Hacker News story title:
13. extract_titles() returns a generator of titles.
14. Created a pipeline.task() function that depends on the extract_titles() function.
15. Called the new function clean_titles(), that returns of a generator of cleaned titles:
  Ensure the title is lower case.
  Remove any punctuation from the title.
16. Created a pipeline.task() function that depends on the clean_titles() function.
17. Called the new function build_keyword_dictionary(), that returns a dictionary of the word frequency of all the HN titles.
  The word frequency should not include stop words.
  I found the words by spliting the titles dictionary on the empty space character.
  Empty words were be ignored.
18. Created a pipeline.task() function that depends on the build_keyword_dictionary() function.
19. Called a new function top_keywords which returns a list of the top 100 tuples described above.
20. Ran the pipeline using pipeline.run(), and printed the output of the new task function.
