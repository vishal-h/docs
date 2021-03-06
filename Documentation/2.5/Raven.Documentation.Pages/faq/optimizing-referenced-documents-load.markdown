# Optimizing referenced documents load

One of the [design principals](http://ravendb.net/docs/theory/document-structure-design) that RavenDB adheres to is the idea that documents are independent, that all the data required to process a document is stored within the document itself.

That said, there are still valid scenarios where you might need to load associated documents, and that leads to a problem. Let us consider the following code:

{CODE optimizing_referenced_documents_load_1@Faq/OptimizingReferencedDocumentsLoad.cs /}

This code is going to make two separate requests to the server, one to load the order, and another to load the customer. This is inefficient, but since document databases don't have joins, you have no other choice.

RavenDB supports the notion of includes. Which is similar to joins, in that it allows you to load additional information from the database based on the information in the loaded document itself.

This is supported using the following API:

{CODE optimizing_referenced_documents_load_2@Faq/OptimizingReferencedDocumentsLoad.cs /}

Or this code:

{CODE optimizing_referenced_documents_load_3@Faq/OptimizingReferencedDocumentsLoad.cs /}

This code will only go to the server once. What actually happen is that RavenDB actually have two channels in which it can return information for a load request. The first is the results channel, which is what is returned from the Load method call. The second is the Includes channel, which contains all the included documents. Those documents are not returned from the Load method call, but they are added to the session unit of work, and subsequent requests to load them can be served directly from the session cache.

The same notion is also supported for queries:

{CODE optimizing_referenced_documents_load_4@Faq/OptimizingReferencedDocumentsLoad.cs /}

Or this code:

{CODE optimizing_referenced_documents_load_5@Faq/OptimizingReferencedDocumentsLoad.cs /}

Again, the customer documents would be loaded into the session, but not returned from the query. That keeps the separation between the different documents but drastically reduces the number of requests that you have to make.

Includes also work for collections. As in the following case:

{CODE optimizing_referenced_documents_load_6@Faq/OptimizingReferencedDocumentsLoad.cs /}

All the user's Roles documents where loaded as part of loading the user document itself.
