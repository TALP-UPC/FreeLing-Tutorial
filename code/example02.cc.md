```C++
#include <iostream>
#include <map>
#include "freeling.h"
using namespace std;

//---------------------------------------------
// Increase the count of key w in map m, or
// set it to 1 if not found
//---------------------------------------------

void add_count (map<wstring,int> &m, const wstring &w) {
   map<wstring,int>::iterator p = m.find(w);

   // key not found in the map, add it with count=1
   if (p==m.end()) m.insert(make_pair(w,1));
   // key previously seen, increase count
   else ++p->second;
}

//---------------------------------------------
// Do whatever is needed with analyzed sentences
//---------------------------------------------

void ProcessSentences(const list<freeling::sentence> &ls) {
   // maps to count lemmas
   map<wstring,int> lemmas;
   map<wstring,int> lemma_bigrams;

   // for each sentence in the list
   for (list<freeling::sentence>::const_iterator s=ls.begin(); s!=ls.end(); ++s) {
      // previous word lemma, for the bigrams
      wstring prev=L"";
      // for each word in the sentence
      for (freeling::sentence::const_iterator w=s->begin(); w!=s->end(); ++w) {
        // count an occurrence more for the current word lemma
        add_count (lemmas, w->get_lemma());
        // count an occurrence more for the pair of previous and current word lemmas.
        if (prev!=L"")
          add_count(lemma_bigrams, prev+L" "+w->get_lemma());
        
        // move to next word
        prev = w->get_lemma();
      }
   }

   // once all lemmas and bigrams are counted, output the results
   wcout << L"========== LEMMA FREQUENCIES (lemma,freq) ====================" << endl;
   for (map<wstring,int>::iterator p=lemmas.begin(); p!=lemmas.end(); ++p)
   	  wcout << p->first << L" " << p->second << endl;  
   wcout << endl;
   wcout << L"========== LEMMA BIGRAM FREQUENCIES (lemma1,lemma2,freq) ==============" << endl;
   for (map<wstring,int>::iterator p=lemma_bigrams.begin(); p!=lemma_bigrams.end(); ++p)
   	  wcout << p->first << L" " << p->second << endl;  
   wcout << endl;
}

//---------------------------------------------
// Set desired options for morphological analyzer
//---------------------------------------------

freeling::maco_options my_maco_options (const wstring &lang, const wstring &lpath) {
  // create options holder 
  freeling::maco_options opt(lang);
  // Provide files for morphological submodules. Note that it is not necessary
  // to set files for modules that will not be used
  opt.UserMapFile = L"";
  opt.LocutionsFile = lpath + L"locucions.dat"; 
  opt.AffixFile = lpath + L"afixos.dat";
  opt.ProbabilityFile = lpath + L"probabilitats.dat"; 
  opt.DictionaryFile = lpath + L"dicc.src";
  opt.NPdataFile = lpath + L"np.dat"; 
  opt.PunctuationFile = lpath + L"../common/punct.dat"; 
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
  

  // get all input text in a single string 
  wstring text=L"";
  wstring line;
  while (getline(wcin,line))
    text = text + line + L"\n";


  // tokenize input line into a list of words
  list<freeling::word> lw = tk.tokenize(text);
  // accumulate list of words in splitter buffer, returning a list of sentences.
  list<freeling::sentence> ls = sp.split(lw);
  // perform morphosyntactic analysis and disambiguation
  morfo.analyze(ls);
  tagger.analyze(ls);

  // do whatever is needed with processed sentences
  ProcessSentences(ls);
}
```
