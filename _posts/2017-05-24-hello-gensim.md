---
author: andromeda
---

So I got obsessed with word2vec a year or two when I saw [this StitchFix engineering blog post](http://multithreaded.stitchfix.com/blog/2015/03/11/word-is-worth-a-thousand-vectors/).

What's word2vec? Big picture: it's an algorithm that chomps through mountains of text and learns what words tend to be near each other. It turns out there's a _lot_ of information you can infer from that; for instance, if you know that "banana" tends to appear near the same words that "apple" does, you can guess that they might have something in common. And extrapolating from there, you can guess that words that appear frequently near "banana" might have something in common with words that appear frequently near "apple"...and so on and so forth.

If (like me) you'd like to see that concept spelled out in way more depth, I commend to you the original paper, ["Efficient Estimation of Word Representations in Vector Space"](https://arxiv.org/abs/1301.3781).

If linear algebra isn't your sort of thing (_de gustibus_, of course), go ahead and skip the paper, but here's one important takeaway: under the hood, the computer learns to represent all the words it sees as _vectors_. This means two really important things:
1. Each word can be located in space. Just like you located points on graph paper back in secondary school math, these words are points that can be plotted (albeit not on graph paper, because the space doesn't have two dimensions; it has hundreds). This means words can be 'near' each other in some meaningful, spatial sense. And it means that, at least in principle, we can visualize that nearness (in practice we need to be clever about how we do it, since visualizing gazillion-dimensional space is kind of tricky).
2. We can add and subtract words - and then see which other words we end up 'near'.

If you'd like to see some examples of how these visualizations work, the StitchFix blog post I linked above has some great ones.

I expect to have a lot to say in subsequent posts about both the lower levels of this (how do neural nets work) and the higher ones (what can we do with this in libraries) but, for the moment, a teaser -

I took [gensim](https://radimrehurek.com/gensim/), the Python library that implements word2vec, and I trained it on our aero-astro theses. And then I asked it...

```
aero.wv.most_similar(positive=['apollo', 'russian'], negative=['american'])
```

...or, in English, "hey software friend, what do you get if you take the Apollo program, remove 'American', and add 'Russian'?" And it said: "[Vostok](https://en.wikipedia.org/wiki/Vostok_programme)".
