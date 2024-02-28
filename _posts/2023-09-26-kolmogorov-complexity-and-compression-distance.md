---
layout: post
---


Alice and Bob play a game of who gets the most number of tales in 20 coin tosses each.

Alice gets the sequence: `HHHTHTHTTHTHHTHTTTTH`, whereas, Bob gets the sequence: `TTTTTTTTTTTTTTTTTTTT`. Alice is perplexed at Bob's result. She senses foul-play and confronts Bob about it. Bob uses the following argument to defend himself:



Total number of 20 coin tosses sequences possible: $$2^{20}$$

Probability of getting a sequence at random from the $$2^{20}$$ sequences: $$2^{-20}$$



Bob claims that since the probability of getting both his and Alice's sequence is the same ($$2^{-20}$$), it proves that there was no foul-play involved. Bob credits his excellent luck. Alice is smart and cannot be easily convinced. She get's back at Bob by claiming that probability cannot be used in this context as it reveals no information regarding the randomness of the obtained sequences. One can take a quick glance at the obtained sequences and easily point out that Alice's sequence is more random than Bob's sequence. Alice needs a way to describe how "hard to describe" or random a string/sequence is. However, this argument lacks mathematical rigor. In this post, we'll help out Alice using a mathematical tool known as Kolmogorov Complexity.



Let's start with describing Bob's sequence in Python,

```py
In [0]: 'T'*20
Out[0]: TTTTTTTTTTTTTTTTTTTT
```

The length of this description in Python comes out to be 6 (as shown below) which is less than the length of the sequence itself.

```py
In [1]: len("'T'*20")
Out[1]: 6
```

If we were to describe Alice's sequence in Python, it would be quite complex (or literal) and the description would probably be equal to or longer than 20 characters. Another thing to keep in mind is that the description language matters too.  If we were to replicate the Python description for Bob's sequence in Javascript, we would get the following description:

```js
>> "T".repeat(20)
"TTTTTTTTTTTTTTTTTTTT"

>> "\"T\".repeat(20)".length
14
```

Notice how changing the description language increased the size of the description from 6 to 14. This tells us that a string's complexity does not only depend on the string but also on the description language. 

Considering the information we have on hand, we can mathematically define Kolmogorov complexity as follows:


$$
KC(x) = \min \{|d| : L(d) = x\}
$$

where, $$L$$ is the language that accepts the program $$d$$ that delivers the same output as the string, $$x$$. I'm gonna go ahead and butcher math for the sake of clarity. The above-mentioned mathematical representation would encompass our Python and Javascript examples in the following way:

$$
KC(\text{'TTTTTTTTTTTTTTTTTTTT'}) = \\ \min \{|d| : \text{Python}(d) = \text{'TTTTTTTTTTTTTTTTTTTT'}\} = 6 \\ where, d \in \{\text{'T'*20}, \text{'TTTTTTTTTTTTTTTTTTTT'}, \text{...}\}\\
$$



$$
KC(\text{'TTTTTTTTTTTTTTTTTTTT'}) = \\ \min \{|d| : \text{Javascript}(d) = \text{'TTTTTTTTTTTTTTTTTTTT'}\} = 14 \\ where, d \in \{\text{'T'.repeat(20)}, \text{'TTTTTTTTTTTTTTTTTTTT'}, \text{...}\}
$$


<hr />

<div>
This defines the Kolmogorov complexity of a string as the length of the shortest program outputting that string. The idea is that Kolmogorov complexity gives us a way to describe the randomness of a string. A string with its Kolmogorov complexity equal to the length of the string will be more random than the string with its Kolmogorov complexity less than the length of the string. Moreover, a string cannot be compressed if its $$KC(x) \geq |x|$$ Another thing to note is that Kolmogorov complexity of a string cannot be computed. There cannot exist a computer that will always guarantee the Kolmogorov complexity for all the strings. It is not a computational problem but rather a fundamentally theoretical one.  To better understand that, take a look at the <a href="https://en.wikipedia.org/wiki/Interesting_number_paradox">interesting number paradox</a>. 
</div>



> The interesting number paradox revolves around the claim that all  natural numbers are interesting. 1 is the first number, so that is  interesting. 2 is the first even number. 3 is the first odd prime  number. 4 is interesting because 4=2×2 and 4=2+2. We can continue in  this fashion and find interesting properties for many numbers. At some  point we might come to some number that does not seem to have an  interesting property. We can call that number the first uninteresting  number. But that, in itself, is an interesting property. In conclusion,  the uninteresting number is, in fact, interesting! [[source](https://nautil.us/kolmogorov-complexity-and-our-search-for-meaning-237158/)]

To summarize, we can never prove that the shortest program we've obtained is indeed the shortest program.

## Eliminating Language Dependency

Currently, we have a strong dependency on the type of language being used to describe the string and that doesn't attest this mathematical representation with consistency. In other words, we need to define complexity such that the definition does not change based on the $$L$$ we pick and depends only on the string. Changing from Python to Javascript in the example above changed our values. So, how do we generalize this and make it language-agnostic? To alleviate this issue, let's assume that there exists a universal language $$U$$ such that it always gives us the shortest description length for all strings. This would imply,


$$
KC_{U}(x) \leq KC_{L}(x) + C
$$

In other words, the complexity of describing a string $$x$$ using $$U$$ versus using an arbitrary language $$L$$ differs by at most a constant factor, $$C$$.





Basically, the theorem says that the Kolmogorov complexity of a string cannot be computed. 
