## Example 01: The Basics: PoS Tagger

The first example we will see shows the basic use of the library: Creating analysis modules, feed them text, and process the output somehow.

For this example, the created modules will be a tokenizer, a splitter, a morphological analyzer, and a PoS tagger. The read text will be run through all of them in cascade, obtaining a data structure with linguistic information about words and sentences in the text.

Then, this data structure will be traversed to simply output its content \(though we could do much more, as we will see later\).

FreeLing library provides two kind of classes: Linguistic object classes and language processing classes. The former are used to contain linguist information about the text and are classes such as `word`, `sentence`, `analysis`, `parse_tree`, etc. The later are classes that get as input some partially analyzed linguistic object \(e.g. a `sentence`\) and enrich it with more linguistic information \(e.g. marking the right PoS tag for each `word`in it, or adding a `parse_tree` to the `sentence`\). Examples of processing classes are `pos_tagger`, `dependency_parser`, or `nec` \(named entity classifier\).

The first program we are going to build is a chain of modules that will input text, tag each word with its right part-of-speech \(that, is a PoS tagger\), and make some kind of processing with the result.

The first thing our program needs to do, is including the library headers:

```C++
#include "freeling.h"
```

Next, first thing we need to do in the main body of the program is telling freeling which UTF8 locale we will be using \(`"default"` stands for `en_US.UTF8`\)

```C++
// set locale to an UTF8 compatible locale
freeling::util::init_locale(L"default");
```

