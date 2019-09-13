
## Example 12: Candidate Corrections for Mispelled Words

FreeLing includes Mans Hulden's [libFOMA](https://code.google.com/archive/p/foma/) Finite-State Technology library, which can be used directly or through FreeLing `compounds` and `alternatives` modules.

The `compounds` module builds a FSM that recognizes the language `L(_L)+`, being `L` the language of a given dictionary. Thus, this module can find very efficiently whether a given word is a compound made of two or more words present in the given dictionary.

FOMA includes also a very fast approximate search algorithm that can return the word in the FSM language closer to the input string. This is used by the `alternatives` module to return a list of candidate words for a string that was not found in the dictionary.

We are going to use [*Example 9*](example09.md) as a starting point.

First, we will modify configuration and invocation options for our `analyzer` to have it perform only morphological analysis, setting `MACO_ProbabilityAssignment = false`. This will deactivate the guesser and thus, unknown words will have no analysis, which will trigger the `alternatives` modules to take action on these words. (Note that this is the default behaviour of `alternatives`, but it can be changed via its configuration file and have it act on all words, only on words with certain PoS, etc...)

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

  // other modules are not needed in this example
  // Sense annotator and WSD config files
  cfg.SENSE_ConfigFile = L"";
  cfg.UKB_ConfigFile = L"";
  // Tagger options
  cfg.TAGGER_HMMFile = L"";
  // Statistical dependency parser & SRL config file
  cfg.DEP_TreelerFile = L"";
  // NEC config file. 
  cfg.NEC_NECFile = L"";
  // Chart parser config file. 
  cfg.PARSER_GrammarFile = L"";
  // Rule based dependency parser config files. 
  cfg.DEP_TxalaFile = L"";
  // Coreference resolution config file.
  cfg.COREF_CorefFile = L"";

  return cfg;
}


///////////////////////////////////////////////////
// Load an ad-hoc set of invoke options

freeling::analyzer::invoke_options fill_invoke() {

  freeling::analyzer::invoke_options ivk;

  // Level of analysis in input and output
  ivk.InputLevel = freeling::TEXT;
  ivk.OutputLevel = freeling::MORFO; 
  
  // activate/deactivate morphological analyzer modules
  ivk.MACO_UserMap = false;
  ivk.MACO_AffixAnalysis = true;
  ivk.MACO_MultiwordsDetection = true;
  ivk.MACO_NumbersDetection = true;
  ivk.MACO_PunctuationDetection = true;
  ivk.MACO_DatesDetection = true;
  ivk.MACO_QuantitiesDetection  = true;
  ivk.MACO_DictionarySearch = true;
  ivk.MACO_CompoundAnalysis = false;
  ivk.MACO_NERecognition = true;
  ivk.MACO_RetokContractions = false;

  // deactivate guesser so words with no analysis do not get one
  // and can be processed by the alternative proposers
  ivk.MACO_ProbabilityAssignment = false;
    
  // other modules are not used in this examples
  ivk.NEC_NEClassification = false; 
  ivk.SENSE_WSD_which = freeling::NO_WSD;
  ivk.TAGGER_which = freeling::NO_TAGGER;
  ivk.DEP_which = freeling::NO_DEP;

  return ivk;
}

