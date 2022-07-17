---
title: 阻止谷歌浏览器翻译数学公式
date: 2022-07-17 22:29:58
tags: MathJax, Katex, Javascript
---

在一切开始之前，让我们看看一些令人不适的翻译。

![难看的翻译](https://s1.ax1x.com/2022/07/17/jIa1HS.png)

![难看的翻译](https://s1.ax1x.com/2022/07/17/jIaBHU.png)

这些翻译对于经常打 Codeforces 和 Atcoder 网站上的比赛的学生们及其熟悉。这些翻译就是谷歌浏览器中自带的翻译工具翻译出来的结果。其中，一个重要的问题就是公式的混乱。

大部分人的解决方法都是不再使用翻译器，或者是把题目内容复制出来，手动调整格式后再进行翻译。对于英语不好的人来说，这是影响他们 rating 的最大因素之一。

为了造福大家，我要研究一下谷歌翻译。

## 找到特征

根据一些资料和对于谷歌翻译使用的文档，我发现了在翻译过程中，所有加上 `translate = "no"` 属性的元素都不会被翻译。那么，我们在所有数学公式元素上添加就可以了！

接下来，我们要找找数学公式在哪里。

### Codeforces

根据开发者工具中的「Elements」，我找到了一段 Codeforces 题面中的 HTML。

```html
<p>Is it possible to make
  <span class="MathJax_Preview" style="color: inherit;"></span>
  <span class="MathJax" id="MathJax-Element-7-Frame" tabindex="0" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><msub><mi>a</mi><mi>i</mi></msub><mo>=</mo><mn>0</mn></math>" role="presentation" style="position: relative;" translate="no">
    <nobr aria-hidden="true">
      <span class="math" id="MathJax-Span-35" style="width: 3.336em; display: inline-block;">
        <span style="display: inline-block; position: relative; width: 2.741em; height: 0px; font-size: 120%;">
          <span style="position: absolute; clip: rect(1.372em, 1002.68em, 2.562em, -999.997em); top: -2.199em; left: 0em;">
            <span class="mrow" id="MathJax-Span-36">
              <span class="msubsup" id="MathJax-Span-37">
                <span style="display: inline-block; position: relative; width: 0.836em; height: 0px;">
                  <span style="position: absolute; clip: rect(3.396em, 1000.54em, 4.17em, -999.997em); top: -3.985em; left: 0em;">
                    <span class="mi" id="MathJax-Span-38" style="font-family: MathJax_Math-italic;">a</span>
                    <span style="display: inline-block; width: 0px; height: 3.991em;"></span>
                  </span>
                  <span style="position: absolute; top: -3.807em; left: 0.539em;">
                    <span class="mi" id="MathJax-Span-39" style="font-size: 70.7%; font-family: MathJax_Math-italic;">i</span>
                    <span style="display: inline-block; width: 0px; height: 3.991em;"></span>
                  </span>
                </span>
              </span>
              <span class="mo" id="MathJax-Span-40" style="font-family: MathJax_Main; padding-left: 0.301em;">=</span>
              <span class="mn" id="MathJax-Span-41" style="font-family: MathJax_Main; padding-left: 0.301em;">0</span></span>
            <span style="display: inline-block; width: 0px; height: 2.205em;"></span>
          </span>
        </span>
        <span style="display: inline-block; overflow: hidden; vertical-align: -0.282em; border-left: 0px solid; width: 0px; height: 1.146em;"></span>
      </span>
    </nobr>
    <span class="MJX_Assistive_MathML" role="presentation">
      <math xmlns="http://www.w3.org/1998/Math/MathML">
        <msub>
          <mi>a</mi>
          <mi>i</mi></msub>
        <mo>=</mo>
        <mn>0</mn></math>
    </span>
  </span>
  <script type="math/tex" id="MathJax-Element-7">a_i = 0</script>for all
  <span class="MathJax_Preview" style="color: inherit;"></span>
  <span class="MathJax" id="MathJax-Element-8-Frame" tabindex="0" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mn>2</mn><mo>&amp;#x2264;</mo><mi>i</mi><mo>&amp;#x2264;</mo><mi>n</mi></math>" role="presentation" style="position: relative;" translate="no">
    <nobr aria-hidden="true">
      <span class="math" id="MathJax-Span-42" style="width: 5.122em; display: inline-block;">
        <span style="display: inline-block; position: relative; width: 4.229em; height: 0px; font-size: 120%;">
          <span style="position: absolute; clip: rect(1.372em, 1004.23em, 2.503em, -999.997em); top: -2.199em; left: 0em;">
            <span class="mrow" id="MathJax-Span-43">
              <span class="mn" id="MathJax-Span-44" style="font-family: MathJax_Main;">2</span>
              <span class="mo" id="MathJax-Span-45" style="font-family: MathJax_Main; padding-left: 0.301em;">≤</span>
              <span class="mi" id="MathJax-Span-46" style="font-family: MathJax_Math-italic; padding-left: 0.301em;">i</span>
              <span class="mo" id="MathJax-Span-47" style="font-family: MathJax_Main; padding-left: 0.301em;">≤</span>
              <span class="mi" id="MathJax-Span-48" style="font-family: MathJax_Math-italic; padding-left: 0.301em;">n</span></span>
            <span style="display: inline-block; width: 0px; height: 2.205em;"></span>
          </span>
        </span>
        <span style="display: inline-block; overflow: hidden; vertical-align: -0.211em; border-left: 0px solid; width: 0px; height: 1.146em;"></span>
      </span>
    </nobr>
    <span class="MJX_Assistive_MathML" role="presentation">
      <math xmlns="http://www.w3.org/1998/Math/MathML">
        <mn>2</mn>
        <mo>≤</mo>
        <mi>i</mi>
        <mo>≤</mo>
        <mi>n</mi></math>
    </span>
  </span>
  <script type="math/tex" id="MathJax-Element-8">2\le i\le n</script>?</p>
```

简单的分析就可以得到，所有公式的内容都在 `class` 为 `MathJax` 的元素中出现。

### Atcoder

同样地，我也看到了 Atcoder 的代码。

```html
<p>
  <var>
    <span>
      <span class="katex">
        <span class="katex-mathml">
          <math xmlns="http://www.w3.org/1998/Math/MathML">
            <semantics>
              <mrow>
                <mn>1</mn></mrow>
              <annotation encoding="application/x-tex">1</annotation></semantics>
          </math>
        </span>
        <span class="katex-html" aria-hidden="true">
          <span class="base">
            <span class="strut" style="height: 0.64444em; vertical-align: 0em;"></span>
            <span class="mord">1</span></span>
        </span>
      </span>
    </span>
  </var>から
  <var>
    <span>
      <span class="katex">
        <span class="katex-mathml">
          <math xmlns="http://www.w3.org/1998/Math/MathML">
            <semantics>
              <mrow>
                <mi>N</mi></mrow>
              <annotation encoding="application/x-tex">N</annotation></semantics>
          </math>
        </span>
        <span class="katex-html" aria-hidden="true">
          <span class="base">
            <span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span>
            <span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span>
        </span>
      </span>
    </span>
  </var>が書かれた
  <var>
    <span>
      <span class="katex">
        <span class="katex-mathml">
          <math xmlns="http://www.w3.org/1998/Math/MathML">
            <semantics>
              <mrow>
                <mi>N</mi></mrow>
              <annotation encoding="application/x-tex">N</annotation></semantics>
          </math>
        </span>
        <span class="katex-html" aria-hidden="true">
          <span class="base">
            <span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span>
            <span class="mord mathnormal" style="margin-right: 0.10903em;">N</span></span>
        </span>
      </span>
    </span>
  </var>枚のカードが裏向きで積まれた山札があり、上から
  <var>
    <span>
      <span class="katex">
        <span class="katex-mathml">
          <math xmlns="http://www.w3.org/1998/Math/MathML">
            <semantics>
              <mrow>
                <mi>i</mi></mrow>
              <annotation encoding="application/x-tex">i</annotation></semantics>
          </math>
        </span>
        <span class="katex-html" aria-hidden="true">
          <span class="base">
            <span class="strut" style="height: 0.65952em; vertical-align: 0em;"></span>
            <span class="mord mathnormal">i</span></span>
        </span>
      </span>
    </span>
  </var>枚目のカードには整数
  <var>
    <span>
      <span class="katex">
        <span class="katex-mathml">
          <math xmlns="http://www.w3.org/1998/Math/MathML">
            <semantics>
              <mrow>
                <msub>
                  <mi>P</mi>
                  <mi>i</mi></msub>
              </mrow>
              <annotation encoding="application/x-tex">P_i</annotation></semantics>
          </math>
        </span>
        <span class="katex-html" aria-hidden="true">
          <span class="base">
            <span class="strut" style="height: 0.83333em; vertical-align: -0.15em;"></span>
            <span class="mord">
              <span class="mord mathnormal" style="margin-right: 0.13889em;">P</span>
              <span class="msupsub">
                <span class="vlist-t vlist-t2">
                  <span class="vlist-r">
                    <span class="vlist" style="height: 0.311664em;">
                      <span class="" style="top: -2.55em; margin-left: -0.13889em; margin-right: 0.05em;">
                        <span class="pstrut" style="height: 2.7em;"></span>
                        <span class="sizing reset-size6 size3 mtight">
                          <span class="mord mathnormal mtight">i</span></span>
                      </span>
                    </span>
                    <span class="vlist-s">&ZeroWidthSpace;</span></span>
                  <span class="vlist-r">
                    <span class="vlist" style="height: 0.15em;">
                      <span class=""></span>
                    </span>
                  </span>
                </span>
              </span>
            </span>
          </span>
        </span>
      </span>
    </span>
  </var>が書かれています。</p>
```

Atcoder 使用的是 KaTeX 进行渲染的。显然，所有的内容全都被包裹在了 `class='katex'` 的元素中。

可能会有读者问，更明显的应该是 `<var>` 标签？当你打开 Atcoder 题解页面的时候，就没有 `<var>` 了。我们需要的是通用的方法。

## 添加属性

两个网站都可以使用 jQuery。事情变得十分简单了。

我使用了用户脚本。它的工作方法是

- 等待页面加载完毕
- 等待 300 毫秒
- 在文档中所有 `.MathJax` 和 `.katex` 元素上添加对应属性。

```javascript
// ==UserScript==
// @name         禁止谷歌翻译公式
// @namespace    https://blog.oimaster.ml/
// @version      0.1
// @description  禁止谷歌翻译公式
// @author       OI-Master
// @match        https://codeforces.com/*
// @match        https://atcoder.jp/*
// @icon         https://www.google.com/s2/favicons?sz=64&domain=codeforces.com
// @grant        none
// ==/UserScript==

$(() => {
    setTimeout(() => {
        $('.MathJax').attr('translate','no');
        $('.katex').attr('translate','no');
    }, 300);
});
```

如果你想使用的话，只需要在 Tampermonkey 等用户脚本管理拓展中复制进去就行了。

## 验证效果

![新的翻译](https://s1.ax1x.com/2022/07/17/jIwZQI.png)

好了许多。希望对大家有所帮助。

