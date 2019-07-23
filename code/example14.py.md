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
  # Tagger options
  cfg.TAGGER_HMMFile = lpath + "tagger.dat"
  cfg.TAGGER_ForceSelect = pyfreeling.RETOK

  # other modules are not needed in this example

  # Sense annotator and WSD config files
  cfg.SENSE_ConfigFile = ""
  cfg.UKB_ConfigFile = ""
  # Tagger options
  cfg.TAGGER_HMMFile = ""
  # Statistical dependency parser & SRL config file
  cfg.DEP_TreelerFile = ""

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
  ivk.OutputLevel = pyfreeling.MORFO 
  
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
  ivk.TAGGER_which = pyfreeling.HMM

  # other modules are not used in this examples
  ivk.NEC_NEClassification = False
  ivk.SENSE_WSD_which = pyfreeling.NO_WSD
  ivk.DEP_which = pyfreeling.NO_DEP

  return ivk

## -----------------------------------------------
## Do whatever is needed with analyzed sentences
## -----------------------------------------------
def ProcessSentences(ls, fname) :

  fextractor = pyfreeling.fex(fname, "")
  lex = pyfreeling.fex_lexicon()

  # for each sentence in list
  ns = 0
  for s in ls :
    # extract features names each word in the sentence
    feats = fextractor.encode_name(s)

    print ("sentence_"+str(ns), end="")
    i = 0
    for w in s :
      # print features for encoded sentence, and store them in the lexicon
      for j in feats[i] :
        lex.add_occurrence(j)
        print(" "+j, end="")
      i = i+1

    print("")
    ns = ns+1

  #  save lexicon 
  lex.save_lexicon(fname.replace(".rgf", ".lex"), 0);


## ----------------------------------------------
## -------------    MAIN PROGRAM  ---------------
## ----------------------------------------------

# set locale to an UTF8 compatible locale 
pyfreeling.util_init_locale("default");

if len(sys.argv) < 2 :
  print("Usage:  ", sys.argv[0], "file.rgf [language] [Freeling-dir]")
  exit(1)

# get name of feature extraction rules file.   
rgfFile = sys.argv[1]

# get requested language from arg1, or English if not provided      
lang = "en"
if len(sys.argv)>2 : lang=sys.argv[2]

# get installation path to use from arg2, or use /usr/local if not provided
ipath = "/usr/local";
if len(sys.argv)>3 : ipath=sys.argv[3]

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
ProcessSentences(ls, rgfFile)
```
