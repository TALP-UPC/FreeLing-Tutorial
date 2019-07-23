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
  ivk.MACO_CompoundAnalysis = False
  ivk.MACO_NERecognition = True
  ivk.MACO_RetokContractions = False
    
  # deactivate guesser so words with no analysis do not get one 
  # and can be processed by the alternative proposers
  ivk.MACO_ProbabilityAssignment = False
  
  # other modules are not used in this example
  ivk.NEC_NEClassification = False 
  ivk.SENSE_WSD_which = pyfreeling.NO_WSD
  ivk.TAGGER_which = pyfreeling.NO_TAGGER
  ivk.DEP_which = pyfreeling.NO_DEP

  return ivk


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

# create alternative porposers
alts_ort = pyfreeling.alternatives(ipath+"/share/freeling/"+lang+"/alternatives-ort.dat")
# comment this out if there is no phonetic encoder for target language
alts_phon = pyfreeling.alternatives(ipath+"/share/freeling/"+lang+"/alternatives-phon.dat");

# load input text
text = "".join(sys.stdin.readlines())

#ls = anlz.analyze_as_sentences(str(text),True)
ls = anlz.analyze(text,True)

ls = alts_ort.analyze(ls)
ls = alts_phon.analyze(ls)

# print results, for each sentence
for s in ls :
  # for each word
  for w in s :
    # print form, and analysis proposed by the morphological analyzer (if any)
    print("FORM:",w.get_form())
    print("   ANALYSIS:", end="")
    for a in w :
      print (" ["+a.get_lemma()+","+a.get_tag()+"]", end="")
    print("")
    
    # print alternative forms proposed by the alternative suggestors
    print("   ALTERNATIVE FORMS:")
    for a in w.get_alternatives() :
      print(" ["+a.get_form()+","+str(a.get_distance())+"]", end="")
    print("")

  # sentence separator
  print("")


```
