
## Example 14: Feature Extractor

NLP applications often use Machine Learning techniques to take decision about sentences or documents (e.g. Sentiment Analysis is usually approached as a ML classification task, where each sentence or paragraph is classified as positive/neutral/negative).

Machine learning algorithms need to be fed with feature vector representations of the objects being classified (i.e., sentences, paragraphs, or documents).

FreeLing includes a feature extraction module (`fex`) that eases the task of converting a text to a set of feature vectors. The module accepts feature pattern definitions that are applied over all words in the text (see [user manual](https://talp-upc.gitbooks.io/freeling-user-manual/content/modules/fex.html) for details).

In this example we are going to illustrate how to convert a text into a set of features using the `fex` module.

We will use [*Example 9*](example09.md) as a starting point, and modify the `ProcessSentences` function to perform feature extraction.

We are going to use only PoS tagging in this example, so modify the functions `fill_config` and `fill_invoke` to deactivate parsing, and other high level processing, as we did in [*Example 12*](example12.md).


Then, make sure to include the feature extractor classes:
```C++
#include "freeling/morfo/fex.h"
#include "freeling/morfo/fex_lexicon.h"
```

Then, replace `ProcessSentences` with the version below, that will create the feature extractor, convert each sentence in the document to a feature vector, print it, and finally, generate a lexicon of seen features with their frequencies.

```C++
void ProcessSentences(list<freeling::sentence> &ls, const wstring &fname) {

  // create feature extractor and feature lexicon 
  freeling::fex fextractor(fname, L"", map<wstring,const freeling::feature_function *>());
  freeling::fex_lexicon lex;

  // for each sentence in list
  int ns=0;
  for (list<freeling::sentence>::iterator s=ls.begin(); s!=ls.end(); ++s, ++ns) {

    // extract features names each word in the sentence
    vector<set<wstring> > feats;
    fextractor.encode_name(*s, feats);      
    
    wcout << L"sentence_" << ns;
    int i=0;
    for (freeling::sentence::const_iterator w=s->begin(); w!=s->end(); ++w, ++i) {
      // print features for encoded sentence, and store them in the lexicon
      for (set<wstring>::iterator j=feats[i].begin(); j!=feats[i].end(); ++j) {
        lex.add_occurrence(*j);
        wcout<<L" "<<*j;
      }
    }
    wcout<<endl; // next sentence
  }

  // save lexicon 
  wstring lname = fname;
  lname.replace(fname.find(L".rgf"), 4, L".lex");
  lex.save_lexicon(lname, 0);
}
```

### Code

Find here the whole code:
    - In [C++](code/example14.cc.md)
    - In [python](code/example14.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

And that the file `features.rgf` contains:
```
# in case we want to use short tag versions
TAGSET tagset-en.dat

## features extracted for all words
RULES ALL
  bow:$w(0)         [0,0]  ALL              # bag-of-words 
  bolt:$l(0)_$T(0)  [0,0]  ALL              # bag-of-pairs (lemma,tag)
  wbg:$w(0)_$w(1)   [0,0]  ALL              # word bigrams
  suf3:{$0}         [0,0]  w matches ...$   
  negation          [0,0]  l is no OR l is not 
ENDRULES

## features extracted for verbs
RULES T matches ^V
  verb:$w(0)       [0,0] ALL        # word x appears as a verb
  verb_prev:$w(-1) [0,0] ALL        # word x preceedes a verb
  verb_next:$w(1)  [0,0] ALL        # word x follows a verb
  verb:$w(0)_prev:$w(-1) [0,0] ALL  # word x preceedes verb y
  verb:$w(0)_next:$w(1)  [0,0] ALL  # word x follows verb y
ENDRULES
```

when running:
```bash
./example14 features.rgf < mytext.txt
```

we would obtain the output:
```
sentence_0 bolt:the_DT bow:the suf3:the wbg:the_big bolt:big_JJ bow:big suf3:big wbg:big_cat bolt:cat_NN bow:cat suf3:cat wbg:cat_eats bolt:eat_VBZ bow:eats suf3:ats verb:eats verb:eats_next:fresh verb:eats_prev:cat verb_next:fresh verb_prev:cat wbg:eats_fresh bolt:fresh_JJ bow:fresh suf3:esh wbg:fresh_fish bolt:fish_NN bow:fish suf3:ish wbg:fish_. bolt:._Fp bow:.
sentence_1 bolt:my_PRP$ bow:my wbg:my_neighbour bolt:neighbour_NN bow:neighbour suf3:our wbg:neighbour_'s bolt:'s_POS bow:'s wbg:'s_dog bolt:dog_NN bow:dog suf3:dog wbg:dog_chased bolt:chase_VBN bow:chased suf3:sed verb:chased verb:chased_next:the verb:chased_prev:dog verb_next:the verb_prev:dog wbg:chased_the bolt:the_DT bow:the suf3:the wbg:the_cat bolt:cat_NN bow:cat suf3:cat wbg:cat_. bolt:._Fp bow:.
sentence_2 bolt:the_DT bow:the suf3:the wbg:the_dog bolt:dog_NN bow:dog suf3:dog wbg:dog_is bolt:be_VBZ bow:is verb:is verb:is_next:eating verb:is_prev:dog verb_next:eating verb_prev:dog wbg:is_eating bolt:eat_VBG bow:eating suf3:ing verb:eating verb:eating_next:meat verb:eating_prev:is verb_next:meat verb_prev:is wbg:eating_meat bolt:meat_NN bow:meat suf3:eat wbg:meat_. bolt:._Fp bow:.
sentence_3 bolt:some_DT bow:some suf3:ome wbg:some_mice bolt:mouse_NNS bow:mice suf3:ice wbg:mice_were bolt:be_VBD bow:were suf3:ere verb:were verb:were_next:hunted verb:were_prev:mice verb_next:hunted verb_prev:mice wbg:were_hunted bolt:hunt_VBD bow:hunted suf3:ted verb:hunted verb:hunted_next:by verb:hunted_prev:were verb_next:by verb_prev:were wbg:hunted_by bolt:by_IN bow:by wbg:by_the bolt:the_DT bow:the suf3:the wbg:the_cat bolt:cat_NN bow:cat suf3:cat wbg:cat_. bolt:._Fp bow:.
```

That is, for every word in the input matching the conditions expressed in the pattern file, features where instantiated with the appropriate values.

The program also produced a file `features.lex` that contains all generated features, with a count of how many times they occurred in the data, and with an integer code associated to each feature. This is useful to filter out features with low occurrence frequency, and to encode the vectors with integer codes instead of strings, as needed by most ML engines.


### Remarks

Note that this module is oriented to developers skilled in machine learning procedures. If you do not understand what this example is about, you probably do not need to use the `fex` module.

For skilled users:  Note that the feature pattern definition language allows to extract per-word features, and that a wide range of context information can be encoded (such as the relative position of certain words in a window around the target word). With this, the classification objects can be all sentence words (e.g. as used in B-I-O task such as Named Entity Recognition), some selected words (e.g. to classify the semantic type of a previously detected Named Entity as person, location, or organization), or the whole sentence (as we did in this example).

For further information please check the `fex` [user manual](https://talp-upc.gitbooks.io/freeling-user-manual/content/modules/fex.html).
