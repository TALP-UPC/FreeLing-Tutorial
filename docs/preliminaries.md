## Preliminaries:  Read this first to properly follow the tutorial examples

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

