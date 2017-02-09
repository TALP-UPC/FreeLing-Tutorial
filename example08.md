
## Example 08: SVO Triple extractor on non-ending text

  In this example, we will input a text of a potentially huge size, and process it line by line. 
  Whenever FreeLing finds one or more whole sentences, we will process them.
  Whole sentences will be detected when the last read line either contained whole sentences, or it contained a partial sentence that satisfactorily completed a sentence started in previou(s) line(s).

 This example is based on [*Example 4*](example04.md), and produces the same output (SVO triples).

 The difference is that instead of loading the whole input text and then processing it afterwards, we will read it line by line as long as they keep coming. We will not wait for the input to end, but we will process sentences as soon as they are complete (even if they are split in several lines).

 To do this, the splitter needs to be able to keep an internal buffer with partial sentences waiting for completion.
This is achieved opening a *session* with the splitter, which will create an internal buffer and return an identifier for it.

So, we not only to create a splitter instance, but we also get a session identifier. Note that this can be used to write a program that uses the same splitter instance to split different texts in parallel, each with its own session.

```C++
freeling::splitter sp(lpath + L"splitter.dat");
freeling::splitter::session_id sid = sp.open_session();
```

So, our main program will get each text line, try to extract sentences from it, and process the extracted sentences.
Note that now the call to the splitter has the session identifier as an extra parameter.
The last `false` parameter tells the splitter to retain words if it looks like the sentence may continue in the next line.

So, we replace the part of loading the text and afterwards process it, with:
```C++
// get plain text input lines while not EOF.
wstring text;
while (getline(wcin,text)) {

   // tokenize input line into a list of words
   list<freeling::word> lw=tk.tokenize(text);
    
   // accumulate list of words in splitter buffer, returning a list of sentences.
   list<freeling::sentence> ls=sp.split(sid, lw, false);
    
   // perform and output morphosyntactic analysis and disambiguation
   morfo.analyze(ls);
   tagger.analyze(ls);
   sen.analyze(ls);
   wsd.analyze(ls);
   parser.analyze(ls)
 
   // do whatever is needed with processed sentences   
   ProcessSentences(ls);
}
```

For each read line, we call the tokenizer that will return a list of `word` with the tokens of the current line.

This list is then given to the splitter that will return a list of sentences. 
The returned list of sentences may be empty (e.g. if the line din't contain a whole sentence). In this case, the splitter will retain the words in its internal buffer waiting for the sentence to continue --and eventually end-- in the next line.
When the splitter finds the end of the sentence, it will create the sentence using words in the buffer and return it.

For instance, with the input
```
The cat eats
fish. The dog barks
aloud under
the tree. The bird flies.
```

When the first line (`The cat eats`) is read, the splitter returns an empty list of sentences and accumulates the words in the buffer.
When the second line (`fish. The dog barks`) is read the splitter detects the end of the first sentence, returns a list with one sentence {*The cat eats fish.*} and retains the words `The dog barks` in the buffer waiting for sentence continuation. 
When the third line (`aloud under`) is read, the splitter just adds the new words to the buffer.
Finally, when the last line is read, the splitter completes the second sentence, and it also detects the whole third sentence, thus returning a list with two sentences: {*The dog barks aloud under the tree.* | *The bird flies.*}

The splitter output goes through all subsequent modules (if the sentence list is empty, all modules simply ignore it). 
Then, we can call function `ProcessSentences` that will extract triples for the sentences detected in the current loop iteration.

If our input text is not never-ending but simply very large, the loop will eventually reach the end of the file and stop.
If the last sentence in the file was missing a clear ending mark (such a dot or a question mark), the splitter will retain it and it will not be processed.
If we want to be sure that the splitter does not retain anything we can add the following code after the end of the loop:

```C++
// No more lines to read. Make sure the splitter doesn't retain anything  
list<freeling::word> lw; 
list<freeling::sentence> ls = sp.split(sid, lw, true);
sp.close_session(sid);
 
// analyze and process sentence(s) which might be lingering in the buffer, if any.
morfo.analyze(ls);
tagger.analyze(ls);
parser.analyze(ls);
sen.analyze(ls);
wsd.analyze(ls);
ProcessSentences(ls); 
```

First, we call the splitter with an empty list of words and the last parameter set to `true`. 
This parameter means "do not expect sentence continuation, flush buffers as if there were a sentence end mark". 
In this way, we force the splitter to build a sentence with all words that may remain in the buffer.

Then, we just need to analyze and process the sentence (if any) as usual.


### Code

Find here the whole code:
  - In [C++](code/example08.cc.md)
  - In [python](code/example08.py.md)

