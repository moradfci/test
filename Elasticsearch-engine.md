# **Elasticsearch-engine location:** 
https://github.com/pfizer/EM-Cognitivesearch/tree/master/Search-engine-using-Elasticsearch-Jipa

## The app exposes 3 rest api:
### * /corpus/v1/documents
accepts `search_term ` as parameter and can be any string, eg: pain, heart, brain, etc.
Bellow are all the terms which can be passed:

 ```parser.add_argument('search', required=True, help='search parameter is required')    
    parser.add_argument('exact', required=False)    
    parser.add_argument('filterBy', required=False)
    parser.add_argument('path', required=False)
    parser.add_argument('titlesearch', required=False)
    parser.add_argument('score_from', required=False)
    parser.add_argument('search_from', required=False)
    parser.add_argument('indexname', required=False)
    parser.add_argument('username', required=False)```

The arguments will later be passed as keywords arguments to the function which will perform the query and return the json response.
This is how the parameters are passed as arguments:
`es_result = querry_elasticsearch_by_term(search_term, exact=exact, filterBy=filterBy, path=path, titlesearch=titlesearch, score_from=score_from, search_from=search_from, indexname=indexname, username=username)`

In the function above `search_term` is argument will always be passed first to the function, it`s a mandatory param, therefore is expected; after `search_term` the function will accept any number of keyword arguments.
keyword arguments = https://docs.python.org/2/glossary.html 

During the next step the search will be done by the function querry_elasticsearch_by_term().
The function is defined  like so:
`def querry_elasticsearch_by_term(search_term, **kwargs):`
`**kwargs` will represent a dictionary which can take an unlimited number of keyword arguments to avoid clustering.
to unpack the `**kwargs` simply use the example bellow to check if they exist and use them:

`  if 'filterBy' in kwargs.keys() and kwargs['score_from'] != None:
    filterBy = kwargs['filterBy']
    payload["query"]["bool"]["filter"] = { "term": { "file.extension": filterBy } }
  if 'score_from' in kwargs.keys() and kwargs['score_from'] != None:
    print("score_from:", kwargs['score_from']) 
    score_from = kwargs['score_from']
  if "search_from" in kwargs.keys() and kwargs['search_from'] != None:
    print("search_from:", kwargs['search_from'])
    search_from = kwargs['search_from']
  if 'indexname' in kwargs.keys() and kwargs['indexname'] != None:
    print("index is: ", kwargs['indexname'])
    indexname = kwargs['indexname']`

Once the json body is built based on the parameters search will be done, again using keyword arguments:
`response = es.search(index=index, body=payload)`
if a parameter `indexname=pubmedtest` will be sent, the response will be rearranged to fit frontend needs with the following function:
`response = response_to_send(response)`

if no `indexname` parameter will be sent, the query will be performed against the `index = "emcoeprod" ` by default.
In this case the response from `emcoeprod` will be trimmed for unnecessary quotes as follows: 
`response = __trim_quotes(response)` 
And then the response will be returned to the front end


## Based on the explanation above, 2 more api`s are opened by this app

* /corpus/v1/document_details
* /corpus/v1/document_preview

The app perform queries in elasticsearch using a connection pool as follows:
`es = Elasticsearch(
        hosts = get_base_url(),
        http_auth = account,
        use_ssl = True,
        verify_certs = False,
        sniff_on_start=True,
        sniff_on_connection_fail=True,
        sniff_timeout=10,
        sniffer_timeout=60,
        connection_class = Urllib3HttpConnection
)`
This was a fast connection with the help of `Urllib3HttpConnection`.
The > account is being read from `config.cfg`