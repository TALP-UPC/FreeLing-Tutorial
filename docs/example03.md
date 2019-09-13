
## Example 03: Adding Word Sense Disambiguation

Now we will continue extending our example program with additional features. In this exercise, we are going to use a sense annotator to associate to each word its possible WordNet synsets, and a word sense disambiguator to select the right meaning among all possible senses for each word.

Then, as post processing we will just print out the possible and selected synsets for each word.
As in [*Example 1*](example01.md), this is simply the output of a WSD system, and we could use one out-of-the-box for that. 
In later examples we'll see how we can further exploit the output of the semantic disambiguator.

We will use [*Example 1*](example01.md) as a strating point and instantiate two new modules in the module creation area of our program:
```C++
// create sense annotator
freeling::senses sen(lpath+L"senses.dat");
// create sense disambiguator
freeling::ukb wsd(lpath+L"ukb.dat");
```

And we need to call them after the text has been PoS tagged.
```C++
sen.analyze(ls);
wsd.analyze(ls);
```

The first module adds to each word a list of synsets found in WordNet for the lemma and PoS selected by the tagger. The second module re-ranks this list so most likely senses according to the context are at the beggining.
So, after these calls, the words in the sentence have been enriched with a sorted list of synsets.

Finally, we can modify our `ProcessSentence` function to output the synsets for each word:
```C++
void ProcessSentences(const list<freeling::sentence> &ls) {
  // for each sentence in list
  for (list<freeling::sentence>::const_iterator is=ls.begin(); is!=ls.end(); ++is) {

    // for each word in sentence
    for (freeling::sentence::const_iterator w=is->begin(); w!=is->end(); ++w) {
      
      // print word form, with PoS and lemma chosen by the tagger
      wcout << L"word '" << w->get_form() << L"'" << endl;
      // print possible analysis in word, output lemma, tag and probability
      wcout << L"   Possible Synsets {" << w->get_senses_string() << L"}" << endl;
      // print best sense and its page rank value.
	  if (not w->get_senses().empty()) {
        list<pair<wstring,double>>::const_iterator best = w->get_senses().begin();
        wcout << L"   Selected Synset ("<< best->first << L"," << best->second << L")"<< endl;
	  }
    }

    // sentence separator
    wcout << endl;
  }
}
```


### Code

Find here the whole code:

