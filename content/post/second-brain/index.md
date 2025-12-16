---
title: 🧠 "AAEA in Denver, Co"
summary: Functions Roshambo Battle Royale

date: 2025-10-29
authors:
  - admin
tags:
  - R
  - Lab
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com)'
---

Youmin's Bot: https://github.com/liyoumin/Geospatial-AgEcon/blob/main/youmin_bot.R

## Mindmaps

Functions take arguments, perform a sequence of operations, sometimes calling other functions (or even itself, if it’s recursive), and often return something useful to us. This is the core of a functional programming language like R. R is a functional language since everything in the R environment is either a data object or a function (hence the term, functional… honest, I don’t make this stuff up).

Programming functions through the process of creating the most sophisticated Rock-Paper-Scissors player ever. The rock-paper-scissors (RPS) game is an extremely simple concept. Two players, choose one of three options, each of which will “win” against only one of the possible three outcomes, losing to another and drawing with itself. So if two players are making choices between “rock,” “paper,” and “scissors” completely at random, over time they have the neutral expectation of winning, losing and drawing exactly 1/3 of the time.

An online game released by the New York Times showed to the world, however, that an RPS artificial intelligence (AI), a bot, can consistently beat human opponents. Why? The reason lies in that humans are predictable, we almost never can play completely randomly, and over time a bot with enough sophistication can detect patterns in our play and exploit those patterns. It is impossible to gain an advantage against a player playing completely randomly.

So, if playing randomly is the optimal strategy, what is the point of an RPS programming competition?  The answer is that a completely random player will only expect to win about 1/3 of the time, with a win:loss ratio of about 1.0 (since you’re drawing 1/3 of time as well). And while it’s trivial to program a bot that plays randomly, against weaker opponents with predictable play you should be able to do much better than a win:loss ratio of 1.0 by exploiting that weakness. But in exploiting that weakness you yourself cannot play randomly, and therefore leave your play open to exploitation itself.

Therefore, identifying the pattern and the predictability of the play of your opponent based on previous turns and outcomes is a solution well-suited to computers. And today you will create your own RPS AI. You will try to maximize your outcomes against a suite of competitors who play with a variety of strategies, and hopefully come out with a bot capable of challenging your classmates for the title of supreme RPS champion!

Mindmaps can be created by simply writing the items as a Markdown list within the `markmap` code block, indenting each item to create as many sub-levels as you need:

<div class="highlight">
<pre class="chroma">
<code>
```markmap {height="200px"}
- Hugo Modules
  - Hugo Blox
  - blox-plugins-netlify
  - blox-plugins-netlify-cms
  - blox-plugins-reveal
```
</code>
</pre>
</div>

renders as

```markmap {height="200px"}
- Hugo Modules
  - Hugo Blox
  - blox-plugins-netlify
  - blox-plugins-netlify-cms
  - blox-plugins-reveal
```

Anh here's a more advanced mindmap with formatting, code blocks, and math:

<div class="highlight">
<pre class="chroma">
<code>
```markmap
- Mindmaps
  - Links
    - [Hugo Blox Docs](https://docs.hugoblox.com/)
    - [Discord Community](https://discord.gg/z8wNYzb)
    - [GitHub](https://github.com/HugoBlox/hugo-blox-builder)
  - Features
    - Markdown formatting
    - **inline** ~~text~~ *styles*
    - multiline
      text
    - `inline code`
    -
      ```js
      console.log('hello');
      console.log('code block');
      ```
    - Math: $x = {-b \pm \sqrt{b^2-4ac} \over 2a}$
```
</code>
</pre>
</div>

renders as

```markmap
- Mindmaps
  - Links
    - [Hugo Blox Docs](https://docs.hugoblox.com/)
    - [Discord Community](https://discord.gg/z8wNYzb)
    - [GitHub](https://github.com/HugoBlox/hugo-blox-builder)
  - Features
    - Markdown formatting
    - **inline** ~~text~~ *styles*
    - multiline
      text
    - `inline code`
    -
      ```js
      console.log('hello');
      console.log('code block');
      ```
    - Math: $x = {-b \pm \sqrt{b^2-4ac} \over 2a}$
```

## Highlighting

<mark>Highlight</mark> important text with `mark`:

```html
<mark>Highlighted text</mark>
```

## Callouts

Use [callouts](https://docs.hugoblox.com/reference/markdown/#callouts) (aka _asides_, _hints_, or _alerts_) to draw attention to notes, tips, and warnings.

By wrapping a paragraph in `{{%/* callout note */%}} ... {{%/* /callout */%}}`, it will render as an aside.

```markdown
{{%/* callout note */%}}
A Markdown aside is useful for displaying notices, hints, or definitions to your readers.
{{%/* /callout */%}}
```

renders as

{{% callout note %}}
A Markdown aside is useful for displaying notices, hints, or definitions to your readers.
{{% /callout %}}

Or use the `warning` callout type so your readers don't miss critical details:

{{% callout warning %}}
A Markdown aside is useful for displaying notices, hints, or definitions to your readers.
{{% /callout %}}

## Did you find this page helpful? Consider sharing it 🙌
