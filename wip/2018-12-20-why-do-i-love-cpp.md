---
layout: post
title: Why do I love c++?
comments: true
tags: [cpp]
---

One of my first tasks at my job as a test engineer at UniKey was to begin overhauling a c++ project on an arm platform. My previous programming experience was pretty limited as an electrical engineer undergrad, and doing things that we did't yet know how to do was the norm. 

My journey of learning c++ is full of experimentation and confusion. I see operators and syntax that I didn't understand. What are lambdas? `[](){ return what?; }`? How many times will I need to look on stack overflow for an explanation of & vs && vs *? How do I make interface-like objects? Templated code is mystifying! I wish c++ had all of the algorithms and iterator tools that I used from python... Seg faults are happening in this large code-base and I don't have a working debugger for this platform? Yes, welcome to hell!

# Farewell to the Former World

Yet with every new discovery, I was filled with a sense of realization. *That* is the way Java does interfaces under the hood? *This* is what Go doesn't allow you to do? My highly abstract code is being completely optimized to literally *nothing*? This isn't hell I've walked into, this is leaving the matrix.

Everything in other languages -clicks-! Immediately you realize why the other languages have the restrictions that they do, how they implement the features that they have. C++ is not just another Object Oriented or Functional Programming language. It is a Generic Programming language of which all else are merely limited or misguided subsets.

![Realization]({{ site.url }}/assets/neo_face.jpg)

But c++ transcends from superior to lovable by accomplishing this not through complexity, but through *simplicity*.

# Shoulders of Giants

Decades have passed since the time Bjarne Stroustrup first created "C with Objects". It is no small miracle that the c++ we have today has remained relatively untarnished by the *bright ideas* proposed by the geniuses around Bjarne. Alexander Stepanov, creator of the c++ Standard Template Library (STL), recollects his account of c++'s evolution:

> Bjarne was resolute in preserving his fundamental goals ... People were telling Bjarne that you should turn c++ into a fully object oriented language... make it automatically garbage collectable... make all the member functions virtual... eliminate global functions... in other words they wanted him to design Java. He listened politely to them and kept maintaining the fundamental integrity of the language.<br/>
CppCon 2014: Bjarne Stroustrup "Make Simple Tasks Simple!"

In other words, he told them to fuck off. None of these features could be simply integrated into the standard while remaining zero cost. His ideas have always been under constant assault from geniuses who think its okay to make sacrifices in runtime performance for something they like. Years of this behavior has led to bloated alternative languages, each one failing to be the sexy improved version of c++. 

The standards committee is filled with complementary personalities ensuring the future evolution of the language. Industry titans from every corner of computer science bringing their own perspective and needs to the table. Optimism balanced with Realism. Determination met with receptivity.

# I'm not locked into a specific build system
# I'm not locked into a specific programming style
# All fundamental tools in the standard are zero overhead
# The language operates implicitly/explicitly when I would expect
# The resulting binaries are extremely optimized


# Common Criticisms 

Any criticism directed at c++ before 2011 is completely invalid. As Herb Sutter mentioned at the end of his cppcon 2015 endnote, "c++ is a fresh new language, simpler than ever". 

The wild pessimism towards c++ that I've seen are from people that aren't embracing the new world of modern c++ that we are now in. I've seen old programmers decide that c style for loops, `for(int i = 0; i < count; ++i)` are more simple than new style range based for loops, `for(auto& element : array)`. They insist that single monolithic 100 line functions are more readable than refactored multi-function programs because they don't have to follow the control flow. The reason these people don't like c++ is because they are lost and are essentially trying to use \-\-c++ from the 90's.

## c++ is dangerous
### Dealing with raw pointers
### Memory leaks
## c++ is not simple
## c++ is old
