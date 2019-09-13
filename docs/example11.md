
## Example 11: Input/Output formats

FreeLing is a library, so its results are stored in data structures and returned to the called application, who is aware of what needs to be done with the results (processed, printed, serialized, etc) and in which format.

However, many applications need to be able to dump and load results of FreeLing analysis, and that is why the library also offers some convenience classes to produce or load usual formats in the NLP community.

In this example, we will see how to use the class `output_conll` to dump FreeLing analysis into a column format similar to that of CoNLL shared tasks.
This module has the advantatge of being configurable, so we can decide which columns and in which order the output is going to consist of.

We are going to use [*Example 9*](example09.md) as a starting point, and simply remove the `ProcessSentences` function and call the output handler to print the analysis instead. 

First, make sure to include the output handler classes:
```C++
#include "freeling/output/output_conll.h"
```

And in [*Example 9*](example09.md), remove function `ProcessSentences`, and instead of calling it after the analysis are completed, use the code:
```C++
// Create output handler and select desired output
freeling::io::output_conll out(L"out.cfg");
// print analysis results in conll format
out.PrintResults(wcout,ls);
```

That will create the output handler instance, and call it to print the results of the analysis.


### Code

Find here the whole code:

* In [C++](code/example11.cc.md)
* In [python](code/example11.py.md)


### Example

Assuming the input file contains the following sentences:

    The big cat eats fresh fish. My neighbour's dog chased the cat.
    The dog is eating meat. Some mice were hunted by the cat.

And if the file `out.cfg` contains, for instance:
```XML
<Type>
conll
</Type>
<Columns>
ID FORM LEMMA TAG DEPHEAD DEPREL SRL
</Columns>
```

we would obtain the output:
```
1 The   the   DT  3 NMOD -                    -  -
2 big   big   JJ  3 NMOD -                    -  -
3 cat   cat   NN  4 SBJ  -                    A0 -
4 eats  eat   VBZ 0 ROOT eat.03|eat.01|eat.02 -  -
5 fresh fresh JJ  6 NMOD -                    -  -
6 fish  fish  NN  4 OBJ  fish.01              A1 -
7 .     .     Fp  4 P    -                    -  -

1 My        my        PRP$ 2 NMOD   -                                  -
2 neighbour neighbour NN   4 NMOD   -                                  -
3 's        's        POS  2 SUFFIX -                                  -
4 dog       dog       NN   5 SBJ    -                                  A0
5 chased    chase     VBD  0 ROOT   tail.01|trail.01|chase.01|track.01 -
6 the       the       DT   7 NMOD   -                                  -
7 cat       cat       NN   5 OBJ    -                                  A1
8 .         .         Fp   5 P      -                                  -

1 The    the  DT  2 NMOD -                    -
2 dog    dog  NN  3 SBJ  -                    A0
3 is     be   VBZ 0 ROOT -                    -
4 eating eat  VBG 3 VC   eat.03|eat.01|eat.02 -
5 meat   meat NN  4 OBJ  -                    A1
6 .      .    Fp  3 P    -                    -

1 Some   some  DT  2 NMOD -       -
2 mice   mouse NNS 3 SBJ  -       A1
3 were   be    VBD 0 ROOT -       -
4 hunted hunt  VBN 3 VC   hunt.01 -
5 by     by    IN  4 LGS  -       A0
6 the    the   DT  7 NMOD -       -
7 cat    cat   NN  5 PMOD -       -
8 .      .     Fp  3 P    -       -

```

We can customize the output at will, selecting different columns, or altering the order in which they are printed.

For instance, with this content for `out.cfg`:
```XML
<Type>
conll
</Type>
<Columns>
ID TAG LEMMA SENSE DEPHEAD DEPREL FORM SPAN_BEGIN SPAN_END
</Columns>
```

We would get the output below:
```
1 DT  the   -          3 NMOD The   0  3
2 JJ  big   01382086-a 3 NMOD big   4  7
3 NN  cat   02121620-n 4 SBJ  cat   8  11
4 VBZ eat   01168468-v 0 ROOT eats  12 16
5 JJ  fresh 01067694-a 6 NMOD fresh 17 22
6 NN  fish  02512053-n 4 OBJ  fish  23 27
7 Fp  .     -          4 P    .     27 28

1 PRP$ my        -          2 NMOD   My        29 31
2 NN   neighbour 10352299-n 4 NMOD   neighbour 32 41
3 POS  's        -          2 SUFFIX 's        41 43
4 NN   dog       02084071-n 5 SBJ    dog       44 47
5 VBD  chase     02001858-v 0 ROOT   chased    48 54
6 DT   the       -          7 NMOD   the       55 58
7 NN   cat       02121620-n 5 OBJ    cat       59 62
8 Fp   .         -          5 P      .         62 63

1 DT  the  -          2 NMOD The    64 67
2 NN  dog  02084071-n 3 SBJ  dog    68 71
3 VBZ be   02604760-v 0 ROOT is     72 74
4 VBG eat  01168468-v 3 VC   eating 75 81
5 NN  meat 07649854-n 4 OBJ  meat   82 86
6 Fp  .    -          3 P    .      86 87

1 DT  some  -          2 NMOD Some   88  92
2 NNS mouse 10335563-n 3 SBJ  mice   93  97
3 VBD be    02604760-v 0 ROOT were   98  102
4 VBN hunt  01143838-v 3 VC   hunted 103 109
5 IN  by    -          4 LGS  by     110 112
6 DT  the   -          7 NMOD the    113 116
7 NN  cat   02121620-n 5 PMOD cat    117 120
8 Fp  .     -          3 P    .      120 121

```

### Remarks

It is important to note that the CoNLL-like output produced by the class `output_conll` can be loaded again into FreeLing by the class `input_conll` instantiated with the same configuration file. The input handler will return a list of sentences with the same content than the original dumped data structures. 

So, this pair of I/O handlers can be used as a way to serialize/deserialize FreeLing data structures to store them to disk, or to send them to other processes using FreeLing in a multi-process application. 

Also, FreeLing provides other output handlers that can dump the produced analysis to formats such as XML, json, or NAF.
