---
layout: post
title: Why do I love c++?
comments: true
tags: [cpp]
---

Encountering a new programming language can be overwhelming for anybody. Just like everyone else, my first time parsing a c++ program was quite confusing. There are so many new operators and syntax that are different than any other language. What are lambdas? `[](){ return what?; }`? How many times will I need to look on stack overflow for an explanation of & vs && vs *? How do I make interface-like objects? Templated code is mystifying! I wish c++ had all of the algorithms and iterator tools that I used from python... Seg faults are happening in this large code-base and I don't have a working debugger for this platform? Yes, welcome to programming hell!

# Farewell to the Former World

Yet with every new discovery, I was filled with a sense of realization. *That* is the way Java does interfaces under the hood? *This* is what Go doesn't allow you to do? My highly abstract code is being completely optimized to literally *nothing*? Rather than venturing into a deep mental quagmire, it felt more like c++ lifted the curtain on every other language's intricacies.

Everything in other languages -clicks-! I begin to realize why the other languages have the restrictions that they do, how they implement the features that they have. So what do all of these new symbols and syntaxes mean, and how could anyone think c++ feels fresh and new in 2019?

# Shoulders of Giants

Decades have passed since the time Bjarne Stroustrup first created "C with Objects". It is no small miracle that the c++ we have today has remained relatively untarnished by the *bright ideas* proposed by the *geniuses* that surrounded Bjarne. Alexander Stepanov, creator of the c++ Standard Template Library (STL), recollects his account of c++'s evolution:

> Bjarne was resolute in preserving his fundamental goals ... People were telling Bjarne that you should turn c++ into a fully object oriented language... make it automatically garbage collectable... make all the member functions virtual... eliminate global functions... in other words they wanted him to design Java. He listened politely to them and kept maintaining the fundamental integrity of the language.<br/>
[CppCon 2014: Bjarne Stroustrup "Make Simple Tasks Simple!"](https://youtu.be/nesCaocNjtQ?t=18)

He was a visionary, not succumbing to the bad advice of those around him that went on to create Java. None of these features could be simply integrated into the standard while remaining zero cost. His ideas have always been under constant assault from geniuses who think its okay to make sacrifices in runtime performance for some feature they like. Years of this behavior has led to bloated alternative languages, each one ultimately failing to be the sexy improved alternative to c++ that it promises to be. 

Today, the c++ standards committee is filled with complementary personalities ensuring the future evolution of the language. Industry titans from every corner of computer science bringing their own perspective and needs to the table. Optimism balanced with Realism. Determination met with receptivity.

# Common Criticisms

All of criticism that I have seen of c++ seems very ignorant of how the language has developed within the last decade. As Herb Sutter mentioned at the end of his CppCon 2014 endnote, ["c++ is a fresh new language, simpler than ever".](https://youtu.be/xnqTKD8uD64?t=5713)

The wild pessimism towards c++ that I've seen are from people that aren't embracing the new world of modern c++ that we are now in. Many people now turn to Rust as a similar alternative to c++ without realizing how similar modern c++ is to Rust. But I want to point out a few features and lack of features that I think are actually great strengths of the language.

## No Universal Build System or Package Manager
C++ lacks a package manager like Java or Rust. Idiomatic c++ projects don't simply import modules from a canonical repository like npm (javascript), Cargo (rust), pip (python). Instead, different solutions emerged from the vacuum for c++ over time. In my experience, package managers end up not really solving the problem they are designed to solve. Anyone who's tried to deploy they're own complex Maven project knows that its a true nightmare.

Similarly, c++ has a variety of choices for build systems. CMake is prevalent, you can use simple makefiles just like c, or you could use countless other custom solutions.
I've found that languages which support a singular build system are really easy to build simple demo apps, but once you need to build something that the default build system doesn't support you're in big trouble. 

## Multiple Implementations
C++ is not like other languages that have monolithic compiler implementations. The commitee defines c++ standards and specifications, but vendors implement the standards in isolation and competition with eachother. I've come to appreciate that different people coming up with multiple implementations end up feeding off of the competition and making the compilers much faster than they otherwise would be alone. 

## Zero Runtime Overhead
One of the main things that holds back new features from entering the next c++ standard are implementation details. The committee refuses to shove in features which would add unnecessary overhead for people that aren't using the feature, or would break backwards compatibility. This is why new languages like Rust can come out in interim periods appearing to have new features which c++ lacks.
However, c++ will always catch up with a better implementation due to their caution. In the meantime, most bleeding edge language features can be supported through separate user libraries anyway.
