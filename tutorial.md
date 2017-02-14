

## 4. Processing non-ending text

In previous chapters we saw how to process a document with FreeLing. The text was first read into a string, and then processed by each of the required modules.  
 But in some scenarios it is not possible to load a whole document into a string \(e.g. when the document is very long\), or we may simply want to process a never-ending incoming stream of text.

This section presents some examples of how to use FreeLing to process non-ending text. We will assume that the text is input one line after another, and that each line may contain several sentences. It is also possible that the last sentence in the line is not finished and we need to wait for the next line to decide whether it is a continuation or a new sentence.

* [Example 08: SVO Triple extractor on non-ending text](example08.md)

## 5. Creating custom pipelines

In previous examples, we have seen how to use FreeLing analysis modules to process text, either as a whole document or in a never-ending stream.

But if one wants to run a long chain of modules, or needs to use it in different places in the application, the code may become messy and unpleasant to maintain.

For this reason, FreeLing provides the `analyzer` meta-module. This module allows the application to create a single instance that will contain all desired modules and call them appropriately. This makes code much simpler, since a single call is enough to get all required processing applied to a text.  
It also makes it simpler to create a series of analysis pipelines \(e.g. one for each language\) inside the same application.

* [Example 09: Simpler code with `analyzer` meta-module](example09.md)

## 6. Other Useful Modules

This section presents a miscelanous of examples about some FreeLing modules that are not typically used in a language processing pipeline, but that may be useful for many applications.

* [Example 10: Language Identifier](example10.md)
* [Example 11: Input/Output formats](example11.md)
* [Example 12: Candidate Corrections for Mispelled Words](example12.md)
* [Example 13: Tagset Manipulation](example13.md)
* [Example 14: Using the Feature Extractor](example14.md)



