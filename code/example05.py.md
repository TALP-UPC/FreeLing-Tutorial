```python

#! /usr/bin/python3

import pyfreeling
import sys

##---------------------------------------------
## Extract lemma and sense of word 'w' and store them
## in 'lem' and 'sens' respectively
##---------------------------------------------
def extract_lemma_and_sense(w) :
   lem = w.get_lemma()
   sens=""
   if len(w.get_senses())>0 :
       sens = w.get_senses()[0][0]
   return lem, sens


## -----------------------------------------------
## Do whatever is needed with analyzed sentences
## -----------------------------------------------
def ProcessSentences(ls) :

    # for each sentence in list
    for s in ls :

       # for each predicate in sentence
       for pred in s.get_predicates() :
          lsubj=""; ssubj=""; ldobj=""; sdobj=""
          # for each argument of the predicate  
          for arg in pred :
              # if the argument is A1, store lemma and synset in ldobj, sdobj
             if arg.get_role()=="A1" : 
                 (ldobj,sdobj) = extract_lemma_and_sense(s[arg.get_position()])
              # if the argument is A0, store lemma and synset in lsubj, subj
             elif arg.get_role()=="A0" :
                 (lsubj,ssubj) = extract_lemma_and_sense(s[arg.get_position()])
             # Get tree node corresponding to the word marked as argument head
             head = s.get_dep_tree().get_node_by_pos(arg.get_position())
             # check if the node has dependency is "by" in passive structure
             if lsubj=="by" and head.get_label=="LGS" :
                 # get first (and only) child, and use it as actual subject 
                 head = head.get_nth_child(0)
                 (lsubj,ssubj) = extract_lemma_and_sense(head.get_word())         

       #if the predicate had both A0 and A1, we found a complete SVO triple. Let's output it.
       if lsubj!="" and ldobj!="" :
          (lpred,spred) = extract_lemma_and_sense(s[pred.get_position()])
          print ("SVO : (pred:   " , lpred, "[" + spred + "]")
          print ("       subject:" , lsubj, "[" + ssubj + "]")
          print ("       dobject:" , ldobj, "[" + sdobj + "]")
          print ("      )")

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
# create dependency parser
parser = pyfreeling.dep_treeler(lpath+"dep_treeler/dependences.dat");

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
# parse sentences
ls = parser.analyze(ls);

# do whatever is needed with processed sentences   
ProcessSentences(ls)
```
