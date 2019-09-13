
## Example 13: Tagset Manipulation

One of the nightmares of NLP application developers is matching tagsets from exisiting data to suit their application needs.

Usually, taggers and parsers are trained on public datasets that use a predetermined set of PoS tags or syntactic labels. When these tools are used, the application has to adapt to the analyzer tagset and not the other way round.

FreeLing includes a `tagset` module that allows to manipulate and transform PoS tags.
The class loads a tagset description, where each tag can be described as a set of morphological features (category, gender, number, cas, tense, etc). 

The module allows to convert from/to a single PoS tag to a set of features. Thus a tag can be decomposed into its features, and those features can be converted to a new tag using a different tagset description (i.e. a different mapping between feature combinations and tags)


In this example we are going to see how to convert a corpus PoS-tagged with one tagset (e.g. PTB tagset) to an EAGLES-like set of tags.

We will assume that the program is passed two arguments: the tagset description file, and the encoding direction (i.e. `tag2msd` or `msd2tag`)

For this, we need to include the `tagset` class:
```C++
#include "freeling/morfo/tagset.h"
```

As usual, do not forget to set FreeLing locale 
```C++
// set locale to an UTF8 compatible locale
freeling::util::init_locale(L"default");
```

And we can create an instance of the tagset managing module using the tagset description file given as an argument to the program:
```C++
// get arguments
wstring fname = freeling::util::string2wstring(argv[1]);
wstring action = freeling::util::string2wstring(argv[2])

// create tagset handling module
freeling::tagset tgs(fname);
```

Now we can enter a loop that will read every word in the tagged input text, and transform the PoS tag using the just created tagset class.
```C++
// convert tags to morphosyntactic descriptions
if (action==L"tag2msd") {
   wstring form,lemma,tag;
   while (wcin >> form >> lemma >> tag) {
      wcout << form << L" " << lemma << L" " << tgs.get_msd_string(tag) << endl;
   }
}
// convert morphosyntactic descriptions to tags
else if (action==L"msd2tag") {   
   wstring form,lemma,msd;
   while (wcin >> form >> lemma >> msd) {
      wcout << form << L" " << lemma << L" " << tgs.msd_to_tag(L"",msd) << endl;
   }
}
```

with that, we can convert any corpus in the right format to a new tagset.


### Code

Find here the whole code:
  - In [C++](code/example13.cc.md)
  - In [python](code/example13.py.md)


### Example

We will assume we have an input file `sentences.ptb` with tagged text, containing the sentence below. This file could have been obtained from some corpus, or tagged any PoS tagger that outputs PTB tags (even FreeLing itself). 
```
The the DT
big big JJ
cat cat NN
on on IN
the the DT
mat mat NN
eats eat VBZ
fishes fish NNS
in in IN
a a DT
dish dish
. . .
```


Then, we can execute:
```
./example13 tagset-pb.dat tag2msd < sentences.ptb > sentences.msd
```

And we will obtain:
```
The the pos=determiner
big big pos=adjective
cat cat pos=noun|type=common|num=singular
on on pos=adposition|type=preposition
the the pos=determiner
mat mat pos=noun|type=common|num=singular
eats eat pos=verb|vform=personal|person=3
fishes fish pos=noun|type=common|num=plural
in in pos=adposition|type=preposition
a a pos=determiner
dish dish pos=noun|type=common|num=singular
. . pos=punctuation|type=period
```

So, we get the same corpus where the PoS tag has been replaced with the corresponding morphological features.

