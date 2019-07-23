```python
#! /usr/bin/python3

import pyfreeling
import sys

## -----------------------------------------------
## Do whatever is needed with analyzed sentences
## -----------------------------------------------
def ProcessSentences(ls, sdb):

    # for each sentence in list
    for s in ls :
        # for each word in sentence
        for w in s :
            # print word form  

            print(w.get_form(), w.get_lemma(), w.get_tag(), end="")

            if len(w.get_senses())>0 :
                syns = w.get_senses()[0][0]
                print(" "+syns, end="")
                # move up to parent until a synset with SUMO == "Animal" is found.
                is_top = False
                is_animal = False
                while not is_top and not is_animal: 
                   si = sdb.get_sense_info(syns)  
                   is_animal = (si.sumo == "Animal=");
                   # climb one more node if possible
                   if len(si.parents)>0 : syns = si.parents[0]
                   else : is_top = True

                if is_animal : print("    *** SUMO 'Animal='", end="")
            # next word
            print("")

        # sentence separator
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

# create sense annotator
sen = pyfreeling.senses(lpath+"senses.dat");
# create sense disambiguator
wsd = pyfreeling.ukb(lpath+"ukb.dat");

# process input text
text = "".join(sys.stdin.readlines())

# tokenize input line into a list of words
lw = tk.tokenize(text)
# split list of words in sentences, return list of sentences
ls = sp.split(lw)

# perform morphosyntactic analysis and disambiguation
ls = morfo.analyze(ls)
ls = tagger.analyze(ls)
# annotate and disambiguate senses     
ls = sen.analyze(ls);
ls = wsd.analyze(ls);

# create semantic DB module
sdb = pyfreeling.semanticDB(lpath+"semdb.dat");
# do whatever is needed with processed sentences   
ProcessSentences(ls, sdb)
```
