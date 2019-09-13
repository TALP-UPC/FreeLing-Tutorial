
## Example 10: Language Identifier

This example shows how to use the language identifier module to find out the language in which a text is written.

This could be a first step in a multilingual processing chain: Get some text, find out the language, and then run it through the analyzer instances created with the appropriate data for that language.

The initial steps are basically the same than in all other examples (see [*Example 1*](example01.md) for details)

Do not forget to include FreeLing headers:
```C++
#include "freeling.h"
```
	
Start your main initializing FreeLing locale:
```C++
// set locale to an UTF8 compatible locale
freeling::util::init_locale(L"default");
```

Set the needed paths:
```C++
// read FreeLing installation path if given, use default otherwise
wstring ipath = L"/usr/local";
if (argc > 1) ipath = freeling::util::string2wstring(argv[1]);

// path to data common to all languages
wstring cpath = ipath+L"/share/freeling/common/";
```

Now we can create an instance of the language identifier:
```C++
// creates a language identifier with the default config file
freeling::lang_ident di(cpath+L"lang_ident/ident.dat");
```  

Check the [user manual](https://talp-upc.gitbooks.io/freeling-user-manual/content/) for details on the format and content of configuration files.

Finally, we can write a loop that reads sentences, one per line, and outputs the language of each of them:
```C++
  set<wstring> candidates=set<wstring>(); // list of languages to consider. 
                                          // Empty -> all known languages
  vector<pair<double,wstring> > result;
  wstring line;
  while (getline(wcin,line)) {
    wcout << L"-----------------------------------" << endl;
    wcout << L"Input text: [" << line << L"]" << endl;

    // classificate the read text line and obtain a sorted 'result' vector <language_code, perplexity>
    di.rank_languages(result, line, candidates);

    // The funcion identify_language will return the code for the best language, 
    // or "none" if no language model yields a small enough perplexity (according
    // to the threshold set in that language model)
    wstring best_l = di.identify_language (line, candidates);
    wcout << L"Best language: "<<best_l<<endl;

    // You can also output get a sorted list of increasing perplexity,
    // in case you want to take the decision yourself.
    wcout<<L"Increasing perplexity list:"<<endl;
    vector<pair<double,wstring> >::iterator i;
    for (i=result.begin(); i!=result.end(); i++)
      wcout<<L"  "<<i->second<<L" "<<i->first<<endl;     
  }
``` 

The language identifier expects a `set<wstring>` containing the codes of languages to be considered. This is to speed up the identification, avoiding to check languages we know the text is not written in (e.g. if your application got the text from a Canadian newspaper, you may assume that it is either in English or French, but no need to consider other choices).
If the list is empty (or the parameter ommited), all available language models are used.

The result of the identifier is ranked by perplexity, which is a common information-theory measure. Roughly, the lower perplexity, the higher probability that the text belongs to that language. 


### Code

Find here the whole code:
    - In [C++](code/example10.cc.md)
    - In [python](code/example10.py.md)


### Example

Assuming the input file contains the following sentences:
``` 
The cat eats fish.
El gato come pescado.
Il gatto mangia pesce.
Le chat mange poisson.
Upmf grhjdfos girms skhupogn.
```

The output would be:
```
-----------------------------------
Input text: [The cat eats fish.]
Best language: en
Decreasing log probability list:
  en 10.1291
  ca 11.8932
  gl 15.5557
  pt 18.2078
  fr 21.6702
  hr 22.8298
  es 24.5168
  it 27.7298
  de 32.602
  sl 36.1853
  cs 36.7898
  sk 55.3557
  ja 21596.1
  zh 33864.6
  ru 85649.8
  bg 111752
  hi 123480
  sr 127602
-----------------------------------
Input text: [El gato come pescado.]
Best language: es
Decreasing log probability list:
  es 8.52691
  gl 8.55545
  pt 10.3512
  it 11.4716
  ca 13.3235
  fr 21.8638
  de 27.4486
  en 27.8389
  hr 31.8612
  cs 34.4912
  sl 36.0055
  sk 46.6498
  ja 25244.2
  zh 40221
  ru 91878.2
  bg 121441
  hi 133212
  sr 139653
-----------------------------------
Input text: [Il gatto mangia pesce.]
Best language: it
Decreasing log probability list:
  it 10.0425
  fr 20.8522
  ca 26.1899
  gl 26.4293
  cs 26.7365
  sl 28.1093
  sk 28.4455
  pt 28.8596
  es 31.6093
  de 32.6849
  hr 34.616
  en 35.5857
  ja 26342.1
  zh 42152.9
  ru 93654.1
  bg 124227
  hi 135997
  sr 143133
-----------------------------------
Input text: [Le chat mange poisson.]
Best language: fr
Decreasing log probability list:
  fr 9.12527
  it 11.9091
  en 12.8855
  sl 12.8907
  cs 14.2737
  pt 14.6708
  sk 15.4319
  ca 15.461
  gl 16.7358
  de 16.7657
  es 17.308
  hr 17.9694
  ja 26342.1
  zh 42152.9
  ru 93654.1
  bg 124227
  hi 135997
  sr 143133
-----------------------------------
Input text: [Upmf grhjdfos girms skhupogn.]
Best language: none
Increasing perplexity list:
  sl 726.191
  en 735.923
  de 750.101
  ca 757.988
  sk 957.714
  pt 1079.76
  hr 1129.77
  cs 1197.76
  it 1343.05
  es 1343.48
  fr 1362.81
  gl 2112.96
  ja 32686
  zh 53469.4
  ru 103198
  bg 139359
  hi 151034
  sr 162152
```