This was done thanks to the mapping described in the following `tagset-ptb.dat` file.  
See [user manual](https://talp-upc.gitbooks.io/freeling-user-manual/content/modules/tagset.html) for details on how to describe tag decompositions.
```XML
<DirectTranslations>
, , pos=punctuation|type=comma
... ... pos=punctuation|type=etc
: : pos=punctuation|type=colon
; ; pos=punctuation|type=semicolon
- - pos=punctuation|type=hyphen
" " pos=punctuation|type=quotation
/ / pos=punctuation|type=slash
. . pos=punctuation|type=period
¿ ¿ pos=punctuation|type=questionmark|punctenclose=open
? ? pos=punctuation|type=questionmark|punctenclose=close
¡ ¡ pos=punctuation|type=exclamationmark|punctenclose=open
! ! pos=punctuation|type=exclamationmark|punctenclose=close
% % pos=punctuation|type=percentage
{ { pos=punctuation|type=curlybracket|punctenclose=open
} } pos=punctuation|type=curlybracket|punctenclose=close
[ [ pos=punctuation|type=squarebracket|punctenclose=open
] ] pos=punctuation|type=squarebracket|punctenclose=close
( ( pos=punctuation|type=parenthesis|punctenclose=open
) ) pos=punctuation|type=parenthesis|punctenclose=close
` ` pos=punctuation|type=quotation|punctenclose=open
' ' pos=punctuation|type=quotation|punctenclose=close
`` `` pos=punctuation|type=quotation|punctenclose=open
'' '' pos=punctuation|type=quotation|punctenclose=close
X X pos=punctuation|type=other
NNP NP pos=noun|type=proper
NNPS NP pos=noun|type=proper|num=plural
NP NP pos=noun|type=proper
CC CC pos=conjunction|type=coordinating
CD CD pos=number
DT DT pos=determiner
EX EX pos=pronoun
IN IN pos=adposition|type=preposition
JJ JJ pos=adjective
JJR JJR pos=adjective|degree=comparative
JJS JJS pos=adjective|degree=superlative
MD MD pos=verb|type=modal
NN NN pos=noun|type=common|num=singular
NNS NNS pos=noun|type=common|num=plural
PDT PDT pos=determiner|type=predeterminer
POS POS pos=adposition|type=possessive
PRP PRP pos=pronoun|type=personal
PRP$ PRP$ pos=pronoun|type=possessive
RB RB pos=adverb|type=general
RBR RBR pos=adverb|type=general|degree=comparative
RBS RBS pos=adverb|type=general|degree=superlative
RP RP pos=particle
TO TO pos=particle|type=to
UH UH pos=interjection
VB VB pos=verb|vform=infinitive
VBD VBD pos=verb|vform=past
VBG VBG pos=verb|vform=gerund
VBN VBN pos=verb|vform=participle
VBP VBP pos=verb|vform=personal
VBZ VBZ pos=verb|vform=personal|person=3
WDT WDT pos=determiner|type=interrogative
WP WP pos=pronoun|type=interrogative
WP$ WP$ pos=pronoun|type=possessive
WRB WRB pos=adverb|type=interrogative
</DirectTranslations>
```

Next, we can encode these morphosyntactic descriptions into a new PoS tag, using the mapping in file `tagset-en.dat`, which describes an EAGLES-like positional tagset for English similar to that used by other languages in FreeLing:
```XML
<DecompositionRules>
Z 2 number type/d:partitive;m:currency;p:percentage;u:unit
W 0 date
I 0 interjection
A 2 adjective degree/S:superlative;C:comparative
C 2 conjunction type/C:coordinating;S:subordinating
D 2 determiner type/P:predeterminer;I:interrogative
N 2 noun type/C:common;P:proper num/S:singular;P:plural;N:invariable
S 2 adposition type/P:preposition;S:possessive
P 2 pronoun type/P:personal;T:interrogative;X:possessive
H 2 particle type/T:to
R 2 adverb type/G:general;I:interrogative degree/C:comparative;S:superlative
V 3 verb type/M:main;D:modal vform/F:personal;P:participle;G:gerund;N:infinitive tense/S:past person/1:1;2:2;3:3 num/S:singular;P:plural
</DecompositionRules>
<DirectTranslations>
Fc Fc pos=punctuation|type=comma
Fs Fs pos=punctuation|type=etc
Fd Fd pos=punctuation|type=colon
Fx Fx pos=punctuation|type=semicolon
Fg Fg pos=punctuation|type=hyphen
Fe Fe pos=punctuation|type=quotation
Fh Fh pos=punctuation|type=slash
Fp Fp pos=punctuation|type=period
Fia Fia pos=punctuation|type=questionmark|punctenclose=open
Fit Fit pos=punctuation|type=questionmark|punctenclose=close
Faa Faa pos=punctuation|type=exclamationmark|punctenclose=open
Fat Fat pos=punctuation|type=exclamationmark|punctenclose=close
Ft Ft pos=punctuation|type=percentage
Fla Fla pos=punctuation|type=curlybracket|punctenclose=open
Flt Flt pos=punctuation|type=curlybracket|punctenclose=close
Fca Fca pos=punctuation|type=squarebracket|punctenclose=open
Fct Fct pos=punctuation|type=squarebracket|punctenclose=close
Fpa Fpa pos=punctuation|type=parenthesis|punctenclose=open
Fpt Fpt pos=punctuation|type=parenthesis|punctenclose=close
Fra Fra pos=punctuation|type=quotation|punctenclose=open
Frc Frc pos=punctuation|type=quotation|punctenclose=close
Fz Fz pos=punctuation|type=other
</DirectTranslations>
``` 

So, we execute again the example program, but using the new tagset description, and converting morphosyntactic descriptions to a single PoS tag:
```
./example13 tagset-en.dat msd2tag < sentences.msd > sentences.pos
```

the obtained result is:
```
The the D0
big big A0
cat cat NCS
on on SP
the the D0
mat mat NCS
eats eat V0F030
fish fish NCS
in in SP
a a D0
dish dish NCS
. . Fp
```


### Remarks

Note that we can run this conversion in a single step, pipelining both processes:
```
cat sentences.ptb | ./example13 tagset-pb.dat tag2msd | ./example13 tagset-en.dat msd2tag > sentences.pos
```

Note also that we could have written a single program that instantiates two tagset modules (one with each file) and applies both conversions without need to output the intermediate stage.


