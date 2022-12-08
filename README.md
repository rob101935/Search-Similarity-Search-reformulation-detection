## Search Similarity and Search reformulation

In search behaviour one common issue is where users alter their search because they are not getting satisfactory results in their search, this is called search reformulation and as discussed in this paper (https://dl.acm.org/doi/abs/10.1145/2396761.2398399) is a key predictor of search abandonment and generally dissatisfaction with search results. 

It makes sense to track sessions where search reformulation occurred for long term monitoring of search algorithm performance and also to illuminate potential future improvement opportunities in seach systems.

## The problem

The data provided provides search terms, search timestamps and resulting product click data (if there was any) in JSON format with the data being arranged as objects that show one browser session for a major supermarket retailer. An example of the data structure is shown below:

`{"search_query":["water","coffee","coffee mate","tide pods","downy","fabric conditioner","polident","hair colour"],"click_events":["20022126_AR","203128_PR","2118_PR","2190_PR","NO CLICKS","NO CLICKS","20437569002_PR","21161_AR"],"query_timestamp":["1631731185","1631731197","1631731232","1631731835","1631731906","1631731917","1631731940","1631731968"],"sessionid":"b9c24cf1-c298-4045-8ae2-2beec4f953b1"}`

It is obvious that we will restrict ourselves to pairs of search terms within the same session, the pairs being the current and following searches rather than any one search in a session versus the others. This is because generally search reformulation happens sequentially, and as evidenced in the data entry above users in this type of search will often search for multiple unrelated products rather than purchasing just one (as supermarket purchases tend to be low priced items)

It is clear we can view the click count as a possible response variable, no clicks indicate abandonment and can be associated with failed searches and therefore can point to a subsequent search reformulation, particularly where semantic similarity with the immediately following search term is high. Low time between searches can also indicate a dissatification with search results and therefore can point to search reformulation so this is another metric we will look at.

## Techniques applied

A commmon method for comparing similarities of search terms is using word embeddings. Recently transformer networks pioneered by techniques such as bert have spawned sentence transformers including those trained on question and answer or query and document pairs. In a similar method used with GLOVE word embeddings we can use these transformers and their embeddings to encode semantic meaning in the form of embedding vectors. Beyond that we can then compare semantic meaning using cosine similarity or dot product to determine their level of similarity. I compared performance using sets of embeddings from two models: from the Setence-Transformers package (for more detail see: https://www.sbert.net/) initally a multilingual model: distiluse-base-multilingual-cased-v2 and then a general purpose english model: all-MiniLM-L6-v2. 

Once these are computed we can then look this as a problem of setting a similarity threshold that will result in correct classification of abandonments (for our purposes searches with no clicks) and successful searches. I also discovered that large time deltas between searches generally were more related to dissimilar searches, even if they did not click results which logically makes sense that the cause or result of these failed searches is not search reformulation, therefore i restricted the training set to only searches with a inter query delta time of 2.5 minutes or less.

I ended up deciding to use all-MiniLM-L6-v2 embeddings as their performance over 5 cross fold validations sets was 67.49% vs 58.41% in terms of Macro F1 score. however it did achieve Precision of 73.5% which I felt was adequate given we are to use this as a metric later, we don't mind if some similar searches are identified that didn't result in abandonment its more important that all searches with relatively high semantic similarity that resulted in abandonment are identified. Refer to the EDA.ipynb for my further details EDA and model comparison. 
