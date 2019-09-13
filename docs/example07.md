
## Example 07: Extracting Triples with Semantic Information

In [*Example 5*](example05.md) we saw how to use the output of a dependency parser and SRL to extract SVO triples. 
Now we are going to generalize these triples using ontological information extracted from the `semanticDB` class presented in [*Example 6*](example06.md).

We are going to access WordNet with the synset chosen by the WSD module, and retrieve all synonyms for each word, plus the code of the equivalent concept in [SUMO](http://www.adampease.org/OP/) ontology.

We only need to modify the `ProcessSentence` function in [*Example 5*](example05.md) to output these additional information for each element of extracted triples. The only changes are in the last part, when a triple is found and output.

```C++
void ProcessSentences(const list<freeling::sentence> &ls, const freeling::semanticDB &sdb) {

  // for each sentence in list
  for (list<freeling::sentence>::const_iterator s=ls.begin(); s!=ls.end(); ++s) {

    // for each predicate in sentence
    for (freeling::sentence::predicates::const_iterator pred=s->get_predicates().begin(); pred!=s->get_predicates().end(); ++pred) { 

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
        // extract lema and sense for the predicate
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
```


### Code

Find here the whole code:
* In [C++](code/example07.cc.md)
* In [python](code/example07.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

The output would be:

```
SVO : (pred:    eat [Eating+]
       subject: cat/true_cat [Feline+]
       dobject: fish [Fish=]
      )
SVO : (pred:    chase/chase_after/dog/give_chase/go_after/tag/tail/track/trail [Pursuing+]
       subject: canis_familiaris/dog/domestic_dog [Canine+]
       dobject: cat/true_cat [Feline+]
      )
SVO : (pred:    eat [Eating+]
       subject: canis_familiaris/dog/domestic_dog [Canine+]
       dobject: meat [Meat=]
      )
SVO : (pred:    hunt/hunt_down/run/track_down [Hunting+]
       subject: cat/true_cat [Feline+]
       dobject: mouse [Mouse=]
      )
```

We can see that the extracted triples are now generalized, since they use WN synonyms and SUMO concept codes to represent the predicates and their arguments.

