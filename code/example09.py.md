```python
#! /usr/bin/python3

import pyfreeling
import sys

##---------------------------------------------
## Load an ad-hoc set of configuration options
##---------------------------------------------

def fill_config(lang, ipath) :

  cfg = pyfreeling.config_options()

  # Language of text to process
  cfg.Lang = lang
 
  # path to language specific data
  lpath = ipath + "/share/freeling/" + cfg.Lang + "/"

  # Tokenizer configuration file
  cfg.TOK_TokenizerFile = lpath + "tokenizer.dat"
  # Splitter configuration file
  cfg.SPLIT_SplitterFile = lpath + "splitter.dat"
  # Morphological analyzer options
  cfg.MACO_Decimal = "."
  cfg.MACO_Thousand = ","
  cfg.MACO_LocutionsFile = lpath + "locucions.dat"
  cfg.MACO_QuantitiesFile = lpath + "quantities.dat"
  cfg.MACO_AffixFile = lpath + "afixos.dat"
  cfg.MACO_ProbabilityFile = lpath + "probabilitats.dat"
  cfg.MACO_DictionaryFile = lpath + "dicc.src"
  cfg.MACO_NPDataFile = lpath + "np.dat"
  cfg.MACO_PunctuationFile = lpath + "../common/punct.dat"
  cfg.MACO_ProbabilityThreshold = 0.001

  # Sense annotator and WSD config files
  cfg.SENSE_ConfigFile = lpath + "senses.dat"
  cfg.UKB_ConfigFile = lpath + "ukb.dat"
  # Tagger options
  cfg.TAGGER_HMMFile = lpath + "tagger.dat"
  cfg.TAGGER_ForceSelect = pyfreeling.RETOK
  # Statistical dependency parser & SRL config file
  cfg.DEP_TreelerFile = lpath + "dep_treeler/dependences.dat"   

  # NEC config file. This module will not be loaded
  cfg.NEC_NECFile = ""
  # Chart parser config file. This module will not be loaded
  cfg.PARSER_GrammarFile = ""
  # Rule based dependency parser config files. This module will not be loaded
  cfg.DEP_TxalaFile = ""
  # Coreference resolution config file. This module will not be loaded
  cfg.COREF_CorefFile = ""

  return cfg



##---------------------------------------------
## Load an ad-hoc set of invoke options
##---------------------------------------------

def fill_invoke() :

  ivk = pyfreeling.invoke_options()

  # Level of analysis in input and output
  ivk.InputLevel = pyfreeling.TEXT
  # We can not request higher analysis levels (e.g. coreference) because 
  # we didn't load the needed modules.
  # But we can use this option to lowe the analysis level at will during
  # our application execution.
  ivk.OutputLevel = pyfreeling.DEP 
  
  # activate/deactivate morphological analyzer modules
  ivk.MACO_UserMap = False
  ivk.MACO_AffixAnalysis = True
  ivk.MACO_MultiwordsDetection = True
  ivk.MACO_NumbersDetection = True
  ivk.MACO_PunctuationDetection = True
  ivk.MACO_DatesDetection = True
  ivk.MACO_QuantitiesDetection  = True
  ivk.MACO_DictionarySearch = True
  ivk.MACO_ProbabilityAssignment = True
  ivk.MACO_CompoundAnalysis = False
  ivk.MACO_NERecognition = True
  ivk.MACO_RetokContractions = False
    
  ivk.SENSE_WSD_which = pyfreeling.UKB
  ivk.TAGGER_which = pyfreeling.HMM

  # since we only created dep_treeler parser, we can not set the parser to use to another one.
  # If we had loaded both parsers, we could change the used parsed at will with this option
  ivk.DEP_which = pyfreeling.TREELER

  # since we did not load the module, setting this to true would trigger an error.
  # if the module was created, we could activate/deactivate it at will with this option.
  ivk.NEC_NEClassification = False 

  return ivk


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

# set config options (which modules to create, with which configuration)
cfg = fill_config(lang, ipath)
# create analyzer
anlz = pyfreeling.analyzer(cfg)

# set invoke options (which modules to use. Can be changed in run time)
ivk = fill_invoke()
# load invoke options into analyzer
anlz.set_current_invoke_options(ivk)

# load input text
text = "".join(sys.stdin.readlines())

#ls = anlz.analyze_as_sentences(str(text),True)
ls = anlz.analyze(text,True)

# do whatever is needed with processed sentences   
ProcessSentences(ls)
```
