---
title: Extension Conferences/Meetings Attended
summary: Regional Conference
date: 2026-05-27
math: true

tags:
  - Water and Waste Management
  - Extension
  - Financial performance
  - Exploring new market opportunities
  - Managing risk (Drought)
  - Enhancing cybersecurity and navigating evolving regulatory and technological landscapes. 
image:
  caption: 'Embed rich media such as videos and LaTeX math'
---

**International Resource Recovery Conference & EXPO on WATER, WASTE MANAGEMENT, ONE HEALTH, and ENERGY - 2026 UF Gainesville, FL**

The organization features a network of over 500 subject matter experts spanning the USA, India, Austria, Canada, Europe, Singapore, and Thailand. On average, its professional collective brings over 20 years of individual industry experience to the mission.

**Southern Economic Association (SEA) Annual Meeting - 2025, Tampa, FL**

One of the oldest regional economics associations in the United States, bringing together scholars to advance applied economic analysis, present theoretical research, and address policy-relevant economic issues across diverse fields. Macro and Micro Economics.

**Florida Agriculture Financial Management Conference - 2025 Orlando, FL**

The event offers a dynamic platform for networking with peers, sharing best practices, and discussing the impact of current financial markets on agribusiness. Whether you manage a large farm, a small operation, or serve agricultural or horticulture clients in a financial or legal capacity, this conference is for you. We welcome farm owners and managers, CFOs, CPAs, controllers, accountants, bookkeepers, lenders, and attorneys who work in the ag sector. Both CPE and CLE credits will be available.

**AWRA Annual Meeting - 2026 Key West, FL**

AWRA Florida Annual Meeting - Key West. This meeting is a great opportunity for students to network with consulting firms and regulatory staff so we invite you to consider attending. Registration is only $25 dollars for students.

**Annual Florida Agricultural Policy Outlook Conference - 2025 Citrus, FL; 2026 Apoka, FL**

Examines critical policy issues facing Florida agribusiness leaders and explores valuable economic insights helpful for making informed business and policy decisions. Optional facility tour included.

## Video

WWM; SEA; FLAFMC

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
## Math

Markdown extension for $\LaTeX$ math. Enable math by setting the `math: true` option in your page's front matter, or enable math for your entire site by toggling math in your `config/_default/params.yaml` file:

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
