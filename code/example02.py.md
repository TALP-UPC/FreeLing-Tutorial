```python
#! /usr/bin/python3

import pyfreeling
import sys

##---------------------------------------------
## Increase the count of key w in map m, or
## set it to 1 if not found
##---------------------------------------------
def add_count(m, w) :
    # key previously seen, increase count 
    if w in m : m[w] = m[w] + 1
    #  key not found in the map, add it with count=1 
    else : m[w] = 1


## -----------------------------------------------
## Do whatever is needed with analyzed sentences
## -----------------------------------------------
def ProcessSentences(ls):
    # maps to count lemma unigrams and bigrams
    lemmas = {}
    lemma_bigrams = {}

    # for each sentence in list
    for s in ls :
        # previous word lemma, for the bigrams
        prev = ""
        # for each word in sentence
        for w in s :
            #  count an occurrence more for the current word lemma
            add_count(lemmas, w.get_lemma())

            # count an occurrence more for the pair of previous and 
            # current word lemmas.
            if prev != "" :
                add_count(lemma_bigrams, prev+" "+w.get_lemma())
                
            # move to next word   
            prev = w.get_lemma()

    # once all lemmas and bigrams are counted, output the results 
    print("========== LEMMA FREQUENCIES (lemma,freq) ====================")
    for p in lemmas :
        print (p, lemmas[p])
    print("")

    print("========== LEMMA BIGRAM FREQUENCIES (lemma1,lemma2,freq) ==============")
    for p in lemma_bigrams :
        print (p, lemma_bigrams[p])
    print("")


## -----------------------------------------------
## Set desired options for morphological analyzer
## -----------------------------------------------
def my_maco_options(lang,lpath) :

    # create options holder 
    opt = pyfreeling.maco_options(lang);

    # Provide files for morphological submodules. Note that it is not 
    # necessary to set file for modules that will not be used.
    opt.UserMapFile = "";
    opt.LocutionsFile = lpath + "locucions.dat"; 
    opt.AffixFile = lpath + "afixos.dat";
    opt.ProbabilityFile = lpath + "probabilitats.dat"; 
    opt.DictionaryFile = lpath + "dicc.src";
    opt.NPdataFile = lpath + "np.dat"; 
    opt.PunctuationFile = lpath + "../common/punct.dat"; 
    return opt;



## ----------------------------------------------
## -------------    MAIN PROGRAM  ---------------
## ----------------------------------------------

# set locale to an UTF8 compatible locale 
pyfreeling.util_init_locale("default");

# get requested language from arg1, or English if not provided      
lang = "en"
if len(sys.argv)>1 : lang=sys.argv[1]

# get installation path to use from arg2, or use /usr/local if not provided
ipath = "/usr/local";
if len(sys.argv)>2 : ipath=sys.argv[2]

# path to language data   
lpath = ipath + "/share/freeling/" + lang + "/"

# create analyzers
tk=pyfreeling.tokenizer(lpath+"tokenizer.dat");
sp=pyfreeling.splitter(lpath+"splitter.dat");

# create the analyzer with the required set of maco_options  
morfo=pyfreeling.maco(my_maco_options(lang,lpath));
#  then, (de)activate required modules   
morfo.set_active_options (False,  # UserMap 
                          True,  # NumbersDetection,  
                          True,  # PunctuationDetection,   
                          True,  # DatesDetection,    
                          True,  # DictionarySearch,  
                          True,  # AffixAnalysis,  
                          False, # CompoundAnalysis, 
                          True,  # RetokContractions,
                          True,  # MultiwordsDetection,  
                          True,  # NERecognition,     
                          False, # QuantitiesDetection,  
                          True); # ProbabilityAssignment                 

# create tagger
tagger = pyfreeling.hmm_tagger(lpath+"tagger.dat",True,2)

# process input text
text = "".join(sys.stdin.readlines())

# tokenize input line into a list of words
lw = tk.tokenize(text)
# split list of words in sentences, return list of sentences
ls = sp.split(lw)

# perform morphosyntactic analysis and disambiguation
ls = morfo.analyze(ls)
ls = tagger.analyze(ls)

# do whatever is needed with processed sentences   
ProcessSentences(ls)
```
