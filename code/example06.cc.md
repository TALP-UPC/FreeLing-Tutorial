```C++
#include <iostream>
#include "freeling.h"
using namespace std;

//---------------------------------------------
// Do whatever is needed with analyzed sentences
//---------------------------------------------

void ProcessSentences(const list<freeling::sentence> &ls, const freeling::semanticDB &sdb) {

  // for each sentence in list
  for (list<freeling::sentence>::const_iterator is=ls.begin(); is!=ls.end(); is++) {

    // for each word in sentence
    for (freeling::sentence::const_iterator w=is->begin(); w!=is->end(); w++) {
      
      wcout << w->get_form() << L" " << w->get_lemma() << L" " << w->get_tag() ;
      if (not w->get_senses().empty()) {
        wstring syns = w->get_senses().begin()->first;      
        wcout << L" " << syns;

        // move up to parent until a synset with SUMO == "Animal" is found.
        bool is_top = false;
        bool is_animal = false;
        while (not is_top and not is_animal) {
          // get additional information about word sense
          freeling::sense_info si = sdb.get_sense_info(syns);
          // check whether it is what we are looking for
          is_animal = (si.sumo == L"Animal=");

          // check whether we can still climb one more step, or we reached the top node.
          if (not si.parents.empty()) syns = si.parents.front();
          else is_top = true;
        }

        if (is_animal) wcout << L"    *** SUMO 'Animal='" ;
      }
      wcout << endl; // next word
    }
    wcout << endl; // sentence separator
  }
}

//---------------------------------------------
// Set desired options for morphological analyzer
//---------------------------------------------

freeling::maco_options my_maco_options (const wstring &lang, const wstring &lpath) {
  // create options holder 
  freeling::maco_options opt(lang);
  // Provide files for morphological submodules. Note that it is not necessary
  // to set files for modules that will not be used
  opt.UserMapFile=L"";
  opt.LocutionsFile=lpath+L"locucions.dat"; opt.AffixFile=lpath+L"afixos.dat";
  opt.ProbabilityFile=lpath+L"probabilitats.dat"; opt.DictionaryFile=lpath+L"dicc.src";
  opt.NPdataFile=lpath+L"np.dat"; opt.PunctuationFile=lpath+L"../common/punct.dat"; 
  return opt;
}


/////////////   MAIN PROGRAM  /////////////////////


int main (int argc, char **argv) {

  // set locale to an UTF8 compatible locale
  freeling::util::init_locale(L"default");

  // get requested language from arg1, or English if not provided
  wstring lang = L"en";
  if (argc > 1) lang = freeling::util::string2wstring(argv[1]);
  // get installation path to use from arg2, or use /usr/local if not provided
  wstring ipath = L"/usr/local";
  if (argc > 2) ipath = freeling::util::string2wstring(argv[2]);

  // path to language data
  wstring lpath = ipath+L"/share/freeling/"+lang+L"/";

  // create analyzers
  freeling::tokenizer tk(lpath+L"tokenizer.dat"); 
  freeling::splitter sp(lpath+L"splitter.dat");

  // create the analyzer with the required set of maco_options
  freeling::maco_options opt = my_maco_options(lang,lpath)
  freeling::maco morfo(opt);  // then, (de)activate required modules

  morfo.set_active_options (false,  // UserMap
                            true,  // NumbersDetection,
                            true,  // PunctuationDetection,
                            true,  // DatesDetection,
                            true,  // DictionarySearch,
                            true,  // AffixAnalysis,
                            false, // CompoundAnalysis,
                            true,  // RetokContractions,
                            true,  // MultiwordsDetection,
                            true,  // NERecognition,
                            false, // QuantitiesDetection,
                            true); // ProbabilityAssignment

  // create a hmm tagger for spanish (with retokenization ability, and forced 
  // to choose only one tag per word)
  freeling::hmm_tagger tagger(lpath+L"tagger.dat", true, FORCE_TAGGER);

  // create sense annotator
  freeling::senses sen(lpath+L"senses.dat");
  // create sense disambiguator
  freeling::ukb wsd(lpath+L"ukb.dat");

  // get all input text in a single string 
  wstring text=L"";
  wstring line;
  while (getline(wcin,line))
     text = text + line + L"\n";

  // tokenize input line into a list of words
  list<freeling::word> lw = tk.tokenize(text);
  // accumulate list of words in splitter buffer, returning a list of sentences.
  list<freeling::sentence> ls = sp.split(lw);
  // perform and output morphosyntactic analysis and disambiguation
  morfo.analyze(ls);
  tagger.analyze(ls);
  // annotate and disambiguate senses
  sen.analyze(ls);
  wsd.analyze(ls);

  // create semantic DB module
  freeling::semanticDB sdb(lpath+L"semdb.dat");
  // do whatever is needed with processed sentences
  ProcessSentences(ls, sdb);
}

```
