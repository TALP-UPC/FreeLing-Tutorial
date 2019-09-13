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
if len(sys.argv)!=3 : 
    print ("Usage:  tagset tagset-file (tag2msd|msd2tag)")
    exit(1)

# get arguments
fname = sys.argv[1]
action = sys.argv[2]

# create tagset handling module
tgs = pyfreeling.tagset(fname);
 
# convert tags to morphosyntactic descriptions
if action=="tag2msd" :
    line = sys.stdin.readline()
    while (line!="") :
        if line!="\n" :
            (form,lemma,tag) = line.rstrip().split(" ")
            print (form, lemma, tgs.get_msd_string(tag) )
        line = sys.stdin.readline()

# convert morphosyntactic descriptions to tags
elif action=="msd2tag" :
    line = sys.stdin.readline()
    while (line!="") :
        if line!="\n" :
            (form,lemma,msd) = line.rstrip().split(" ")
            print (form, lemma, tgs.msd_to_tag("",msd) )
        line = sys.stdin.readline()

else :
    print ("Invalid action "+action)
  
```
