# Building a modern DarkWeb Crawler

### What is a crawler ?

A web crawler is a computer program that browse the internet to index existing pages, images, PDF, ... and allow user to search them using a search engine. It's basically the technology behind the famous google search engine.

Typically a efficient web crawler is designed to be distributed: instead of a single program that runs on a dedicated server, it's multiples instances of several programs that run on several servers (eg: on the cloud) that allows better task repartition, increased performances and increased bandwidth.

But distributed softwares does not come without drawbacks: there is factors that may add extra latency to your program and may decrease performances such as network latency, synchronization problems, poorly designed communication protocol, etc...

To be efficient, a distributed web crawler has to be well designed: it is important to eliminate as many bottlenecks as possible: as french admiral Olivier Lajous has said:

> The weakest link determines the strength of the whole chain.


# Trandoshan : A crawler

### What is the darkweb ?

- I won't be too technical to describe what the dark web is, since it may need is own article.

- The web is designed is composed of 3 layers and we can think of it like an iceberg:

- The Surface Web, or Clear Web is the part that we browse everyday. It's indexed by popular web crawler such as Google, Qwant, Duckduckgo, etc...

- The Deep Web is a part of the web non indexed, It means that you cannot find these websites using a search engine but you'll need to access them by knowing the associated URL / IP address.

- The Dark Web is a part of the web that you't cannot access using a regular browser. You'll need to use a particular application or a special proxy. The most famous dark web is the hidden services built on the tor network. They can be accessed using special URL who ends with .onion


### Iceberg

![image](https://user-images.githubusercontent.com/86452055/149205463-88cd3944-e395-4f0b-8fbe-d0a6163ff657.png)


### Trandoshan design

![image](https://user-images.githubusercontent.com/86452055/149205546-1a0a9ac0-de31-4b63-98df-9df5f2d7ad75.png)


Before talking about the responsibility of each process it is important to understand how they talk to each others.

The inter process communication (IPC) is mainly done using a messaging protocol known as NATS (yellow line in the diagram) based on the producers / consumers pattern. Each message in NATS has a subject (like an email) that allow other process to identify it and therefore to read only messages they want to read. NATS allowing scaling: for example they can be 10 crawler processes reading URL from the messaging server. Each of these process will receive an unique URL to crawl. This allow process concurrency (many instances can run at the same time without any bugs) and therefore increase performances.

Trandoshan is divided in 4 principal processes:

**Crawler**: The process responsible of crawling pages: it read URLs to crawl from NATS (message identified by subject "todoUrls"), crawl the page, and extract all URLs present in the page. These extracted URLs are sent to NATS with subject "crawledUrls", and the page body (the whole content) is sent to NATS with subject "content".

**Scheduler**: The process responsible of URL approval: this process read the "crawledUrls" messages, check if the URL is to be crawled (if the URL has not been already crawled) and If so, send the URL to NATS with subject "todoUrls"

**Persister**: The process responsible of content archiving: it read page content (message identified by subject "content") and store them into a NoSQL database (MongoDB).

**API**: The process used by other processes to gather informations. For example it is used by the Scheduler to determinate if a page has been already crawled. Instead of directly calling the database to check if an URL exist (which would add extra coupling to the database technology) the scheduler use to API: this allow sort of abstraction between database / processes.

