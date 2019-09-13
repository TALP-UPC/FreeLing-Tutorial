
## Example 05: Better Triples Using the Semantic Role Labeller

In [*Example 4*](example04.md) we saw an easy way to extract SVO triples, but we also saw that some sentences, such as passive sentences or compound verb tenses, have a different tree structure which would require programming more specific patterns.

In this example, we'll see how a semantic role labeller can save us the work, since it will identify the arguments of a verb, regardless of the surface syntactic structure or verb tense.

The code will be the same than we used in [*Example 4*](example04.md), since the dependency parser we used there not only produces dependency trees but also performs SRL.

So, after running the parser, sentences contain not also the dependency tree we used before, but also a list of predicates with their arguments. 
To extract the triplets we will modify  our `ProcessSentence` function to combine the information in the tree with the information in the predicate list.

```C++
//---------------------------------------------
// Do whatever is needed with analyzed sentences
//---------------------------------------------

void ProcessSentences(const list<freeling::sentence> &ls) {

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
        // process found triple
        wcout << L"SVO : (pred:    " << lpred << L" [" << spred << L"]" << endl;
        wcout << L"       subject: " << lsubj << L" [" << ssubj << L"]" << endl; 
        wcout << L"       dobject: " << ldobj << L" [" << sdobj << L"]" << endl;
        wcout << L"      )" << endl;
      }      
    }
  }
}
```

### Code

Find here the whole code:

* In [C++](code/example05.cc.md)
* In [python](code/example05.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

The output would be:
```
SVO : (pred:    eat [01168468-v]
       subject: cat [02121620-n]
       dobject: fish [02512053-n]
      )
SVO : (pred:    chase [02001858-v]
       subject: dog [02084071-n]
       dobject: cat [02121620-n]
      )
SVO : (pred:    eat [01168468-v]
       subject: dog [02084071-n]
       dobject: meat [07649854-n]
      )
SVO : (pred:    hunt [01143838-v]
       subject: cat [02121620-n]
       dobject: mouse [10335563-n]
      )
```

We can see that now we get the SVO triples for all four sentences, despite the compound tense and the passive structure of the last two sentences.

