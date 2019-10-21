---
layout: post
title:  "A Trial on Purely Rule-Based Legal English Processing - 1"
date:   2017-10-22 01:24 +1000
categories: work
---

# Why
We are doing some NLP tasks on the WCO (World Customs Organization / Organisation Mondiale des Douanes)'s Harmonized System 
Nomenclature.

# Why is English a bad language for NLP tasks
Well, we all know that English is a highly Romanized German language (as in contrast to Romansh, lol), and many different sources 
of roots and grammar rules can be found in English (including Greek, Roman languages, German languages, Celtic languages, etc.), 
hence it is a 
difficult task for a common NLP approach (i.e. statistical) to achieve a good result (please don't take BERT into account, that's 
something else).

After the long evolution of English, it has reached some point in between of synthetic language and isolated language, 
the loss of inflectionaly features has made 
it is 
an impossible task to infer the POS of a word without considering its context (looking at highly inflectional languages). And as we 
all know, there is nothing that really does **consider** the context of a word, the so-called best tools (namely Stanford Core NLP 
Parser and spaCy) both failed to produce correct syntax tree of sentences found in that HS Nomenclature.

# What is Legal English
Basically, Legal English is something that lawyers use to write legal text. Unlike vulgar English (that most statistical tools used
to train their models), Legal English has a way more strict set of syntactical rules.

Then the idea came to my mind that, why not consider Legal English as a badly designed formal language? Is it possible to achieve
the parsing goal without evening thinking of the semantics of the sentences?

I would say yes.

Consider a relatively complex Heading (not even a complete sentence tho) from the Nomenclature:

> Pig fat, free of lean meat, and poultry fat, not rendered or
otherwise extracted, fresh, chilled, frozen, salted, in brine, dried
or smoked. 

For anyone who has passed the English grammar exams (IELTS / TOEFL / GMAT / etc.) can parse this sentence by their brain. 

And we can almost directly write the syntax rule for it after parsing it:

```
S -> And-List-of NP ("," Or-List-of ADJs)+
And-List-of 'a -> 'a "and" 'a | 'a "," And-List-of 'a
Or-List-of 'a -> 'a "or" 'a | 'a "," Or-List-of 'a
NP -> NN NN | NP "," ADJP ","
ADJs -> ADJ | ADV ADJ
```

TBC.
