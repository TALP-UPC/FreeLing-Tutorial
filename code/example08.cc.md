```C++
#include <iostream>
#include "freeling.h"
using namespace std;

//---------------------------------------------
// Do whatever is needed with analyzed sentences
//---------------------------------------------

void ProcessSentences(const list<freeling::sentence> &ls) {

  // for each sentence in list
  for (list<freeling::sentence>::const_iterator s=ls.begin(); s!=ls.end(); ++s) {
    for  (freeling::dep_tree::const_iterator p=s->get_dep_tree().begin(); p!=s->get_dep_tree().end(); p++) {
      // if it is a verb, check dependants
      if  (p->get_word().get_tag()[0]==L'V') {
        wstring ssubj, lsubj;
        wstring sdobj, ldobj;
        for (freeling::dep_tree::const_sibling_iterator ch = p.sibling_begin(); ch!=p.sibling_end(); ch++) { 
          if (ch->get_label()==L"SBJ") {
            lsubj = ch->get_word().get_lemma();
            if (not ch->get_word().get_senses().empty()) 
              ssubj = ch->get_word().get_senses().begin()->first;
          }
          else if (ch->get_label()==L"OBJ") {
            ldobj = ch->get_word().get_lemma();
            if (not ch->get_word().get_senses().empty()) 
              sdobj = ch->get_word().get_senses().begin()->first;
          }
        }
        // if we found a SVO triple, output it
        if (lsubj!=L"" and ldobj!=L"") {
          wstring spred, lpred;
          lpred = p->get_word().get_lemma();
          if (not p->get_word().get_senses().empty()) 
            spred = p->get_word().get_senses().begin()->first;
          wcout << L"SVO : (pred:    " << lpred << L" [" << spred << L"]" << endl;
          wcout << L"       subject: " << lsubj << L" [" << ssubj << L"]" << endl; 
          wcout << L"       dobject: " << ldobj << L" [" << sdobj << L"]" << endl;
          wcout << L"      )" << endl;
        }
      }
    }
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
  freeling::splitter::session_id sid=sp.open_session();

  // create the analyzer with the required set of maco_options
  freeling::maco morfo(my_maco_options(lang,lpath)); 
  // then, (de)activate required modules
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
  // create dependency parser
  freeling::dep_treeler parser(lpath+L"dep_treeler/dependences.dat");

 // get plain text input lines while not EOF.
  wstring text;
  while (getline(wcin,text)) {

    // tokenize input line into a list of words
    list<freeling::word> lw=tk.tokenize(text);
    
    // accumulate list of words in splitter buffer, returning a list of sentences.
    list<freeling::sentence> ls=sp.split(sid, lw, false);
    
    // perform and output morphosyntactic analysis and disambiguation
    morfo.analyze(ls);
    tagger.analyze(ls);
    sen.analyze(ls);
    wsd.analyze(ls);
    parser.analyze(ls);
 
    // do whatever is needed with processed sentences   
    ProcessSentences(ls);
  }

  // No more lines to read. Make sure the splitter doesn't retain anything  
  list<freeling::word> lw; 
  list<freeling::sentence> ls = sp.split(sid, lw, true);
  sp.close_session(sid);
 
  // analyze and process sentence(s) which might be lingering in the buffer, if any.
  morfo.analyze(ls);
  tagger.analyze(ls);
  parser.analyze(ls);
  sen.analyze(ls);
  wsd.analyze(ls);
  ProcessSentences(ls); 
}
```
