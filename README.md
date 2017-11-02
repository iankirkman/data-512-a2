# data-512-a2
Assignment 2 for DATA 512 (Human-Centered Data Science). See: https://wiki.communitydata.cc/HCDS_(Fall_2017)/Assignments#A2:_Bias_in_data

Released by: Ian Kirkman ikirkman@uw.edu 11/1/2017

## Project Goal
The goal of this assignment is to explore the ramifications of bias in data. Given the known demographics of english Wikipedia editors (see "Nationality" in https://en.wikipedia.org/wiki/Wikipedia:Wikipedians), we anticipate a bias that affects both the scope and quality of english Wikipedia articles for political figures from various countries. We will analyze the coverage and quality metrics of these articles on political figures, and reflect on our findings.

## License Info
 - The content of this project directory is released under the [MIT license](LICENSE.md).

## Data Sources
The [Population Reference Bureau (PRB)](http://www.prb.org/DataFinder/Topic/Rankings.aspx?ind=14) website contains a dataset with population counts by country circa mid-2015. A CSV file can be downloaded directly from the link by clicking the Excel icon in the top right side of the page. The CSV file must then be uploaded to the project data directory at the path specified in the inputs section of the project [jupyter notebook](hcds-a2-bias.ipynb). The only fields we use from this data are `Location` and `Data`, which correspond to 'Country' and 'Population' on our final dataset, respectively.

The english Wikipedia page data for political figures by country was provided by Oliver Keyes on [Figshare](https://figshare.com/articles/Untitled_Item/5513449). The CSV file can be downloaded via the download button on the top left, and then uploaded to the project data directory at the path specified in the inputs section of the project [jupyter notebook](hcds-a2-bias.ipynb). Keyes's code and data notes at available at the link. We will use each field from this dataset, with the following mapping to our final output: 'country' to 'Country', 'page' to 'Article_Name', and 'rev_id' to 'Revision_ID'.

For the article quality score predicitons, we will be accessing the [Objective Revision Evaluation Service (ORES) API](https://www.mediawiki.org/wiki/ORES) to collect predictions by article (matched on Revision ID). We use the endpoint and parameters specified in Section 1.1 of the notebook to call the API with multiple Revision IDs smushed together with a vertical line delimiter. The user-input parameters in the notebook specify the project ('enwiki') and model ('wp10') for the API call. Version 3 is assumed and hard-coded into all API calls. Revision IDs are added to the parameters in chunks of 50 after some initial processing steps from the Figshare data.

ORES API calls return a nested dictionary. To access the score prediction for a given article (using Rev_ID_001 as an example), we pull: `api_results[ores_params['project']]['scores'][Rev_ID_001][ores_params['model']]['score']['prediction']`. 

ORES score predictions are classified as (ordered from best to worst):
- `FA`: Featured article
- `GA`: Good article
- `B`: B-class article
- `C`: C-class article
- `Start`: Start-class article
- `Stub`: Stub-class article

See the ORES API linked above for further details.

Raw ORES data from the API calls is saved in the [DATA](DATA/) folder.

**Additional Data Source Notes and Assumptions:**
- The column order is assumed to be consistent for any data download from the first two sources. Passive header checking has been added as print statements in Sections 1.3 and 2 of the notebook.
- Note that the Wikipedia page data on Figshare has updated a field name from 'last_edit' to 'rev_id'. This change is not currently reflected in the page data documentation.
- Not all of the country names matched between the Wikipedia page data and the PRB Population data. Unmatched countries were excluded, and excluded data was tracked and reconciled via print statements in the notebook.
- Some of the ORES API calls return an error dictionary instead of returning a score prediction. Those error dictionaries are saved in place on the raw data, and dealt with in our processing steps of Section 2.


## Processing Steps
See the project [jupyter notebook](hcds-a2-bias.ipynb) for detailed processing steps.

## Output
The raw data is processed into a single [csv table](OUTPUT/processed_wp_prb_ores_data.csv) with the following fields:

The final combined dataset exported to CSV has headers that correspond to each source value:

| Column | Value Source |
| :--- | :--- |
| Country	| WP.country & PRB.Location  |
| Article_Name	| WP.page |
| Revision_ID	| WP.rev_id & ORES.revid |
| Article_Quality	| ORES.prediction |
| Population	| PRB.Data |

To analyze the bias in english Wikipedia articles, we compute two metrics for each country in the combined data. To assess coverage of articles in a country, we compute an articles-per-population proportion (reported as a percentage). To assess the quality of articles in a given country, we compute the proportion of articles that are high quality (those that are classified as 'FA' or 'GA', also reported as a percentage).

We can use these two metrics to obtain the following country-ranking visualization tables, embedded in the project [jupyter notebook](hcds-a2-bias.ipynb):
Top Ten Countries by Coverage Proportion (Articles-to-Population)
Bottom Ten Countries by Coverage Proportion (Articles-to-Population)
Top Ten Countries by Proportion of High Quality Articles
Bottom Ten Countries by Proportion of High Quality Articles


## Known Issues

- See Above: Additional Data Source Notes and Assumptions.
- No tie-breaker logic has been added to country ranking tables.
