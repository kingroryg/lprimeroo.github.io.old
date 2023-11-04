---
layout: post
---

Before we understand what the _Kullback-Leibler (or KL) divergence_ is, we need to understand what the term _entropy_ means or represents in the field of information theory and statistics.

## Entropy

This term first found it's way into a myriad number of statistical applications (ignoring the relation to thermodynamics) through a paper published by the legendary <a href="https://en.wikipedia.org/wiki/Claude_Shannon">Claude Shannon</a> in 1948 called "A Mathematical Theory of Communication"[<a href="http://math.harvard.edu/~ctm/home/text/others/shannon/entropy/entropy.pdf">pdf</a>]. The main problem being targeted here was the communication of bits in a network. With the advent of the digital age in the mid-1900s, it became increasingly crucial to efficiently send bits(or information) from a source to a destination.

We know that a 2 digit binary number can represent 2<sup>2</sup> pieces of information. If we add another digit to it, we get 2<sup>3</sup> ways of representing the same information. In a nutshell, a single bit can reduce our uncertainty of knowing something by a factor of 2 or we get twice the number of ways to represent the same information. However in our bit sequence, there will be certain combinations which aren't used at all or maybe some of them represent an error code of some sort. What if we used a 3-bit uniform sequence to represent 6 pieces of information? Not that efficient, right? Wouldn't it be better if we devise a code to reduce the number of bits and preserve only the critical information at the same time?

This is where _entropy_ comes in. Looking at the information, it tells us the ideal number of bits required to represent a particular information for transmission.

Let's consider an example.

