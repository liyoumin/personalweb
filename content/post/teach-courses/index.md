---
title: 👩🏼‍🏫 Florida Agriculture Financial Management Conference (FLAFMC)
summary: Embed videos, podcasts, code, LaTeX math, and even test students!
date: 2025-09-30
math: true

tags:
  - Analyzing financial performance
  - Exploring new market opportunities
  - Managing risk
  - Enhancing cybersecurity and navigating evolving regulatory and technological landscapes. 
image:
  caption: 'Embed rich media such as videos and LaTeX math'
---

**Florida Agriculture Financial Management Conference**

The event offers a dynamic platform for networking with peers, sharing best practices, and discussing the impact of current financial markets on agribusiness. Whether you manage a large farm, a small operation, or serve agricultural or horticulture clients in a financial or legal capacity, this conference is for you. We welcome farm owners and managers, CFOs, CPAs, controllers, accountants, bookkeepers, lenders, and attorneys who work in the ag sector. Both CPE and CLE credits will be available.


## Video

FLAFMC isn’t just about education…it’s about building relationships, discovering solutions, and shaping the future of Florida’s agricultural communities. 

**Youtube**:

    {{</* youtube D2vj0WcvH5c */>}}

{{< youtube D2vj0WcvH5c >}}

**Bilibili**:

    {{</* bilibili BV1WV4y1r7DF */>}}

{{< bilibili BV1WV4y1r7DF >}}

**Video file**

Videos may be added to a page by either placing them in your `assets/media/` media library or in [page's folder](https://gohugo.io/content-management/page-bundles/),:

    {{</* video src="my_video.mp4" controls="yes" */>}}

## Podcast



    {{</* audio src="ambient-piano.mp3" */>}}

Try it out:

{{< audio src="ambient-piano.mp3" >}}


## Math

Hugo Blox Builder supports a Markdown extension for $\LaTeX$ math. Enable math by setting the `math: true` option in your page's front matter, or enable math for your entire site by toggling math in your `config/_default/params.yaml` file:

```yaml
features:
  math:
    enable: true
```

To render _inline_ or _block_ math, wrap your LaTeX math with `$...$` or `$$...$$`, respectively.

Example **math block**:

```latex
$$
\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}
$$
```

renders as

$$\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}$$

Example **inline math** `$\nabla F(\mathbf{x}_{n})$` renders as $\nabla F(\mathbf{x}_{n})$.

Example **multi-line math** using the math linebreak (`\\`):

```latex
$$f(k;p_{0}^{*}) = \begin{cases}p_{0}^{*} & \text{if }k=1, \\
1-p_{0}^{*} & \text{if }k=0.\end{cases}$$
```

renders as

$$
f(k;p_{0}^{*}) = \begin{cases}p_{0}^{*} & \text{if }k=1, \\
1-p_{0}^{*} & \text{if }k=0.\end{cases}
$$

## Code

Hugo Blox Builder utilises Hugo's Markdown extension for highlighting code syntax. The code theme can be selected in the `config/_default/params.yaml` file.


    ```python
    import pandas as pd
    data = pd.read_csv("data.csv")
    data.head()
    ```

renders as

```python
import pandas as pd
data = pd.read_csv("data.csv")
data.head()
```

## Inline Images

```go
{{</* icon name="python" */>}} Python
```

renders as

{{< icon name="python" >}} Python

## Did you find this page helpful? Consider sharing it 🙌
