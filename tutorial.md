# FreeLing Tutorial

This tutorial presents some examples of how to use FreeLing main classes for simple language processing objectives.

The goal of this tutorial is not covering all FreeLing possibilities, but get you started with the library.  
Once you understand the basics, you are encouraged to inspect further modules and devise your own ways to exploit FreeLing capabilities.

## 0. Preliminaries:  Read this first to properly follow the tutorial examples

This tutorial presents examples about how to use different FreeLing modules.

Each example is a simple program, and most of them share a significant part of the code with other examples.   
When that is the case, it is explicitly stated, and only the differences are described. So, when an example description reads "this example is based in _Example X_", you should follow _Example X_ first and understand the code, otherwise you will not understand which parts are changing and which are not.

Complete code for all examples is provided. Examples are C++ programs, which need to be compiled with FreeLing library. So, you'll need to have FreeLing installed in your computer if you want to compile the examples.

To compile the examples, you need to use the command:

```bash
g++ example.cc -o example -lfreeling -std=c++0x
```

If you installed FreeLing somewhere different than `/usr` or `/usr/local` you will need to add appropriate `-I` and `-L` flags to tell the compiler where to locate the library. For instance, if you installed FreeLing in `/my/freeling/install`, you need to do:

```bash
g++ example.cc -o example -lfreeling -std=c++0x -L/my/freeling/install/lib -I/my/freeling/install/include
```

Once the program is compiled, you can run it with:

```bash
./example < mytext.txt
```

If you have installed FreeLing in a custom directory, the loader may fail to find the library.   
To fix this, execute:

```bash
export LD_LIBRARY_PATH=/my/freeling/install/lib
```

Most example programs expect the language of the input text as first argument, and the directory of FreeLing installation as second argument. If not provided, these arguments default to `en` \(English\) for the language and to `/usr/local` for the directory.  
So, different ways to run the example programs could be:

```bash
./example < mytext.txt                       ## (default values: English, FreeLing installed in '/usr/local')
./example en /my/installed/FL < mytext.txt   ## (English, FreeLing installed in '/my/installed/FL')
./example es < mytext.txt                    ## (Spanish, FreeLing installed in '/usr/local')
./example it /usr < mytext.txt               ## (Italian, FreeLing installed in '/usr')
```

Some programs expect different argument. In those cases, the execution command is explicitly described.

## 1. The Basics: Morphology and PoS Tagging

This section introduces the basic FreeLing processing modules, and how to handle the data structure they return.

* [Example 01: PoS tagger](./example01.md)
* [Example 02: Post-processing](./example02.md).

## 2. Adding Syntax and Semantics

In this section, we will extend the basic examples seen so far with the use of a word sense disambiguation module and a syntactic parser, and we will illustrate how to traverse the enriched data structure.

* [Example 03: Adding Word Sense Disambiguation](./example03.md)
* [Example 04: Using the Parser to Extract SVO Triples](./example04.md)
* [Example 05: Better Triples Using the Semantic Role Labeller](./example05.md).

## 3. Advanced semantics with WordNet

This section introduces the use of auxiliary classes to browse ontological information.

* [Example 06: Navigating WordNet](./example06.md)
* [Example 07: Extracting Triples with Semantic Information](./example07.md)

## 4. Processing non-ending text

In previous chapters we saw how to process a document with FreeLing. The text was first read into a string, and then processed by each of the required modules.  
 But in some scenarios it is not possible to load a whole document into a string \(e.g. when the document is very long\), or we may simply want to process a never-ending incoming stream of text.

This section presents some examples of how to use FreeLing to process non-ending text. We will assume that the text is input one line after another, and that each line may contain several sentences. It is also possible that the last sentence in the line is not finished and we need to wait for the next line to decide whether it is a continuation or a new sentence.

* [Example 08: SVO Triple extractor on non-ending text](./example08.md)

## 5. Creating custom pipelines

In previous examples, we have seen how to use FreeLing analysis modules to process text, either as a whole document or in a never-ending stream.

But if one wants to run a long chain of modules, or needs to use it in different places in the application, the code may become messy and unpleasant to maintain.

For this reason, FreeLing provides the `analyzer` meta-module. This module allows the application to create a single instance that will contain all desired modules and call them appropriately. This makes code much simpler, since a single call is enough to get all required processing applied to a text.  
It also makes it simpler to create a series of analysis pipelines \(e.g. one for each language\) inside the same application.

* [Example 09: Simpler code with `analyzer` meta-module](./example09.md)

## 6. Other Useful Modules

This section presents a miscelanous of examples about some FreeLing modules that are not typically used in a language processing pipeline, but that may be useful for many applications.

* [Example 10: Language Identifier](./example10.md)
* [Example 11: Input/Output formats](./example11.md)
* [Example 12: Candidate Corrections for Mispelled Words](./example12.md)
* [Example 13: Tagset Manipulation](./example13.md)
* [Example 14: Using the Feature Extractor](./example14.md)