* In [C++](code/example03.cc.md)
* In [python](code/example03.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

The output would be:
```
word 'The'
  Possible Synsets {}
word 'big'
  Possible Synsets {01382086-a:0.00268966/01890752-a:0.00115455/01890187-a:0.0010466/01111418-a:0.0010063/01510444-a:0.000966933/01191780-a:0.000929398/00173391-a:0.000919019/02402439-a:0.00091364/01114658-a:0.000910303/00579622-a:0.00087967/01276872-a:0.000863808/01453084-a:0.000843215/01488616-a:0.000838392}
  Selected Synset (01382086-a,0.00268966)
word 'cat'
  Possible Synsets {02121620-n:0.00171637/02985606-n:0.00170318/02127808-n:0.0014893/09900153-n:0.00144523/00901476-n:0.00141993/10153414-n:0.0014143/03608870-n:0.00138002/02983507-n:0.0013799}
  Selected Synset (02121620-n,0.00171637)
word 'eats'
  Possible Synsets {01168468-v:0.00445074/01157517-v:0.00443505/01166351-v:0.00432526/01179865-v:0.00415433/00274283-v:0.00375442/01766273-v:0.00368628}
  Selected Synset (01168468-v,0.00445074)
word 'fresh'
  Possible Synsets {01067694-a:0.0011371/02370083-a:0.00104037/00205295-a:0.00103304/01357027-a:0.00101317/02434797-a:0.000991617/01906320-a:0.000973927/01071198-a:0.000973312/00418198-a:0.000964367/01687167-a:0.000946948/01641648-a:0.000945094/02554420-a:0.000942024/01073707-a:0.000914425}
  Selected Synset (01067694-a,0.0011371)
word 'fish'
  Possible Synsets {02512053-n:0.00335443/08688076-n:0.00332412/09753792-n:0.00329016/07775375-n:0.00313735}
  Selected Synset (02512053-n,0.00335443)
word '.'
  Possible Synsets {}

word 'My'
  Possible Synsets {}
word 'neighbour'
  Possible Synsets {10352299-n:0.00632706/09368224-n:0.0057613}
  Selected Synset (10352299-n,0.00632706)
word ''s'
  Possible Synsets {}
word 'dog'
  Possible Synsets {02084071-n:0.00195676/09886220-n:0.00173153/07676602-n:0.00169912/10114209-n:0.00162535/03901548-n:0.00161203/10023039-n:0.00159417/02710044-n:0.00155698}
  Selected Synset (02084071-n,0.00195676)
word 'chased'
  Possible Synsets {02001858-v:0.00334444/02535093-v:0.00291128/01277649-v:0.00280385/01583881-v:0.00269432}
  Selected Synset (02001858-v,0.00334444)
word 'the'
  Possible Synsets {}
word 'cat'
  Possible Synsets {02121620-n:0.00171637/02985606-n:0.00170318/02127808-n:0.0014893/09900153-n:0.00144523/00901476-n:0.00141993/10153414-n:0.0014143/03608870-n:0.00138002/02983507-n:0.0013799}
  Selected Synset (02121620-n,0.00171637)
word '.'
  Possible Synsets {}

word 'The'
  Possible Synsets {}
word 'dog'
  Possible Synsets {02084071-n:0.00195676/09886220-n:0.00173153/07676602-n:0.00169912/10114209-n:0.00162535/03901548-n:0.00161203/10023039-n:0.00159417/02710044-n:0.00155698}
  Selected Synset (02084071-n,0.00195676)
word 'is'
  Possible Synsets {02604760-v:0.00247978/02620587-v:0.00211987/02655135-v:0.00199313/02702508-v:0.00194539/02603699-v:0.00194327/02445925-v:0.00192557/02744820-v:0.0017768/02664769-v:0.00177188/02268246-v:0.00174887/02614181-v:0.00173693/02697725-v:0.00173175/02749904-v:0.00165705/02616386-v:0.00165004}
  Selected Synset (02604760-v,0.00247978)
word 'eating'
  Possible Synsets {01168468-v:0.00445074/01157517-v:0.00443505/01166351-v:0.00432526/01179865-v:0.00415433/00274283-v:0.00375442/01766273-v:0.00368628}
  Selected Synset (01168468-v,0.00445074)
word 'meat'
  Possible Synsets {07649854-n:0.00439533/05921123-n:0.00431161/13137010-n:0.00384909}
  Selected Synset (07649854-n,0.00439533)
word '.'
  Possible Synsets {}

word 'Some'
  Possible Synsets {}
word 'mice'
  Possible Synsets {10335563-n:0.00328019/02330245-n:0.00302807/03793489-n:0.00297466/14289387-n:0.00276138}
  Selected Synset (10335563-n,0.00328019)
word 'were'
  Possible Synsets {02604760-v:0.00247978/02620587-v:0.00211987/02655135-v:0.00199313/02702508-v:0.00194539/02603699-v:0.00194327/02445925-v:0.00192557/02744820-v:0.0017768/02664769-v:0.00177188/02268246-v:0.00174887/02614181-v:0.00173693/02697725-v:0.00173175/02749904-v:0.00165705/02616386-v:0.00165004}
  Selected Synset (02604760-v,0.00247978)
word 'hunted'
  Possible Synsets {01143838-v:0.00219794/02003601-v:0.00193041/01316401-v:0.0017863/02034147-v:0.00176526/02056691-v:0.00164133/01144657-v:0.00162329/01878376-v:0.00157469}
  Selected Synset (01143838-v,0.00219794)
word 'by'
  Possible Synsets {}
word 'the'
  Possible Synsets {}
word 'cat'
  Possible Synsets {02121620-n:0.00171637/02985606-n:0.00170318/02127808-n:0.0014893/09900153-n:0.00144523/00901476-n:0.00141993/10153414-n:0.0014143/03608870-n:0.00138002/02983507-n:0.0013799}
  Selected Synset (02121620-n,0.00171637)
word '.'
  Possible Synsets {}
```
