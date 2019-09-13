
## Example 02: Post-processing

In the previous example  we saw how to PoS tag an input text and print the categories selected by the tagger for each word.

But this is tipically the output of any PoS tagger package, so we don't need to write our own program for that (we could use the script `analyze` provided with FreeLing, or any other tagger available in the Internet).

Using FreeLing as a library becomes handy when we want to do some post processing with the output of the tagger.
Once we have used FreeLing processing classes to annotate a text, we obtain a data structure that we can process depending on the needs of our application.

In this example, we will modify the `ProcessSentences` function in [*Example 1*](./example01.md) to do something more than simply output the content of the linguistic analyzers output. We will start with something simple, as for instance, counting how many times each lemma occurs in the input text (i.e. lemma unigram frequencies), and how many times two lemmas appear in consecutive positions in the text (i.e. lemma bigram frequencies).

We will use two map variable to store the lemma and lemma bigram counts, so we need to include the proper STL library
```C++
#include <map>
```

Then, we can replace function `ProcessSententes` with the code below:
```C++
void ProcessSentences(const list<freeling::sentence> &ls) {
   // maps to count lemmas
   map<wstring,int> lemmas;
   map<wstring,int> lemma_bigrams;

   // for each sentence in the list
   for (list<freeling::sentence>::const_iterator s=ls.begin(); s!=ls.end(); ++s) {
      // previous word lemma, for the bigrams
      wstring prev=L"";
      // for each word in the sentence
      for (freeling::sentence::const_iterator w=s->begin(); w!=s->end(); ++w) {
         // count an occurrence more for the current word lemma
		 add_count (lemmas, w->get_lemma());
         // count an occurrence more for the pair of previous and current word lemmas.
         if (prev!=L"")
            add_count(lemma_bigrams, prev+L" "+w->get_lemma());

         // move to next word
         prev = w->get_lemma();
      }
   }

   // once all lemmas are counted, output the results
   wcout << L"========== LEMMA FREQUENCIES (lemma,freq) ====================" << endl;
   for (map<wstring,int>::iterator p=lemmas.begin(); p!=lemmas.end(); ++p)
   	  wcout << p->first << L" " << p->second << endl;  
   wcout << endl;
   wcout << L"========== LEMMA BIGRAM FREQUENCIES (lemma1,lemma2,freq) ==============" << endl;
   for (map<wstring,int>::iterator p=lemma_bigrams.begin(); p!=lemma_bigrams.end(); ++p)
   	  wcout << p->first << L" " << p->second << endl;  
   wcout << endl;
}
```

The function `add_count` checks whether the given key is in the map, and increases the count if it is found.  If it is not found, the count is initialized to 1.
```C++
void add_count (map<wstring,int> &m, const wstring &w) {
   map<wstring,int>::iterator p = m.find(w);
   // key not found in the map, add it with count=1
   if (p==m.end()) m.insert(make_pair(w,1));
   // key previously seen, increase count
   else ++p->second;
}
```


### Code

Find here the whole code:
* In [C++](code/example02.cc.md)
* In [python](code/example02.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

The output would be:
```
========== LEMMA FREQUENCIES (lemma,freq) ====================
's 1
. 4
be 2
big 1
by 1
cat 3
chase 1
dog 2
eat 2
fish 1
fresh 1
hunt 1
meat 1
mouse 1
my 1
neighbour 1
some 1
the 4

========== LEMMA BIGRAM FREQUENCIES (lemma1,lemma2,freq) ==============
's dog 1
be eat 1
be hunt 1
big cat 1
by the 1
cat . 2
cat eat 1
chase the 1
dog be 1
dog chase 1
eat fresh 1
eat meat 1
fish . 1
fresh fish 1
hunt by 1
meat . 1
mouse be 1
my neighbour 1
neighbour 's 1
some mouse 1
the big 1
the cat 2
the dog 1
```