<p align="center" markdown="1">
![](https://spaceplace.nasa.gov/glossary/en/searching-earth.en.png)
</p>
Suppose, you are an official at NASA and your job is to receive the temperature levels of a particular planet from a fellow astronaut stationed at the International Space Station. Years of observation of this planet's temperatures levels have concluded that it can only have 8 different levels of it.

```py
possible_temperatures_levels = [100, 105, 110, 145, 160, 162, 174, 180]
```

Each of them has a certain probability of occurring attached to it. It's clear that some of them occur more commonly than the others.

```py
# (of course, they all sum up to 1)
temperature_probs = [0.30, 0.40, 0.05, 0.06, 0.07, 0.02, 0.05, 0.05]
```
Let's consider a trivial example at first.

Suppose, the astronaut wants to convey today's temperature to the NASA official. He could do it by sending the string - "Today's temperature is 105&deg;C". This string is 28 characters or 8 x 28 = 224 bits long. Do we really need these many bits? If we shorten the string to just "105", we still require 3 x 8 = 24 bits to convey the information. Since we have 8 different temperature levels, the ideal way to go about it would be to use 3 bits where each 3-bit pattern represents a specific temperature level. Using 24 bits to represent information worth only 3 bits is a sin. Also, note that if a receiver successfully receives the information of the temperature being 105&deg;C, it would have reduced its uncertainty(or confusion) by a factor of 8(as mentioned earlier).

Well, this was easy. But how do we find the magic number of 3 mathematically? That is easy too, right? Just take a $$\log_2$$ of the number of available unique pieces of information (eg. $$\log_2 8 = 3$$) and we're sorted. Well, no!

That would only work if the probabilities for all the temperatures levels is same. It is to be noted that in the case of uneven probabilities(as in our example), we can further save up on the number of bits being transferred and that is what entropy addresses. The intuition here is that we don't need as many bits for something that occurs much more frequently compared to something that occurs rarely. If 105&deg;C occurs 60% of the time and 162&deg;C occurs only 2% of the time, then it doesn't makes sense to use the same bit code(encoding) for representing both these pieces of information.

I hope you understand the problem now.

If the astronaut sends us a continuous stream of information about the temperatures such as,

```
100,105,110,162,100......and so on.
```
where each number/temperature is obviously represented by some kind of a bit code.

The frequency of occurrence of each temperature in this stream is dependent on the given probability distribution. The NASA official gets puzzled and ponders over a question - "How many bits or guesses do I need on an average to correctly predict a temperature level in a stream?". Rephrasing his concerns in layman's terms, he wants to know that on an average, how many questions does he need to ask to correctly predict or tell the temperature value currently received by him? This is known as entropy. The average number of times the NASA official will encounter uncertainty before correctly predicting something.

Mathematically, it is represented as

$$
H(p) =
\begin{cases}
- \sum_{i} p_i\log_2 p_i  & \text{for a discrete set of probabilities} \\\\
- \int_{-\infty}^{\infty} p_i\log_2 p_i & \text{for a continuous set of probabilities}
\end{cases}
$$

Throughout this post, we deal with a discrete set of probabilities.

One may argue that in order to find the number of "guesses" to rightly predict something from the stream, we can just multiply the probabilities. For example,

```py
import numpy as np
np.prod([0.30, 0.40, 0.05, 0.06, 0.07, 0.02, 0.05, 0.05])
# result = 0.0000000012600000000000002
```
However, calculating the product serves no purpose as the result is extremely tiny and doesn't provide a lot of insights. Multiplication is also an expensive operation. A simple way to tackle this is to convert this into a sum by using $$\log$$. Utilizing the property,

$$
\log(xy) = \log(x) + \log(y)
$$

we get,

$$
\log_2 (0.30 * 0.40 * 0.05 * 0.06 * 0.07 * 0.02 * 0.05 * 0.05) = \log_2 (0.0000000012600000000000002)\\
\log_2 0.30 + \log_2 0.40 + \ldots + \log_2 0.05 = \log_2 (0.0000000012600000000000002)\\
\log_2 0.30 + \log_2 0.40 + \ldots + \log_2 0.05 = -29.5639291203\\
- (\log_2 0.30 + \log_2 0.40 + \ldots + \log_2 0.05 ) = 29.5639291203\\
\text{or}\\
- \sum_{i} \log_2 p_i = 29.5639291203\\
$$

Entropy means the average number of bits/guesses. Had the probabilities been similar for all the temperatures, we could have just done

$$
- \frac{1}{8}(\sum_{i} \log_2 (\frac{1}{8})) = -\log_2 (\frac{1}{8}) = 3.0 \\
$$

But since we already have a given probability distribution, we multiply each of the $$\log_2 p_i$$ terms with their respective probabilities. Why? Because when we calculate an average of a set of numbers, we assume that each of the numbers in the set are equally likely to be closer to the average. But when the probabilities for each number in the set are mentioned, things are done differently.

$$
- (0.30(\log_2 0.30) + 0.40(\log_2 0.40) + 0.05(\log_2 0.05) + 0.06(\log_2 0.06) + 0.07(\log_2 0.07) + \\0.02(\log_2 0.02) + 0.05(\log_2 0.05) + 0.05(\log_2 0.05)) \\
\text{or}\\
- \sum_{i} p_i\log_2 p_i = 2.323115964316818 = \text{~}2.3\\
$$

Obtaining an entropy roughly equal to 2.3 tells us that we need to ask questions on an average of 2.3 times or need 2.3 bits per information in order to clearly determine what the message stands for and efficiently transmit it. This also means that we only have 2.3 bits of useful information out of the 3 bits if we would have used the coding scheme described earlier (we'd be wasting 0.7 bits if we used a 3-bit pattern on this sample).

For folks who can't get their head around what 2.3 bits would look like, don't think in terms of a binary number but in terms of the portion or part of an information. Let me propose 2 ways of looking at it which will hopefully make this more lucid.

* Considering the planetary temperature example mentioned above, we know that we have 8 different kinds of information and 2.3 is our entropy. Now if we use 2.3 bits per information, our message would be 2.3 x 8 = 18.4 bits long which is not possible. But if we send the message 10 times, we can do it in 184 bits in contrast to the 3 x 8 x 10 = 240 bits. Scale of transmission is tackled here.

* Consider a state election where 4 candidates (3 women and 1 man) are contesting. Suppose we find some dirt on the only male candidate and he get's eliminated from the race. We still have 3 candidates we can vote for. The dirt certainly gave me more than 0 bits of information because I know more than I knew before. But at the same time, it did not give me 1-complete bit of information because if that would have happened, I would have reduced my uncertainty by a factor of 2 and be left with only 2 candidates in total (refer to the third paragraph in this post).

## Cross-Entropy

Since we now know what entropy is, cross entropy should be fairly easy to understand. A Wikipedia lift tells us that,

> Cross entropy between two probability distributions  p and q over the same underlying set of events measures the average number of bits needed to identify an event drawn from the set, if a coding scheme is used that is optimized for an "unnatural" probability distribution q, rather than the "true" distribution  p.

In simple terms,

Given that $$H(p) = - \sum_{i} p_i\log_2 p_i$$ represents entropy, we can say that $$H(p, q) = - \sum_{i} p_i\log_2 q_i$$ represents cross entropy. You ask how? Well, consider you have 2 probability distributions _p_ and _q_ where _p_ represents a true distribution and _q_ represents a predicted distribution. Both _p_ and _q_ are distributions over the same set of events. Imagine the last step of a neural network wherein you obtain a probability distribution(or _q_) and then compare it with a distribution you already know beforehand(or _p_) where both the distributions describe the same set of classes such animals, plants, cars, or whatever your model is trained on.

In entropy, we find the optimal/average number of bits we require to transmit an information using a probability distribution applied to its set of events. If we use the probability distribution of _q_ to devise a code for the information from _p_, then the optimal/average number of bits we'll need is known as the cross entropy. This is quite insightful. Cross-entropy tells us how different the 2 distributions are. The closer they are, the closer will their entropies be.

Hence, it is represented as follows.

$$
H(p, q) = - \sum_{i} p_i\log_2 q_i
$$

Let's write some code to better understand this.

```py
import numpy as np
obtained_probability = np.random.dirichlet(np.ones(8))
true_probability = np.random.dirichlet(np.ones(8))

assert sum(obtained_probability), sum(true_probability) == 1.0

print("Obtained Probability: ", obtained_probability)
# [0.22374847 0.01351315 0.26085442 0.04087474 0.05121301 0.04455142 0.14301258 0.22223221]
print("True Probability: ", true_probability)
# [0.29786589 0.01631927 0.00065958 0.27396781 0.11610311 0.10891934 0.01762108 0.16854391]

plt.bar(np.arange(8), obtained_probability, width=0.35, color="r")
plt.bar(np.arange(8) + 0.35, true_probability, width=0.35, color="b")
plt.show()
```
<!-- <p align="center" markdown="1">
![](http://oi65.tinypic.com/29gkg2e.jpg){:height="350px" width="450px"}
</p>

All we've done till now is generate 2 random probability distributions such that they add up to 1 and plotted them. -->

Below, we've written certain routines to calculate entropies and cross-entropies for these 2 distributions.

```py
def entropy(x):
    return -sum([x_i * log(x_i, 2) for x_i in x])

def cross_entropy(p, q):
    return -sum([p_i * log(q_i, 2) for p_i, q_i in zip(p, q)])

entropy(true_probability) # 2.380762398444909
entropy(obtained_probability) # 2.5644798655740377
cross_entropy(true_probability, obtained_probability) # 3.4115389365100524
cross_entropy(true_probability, true_probability) # 2.380762398444909
```

Notice, the cross-entropy will always be higher than the individual entropies since we've performed the encoding using the wrong distribution. Also, when both the distributions are same, the cross-entropy and entropy are equal to each other.

## Kullback-Leibler divergence

Yayy! So, we've accomplished most of the crucial stuff required to understand KL divergence.

The KL divergence between two probability distributions is just the number of extra bits we need if we encode the information represented by one distribution using the probability distribution of the other. It is the difference between cross entropy and entropy. Yes, that's it. You can take a guess regarding how it is mathematically represented and you'll probably be right.

$$
KLD = cross entropy - entropy = H(p, q) - H(p) \\
= (- \sum_{i} p_i\log_2 q_i) - (- \sum_{i} p_i\log_2 p_i)\\
= \sum_{i} p_i\log_2 p_i - \sum_{i} p_i\log_2 q_i\\
\text{using } \log(x \div y) = \log(x) - \log(y) \text{ we get,}\\
KLD = \sum_{i} p_i log_2 (\frac{p_i}{q_i})\\

$$

Lesser the cross-entropy, lesser will be the KL divergence. The term _divergence_ in itself is quite self-explanatory. The factor by which one distribution diverges from the other is basically what KL divergence is.

Writing the code for it should be easy now.

```py
def kld(p, q):
    return cross_entropy(p, q) - entropy(p)

kld(true_probability, obtained_probability) # 1.0307765380651435
```

In our case, the divergence comes out to be a factor of 1.0307765380651435.

## Frequently Asked Questions

**Q1: How is cross-entropy different from KL divergence?**

Cross-entropy measures the average number of bits required to describe data using one probability distribution when optimized for another distribution. It quantifies the dissimilarity between two distributions. In contrast, KL divergence measures the additional bits needed to encode information using one distribution compared to another, indicating the difference between them.


**Q2: When are these concepts used in practical applications?**

Entropy, cross-entropy, and KL divergence find applications in information theory, machine learning, data compression, and more. They play a crucial role in optimizing data representation, model training, and assessing the similarity of probability distributions in various fields.


**Q3: Why is entropy important in data transmission?**

Entropy is crucial in data transmission because it helps determine the optimal encoding length for information. By using shorter codes for more frequent events, data can be transmitted more efficiently, reducing the required bandwidth and minimizing transmission errors.


**Q4: What is the significance of KL divergence in machine learning?**

In machine learning, KL divergence is used to assess the dissimilarity between predicted probability distributions and actual data. It helps fine-tune models by quantifying the information loss when approximating one distribution with another. Reducing KL divergence is a common goal in model training.


**Q5: Can you provide a real-world example of KL divergence?**

Suppose you have a machine learning model that predicts the probability distribution of diseases in a population. The true distribution is based on medical records. KL divergence can help quantify how well the model's predictions align with the actual distribution, highlighting areas where the model needs improvement.


That wraps up the explanation of KL divergence. Read the next chapter about Kolmogorov complexity <a href="https://smunshi.net/posts/kolmogorov-complexity">here</a>.
