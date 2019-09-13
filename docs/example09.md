
## Example 09: Simpler code with `analyzer` meta-module

This example shows how to use `analyzer` meta-module to create a single instance that contains all required analysis modules, this simplifying the code and the management language processing module instances.

The drawback is that configuration for the meta-module needs to contain information about configuration of each module that it contains. For this reason, there are two additional classes that will hold the needed configuration options.

We will write a program that performs syntactic parsing, as the example presented in [*Example 05*](example05.md), but instead of calling every module, we will use a single instance of `analyzer` meta-module.

For that, we need to fill the required configuration options, and use them to create an instace of `analyzer`.
```C++
// set config options (which modules to create, with which configuration)
freeling::analyzer::config_options cfg = fill_config(lang, ipath);
// create analyzer
freeling::analyzer anlz(cfg);
```

The instance of `analyzer::config_options` will contain all instantiation time options (e.g. configuration files to be used to create each analysis module).

 Once the analyzer is created, we may set or change its invoke options.
 This class contains options that may be changed in run time (as for instance activating or deactivating a module, or changing the target level of analysis). 

```C++
// set invoke options (which modules to use)
freeling::analyzer::invoke_options ivk = fill_invoke();
// load invoke options into analyzer
anlz.set_current_invoke_options(ivk);
```

Once the analyzer is created and configured, we can use it to process text:
```C++
// load document to analyze
wstring text;  
wstring line;
while (getline(wcin,line)) 
   text = text + line + L"\n";

// analyze text, leave result in ls
list<freeling::sentence> ls;
anlz.analyze(text,ls);

// do whatever is needed with the results
ProcessSentences(ls);
```

If we use the `ProcessSentences` function from [*Example 05*](example05.md), we will get the same results with a much shorter code.


All we need to complete the program are the functions `fill_config` and `fill_invoke` that will set the needed options.

In this example, we are hardcoding the options, but they could be loaded from some configuration file, or set in any other way your application may need.

One important remark about `config_options` is that it determines which modules are instantiated. If the configuration file for a module is left blank, it will not be loaded, and thus it will not be available later. 
If it is loaded, the application will be able to choose via `invoke_options` whether it must used or not.

```C++
///////////////////////////////////////////////////
// Load an ad-hoc set of configuration options

freeling::analyzer::config_options fill_config(const wstring &lang, const wstring &ipath) {

  freeling::analyzer::config_options cfg;

  // Language of text to process
  cfg.Lang = lang;
 
  // path to language specific data
  wstring lpath = ipath + L"/share/freeling/" + cfg.Lang + L"/";

  // Tokenizer configuration file
  cfg.TOK_TokenizerFile = lpath + L"tokenizer.dat";
  // Splitter configuration file
  cfg.SPLIT_SplitterFile = lpath + L"splitter.dat";
  // Morphological analyzer options
  cfg.MACO_Decimal = L".";
  cfg.MACO_Thousand = L",";
  cfg.MACO_LocutionsFile = lpath + L"locucions.dat";
  cfg.MACO_QuantitiesFile = lpath + L"quantities.dat";
  cfg.MACO_AffixFile = lpath + L"afixos.dat";
  cfg.MACO_ProbabilityFile = lpath + L"probabilitats.dat";
  cfg.MACO_DictionaryFile = lpath + L"dicc.src";
  cfg.MACO_NPDataFile = lpath + L"np.dat";
  cfg.MACO_PunctuationFile = lpath + L"../common/punct.dat";
  cfg.MACO_ProbabilityThreshold = 0.001;

  // Sense annotator and WSD config files
  cfg.SENSE_ConfigFile = lpath + L"senses.dat";
  cfg.UKB_ConfigFile = lpath + L"ukb.dat";
  // Tagger options
  cfg.TAGGER_HMMFile = lpath + L"tagger.dat";
  cfg.TAGGER_ForceSelect = freeling::RETOK;
  // Statistical dependency parser & SRL config file
  cfg.DEP_TreelerFile = lpath + L"dep_treeler/dependences.dat";   

  // NEC config file. This module will not be loaded
  cfg.NEC_NECFile = L"";
  // Chart parser config file. This module will not be loaded
  cfg.PARSER_GrammarFile = L"";
  // Rule based dependency parser config files. This module will not be loaded
  cfg.DEP_TxalaFile = L"";
  // Coreference resolution config file. This module will not be loaded
  cfg.COREF_CorefFile = L"";

  return cfg;
}
```


And function `fill_invoke` could look like:
```C++
///////////////////////////////////////////////////
// Load an ad-hoc set of invoke options

freeling::analyzer::invoke_options fill_invoke() {

  freeling::analyzer::invoke_options ivk;

  // Level of analysis in input and output
  ivk.InputLevel = freeling::TEXT;
  // We can not request higher analysis levels (e.g. coreference) because 
  // we didn't load the needed modules.
  // But we can use this option to lowe the analysis level at will during
  // our application execution.
  ivk.OutputLevel = freeling::DEP; 
  
  // activate/deactivate morphological analyzer modules
  ivk.MACO_UserMap = false;
  ivk.MACO_AffixAnalysis = true;
  ivk.MACO_MultiwordsDetection = true;
  ivk.MACO_NumbersDetection = true;
  ivk.MACO_PunctuationDetection = true;
  ivk.MACO_DatesDetection = true;
  ivk.MACO_QuantitiesDetection  = true;
  ivk.MACO_DictionarySearch = true;
  ivk.MACO_ProbabilityAssignment = true;
  ivk.MACO_CompoundAnalysis = false;
  ivk.MACO_NERecognition = true;
  ivk.MACO_RetokContractions = false;
    
  ivk.SENSE_WSD_which = freeling::UKB;
  ivk.TAGGER_which = freeling::HMM;

  // since we only created dep_treeler parser, we can not set the parser to use to another one.
  // If we had loaded both parsers, we could change the used parsed at will with this option
  ivk.DEP_which = freeling::TREELER;

  // since we did not load the module, setting this to true would trigger an error.
  // if the module was created, we could activate/deactivate it at will with this option.
  ivk.NEC_NEClassification = false; 

  return ivk;
}
```


### Code

Find here the whole code:
    - In [C++](code/example09.cc.md)
    - In [python](code/example09.py.md)