```

Next, we will create two instances of the `alternatives` module:  One using the morphological dictionary and a basic String Edit Distance (SED) matrix to find the closest words, and another using a phonetic dictionary and a phonetic-based SED matrix.

```C++
// create alternative porposers
freeling::alternatives alts_ort(ipath+L"/share/freeling/"+lang+L"/alternatives-ort.dat");
// comment this out if there is no phonetic encoder for target language
freeling::alternatives alts_phon(ipath+L"/share/freeling/"+lang+L"/alternatives-phon.dat");
```

You can tune the behaviour of these modules by changing the options on their respective configuration files. 
See the [user manual](https://talp-upc.gitbooks.io/freeling-user-manual/content/modules/alternatives.html) for details.

Once the alternative suggestion modules have been created, we only need to run the document through them, as we would do with any other module:

```C++
// propose alternative forms based on ortographical distance
alts_ort.analyze(ls);
// propose alternative forms based on phonetic distance.
// Comment this out if there is no phonetic encoder for your language    
alts_phon.analyze(ls);
```

Then we can check and use the obtained alternatives (e.g. to pick the best one, or simply to dump them as in this example)

```C++
// for each sentence
for (list<freeling::sentence>::iterator s=ls.begin(); s!=ls.end(); s++) {
   // for each word
   for (freeling::sentence::iterator w=s->begin(); w!=s->end(); w++) {
      // print form, and analysis proposed by the morphological analyzer (if any)
      wcout<<L"FORM: "<<w->get_form()<<endl; 
      wcout<<L"   ANALYSIS:";
      for (freeling::word::iterator a=w->begin(); a!=w->end(); a++) 
         wcout<<L" ["<<a->get_lemma()<<L","<<a->get_tag()<<L"]";
      wcout<<endl;

      // print alternative forms proposed by the alternative suggestors
      wcout<<L"   ALTERNATIVE FORMS:";
      for (list<pair<wstring,int> >::iterator a=w->alternatives_begin(); a!=w->alternatives_end(); a++) 
        wcout<<L" ["<<a->first<<L","<<a->second<<L"]";
      wcout<<endl; 
   }
   wcout<<endl;
}
```


### Code

Find here the whole code:
    - In [C++](code/example12.cc.md)
    - In [python](code/example12.py.md)


### Example

Assuming the input file contains the following sentences:

    The beeg catt eates freshh feesh.

we would obtain the output:
```
FORM: the
   ANALYSIS: [the,DT]
   ALTERNATIVE FORMS:
FORM: beeg
   ANALYSIS:
   ALTERNATIVE FORMS: [big,100] [beg,350] [bag,600] [bead,800] [bug,850] [bid,900] [dig,900] 
                      [beak,1000] [beef,1000] [been,1000] [beep,1000] [beer,1000] [bees,1000] 
                      [beet,1000] [berg,1000] [bee,1000] [be,1000] [bing,1100] [mig,1100] [pig,1100] 
                      [bed,1150] [deg,1150]
FORM: catt
   ANALYSIS:
   ALTERNATIVE FORMS: [cat,0] [kat,0] [kas,500] [kit,500] [cot,600] [cut,750] [cap,800] [batt,1000]
                      [cant,1000] [cart,1000] [cash,1000] [cast,1000] [cath,1000] [cats,1000] 
                      [kats,1000] [khat,1000] [kiss,1000] [kite,1000] [matt,1000] [watt,1000] 
                      [att,1000] [cad,1000] [can't,1000] [caste,1000] [catch,1000] [gat,1000] 
                      [capped,1000] [cashed,1000] [cattle,1000]
FORM: eates
   ANALYSIS:
   ALTERNATIVE FORMS: [eases,500] [ekes,800] [bates,1000] [dates,1000] [eaten,1000] [eater,1000] 
                      [eaves,1000] [fates,1000] [fetes,1000] [gates,1000] [hates,1000] [mates,1000] 
                      [metes,1000] [pates,1000] [rates,1000] [sates,1000] [eaters,1000] [eats,1000] 
                      [elates,1000] [assess,1100] [it's,1100] [itches,1100] [its,1100]
FORM: freshh
   ANALYSIS:
   ALTERNATIVE FORMS: [fresh,1000] 
FORM: feesh
   ANALYSIS:
   ALTERNATIVE FORMS: [fish,100] [phis,600] [fes,850] [flesh,1000] [fresh,1000] [feat,1000] [fees,1000] 
                      [feet,1000] [fete,1000] [fee,1000] [fey,1000] [cease,1100] [fishy,1100] [fished,1100] 
                      [fit,1100] [peace,1200] [piece,1200] [pees,1200] [sash,1200] 
FORM: .
   ANALYSIS: [.,Fp]
   ALTERNATIVE FORMS:
```

