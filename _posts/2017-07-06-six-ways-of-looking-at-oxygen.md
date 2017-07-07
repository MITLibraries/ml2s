---
author: andromeda
---
<script src="https://d3js.org/d3.v4.min.js"></script>

[In a previous post]({{ site.baseurl }}{% post_url 2017-05-24-hello-gensim %}
) I talked about using the [word2vec](https://arxiv.org/abs/1301.3781) algorithm to find clusters of meaning in MIT thesis data ([see the code](https://github.com/MITLibraries/mundaneum)).

I started by ingesting the [aeronautical & aerospace engineering](http://aeroastro.mit.edu/) theses, because that department is first alphabetically. This is both awesome and a strategic miscalculation, since I don't know anything about AeroAstro, and thus found myself with a corpus and no idea what questions to ask of it. Luckily, my library colleague Christine Moulen is an MIT AeroAstro grad, so she gave me lots of cool ideas for things to explore, and thus we found "oxygen".

Here's what oxygen looks like if everything you know about the world is AeroAstro:

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/oxygen_aeroastro.h2.t65.json{% endcapture %}

<svg width="700" height="500" id="svg_aeroastro"></svg>
<script>
  var datafile = '{{ datafile }}';
  {% include d3_word_net.html svg_id='svg_aeroastro' datafile=datafile %}
</script>

Here, links indicate similarity of meaning. Words are linked if their similarity is above a certain threshold (which varies per graph &mdash; always above 0.45, below which there doesn't seem subjectively to be any meaning similarity, but sometimes higher to make the graph readable by limiting the number of nodes). More intensely colored links show greater similarity. We proceed a couple of hops out from "oxygen" to illustrate its neighborhood. We can see that "oxygen" is similar in meaning to 'hydrogen' and 'water'...and 'propellant'...and ['hypergolic'](https://www.merriam-webster.com/dictionary/hypergolic)...

If all we know about the world is AeroAstro, oxygen is rocket fuel.

Which was immediately striking to me because, when I was busy not being an AeroAstro major in college, I came awfully close to being a chemistry major. And if everything I knew about the world were chemistry, I don't think that "oxygen" would mean "rocket fuel".

So I asked the corpus, ingesting other departments' theses and probing the area most similar to "oxygen".

## Chemistry

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/oxygen_chemistry.h2.t48.json{% endcapture %}

<svg width="700" height="500" id="svg_chemistry"></svg>
<script>
  var datafile = '{{ datafile }}';
  {% include d3_word_net.html svg_id='svg_chemistry' datafile=datafile %}
</script>

If your whole world is chemistry, "oxygen" is like "nitrogen" and "chlorine"..."oxygen" is an element. And a highly reactive one, at that!

(Side note: it's also quite similar to "02", which is a clear OCR error for "O2"...but at least this means the neural net picked up that molecular and atomic oxygen are quite similar.)

## Biology

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/oxygen_biology.h1.t4355.json{% endcapture %}

<svg width="700" height="500" id="svg_biology"></svg>
<script>
  var datafile = '{{ datafile }}';
  {% include d3_word_net.html svg_id='svg_biology' datafile=datafile %}
</script>

If your whole world is biology, "oxygen" is like "nutrient" and "energy"...it's fuel again, but not for rockets &mdash; for organisms.

## Physics

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/oxygen_physics.h2.t50.json{% endcapture %}

<svg width="700" height="500" id="svg_physics"></svg>
<script>
  var datafile = '{{ datafile }}';
  {% include d3_word_net.html svg_id='svg_physics' datafile=datafile %}
</script>

I'll be honest: I have no idea what oxygen looks like if your world is physics. Apparently it's similar to 174-ytterbium. I thought, maybe it's in the decay chain? But apparently 174 is one of the few isotopes of ytterbium that is _not_ radioactive. Oh well. Sometimes machine learning isn't illuminating.

## Nuclear engineering

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/oxygen_nuke.h2.t47.json{% endcapture %}

<svg width="700" height="500" id="svg_nuke"></svg>
<script>
  var datafile = '{{ datafile }}';
  {% include d3_word_net.html svg_id='svg_nuke' datafile=datafile %}
</script>

I don't really know how nuclear engineering works, but I believe there's a coherent cluster of meaning there. Any nuclear engineers able to apply an interpretable label to this one?

## Earth, Atmospheric, and Planetary Science

{% capture datafile %}{{ site.url }}{{ site.baseurl }}/data/oxygen_eaps.h2.t495.json{% endcapture %}

<svg width="700" height="500" id="svg_eaps"></svg>
<script>
  var datafile = '{{ datafile }}';
  {% include d3_word_net.html svg_id='svg_eaps' datafile=datafile %}
</script>

As with nuclear engineering, I don't have the domain knowledge to interpret this one, but I buy that there's something coherent going on here; those look like a lot of elements and gases. (Maybe the gaseous cluster here is even from more atmospheric-sciency theses, and the elemental one more from earth sciences?)

## Acknowledgements

Thanks go to:
* Christine Moulen, as mentioned above;
* My other colleague Helen Bailey, for writing the first version of the d3 scripts used here (I don't know a thing about datavis and this is my first d3 so it was great having her scripts to crib off of);
* Wallace Stevens, for [Thirteen Ways of Looking at a Blackbird](https://www.poetryfoundation.org/poems/45236/thirteen-ways-of-looking-at-a-blackbird), and Henry Louis Gates, for bringing that poem and other things to my attention via [Thirteen Ways of Looking at a Black Man](https://www.amazon.com/Thirteen-Ways-Looking-Black-Man/dp/0679776664).
