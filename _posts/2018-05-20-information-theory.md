---
layout: post
title: 'Information Theory: The Phantom of the Bits'
tags: [CS]
categories: [English]
---
I've been reading *Information Theory* by David Mackey every now and then during this term. It is a big book (at least to me), over 500 pages thick, and has many time-killing exercises. The frustrating part of it is that, **every** page is worth reading and putting efforts into. No immediately obvious way exists for me to master the book in like one week, which is the average time that I get bored and eventually abandon a book I'm reading.

So why read it if it takes so much time? Time, above all, is what we programmers (if not everyone) values most and are (will be) paid for. Why torture yourself with some old theories full of mathematical formulas, while you can learn some new language or fancy frameworks that are actually **useful**? If you are asking yourself the above question, you can now close this web page and do whatever fits you. Nothing prevents you from doing so, but if your curiosity craves more, then I'm glad to help.

I'm going to throw you some *realistic* conclusions without giving the proof. Watch out, you may already have some of them in mind, but this will be *formal* conclusion that you have always longed for.

## Noisy Channel

Consider the case when we're communicating over a noisy network, where the server tries to send the client some sequence of bits (message) but there's a probability of error for every bit sent. So the received message will contain some noise that makes it hard to recognize the original message.

Since unreliability is the nature of the channel, we would like the client to somehow *deduce* the original message from the tainted one. To tackle this problem, suppose we have an encoder at the server side and a decoder at the client side, and we design an encoding so that we can recover the original message from tainted message as closely as possible.

An example encoding would be to repeat every bit multiple times in the encoding process, and choose the bit with most occurrences among them as the *correct* bit when decoding it. This introduces **redundancy**, since we send more bits than we originally had now. Here I'll also give you the concept of transmission speed: the information each encoded bit conveys. For the example encoding, if we repeat each bit n times, then the transmission **rate** would be 1/n now, since we have to send n bits for a message containing only one bit. The **bit error probability** is the probability of a bit in decoded message being different from the original message.

So our goal would be to reduce the bit error probability as much as possible. A natural idea would be to repeat ourselves many times, so that we can be more confident of our result. But how *many*? Remember that the more times we repeat ourselves, the smaller our rate would be. So if you want to reduce the error as much as possible, you end up having a rate so slow that it's not practical at all! The error-rate trade-off seems unavoidable and leaves us in a  desperate situation, but not yet.

In the name of *perfect mathematical deductions*, we claim that for every such noisy channel with a fixed error probability, we can find an encoding with transmission speed $$1/C$$ (a constant) such that the bit error probability can be  made *arbitrarily small*. What this means is that it's theoretically possible to communicate reliably through an unreliable channel at a relatively acceptable rate! Forget about repeating yourself infinite times, that's for someone who doesn't *know* information theory.

## Source Coding

We all know that in the world of computer every piece of information is represented as *bits*, namely 0's and 1's. Take English text as an example: A sentence in English is basically a sequence of characters. In computer we usually encode the characters according to the famous ASCII standard. Every character has its own according 7-bit representation, so the sentence can be represented just as a sequence of bits.

A smart reader might already notice something unreasonable and ask the following question: Why do we need 7 bits for English text? The number of alphabets and symbols clearly aren't that many as 2^7 = 128, so there's an obvious waste there.

Indeed, the ASCII contains special symbols other than English and is a naive way to encode anything. If one thinks harder, he may come with the idea that we should encode frequent letters like 'a' and 'e' with shorter bits and less frequent ones like 'z' with longer bits, and with that comes the Huffman code. Huffman code is a neat way to encode symbols and can be proved to be able to express every symbol in less than $$H(x) + 1$$ bits on average, where $$H(x)$$ is the entropy of the symbol.

Entropy is just a way to measure the **information** contained in a symbol. We see that Huffman encoding is already pretty close to an *optimal* encoding that represents information in just the needed number of bits. However, this is still not the whole story. If you think more about English words, you can see that there is a lot more redundancy than just letter frequency. For example, if you see a 't' at the beginning of a word, then there's a large probability that you'll see a 'h' after it, so a prefix of 'th' isn't really that useful. However, if you see a prefix of 'ze', then you can almost be sure what the word would be. (zeal!)

Actually it is estimated that in English the average information in each letter is approximately 1 bit. This is obviously beyond what a vanilla Huffman code could achieve. Like, how could we represent one letter using one bit? But indeed we can if the length of the string is long enough. *Shannon's Source Coding Theorem* claims that we can represent any long enough string with $$H(x)$$ bits per symbol on average with negligible error probability. By error here we mean that there's a small number of string that we won't be able to represent here, but we can *mathematically* ignore them since they can be made really improbable.

## TBC

It takes longer than I expected to just illustrate some beautiful conclusions, and hopefully I've done well. I may write some more articles giving more insights about what information theory promises us, and perhaps also give intuitive proof about the theorems mentioned here (they're not so hard).

For those that can't wait for my unpromising subsequent articles, I encourage you to read the book yourself. It's available [online](http://www.inference.org.uk/mackay/itila/book.html) in pdf form (free!). It's an excellent book, and certainly covers much more things than I could ever write here.
