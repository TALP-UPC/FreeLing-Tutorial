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
   if len(w.get_senses())>0 : sens = w.get_senses()[0][0]
   return lem, sens

## -----------------------------------------------
## Do whatever is needed with analyzed sentences
## -----------------------------------------------
def ProcessSentences(ls):

    # for each sentence in list
    for s in ls :

        # for each node in dependency tree
        dt = s.get_dep_tree()
        node = dt.begin()
        while node != dt.end() :
            ssubj=""; lsubj=""; sdobj=""; ldobj=""
            # if it is a verb, check dependants
            if node.get_word().get_tag()[0]=='V' :
                for ch in range(0,node.num_children()) :
                    child = node.nth_child(ch)
                    if child.get_label()=="SBJ" :
                       (lsubj,ssubj) = extract_lemma_and_sense(child.get_word())
                    elif child.get_label()=="OBJ" :
                       (ldobj,sdobj) = extract_lemma_and_sense(child.get_word())

                if lsubj!="" and ldobj!="" :
                   (lpred,spred) =  extract_lemma_and_sense(node.get_word())
                   print ("SVO : (pred:   " , lpred, "[" + spred + "]")
                   print ("       subject:" , lsubj, "[" + ssubj + "]")
                   print ("       dobject:" , ldobj, "[" + sdobj + "]")
                   print ("      )")

            node.incr()



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
sid=sp.open_session()

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
text = sys.stdin.readline()
while text!="" :
   # tokenize input line into a list of words
   lw = tk.tokenize(text)
   # accumulate list of words in splitter buffer, returning a list of sentences
   ls = sp.split(sid, lw, False)

   # perform and output morphosyntactic analysis and disambiguation 
   ls = morfo.analyze(ls);
   ls = tagger.analyze(ls);
   ls = sen.analyze(ls)
   ls = wsd.analyze(ls)
   ls = parser.analyze(ls);

   # do whatever is needed with processed sentences 
   ProcessSentences(ls)

   # next input line
   text = sys.stdin.readline()

# No more lines to read. Make sure the splitter doesn't retain anything
lw = []
ls = sp.split(sid, lw, True)
sp.close_session(sid)

# analyze and process sentence(s) which might be lingering in the buffer, if any
ls = morfo.analyze(ls);
ls = tagger.analyze(ls);
ls = sen.analyze(ls)
ls = wsd.analyze(ls)
ls = parser.analyze(ls);

# do whatever is needed with processed sentences   
ProcessSentences(ls)
```
