## Processing non-ending text

In previous chapters we saw how to process a document with FreeLing. The text was first read into a string, and then processed by each of the required modules.
But in some scenarios it is not possible to load a whole document into a string \(e.g. when the document is very long\), or we may simply want to process a never-ending incoming stream of text.

This section presents some examples of how to use FreeLing to process non-ending text. We will assume that the text is input one line after another, and that each line may contain several sentences. It is also possible that the last sentence in the line is not finished and we need to wait for the next line to decide whether it is a continuation or a new sentence.

* [Example 08: SVO Triple extractor on non-ending text](example08.md)
