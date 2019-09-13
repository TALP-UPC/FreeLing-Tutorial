
## Example 04: Using the Parser to Extract SVO Triples

In this example, we will use the parser to obtain a syntactic analysis of each sentece, and then process the obtained trees to detect some relevant patterns, such as Subject-verb-object triples describing who did what to who.

We will use code from [*Example 3*](example03.md) as a starting point, and add code to create a parser instance.

```C++
// Create dependency parser & SRL
freeling::dep_treeler parser(lpath+L"dep_treeler/dependences.dat");
```

  Here we use a statistical dependency parser, though FreeLing has other parsers  (one rule-based dependency parser and one rule-based constituency parser) that you can use.
  
We also must add a call to the parser after the other modules.

```C++
// parse sentences
parser.analyze(ls);
```

After calling the parser, `sentence` objects are enriched with a dependency tree  we can traverse.
Next step is modifying our `ProcessSentence` function search for SVO triples and output them:

```C++
void ProcessSentences(const list<freeling::sentence> &ls) {

  // for each sentence in list
  for (list<freeling::sentence>::const_iterator s=ls.begin(); s!=ls.end(); ++s) {
    for  (freeling::dep_tree::const_iterator p=s->get_dep_tree().begin(); p!=s->get_dep_tree().end(); p++) {
      // if it is a verb, check dependants
      if  (p->get_word().get_tag()[0]==L'V') {
        wstring ssubj, lsubj;
        wstring sdobj, ldobj;
        for (freeling::dep_tree::const_sibling_iterator ch = p.sibling_begin(); ch!=p.sibling_end(); ch++) { 
          if (ch->get_label()==L"SBJ") 
            extract_lemma_and_sense(ch->get_word(), lsubj, ssubj);

          else if (ch->get_label()==L"OBJ") 
            extract_lemma_and_sense(ch->get_word(), ldobj, sdobj);
        }
        // if we found a SVO triple, output it
        if (lsubj!=L"" and ldobj!=L"") {
          wstring spred, lpred;
          extract_lemma_and_sense(p->get_word(), lpred, spred);

          wcout << L"SVO : (pred:    " << lpred << L" [" << spred << L"]" << endl;
          wcout << L"       subject: " << lsubj << L" [" << ssubj << L"]" << endl; 
          wcout << L"       dobject: " << ldobj << L" [" << sdobj << L"]" << endl;
          wcout << L"      )" << endl;
        }
      }
    }
  }
}
```

The function `extract_lemma_and_sense` is just to make code nicer. and looks like this
```C++
//---------------------------------------------
// Extract lemma and sense of word 'w' and store them
// in 'lem' and 'sens' respectively
//---------------------------------------------

void extract_lemma_and_sense(const freeling::word &w, wstring &lem, wstring &sens) {
   lem = w.get_lemma();
   if (not w.get_senses().empty()) sens = w.get_senses().begin()->first;
   else sens=L"";
}
```

### Code

Find here the whole code:
    - In [C++](code/example04.cc.md)
    - In [python](code/example04.py.md)


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
```

### Remarks

- We can see that the SVO triples for the first two sentences were correctly extracted, but not the triples for the last two sentences.
    The reason is that these sentences have either a compound verb tense (*is eating*) or a passive construction, which have different tree structures than active present tense sentences. Since our program looks only for a particular structure, it didn't find the triples for those cases.
    In [*Example 5*](example05.md) we will see how this can be overcome using a Semantic Role Labeller. 

- Finally, note that the code looks for nodes in the tree with syntactic function `SBJ` and `OBJ`, which are the labels used by the English parser. If you want to use this program for a different language, you may need to change these labels to the appropriate ones.

