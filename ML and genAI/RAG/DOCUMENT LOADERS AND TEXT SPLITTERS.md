

# DOCUMENT LOADERS 

imp types there are like 500 more 
https://docs.langchain.com/oss/python/integrations/document_loaders 


1. TEXT LOADER
2. PyPDF Loaders
3. WebBase Loaders
4. CSVLoaders

--- 

Also with all the loaders we have the integration to the **.load()** and the **.lazy_loader()** 

```python
from langchain_community.document_loaders.csv_loader import CSVLoader

loader = CSVLoader(
    ...  # Integration-specific parameters here
)
# Load all documents
documents = loader.load()           # simple lopa laganeka 

# For large datasets, lazily load documents
for document in loader.lazy_load(): # generators 
    print(document) 
```
--- 
 
#### text loaders 
returns a list of object from the document, 
has the page_content and metadata. 

#### PyPDF Loaders 
Converts the pdf to docs page wise. page wise conversion only.  
	number of pages -> number of retreived docs. 

#### Webbaseloader 
scrapper and shi

--- 
--- 
--- 
# TEXT SPLITERS

> link `https://docs.langchain.com/oss/python/integrations/splitters` <- best resource

>                        DIVIDE AND CONQUER CHIEF


**Text splitters** break large docs into smaller chunks that will be retrievable individually and fit within model context window limit. 

THERE ARE TYPES: 

* `CharacterTextSplitter`               -> length based  and token based
*  `RecursiveCharacterTextSplitter`  -> length based but completes the word before hard split. based on the basis of.  `\n\n` paragraph based. `\n` line based. ` ` space based.  

* `DocumentStrucureBased`                   -> structure based like code, json, yaml etc. 
* `Semantic meaning based`                  -> 



--- 
