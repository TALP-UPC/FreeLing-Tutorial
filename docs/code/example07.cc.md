```C++
#include <iostream>
#include "freeling.h"
using namespace std;

//---------------------------------------------
// Extract lemma and sense of word 'w' and store them
// in 'lem' and 'sens' respectively
//---------------------------------------------

void extract_lemma_and_sense(const freeling::word &w, wstring &lem, wstring &sens) {
   lem = w.get_lemma();
   if (not w.get_senses().empty()) sens = w.get_senses().begin()->first;
   else sens=L"";
}

//---------------------------------------------
// Do whatever is needed with analyzed sentences
//---------------------------------------------

void ProcessSentences(const list<freeling::sentence> &ls, const freeling::semanticDB &sdb) {

  // for each sentence in list
  for (list<freeling::sentence>::const_iterator s=ls.begin(); s!=ls.end(); ++s) {

    // for each predicate in sentence
    for (freeling::sentence::predicates::const_iterator pred=s->get_predicates().begin(); pred!=s->get_predicates().end(); ++pred) { 
      // extract lema and sense for the predicate

      wstring lsubj=L"", ssubj=L"", ldobj=L"", sdobj=L"";
      // for each argument of the predicate
      for (freeling::predicate::const_iterator arg=pred->begin(); arg!=pred->end(); ++arg) {

         // if the argument is A1, store lemma and synset in ldobj, sdobj
         if (arg->get_role()==L"A1") {
           extract_lemma_and_sense((*s)[arg->get_position()], ldobj, sdobj);
         }

         // if the argument is A0, store lemma and synset in lsubj, ssubj
         else if (arg->get_role()==L"A0") {
           extract_lemma_and_sense((*s)[arg->get_position()], lsubj, ssubj);

           // Get tree node corresponding to the word marked as argument head:
           freeling::dep_tree::const_iterator head = s->get_dep_tree().get_node_by_pos(arg->get_position());
           // check if the node has dependency is "by" in passive structure
           if (lsubj == L"by" and head->get_label() == L"LGS") {
             // advance to its child (it is a preorder iterator, so advancing 
             // one step moves us to the child)
             ++head;
             // use child as actual subject
             extract_lemma_and_sense(head->get_word(), lsubj, ssubj);
           }
         }
      }
      
      // if the predicate had both A0 and A1, we found a complete SVO triple. Let's output it.
      if (lsubj!=L"" and ldobj!=L"") {        
        wstring lpred, spred;
        extract_lemma_and_sense((*s)[pred->get_position()], lpred, spred);
        // if we found a synset for the predicate, obtain lemma synonyms and SUMO link
        if (spred!=L"") {
          freeling::sense_info ipred = sdb.get_sense_info(spred);
          lpred =  freeling::util::list2wstring(ipred.words,L"/") + L" [" + ipred.sumo + L"]";
        }
        // if we found a synset for the subject, obtain lemma synonyms and SUMO link
        if (ssubj!=L"") {
          freeling::sense_info isubj = sdb.get_sense_info(ssubj);
          lsubj =  freeling::util::list2wstring(isubj.words,L"/") + L" [" + isubj.sumo + L"]";
        } 
        // if we found a synset for the object, obtain lemma synonyms and SUMO link
        if (sdobj!=L"") {
          freeling::sense_info idobj = sdb.get_sense_info(sdobj);
          ldobj =  freeling::util::list2wstring(idobj.words,L"/") + L" [" + idobj.sumo + L"]";
        }
        wcout << L"SVO : (pred:    " << lpred << endl;
        wcout << L"       subject: " << lsubj << endl; 
        wcout << L"       dobject: " << ldobj << endl;
        wcout << L"      )" << endl;
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

  // create the analyzer with the required set of maco_options
  freeling::maco_options opt = my_maco_options(lang,lpath);
  freeling::maco morfo(opt);

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
  // parse sentences
  parser.analyze(ls);

  // create semantic DB module
  freeling::semanticDB sdb(lpath+L"semdb.dat");
  // do whatever is needed with processed sentences
  ProcessSentences(ls,sdb);
}

```
