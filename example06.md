
## Example 06: Navigating WordNet

This example is an extension of [*Example 3*](example03.md) but that not merely outputs the synsets proposed by the WSD, but uses them as starting points to navigate WordNet using FreeLing classes. 
This is done using the class `semanticDB` to obtain additional semantic information and to navigate WordNet.

In particular, we are going to detect which words in the text refer to animals.
For that, we will climb up the hypernym chain until we reach the synset for concept *animal*.
We could do that using simply WordNet information, but for illustration purposes we are going to do that using [SUMO](http://www.adampease.org/OP/). SUMO is a general ontology that has been linked to WordNet, so, from each WN synset we can access the corresponding SUMO node and navigate the SUMO ontology (provided we have it downloaded)

FreeLing includes WordNet-to-SUMO links, so instead of checking if we reached the WN synset for *animal*, we will check if we reached a synset which is linked to SUMO concept `[Animal=]` (that is the label assigned to that concept in SUMO)

We only need to modify our `ProcessSentence` function to climb up wordnet taxonomy until either *animal* or the top node is reached:
```C++

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
```

The WordNet navigation oculd be done using native WN library, but here we are using `freeling::semanticDB`, a utility class provided by FreeLing that allows basic WN navigation (for full-fledged navigation you'd need to install WN library)

This class loads a file with information about WN taxonomy (which synsets are parents of which) and links to other ontologies (such as SUMO or OpenCYC). Then, the `semanticDB` instance can be queried to obtain information about a particular synset, such as its parents, or the code of the equivalent concept in SUMO.

In our example, the `ProcessSentences` function expects an already created instance of `semanticDB`, thus, our main program needs to create it before calling the function:
```C++
// create semantic DB module
freeling::semanticDB sdb(lpath+L"semdb.dat");
// do whatever is needed with processed sentences
ProcessSentences(ls, sdb);
```


### Code

Find here the whole code:
 - In [C++](code/example06.cc.md)
 - In [python](code/example06.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

The output would be:
```
The the DT
big big JJ 01382086-a
cat cat NN 02121620-n    *** SUMO 'Animal='
eats eat VBZ 01168468-v
fresh fresh JJ 01067694-a
fish fish NN 02512053-n    *** SUMO 'Animal='
. . Fp

My my PRP$
neighbour neighbour NN 10352299-n
's 's POS
dog dog NN 02084071-n    *** SUMO 'Animal='
chased chase VBD 02001858-v
the the DT
cat cat NN 02121620-n    *** SUMO 'Animal='
. . Fp

The the DT
dog dog NN 02084071-n    *** SUMO 'Animal='
is be VBZ 02604760-v
eating eat VBG 01168468-v
meat meat NN 07649854-n
. . Fp

Some some DT
mice mouse NNS 10335563-n
were be VBD 02604760-v
hunted hunt VBN 01143838-v
by by IN
the the DT
cat cat NN 02121620-n    *** SUMO 'Animal='
. . Fp

```

  Thus, we get as output the PoS, lemma, and synset for each word, plus a mark for those words with a sense under concept *animal* in the hypernymy hierarchy.

   Note that in this example, two words are wrongly disambiguated by the WSD algorithm: *fish* should have a `food` meaning and not an `animal` one, and *mice* should have `animal` sense but has a different one (*10335563-n* = *timid person*).
   This is caused by the fact that the text is short and WSD typically requires longer contexts, and also because state-of-the-art precision of WSD modules is between 70% and 80%, so, a certain amount of noise is inevitable.
   
   
   
