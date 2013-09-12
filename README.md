Digital-prosopography
=====================

Prosopography project using Gate and RDF

This project aims for extracting entities using Gate and querying database using RDF.


Installation
============

Install Apache Tomcat server:
```
sudo apt-get install tomcat6
```
Install mysql:
```
sudo apt-get install mysql-server mysql-client
```
Install sesame server:
```
go to http://www.openrdf.org/doc/sesame/users/ch02.html and follow the instruction under section "2.2.2. Installation under Tomcat 4 or 5"
```
Install GATE:
```
Download the jar file from http://gate.ac.uk/download/ and follow the corresponding instruction.
```

The Overall Process
===================

The raw file is uploaded through the JSF page  
The file is inserted into the article table in MySQL database  
GATE is invoked in the embedded mode with Tomcat server.  
Corresponding gapp files are loaded and the raw file is splitted and keywords are extracted to generate the xml files.  
The generated xml files and nq files are also stored in the MySQL database.  
Then the nq files are inserted into the RDF repository of Sesame.  

Detailed Workflow
=================

Upload Page  
The upload page is developed with Java Server Faces+ MyFaces+Tomcat technologies.  
Each raw file will be stored in MySQL DB before processed by GATE.  

Gate  
After the upload process, tomcat invoke preprocess script to clean up the uploaded document. Then Gate takes the output of preprocess and performs entity extraction.  
The results generated from GATE is sent to a post-process script, making sure the xml file saved is as clean as possible.  
The post-process script at the same time convert content of xml file to corresponding sparql insert queries in “.nq” files as inputs to the sesame server.  
 
Sesame RDF  
Sesame is Java open-source project which manages storage and querying of RDF data.  
Sesame provides SPARQL query interface.  
Native Sesame storage is utilized to store the nq files generated by GATE.  

GATE Workflow Summary
=====================

Gate is a natural language processing tool used in Digital Prosopography project that scans through the uploaded document from the above section, extracts name entities, and uses certian algorithm to make logical conclusion of the relations between entities. The output file generated from the algorithm will be in xml format with Event-Entity tree structure. The following plugins are the core of Gate and it’s function.

ANNIE Plugins:  
- English Tokeniser: Tokenize word, numbers, Symbol, Punctuation, and spaces
- Gazetteer: Gate built in database. Can create new tables, add new rows, etc. Machine learning. Lst file is under $gatehome/plugins/ANNIE/resources/gazetteer/  Lst file is under $gatehome/plugins/ANNIE/resources/gazetteer/
- Sentence Splitter: Split paragraph to multiple sentences based on regex
- Part Of Speech Tagger: Brill Tagger method
- Pronominal Coreference: Pronoun resolution
 
Customized Plugins:
- Noun: Extract Tokens that are nouns
- Verb: Extract Tokens that are verbs
- Event Separator: Separate a single sentence into multiple segments for event extraction
- Event: Extract a single event
 
Optional Plugins:
- Birth and Death: Extract birth and death verbs specifically
- Birth and Death Event: Extract birth and death event

Gate runs through the plugins specified on the gapp file on the input document, and the output is saved as a xml file. (Note that a “.gapp” file is a GATE config file that specifies which plugin to run).  

Customizing for bakers  
Instead of calling Customized Plugins, we call the Optional Plugins shown above. Thus at this stage we only extract birth and death date & location.

RDF Event Modelling
===================

Quad – Subject, Predict, Object, Context
 
Model 1: (EventID as Subject)  
Subject – EventID  
Predict – [hasSubject, hasAction, hasObject, hasDate, hasLocation]  
Object – any object/other EventID  
Context – Source (usually the name of the book)  
```
Example RDF:
<http://humanhistoryproject.ca/Aaron_Pietro_2013-08-29_00:43:11_1> <http://humanhistoryproject.ca/hasSubject> <http://humanhistoryproject.ca/Aaron_Pietro> .
<http://humanhistoryproject.ca/Bakers> .

<http://humanhistoryproject.ca/Aaron_Pietro_2013-08-29_00:43:11_1> <http://humanhistoryproject.ca/hasAction> <http://humanhistoryproject.ca/born> 
<http://humanhistoryproject.ca/Bakers> .

<http://humanhistoryproject.ca/Aaron_Pietro_2013-08-29_00:43:11_1> <http://humanhistoryproject.ca/hasTime> <http://humanhistoryproject.ca/1480> <http://humanhistoryproject.ca/Bakers> .
```
Model 2: (EventID as context)  
Subject – any person/EventID  
Predict – the action, verb  
Object – any object  
Context – EventID  
 
Reasons for choosing Model 1:  
Model structure is flat, easy to search, compute time small. Can have context/source specified at any level. Flexible, can add new namespaces as you go, i.e., does not restrict a certain event adding new namespace. The disadvantage of model 1 is that the event ID is always the subject in the quad modeling, which is difficult to interpret by users. 
 
Things to concern about:  
Verb hypernym/hyponym. Alg for restricting a good set of verbs  
Exact same event duplicates with different EventID  
Date location various format hard to perform search alg  
 




