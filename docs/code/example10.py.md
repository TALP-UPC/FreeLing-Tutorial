```python
#! /usr/bin/python3

import pyfreeling
import sys


## ----------------------------------------------
## -------------    MAIN PROGRAM  ---------------
## ----------------------------------------------

# set locale to an UTF8 compatible locale 
pyfreeling.util_init_locale("default");

# get installation path to use from arg2, or use /usr/local if not provided
ipath = "/usr/local";
if len(sys.argv)>1 : ipath=sys.argv[1]

# path to language data   
cpath = ipath + "/share/freeling/common/"

# creates a language identifier with the default config file
di = pyfreeling.lang_ident(cpath+"lang_ident/ident.dat")
  
#list of languages to consider.  Empty -> all known languages
candidates = []
   
line = sys.stdin.readline()                             
while (line!="") :
    line = line.rstrip()
    print ("-----------------------------------")
    print ("Input text: [" + line + "]")

    # The funcion identify_language will return the code for the best language, 
    # or "none" if no language model yields a small enough perplexity (according
    # to the threshold set in that language model)
    best_l = di.identify_language (line)
    print ("Best language:", best_l)

    # You can also get a sorted list of increasing perplexity,
    # in case you want to take the decision yourself.
    result = di.rank_languages(line)
    print("Increasing perplexity list:")
    for i in result : print(i[1],i[0])

    # next sentence
    line = sys.stdin.readline()                             



```
