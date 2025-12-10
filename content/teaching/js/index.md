---
title:  Agricultural Data Analysis
summary: Easily learn how to use excel to do data process in agricultural economics!
date: 2024-08-16
type: docs
math: false
tags:
  - Excel; R
image:
  caption: 'Instructor: Prof. Misti Sharp.  AEB 3550 - Agricultural Data Analysis for second or third year ungergraduate students'
---

Machine learning methods are increasingly used in agricultural economics to model non-linear relationships, improve prediction, classify heterogeneous farmers/consumers, and analyze high-dimensional climate or remote-sensing data. The distinction between supervised and unsupervised learning depends on whether the outcome variable is known.

**Consumer Perception Impacts on Olive Oil Consumption Choice: A Case Study Using Machine Learning Approach**

On this page, you'll find my presentation and some useful technical content.

## Video

Teach course by sharing videos.

**Video file**

Project presentation:

{{< video src="ambient-paino.mp4" controls="yes" >}}

If `ambient-paino.mp4` is placed in this page's folder (page bundle) or in `assets/media/`, Hugo Blox will automatically find it.

---


## Podcast

You can add a podcast or music to a page by placing the MP3 file in the page's folder or the media library folder and then embedding the audio on your page with the _audio_ shortcode: 

My oaino music is coming out:

    {{</* audio src="ambient-piano.mp3" */>}}

Try it out:

{{< audio src="ambient-piano.mp3" >}}

## Test students

Provide a simple yet fun self-assessment by revealing the solutions to challenges with the `spoiler` shortcode:

```markdown
{{</* spoiler text="👉 Click to view the solution" */>}}
You found me!
{{</* /spoiler */>}}
```

renders as

{{< spoiler text="👉 Click to view the solution" >}} You found me 🎉 {{< /spoiler >}}

## Math
Markdown extension for $\LaTeX$ math. You can enable this feature by toggling the `math` option in your `config/_default/params.yaml` file.

To render _inline_ or _block_ math, wrap your LaTeX math with `{{</* math */>}}$...${{</* /math */>}}` or `{{</* math */>}}$$...$${{</* /math */>}}`, respectively.

{{% callout note %}}
We wrap the LaTeX math in the Hugo Blox _math_ shortcode to prevent Hugo rendering our math as Markdown.
{{% /callout %}}

Example **math block**:

```latex
{{</* math */>}}
$$
\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}
$\begin{flalign}
&\text{First stage:} \quad 
\Delta \ln Q_{D,t}
=
\alpha_0
+ \sum_{k=0}^{2} 
  \Big(
  \gamma^{D}_{k} \, \Delta \ln (\mathrm{tr}^{D}_{t-k}+1)
    + \gamma^{A}_{k} \, \Delta (\mathrm{tr}^{A}_{t-k}+1)
  \Big)
+ \mu_m + \lambda_y + u_{t},&
\label{eq:first_stage}
\\[0.6em]
&\text{2SLS:} \quad
\Delta \ln P^{mi}_{t}
=
\beta_0
+ \sum_{k=0}^{2}
\big(
\beta_{1,k} \, \widehat{\Delta \ln Q_{D,t-k}}
+ \beta_{2,k} \, \Delta \ln (\mathrm{P}^{D}_{t-k}) 
+ \beta_{3,k} \, \Delta \ln (\mathrm{P}^{A}_{t-k})
\big)
+ \mu_m + \lambda_y + \varepsilon_{t},&
\label{eq:second_stage}
\\[0.6em]
&\text{Reduced form:} \quad
\Delta P^{milk}_{t}
=
\theta_0
+ \sum_{k=0}^{2} 
  \Big(
    \theta^{A}_{k} \, \Delta (\mathrm{tr}^{A}_{t-k}+1)
  + \theta^{D}_{k} \, \Delta (\mathrm{tr}^{D}_{t-k}+1)
  \Big)
+ \mu_m + \lambda_y + \nu_{t}.&
\label{eq:reduced_form}
\end{flalign}$
{{</* /math */>}}
```

renders as

{{< math >}}
$$\gamma_{n} = \frac{ \left | \left (\mathbf x_{n} - \mathbf x_{n-1} \right )^T \left [\nabla F (\mathbf x_{n}) - \nabla F (\mathbf x_{n-1}) \right ] \right |}{\left \|\nabla F(\mathbf{x}_{n}) - \nabla F(\mathbf{x}_{n-1}) \right \|^2}$$
{{< /math >}}

Example **inline math** `{{</* math */>}}$\nabla F(\mathbf{x}_{n})${{</* /math */>}}` renders as {{< math >}}$\nabla F(\mathbf{x}_{n})${{< /math >}}.

Example **multi-line math** using the math linebreak (`\\`):

```latex
{{</* math */>}}
$$f(k;p_{0}^{*}) = \begin{cases}p_{0}^{*} & \text{if }k=1, \\
1-p_{0}^{*} & \text{if }k=0.\end{cases}$$
{{</* /math */>}}
```

renders as

{{< math >}}

$$
f(k;p_{0}^{*}) = \begin{cases}p_{0}^{*} & \text{if }k=1, \\
1-p_{0}^{*} & \text{if }k=0.\end{cases}
$$

{{< /math >}}

## Code

Builder utilises Hugo's Markdown extension for highlighting code syntax. The code theme can be selected in the `config/_default/params.yaml` file.


    ```python
    import pandas as pd
    data = pd.read_csv("data.csv")
    data.head()
    ```

renders as

```R
import pandas as pd
data = pd.read_csv("data.csv")
data.head()
```

## Inline Images

```go
{{</* icon name="R" */>}} R
```

renders as

{{< icon name="R" >}} R