Then, we can start creating modules. Most module constructors require a configuration file. We will use configuration files in default freeling language data directories \(`/usr` if you installed from a binary package, or `/usr/local` if you compiled from source. If you installed FreeLing in a custom directory you'll need to give it to the program\).

Note that this is just an example. You can write your own configuration files and place them wherever you like.  
See [user manual](https://talp-upc.gitbooks.io/freeling-user-manual/content/) for details on the format and content of configuration files for each available module.

The program we are writing will expect up to two optional parameters: The language of the input text \(defaults to English\) and the directory where FreeLing was installed \(defaults to `/usr/local`\) and the language data can be found.

```C++
// get requested language from arg1, or use 'English' if not provided
wstring lang = L"en";
if (argc > 1) lang = freeling::util::string2wstring(argv[1]);
// get installation path to use from arg2, or use /usr/local if not provided
wstring ipath = L"/usr/local";
if (argc > 2) ipath = freeling::util::string2wstring(argv[2]);

// path to language data
wstring lpath = ipath+L"/share/freeling/"+lang+L"/";
```

Note that this is a very basic argument control, and that the program will crash if wrong parameters are given \(e.g. non-exisiting language or wrong installation path\).   
You can adapt this path setting to your convenience \(from hard-coding it to your needs to doing a sophisticated argument control.\)

Once we have the path for needed configuration files, we can start creating instances of required analyzers:

```C++
// create analyzers
freeling::tokenizer tk(lpath + L"tokenizer.dat");
freeling::splitter sp(lpath + L"splitter.dat");
```

So far, we created a tokenizer and a splitter.  
Next, we will create a morphological analyzer and a PoS tagger.

```C++
// create morphological analyzer with desired options
freeling::maco morfo(my_maco_options(lang,lpath)); 
// activate/deactivate desired morphological submodules
morfo.set_active_options (false,// UserMap
                          true, // NumbersDetection,
                          true, //  PunctuationDetection,
                          true, //  DatesDetection,
                          true, //  DictionarySearch,
                          true, //  AffixAnalysis,
                          false, //  CompoundAnalysis,
                          true, //  RetokContractions,
                          true, //  MultiwordsDetection,
                          true, //  NERecognition,
                          false, //  QuantitiesDetection,
                          true);  //  ProbabilityAssignment

// create a hmm tagger
freeling::hmm_tagger tagger(lpath+L"tagger.dat", true, FORCE_TAGGER); 
```

Method `set_active_options` is used to activate/deactivate specific morphological analysis submodules. It can be called anytime to modify the behaviour of the analyzer in subsequent calls.

Note the call to function `my_maco_options`. This function is not a member of any freeling class, we just wrote it to make the code less messy. It creates a `maco_options` object with the configuration we want for our morphological analyzer. This is because the morphological analyzer is not a single module, but a cascade of different modules, each with its own configuration options.  
  We can write `my_maco_options` as follows:

```C++
freeling::maco_options my_maco_options(const wstring &lang, const wstring &path) {
   // create options pack
   freeling::maco_options opt(lang);
   // fill it with files for morphological submodules. Only those that are going
   // to be used need to be provided.
   opt.UserMapFile = L"";
   opt.LocutionsFile = path + L"locucions.dat"; 
   opt.AffixFile = path + L"afixos.dat";
   opt.ProbabilityFile = path + L"probabilitats.dat"; 
   opt.DictionaryFile = path + L"dicc.src";
   opt.NPdataFile = path + L"np.dat";
   opt.PunctuationFile = path + L"../common/punct.dat";

   return opt;
}
```

Once all processing classes are created, we can start reading and processing text.

In this first example, We will assume we have a relatively short text that can be completely loaded in memory and then processed:

```C++
// get all input text in a single string, keeping line breaks
wstring text=L"";
wstring line;
while (getline(wcin,line))
   text = text + line + L"\n";

// tokenize input line into a list of words
list<freeling::word> lw = tk.tokenize(text);
// split list of words into sentences.
list<freeling::sentence> ls = sp.split(lw);
// perform morphosyntactic analysis and disambiguation
morfo.analyze(ls);
tagger.analyze(ls);

// do whatever is needed with processed sentences
ProcessSentences(ls);
```

After the text is processed, the function `ProcessSentences` will do the required handling of the analysis produced by FreeLing modules.  
In this first example we are simply going to traverse the data structure returned by the tagger and print morphological information associated to each word.

```C++
void ProcessSentences(const list<freeling::sentence> &ls) {

  // for each sentence in list
  for (list<freeling::sentence>::const_iterator is=ls.begin(); is!=ls.end(); ++is) {

    // for each word in sentence
    for (freeling::sentence::const_iterator w=is->begin(); w!=is->end(); ++w) {

      // print word form, with PoS and lemma chosen by the tagger
      wcout << L"word '" << w->get_form() << L"'" << endl;
      // print possible analysis in word, output lemma, tag and probability
      wcout << L"  Possible analysis: {";
      for (freeling::word::const_iterator a=w->analysis_begin(); a!=w->analysis_end(); ++a)     
        wcout << L" (" << a->get_lemma() << L"," << a->get_tag() << L")";
      wcout << L" }" << endl;
      wcout << L"  Selected analysis: ("<< w->get_lemma() << L", " << w->get_tag() << L")" << endl;
    }

    // sentence separator
    wcout << endl;
   }
}
```

### Code

Find here the whole code:

    * In [C++](code/example01.cc.md)
    * In [python](code/example01.py.md)

### Example

Assuming the input file contains the following sentences:

```
The big cat eats fresh fish. My neighbour's dog chased the cat.
The dog is eating meat. Some mice were hunted by the cat.
```

The output would be:

```
word 'The'
  Possible analysis: { (the,DT) }
  Selected analysis: (the, DT)
word 'big'
  Possible analysis: { (big,JJ) }
  Selected analysis: (big, JJ)
word 'cat'
  Possible analysis: { (cat,NN) }
  Selected analysis: (cat, NN)
word 'eats'
  Possible analysis: { (eat,VBZ) }
  Selected analysis: (eat, VBZ)
word 'fresh'
  Possible analysis: { (fresh,JJ) }
  Selected analysis: (fresh, JJ)
word 'fish'
  Possible analysis: { (fish,NN) (fish,VBP) (fish,NNS) (fish,VB) }
  Selected analysis: (fish, NN)
word '.'
  Possible analysis: { (.,Fp) }
  Selected analysis: (., Fp)

word 'My'
  Possible analysis: { (my,PRP$) }
  Selected analysis: (my, PRP$)
word 'neighbour'
  Possible analysis: { (neighbour,NN) (neighbour,VB) (neighbour,VBP) }
  Selected analysis: (neighbour, NN)
word ''s'
  Possible analysis: { ('s,POS) (be,VBZ) (have,VBZ) }
  Selected analysis: ('s, POS)
word 'dog'
  Possible analysis: { (dog,NN) (dog,VB) (dog,VBP) }
  Selected analysis: (dog, NN)
word 'chased'
  Possible analysis: { (chase,VBN) (chase,VBD) }
  Selected analysis: (chase, VBD)
word 'the'
  Possible analysis: { (the,DT) }
  Selected analysis: (the, DT)
word 'cat'
  Possible analysis: { (cat,NN) }
  Selected analysis: (cat, NN)
word '.'
  Possible analysis: { (.,Fp) }
  Selected analysis: (., Fp)

word 'The'
  Possible analysis: { (the,DT) }
  Selected analysis: (the, DT)
word 'dog'
  Possible analysis: { (dog,NN) (dog,VB) (dog,VBP) }
  Selected analysis: (dog, NN)
word 'is'
  Possible analysis: { (be,VBZ) }
  Selected analysis: (be, VBZ)
word 'eating'
  Possible analysis: { (eat,VBG) }
  Selected analysis: (eat, VBG)
word 'meat'
  Possible analysis: { (meat,NN) }
  Selected analysis: (meat, NN)
word '.'
  Possible analysis: { (.,Fp) }
  Selected analysis: (., Fp)

word 'Some'
  Possible analysis: { (some,DT) (some,RB) }
  Selected analysis: (some, DT)
word 'mice'
  Possible analysis: { (mouse,NNS) }
  Selected analysis: (mouse, NNS)
word 'were'
  Possible analysis: { (be,VBD) }
  Selected analysis: (be, VBD)
word 'hunted'
  Possible analysis: { (hunt,VBD) (hunt,VBN) }
  Selected analysis: (hunt, VBN)
word 'by'
  Possible analysis: { (by,IN) (by,RP) (by,RB) }
  Selected analysis: (by, IN)
word 'the'
  Possible analysis: { (the,DT) }
  Selected analysis: (the, DT)
word 'cat'
  Possible analysis: { (cat,NN) }
  Selected analysis: (cat, NN)
word '.'
  Possible analysis: { (.,Fp) }
  Selected analysis: (., Fp)
```



