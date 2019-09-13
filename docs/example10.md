
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

* In [C++](code/example10.cc.md)
* In [python](code/example10.py.md)


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
Increasing perplexity list:
en 10.129120110158526
ca 11.893200093643978
gl 15.55565177390918
pt 18.20783137152024
fr 21.6701832798073
hr 22.82976729442382
es 24.51677717565222
it 27.729849011121267
de 32.6019832996728
sl 36.18534081293369
cs 36.78979963643931
sk 55.355669631765174
ja 21596.079092969994
zh 33864.57423243041
ru 85649.82372824152
bg 111752.19745525921
hi 123480.2137849647
sr 127601.75649662777
-----------------------------------
Input text: [El gato come pescado.]
Best language: es
Increasing perplexity list:
es 8.526913928535956
gl 8.555448072035988
pt 10.351249426974697
it 11.471587528325122
ca 13.323479578286348
fr 21.863841664778857
de 27.448648848291384
en 27.838883782646917
hr 31.861240908216487
cs 34.49116216150059
sl 36.00546245621243
sk 46.64975373503211
ja 25244.23352788273
zh 40220.9684017926
ru 91878.18575788377
bg 121441.36282456886
hi 133212.0039495265
sr 139652.9665044314
-----------------------------------
Input text: [Il gatto mangia pesce.]
Best language: it
Increasing perplexity list:
it 10.042528049061511
fr 20.85222004096425
ca 26.18988220296434
gl 26.42927443083413
cs 26.736473792186963
sl 28.109329936470147
sk 28.44547183827767
pt 28.859620672218373
es 31.60933383450606
de 32.68489233102824
hr 34.61596738361512
en 35.585739539989895
ja 26342.053592919012
zh 42152.86842124164
ru 93654.09198067486
bg 124226.70524477196
hi 135996.78174675125
sr 143132.84551381494
-----------------------------------
Input text: [Le chat mange poisson.]
Best language: fr
Increasing perplexity list:
fr 9.125266846180677
it 11.909092649850852
en 12.88545287857784
sl 12.890748421435225
cs 14.273701439003112
pt 14.670756465940318
sk 15.431940882726078
ca 15.460952344079542
gl 16.735792168708404
de 16.76568199538248
es 17.308034365663406
hr 17.969449963856448
ja 26342.053592919012
zh 42152.86842124164
ru 93654.09198067486
bg 124226.70524477196
hi 135996.78174675125
sr 143132.84551381494
-----------------------------------
Input text: [Upmf grhjdfos girms skhupogn.]
Best language: none
Increasing perplexity list:
sl 726.190602634379
en 735.9230895433196
de 750.1007301483637
ca 757.987983007315
sk 957.7138614290852
pt 1079.7648780393256
hr 1129.766803821508
cs 1197.758081554562
it 1343.051957029233
es 1343.4833411163802
fr 1362.80876221829
gl 2112.959528213961
ja 32686.003619199797
zh 53469.40214330654
ru 103198.14565962118
bg 139359.2614128819
hi 151033.97532073915
sr 162151.95399340615
```


