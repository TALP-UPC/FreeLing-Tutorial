## Creating custom pipelines

In previous examples, we have seen how to use FreeLing analysis modules to process text, either as a whole document or in a never-ending stream.

But if one wants to run a long chain of modules, or needs to use it in different places in the application, the code may become messy and unpleasant to maintain.

For this reason, FreeLing provides the `analyzer` meta-module. This module allows the application to create a single instance that will contain all desired modules and call them appropriately. This makes code much simpler, since a single call is enough to get all required processing applied to a text.  
It also makes it simpler to create a series of analysis pipelines \(e.g. one for each language\) inside the same application.

* [Example 09: Simpler code with `analyzer` meta-module](example09.md)
