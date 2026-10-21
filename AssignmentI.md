<div id="doc" class="markdown-body container-fluid comment-enabled" data-hard-breaks="true" lang="en-US"><h1 class="part" data-startline="1" data-endline="1" id="Assignment1-RISC-V-Assembly-and-Instruction-Pipeline" data-id="Assignment1-RISC-V-Assembly-and-Instruction-Pipeline"><a class="anchor hidden-xs" href="#Assignment1-RISC-V-Assembly-and-Instruction-Pipeline" title="Assignment1-RISC-V-Assembly-and-Instruction-Pipeline" smoothhashscroll="" data-size="53" data-position="2"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="2" data-size="53">Assignment1: RISC-V Assembly and Instruction Pipeline</span></h1><div class="alert alert-danger part" data-startline="3" data-endline="4" data-position="57" data-size="66">
<p data-position="67" data-size="0"><span data-position="67" data-size="52">Check the sample pages, making the title consistent.</span></p>
</div><div class="alert alert-success part" data-startline="6" data-endline="7" data-position="124" data-size="41">
<p data-position="135" data-size="0"><span data-position="135" data-size="26">I change the title thanks.</span></p>
</div><h2 class="part" data-startline="10" data-endline="10" id="Quiz-1-Problem-B" data-id="Quiz-1-Problem-B"><a class="anchor hidden-xs" href="#Quiz-1-Problem-B" title="Quiz-1-Problem-B" smoothhashscroll="" data-size="16" data-position="170"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="170" data-size="16">Quiz 1 Problem B</span></h2><h3 class="part" data-startline="11" data-endline="11" id="Introduction" data-id="Introduction"><a class="anchor hidden-xs" href="#Introduction" title="Introduction" smoothhashscroll="" data-size="12" data-position="191"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="191" data-size="12">Introduction</span></h3><p class="part" data-startline="13" data-endline="13" data-position="205" data-size="0"><a href="https://en.wikipedia.org/wiki/Bfloat16_floating-point_format" target="_blank" rel="noopener ugc nofollow"><span data-position="206" data-size="8">bfloat16</span></a><span data-position="278" data-size="52">, also known as brain float 16, originated from the </span><a href="https://zh.wikipedia.org/zh-tw/%E8%B0%B7%E6%AD%8C%E5%A4%A7%E8%84%91" target="_blank" rel="noopener ugc nofollow"><span data-position="330" data-size="17">Google Brain team</span></a><span data-position="386" data-size="96">, which introduced this new floating-point standard to accelerate machine learning computations.</span></p><p class="part" data-startline="15" data-endline="15" data-position="483" data-size="0"><span data-position="483" data-size="385">We know that floating-point numbers consist of three parts: Sign, Exponent, and Fraction (or Mantissa). The Exponent represents the range of values that can be expressed (also called dynamic range), and the Fraction represents the precision of the values. In a limited hardware space, a trade-off between these two properties is inevitable. The length of bfloat16 is only half that of </span><a href="" target="_blank"><span data-position="869" data-size="7">float32</span></a><span data-position="967" data-size="361">. When performing calculations on the same CPU using bfloat16, the ideal throughput would be double that of float32. Therefore, if calculations are performed using bfloat16 under the same throughput, the CPU’s processing speed will ideally increase by 2x. Additionally, in terms of storage space, bfloat16 saves approximately half the space compared to float32.</span></p><p class="part" data-startline="19" data-endline="19" data-position="1331" data-size="0"><span data-position="1331" data-size="22">Although the existing </span><a href="https://zh.wikipedia.org/zh-tw/IEEE_754" target="_blank" rel="noopener ugc nofollow"><span data-position="1354" data-size="8">IEEE 754</span></a><span data-position="1404" data-size="75"> standard already defines a half-precision floating-point format, known as </span><a href="https://zh.wikipedia.org/zh-tw/%E5%8D%8A%E7%B2%BE%E5%BA%A6%E6%B5%AE%E7%82%B9%E6%95%B0" target="_blank" rel="noopener ugc nofollow"><span data-position="1480" data-size="7">float16</span></a><span data-position="1576" data-size="123">, which has similar advantages over float32 in terms of computation speed and storage space, why do we still need bfloat16?</span></p><p class="part" data-startline="21" data-endline="21" data-position="1700" data-size="0"><span data-position="1700" data-size="186">The first reason: float16’s dynamic range is not as wide as bfloat16. Google explains that “neural networks are more sensitive to the size of the exponent than the size of the mantissa.”</span></p><p class="part" data-startline="23" data-endline="23" data-position="1888" data-size="0"><span data-position="1888" data-size="405">The second reason: the area of a float16 multiplier is twice that of a bfloat16 multiplier. This is because the physical size of a hardware multiplier increases with the square of the mantissa width. Since float16 has a mantissa of 10 bits and bfloat16 has a mantissa of 7 bits, squaring these values results in 100 and 49, respectively. Dividing 100 by 49 gives an area size approximately 2 times larger.</span></p><p class="part" data-startline="26" data-endline="26" data-position="2296" data-size="0"><span data-position="2296" data-size="415">The third reason: conversion between float16 and float32 is more difficult than between bfloat16 and float32. This is because the formats of float16 and float32 differ entirely in both exponent size and mantissa size, so during conversion, both the exponent and mantissa must be adjusted. In contrast, the only difference between bfloat16 and float32 is in the mantissa size, making bfloat16 much easier to convert.</span></p><p class="part" data-startline="28" data-endline="29" data-position="2713" data-size="0"><span data-position="2713" data-size="16">bfloat16 format：</span><br>
<span class="mathjax code-block-wrapper code-block-toolbar-handled" data-raw-code="XGJlZ2lue2VxdWF0aW9ufVxvdmVyYnJhY2V7XHVuZGVyYnJhY2V7MH1fXHRleHR7MSBiaXR9fV5cdGV4dHtzaWdufVxtaWRcb3ZlcmJyYWNle1x1bmRlcmJyYWNlezAwMDAwMDAwfV9cdGV4dHs4IGJpdHN9fV5cdGV4dHtleHBvbmVudH1cbWlkXG92ZXJicmFjZXtcdW5kZXJicmFjZXswMDAwMDAwfV9cdGV4dHs3Yml0c319Xlx0ZXh0e2ZyYWN0aW9ufVxlbmR7ZXF1YXRpb259" style="margin-bottom: 0px; display: block;"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div><mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="0" style="font-size: 123.5%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-mover><mjx-over style="padding-bottom: 0.006em; padding-left: 0.289em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c73"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c67"></mjx-c><mjx-c class="mjx-c6E"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 1.8em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base style="padding-left: 0.65em;"><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 1.8em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 0.202em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c31"></mjx-c><mjx-c class="mjx-c20"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c2223"></mjx-c></mjx-mo><mjx-mover space="4"><mjx-over style="padding-bottom: 0.006em; padding-left: 0.595em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c78"></mjx-c><mjx-c class="mjx-c70"></mjx-c><mjx-c class="mjx-c6F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c74"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 4em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 4em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 1.163em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c38"></mjx-c><mjx-c class="mjx-c20"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c2223"></mjx-c></mjx-mo><mjx-mover space="4"><mjx-over style="padding-bottom: 0.087em; padding-left: 0.56em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c66"></mjx-c><mjx-c class="mjx-c72"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c63"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c6F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 3.5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 3.5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 1.002em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c37"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>0</mn><mo>⏟</mo></munder></mrow><mtext>1 bit</mtext></munder><mo>⏞</mo></mover></mrow><mtext>sign</mtext></mover><mo>∣</mo><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>00000000</mn><mo>⏟</mo></munder></mrow><mtext>8 bits</mtext></munder><mo>⏞</mo></mover></mrow><mtext>exponent</mtext></mover><mo>∣</mo><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>0000000</mn><mo>⏟</mo></munder></mrow><mtext>7bits</mtext></munder><mo>⏞</mo></mover></mrow><mtext>fraction</mtext></mover></math></mjx-assistive-mml></mjx-container></span></p><p class="part" data-startline="31" data-endline="32" data-position="2945" data-size="0"><span data-position="2945" data-size="15">float16 format：</span><br>
<span class="mathjax code-block-wrapper code-block-toolbar-handled" data-raw-code="XGJlZ2lue2VxdWF0aW9ufVxvdmVyYnJhY2V7XHVuZGVyYnJhY2V7MH1fXHRleHR7MSBiaXR9fV5cdGV4dHtzaWdufVxtaWRcb3ZlcmJyYWNle1x1bmRlcmJyYWNlezAwMDAwfV9cdGV4dHs1IGJpdHN9fV5cdGV4dHtleHBvbmVudH1cbWlkXG92ZXJicmFjZXtcdW5kZXJicmFjZXswMDAwMDAwMDAwfV9cdGV4dHsxMGJpdHN9fV5cdGV4dHtmcmFjdGlvbn1cZW5ke2VxdWF0aW9ufQ" style="margin-bottom: 0px; display: block;"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div><mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="1" style="font-size: 123.5%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-mover><mjx-over style="padding-bottom: 0.006em; padding-left: 0.289em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c73"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c67"></mjx-c><mjx-c class="mjx-c6E"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 1.8em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base style="padding-left: 0.65em;"><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 1.8em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 0.202em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c31"></mjx-c><mjx-c class="mjx-c20"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c2223"></mjx-c></mjx-mo><mjx-mover space="4"><mjx-over style="padding-bottom: 0.006em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c78"></mjx-c><mjx-c class="mjx-c70"></mjx-c><mjx-c class="mjx-c6F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c74"></mjx-c></mjx-mtext></mjx-over><mjx-base style="padding-left: 0.155em;"><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 2.5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 2.5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 0.413em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c35"></mjx-c><mjx-c class="mjx-c20"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c2223"></mjx-c></mjx-mo><mjx-mover space="4"><mjx-over style="padding-bottom: 0.087em; padding-left: 1.31em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c66"></mjx-c><mjx-c class="mjx-c72"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c63"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c6F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 1.575em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c31"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>0</mn><mo>⏟</mo></munder></mrow><mtext>1 bit</mtext></munder><mo>⏞</mo></mover></mrow><mtext>sign</mtext></mover><mo>∣</mo><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>00000</mn><mo>⏟</mo></munder></mrow><mtext>5 bits</mtext></munder><mo>⏞</mo></mover></mrow><mtext>exponent</mtext></mover><mo>∣</mo><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>0000000000</mn><mo>⏟</mo></munder></mrow><mtext>10bits</mtext></munder><mo>⏞</mo></mover></mrow><mtext>fraction</mtext></mover></math></mjx-assistive-mml></mjx-container></span></p><p class="part" data-startline="34" data-endline="35" data-position="3177" data-size="0"><span data-position="3177" data-size="16">float 32 format：</span><br>
<span class="mathjax code-block-wrapper code-block-toolbar-handled" data-raw-code="XGJlZ2lue2VxdWF0aW9ufVxvdmVyYnJhY2V7XHVuZGVyYnJhY2V7MH1fXHRleHR7MSBiaXR9fV5cdGV4dHtzaWdufVxtaWRcb3ZlcmJyYWNle1x1bmRlcmJyYWNlezAwMDAwMDAwfV9cdGV4dHs4IGJpdHN9fV5cdGV4dHtleHBvbmVudH1cbWlkXG92ZXJicmFjZXtcdW5kZXJicmFjZXswMDAwMDAwMDAwMDAwMDAwMDAwMDAwMH1fXHRleHR7MjNiaXRzfX1eXHRleHR7ZnJhY3Rpb259XGVuZHtlcXVhdGlvbn0" style="margin-bottom: 0px; display: block;"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div><mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" display="true" tabindex="0" ctxtmenu_counter="2" style="font-size: 123.5%; position: relative;"><mjx-math display="true" class="MJX-TEX" aria-hidden="true" style="margin-left: 0px; margin-right: 0px;"><mjx-mover><mjx-over style="padding-bottom: 0.006em; padding-left: 0.289em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c73"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c67"></mjx-c><mjx-c class="mjx-c6E"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 1.8em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base style="padding-left: 0.65em;"><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 1.8em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 0.202em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c31"></mjx-c><mjx-c class="mjx-c20"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c2223"></mjx-c></mjx-mo><mjx-mover space="4"><mjx-over style="padding-bottom: 0.006em; padding-left: 0.595em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c78"></mjx-c><mjx-c class="mjx-c70"></mjx-c><mjx-c class="mjx-c6F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c65"></mjx-c><mjx-c class="mjx-c6E"></mjx-c><mjx-c class="mjx-c74"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 4em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 4em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 1.163em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c38"></mjx-c><mjx-c class="mjx-c20"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c2223"></mjx-c></mjx-mo><mjx-mover space="4"><mjx-over style="padding-bottom: 0.087em; padding-left: 4.56em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c66"></mjx-c><mjx-c class="mjx-c72"></mjx-c><mjx-c class="mjx-c61"></mjx-c><mjx-c class="mjx-c63"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c6F"></mjx-c><mjx-c class="mjx-c6E"></mjx-c></mjx-mtext></mjx-over><mjx-base><mjx-texatom texclass="OP"><mjx-mover><mjx-over style="padding-bottom: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DE" style="width: 11.5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-over><mjx-base><mjx-munder><mjx-row><mjx-base><mjx-texatom texclass="OP"><mjx-munder><mjx-row><mjx-base><mjx-mn class="mjx-n"><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c><mjx-c class="mjx-c30"></mjx-c></mjx-mn></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.105em;"><mjx-mo class="mjx-n"><mjx-stretchy-h class="mjx-c23DF" style="width: 11.5em;"><mjx-beg><mjx-c></mjx-c></mjx-beg><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-mid><mjx-c></mjx-c></mjx-mid><mjx-ext><mjx-c></mjx-c></mjx-ext><mjx-end><mjx-c></mjx-c></mjx-end></mjx-stretchy-h></mjx-mo></mjx-under></mjx-row></mjx-munder></mjx-texatom></mjx-base></mjx-row><mjx-row><mjx-under style="padding-top: 0.062em; padding-left: 4.825em;"><mjx-mtext class="mjx-n" size="s"><mjx-c class="mjx-c32"></mjx-c><mjx-c class="mjx-c33"></mjx-c><mjx-c class="mjx-c62"></mjx-c><mjx-c class="mjx-c69"></mjx-c><mjx-c class="mjx-c74"></mjx-c><mjx-c class="mjx-c73"></mjx-c></mjx-mtext></mjx-under></mjx-row></mjx-munder></mjx-base></mjx-mover></mjx-texatom></mjx-base></mjx-mover></mjx-math><mjx-assistive-mml unselectable="on" display="block"><math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>0</mn><mo>⏟</mo></munder></mrow><mtext>1 bit</mtext></munder><mo>⏞</mo></mover></mrow><mtext>sign</mtext></mover><mo>∣</mo><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>00000000</mn><mo>⏟</mo></munder></mrow><mtext>8 bits</mtext></munder><mo>⏞</mo></mover></mrow><mtext>exponent</mtext></mover><mo>∣</mo><mover><mrow data-mjx-texclass="OP"><mover><munder><mrow data-mjx-texclass="OP"><munder><mn>00000000000000000000000</mn><mo>⏟</mo></munder></mrow><mtext>23bits</mtext></munder><mo>⏞</mo></mover></mrow><mtext>fraction</mtext></mover></math></mjx-assistive-mml></mjx-container></span></p><h3 class="part" data-startline="37" data-endline="37" id="Thinking" data-id="Thinking"><a class="anchor hidden-xs" href="#Thinking" title="Thinking" smoothhashscroll="" data-size="8" data-position="3430"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="3430" data-size="8">Thinking</span></h3><p class="part" data-startline="39" data-endline="39" data-position="3440" data-size="0"><span data-position="3440" data-size="70">The process for converting between float32 and bfloat16 is as follows:</span></p><div class="part code-block-wrapper code-block-toolbar-handled" data-startline="40" data-endline="50" data-infoprefix-length="7" data-position="3511"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-diagram" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-diagram" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-diagram" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-zoom-button ui-code-block-toolbar-type-diagram" data-state="closed"><i class="inline-flex ph ph-magnifying-glass-plus" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-download-image-button ui-code-block-toolbar-type-diagram" data-state="closed"><i class="inline-flex ph ph-download-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div>
        <pre class="click-event-handled mermaid plugin-rendered cursor-zoom-in" data-raw-code="JSV7aW5pdDogeyd0aGVtZSc6ICduZXV0cmFsJ30gfSUlCmdyYXBoIFRECiAgc3RhcnQoRlAzMikgLS0-IGlkMntJcyBpdCBOYU4_fQogIGlkMi0tfFllc3wtLT5pZDNbIlNoaWZ0IDE2IGJpdHMgcmlnaHQKICBzZXQgdG8gUU5hTiJdLS0-b3ZlcihiZmxvYXQxNikKICBpZDItLXxOb3wtLT5pZDR7IklzIHRoZSBMU0Igb2YgdGhlIGxlZnRtb3N0IDE2IGJpdHMgMT8ifQogIGlkNC0tfFllc3wtLT5pZDVbU2hpZnQgMTYgYml0cyByaWdodDxicj5hZGQgMV0tLT5vdmVyCiAgaWQ0LS18Tm98LS0-aWQ2W1NoaWZ0IDE2IGJpdHMgcmlnaHRdLS0-b3ZlcgogIG92ZXItLT5pZDdbU2hpZnQgMTYgYml0cyBsZWZ0XS0tPnN0YXJ0Cg"><svg id="mermaid-1792520053371-loci1rvfl" width="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" class="flowchart" style="max-width: 746.7379760742188px;" viewBox="0 0 746.7379760742188 955.6875" role="graphics-document document" aria-roledescription="flowchart-v2"><style>#mermaid-1792520053371-loci1rvfl{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#000000;}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#mermaid-1792520053371-loci1rvfl .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#mermaid-1792520053371-loci1rvfl .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#mermaid-1792520053371-loci1rvfl .error-icon{fill:#552222;}#mermaid-1792520053371-loci1rvfl .error-text{fill:#552222;stroke:#552222;}#mermaid-1792520053371-loci1rvfl .edge-thickness-normal{stroke-width:1px;}#mermaid-1792520053371-loci1rvfl .edge-thickness-thick{stroke-width:3.5px;}#mermaid-1792520053371-loci1rvfl .edge-pattern-solid{stroke-dasharray:0;}#mermaid-1792520053371-loci1rvfl .edge-thickness-invisible{stroke-width:0;fill:none;}#mermaid-1792520053371-loci1rvfl .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-1792520053371-loci1rvfl .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-1792520053371-loci1rvfl .marker{fill:#666;stroke:#666;}#mermaid-1792520053371-loci1rvfl .marker.cross{stroke:#666;}#mermaid-1792520053371-loci1rvfl svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-1792520053371-loci1rvfl p{margin:0;}#mermaid-1792520053371-loci1rvfl .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#000000;}#mermaid-1792520053371-loci1rvfl .cluster-label text{fill:#333;}#mermaid-1792520053371-loci1rvfl .cluster-label span{color:#333;}#mermaid-1792520053371-loci1rvfl .cluster-label span p{background-color:transparent;}#mermaid-1792520053371-loci1rvfl .label text,#mermaid-1792520053371-loci1rvfl span{fill:#000000;color:#000000;}#mermaid-1792520053371-loci1rvfl .node rect,#mermaid-1792520053371-loci1rvfl .node circle,#mermaid-1792520053371-loci1rvfl .node ellipse,#mermaid-1792520053371-loci1rvfl .node polygon,#mermaid-1792520053371-loci1rvfl .node path{fill:#eee;stroke:#999;stroke-width:1px;}#mermaid-1792520053371-loci1rvfl .rough-node .label text,#mermaid-1792520053371-loci1rvfl .node .label text,#mermaid-1792520053371-loci1rvfl .image-shape .label,#mermaid-1792520053371-loci1rvfl .icon-shape .label{text-anchor:middle;}#mermaid-1792520053371-loci1rvfl .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#mermaid-1792520053371-loci1rvfl .rough-node .label,#mermaid-1792520053371-loci1rvfl .node .label,#mermaid-1792520053371-loci1rvfl .image-shape .label,#mermaid-1792520053371-loci1rvfl .icon-shape .label{text-align:center;}#mermaid-1792520053371-loci1rvfl .node.clickable{cursor:pointer;}#mermaid-1792520053371-loci1rvfl .root .anchor path{fill:#666!important;stroke-width:0;stroke:#666;}#mermaid-1792520053371-loci1rvfl .arrowheadPath{fill:#333333;}#mermaid-1792520053371-loci1rvfl .edgePath .path{stroke:#666;stroke-width:2.0px;}#mermaid-1792520053371-loci1rvfl .flowchart-link{stroke:#666;fill:none;}#mermaid-1792520053371-loci1rvfl .edgeLabel{background-color:white;text-align:center;}#mermaid-1792520053371-loci1rvfl .edgeLabel p{background-color:white;}#mermaid-1792520053371-loci1rvfl .edgeLabel rect{opacity:0.5;background-color:white;fill:white;}#mermaid-1792520053371-loci1rvfl .labelBkg{background-color:rgba(255, 255, 255, 0.5);}#mermaid-1792520053371-loci1rvfl .cluster rect{fill:hsl(0, 0%, 98.9215686275%);stroke:#707070;stroke-width:1px;}#mermaid-1792520053371-loci1rvfl .cluster text{fill:#333;}#mermaid-1792520053371-loci1rvfl .cluster span{color:#333;}#mermaid-1792520053371-loci1rvfl div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(-160, 0%, 93.3333333333%);border:1px solid #707070;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-1792520053371-loci1rvfl .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#000000;}#mermaid-1792520053371-loci1rvfl rect.text{fill:none;stroke-width:0;}#mermaid-1792520053371-loci1rvfl .icon-shape,#mermaid-1792520053371-loci1rvfl .image-shape{background-color:white;text-align:center;}#mermaid-1792520053371-loci1rvfl .icon-shape p,#mermaid-1792520053371-loci1rvfl .image-shape p{background-color:white;padding:2px;}#mermaid-1792520053371-loci1rvfl .icon-shape rect,#mermaid-1792520053371-loci1rvfl .image-shape rect{opacity:0.5;background-color:white;fill:white;}#mermaid-1792520053371-loci1rvfl :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style><g><marker id="mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd" class="marker flowchart-v2" viewBox="0 0 10 10" refX="5" refY="5" markerUnits="userSpaceOnUse" markerWidth="8" markerHeight="8" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-1792520053371-loci1rvfl_flowchart-v2-pointStart" class="marker flowchart-v2" viewBox="0 0 10 10" refX="4.5" refY="5" markerUnits="userSpaceOnUse" markerWidth="8" markerHeight="8" orient="auto"><path d="M 0 5 L 10 10 L 10 0 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-1792520053371-loci1rvfl_flowchart-v2-circleEnd" class="marker flowchart-v2" viewBox="0 0 10 10" refX="11" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="mermaid-1792520053371-loci1rvfl_flowchart-v2-circleStart" class="marker flowchart-v2" viewBox="0 0 10 10" refX="-1" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="mermaid-1792520053371-loci1rvfl_flowchart-v2-crossEnd" class="marker cross flowchart-v2" viewBox="0 0 11 11" refX="12" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="mermaid-1792520053371-loci1rvfl_flowchart-v2-crossStart" class="marker cross flowchart-v2" viewBox="0 0 11 11" refX="-1" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><g class="root"><g class="clusters"></g><g class="edgePaths"><path d="M399.281,54.579L390.241,59.983C381.201,65.386,363.12,76.193,354.15,85.18C345.18,94.167,345.32,101.334,345.39,104.917L345.461,108.501" id="L_start_id2_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M301.664,192.312L268.666,205.708C235.669,219.104,169.674,245.896,136.677,288.625C103.68,331.354,103.68,390.021,103.68,448.688C103.68,507.354,103.68,566.021,103.68,600.854C103.68,635.688,103.68,646.688,103.68,652.188L103.68,657.688" id="L_id2_id3_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M103.68,739.688L103.68,743.854C103.68,748.021,103.68,756.354,135.549,767.387C167.418,778.42,231.156,792.152,263.025,799.018L294.894,805.884" id="L_id3_over_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M379.537,202.19L393.9,213.939C408.264,225.689,436.991,249.188,451.43,266.521C465.868,283.854,466.017,295.021,466.091,300.604L466.165,306.188" id="L_id2_id4_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M409.678,531.647L398.905,547.154C388.132,562.66,366.585,593.674,355.812,614.681C345.039,635.688,345.039,646.688,345.039,652.188L345.039,657.688" id="L_id4_id5_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M345.039,739.688L345.039,743.854C345.039,748.021,345.039,756.354,345.039,764.021C345.039,771.688,345.039,778.688,345.039,782.188L345.039,785.688" id="L_id5_over_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M522.759,531.647L533.366,547.154C543.972,562.66,565.185,593.674,575.792,616.681C586.398,639.688,586.398,654.688,586.398,662.188L586.398,669.688" id="L_id4_id6_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M586.398,727.688L586.398,733.854C586.398,740.021,586.398,752.354,554.529,765.387C522.66,778.42,458.922,792.152,427.053,799.018L395.184,805.884" id="L_id6_over_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M345.039,843.688L345.039,847.854C345.039,852.021,345.039,860.354,363.771,869.211C382.503,878.068,419.968,887.449,438.7,892.139L457.432,896.829" id="L_over_id7_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path><path d="M644.125,895.136L659.894,890.728C675.663,886.32,707.201,877.504,722.969,864.429C738.738,851.354,738.738,834.021,738.738,816.688C738.738,799.354,738.738,782.021,738.738,762.688C738.738,743.354,738.738,722.021,738.738,698.688C738.738,675.354,738.738,650.021,738.738,608.021C738.738,566.021,738.738,507.354,738.738,448.688C738.738,390.021,738.738,331.354,738.738,285.547C738.738,239.74,738.738,206.792,738.738,175.844C738.738,144.896,738.738,115.948,693.739,93.844C648.739,71.741,558.74,56.482,513.74,48.852L468.741,41.223" id="L_id7_start_0" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" style="" marker-end="url(#mermaid-1792520053371-loci1rvfl_flowchart-v2-pointEnd)"></path></g><g class="edgeLabels"><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(103.6796875, 448.6875)"><g class="label" transform="translate(-20.59375, -12)"><foreignObject width="41.1875" height="24"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"><p>|Yes|</p></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(465.71875, 272.6875)"><g class="label" transform="translate(-18.4921875, -12)"><foreignObject width="36.984375" height="24"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"><p>|No|</p></span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(345.0390625, 624.6875)"><g class="label" transform="translate(-20.59375, -12)"><foreignObject width="41.1875" height="24"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"><p>|Yes|</p></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(586.3984375, 624.6875)"><g class="label" transform="translate(-18.4921875, -12)"><foreignObject width="36.984375" height="24"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"><p>|No|</p></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" class="labelBkg" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node default" id="flowchart-start-0" transform="translate(432.0390625, 35)"><rect class="basic label-container" style="" rx="5" ry="5" x="-32.7578125" y="-27" width="65.515625" height="54"></rect><g class="label" style="" transform="translate(-17.7578125, -12)"><rect></rect><foreignObject width="35.515625" height="24"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>FP32</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-id2-1" transform="translate(345.0390625, 173.84375)"><polygon points="61.84375,0 123.6875,-61.84375 61.84375,-123.6875 0,-61.84375" class="label-container" transform="translate(-61.84375,61.84375)"></polygon><g class="label" style="" transform="translate(-34.84375, -12)"><rect></rect><foreignObject width="69.6875" height="24"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>Is it NaN?</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-id3-3" transform="translate(103.6796875, 700.6875)"><rect class="basic label-container" style="" x="-95.6796875" y="-39" width="191.359375" height="78"></rect><g class="label" style="" transform="translate(-65.6796875, -24)"><rect></rect><foreignObject width="131.359375" height="48"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>Shift 16 bits right<br>set to QNaN</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-over-4" transform="translate(345.0390625, 816.6875)"><rect class="basic label-container" style="" rx="5" ry="5" x="-46.234375" y="-27" width="92.46875" height="54"></rect><g class="label" style="" transform="translate(-31.234375, -12)"><rect></rect><foreignObject width="62.46875" height="24"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>bfloat16</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-id4-6" transform="translate(465.71875, 448.6875)"><polygon points="139,0 278,-139 139,-278 0,-139" class="label-container" transform="translate(-139,139)"></polygon><g class="label" style="" transform="translate(-100, -24)"><rect></rect><foreignObject width="200" height="48"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table; white-space: break-spaces; line-height: 1.5; max-width: 200px; text-align: center; width: 200px;"><span class="nodeLabel"><p>Is the LSB of the leftmost 16 bits 1?</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-id5-8" transform="translate(345.0390625, 700.6875)"><rect class="basic label-container" style="" x="-95.6796875" y="-39" width="191.359375" height="78"></rect><g class="label" style="" transform="translate(-65.6796875, -24)"><rect></rect><foreignObject width="131.359375" height="48"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>Shift 16 bits right<br>add 1</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-id6-11" transform="translate(586.3984375, 700.6875)"><rect class="basic label-container" style="" x="-95.6796875" y="-27" width="191.359375" height="54"></rect><g class="label" style="" transform="translate(-65.6796875, -12)"><rect></rect><foreignObject width="131.359375" height="24"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>Shift 16 bits right</p></span></div></foreignObject></g></g><g class="node default" id="flowchart-id7-14" transform="translate(552.71875, 920.6875)"><rect class="basic label-container" style="" x="-91.40625" y="-27" width="182.8125" height="54"></rect><g class="label" style="" transform="translate(-61.40625, -12)"><rect></rect><foreignObject width="122.8125" height="24"><div xmlns="http://www.w3.org/1999/xhtml" style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;"><span class="nodeLabel"><p>Shift 16 bits left</p></span></div></foreignObject></g></g></g></g></g></svg></pre>
      </div><p class="part" data-startline="52" data-endline="52" data-position="3879" data-size="0"><span data-position="3879" data-size="690">The first step in converting float32 to bfloat16 is to check if the value is NaN. If it is NaN, then the bfloat16 conversion will also result in NaN. NaN can be further classified into QNaN (Quiet NaN) and SNaN (Signaling NaN). The difference between the two is that QNaN represents an undefined result and does not cause a program interruption, while SNaN represents an uninitialized value and will cause a program interruption. Since we want to avoid interruptions during machine learning model computations, NaN must be forcibly converted to QNaN. By definition, if the Most Significant Bit (MSB) of the mantissa is 1, it is classified as QNaN; if the MSB is 0, it is classified as SNaN.</span></p><p class="part" data-startline="54" data-endline="54" data-position="4571" data-size="0"><span data-position="4571" data-size="291">If the float32 value is not NaN, the mantissa needs to be checked and rounded if necessary. We use the “Round to Nearest, ties to Even” rounding mode. If the Least Significant Bit (LSB) of bfloat16 is 1, we add 1 to the binary representation of bfloat16; otherwise, no addition is necessary.</span></p><p class="part" data-startline="56" data-endline="56" data-position="4864" data-size="0"><span data-position="4864" data-size="91">Finally, shift the entire value to the right by 16 bits to convert it into bfloat16 format.</span></p><p class="part" data-startline="58" data-endline="58" data-position="4957" data-size="0"><span data-position="4957" data-size="83">To convert bfloat16 back to float32, simply shift the value to the left by 16 bits.</span></p><h3 class="part" data-startline="60" data-endline="60" id="C-program" data-id="C-program"><a class="anchor hidden-xs" href="#C-program" title="C-program" smoothhashscroll="" data-size="9" data-position="5046"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="5046" data-size="9">C program</span></h3><div class="alert alert-danger part" data-startline="62" data-endline="63" data-position="5057" data-size="66">
<p data-position="5067" data-size="0"><span data-position="5067" data-size="8">Why was </span><code data-position="5076" data-size="5">union</code><span data-position="5082" data-size="37"> used rather than arbitrary pointers?</span></p>
</div><div class="alert alert-info part" data-startline="65" data-endline="66" data-position="5124" data-size="415">
<p data-position="5132" data-size="0"><span data-position="5132" data-size="402">A union allows different data types to share the same memory space, enabling direct conversion between a floating-point number and its corresponding binary representation. This allows seamless switching between the memory representation of a float and a uint32_t. In contrast, if type casting or pointer operations were used to achieve the same effect, such an approach might lead to undefined behavior.</span></p>
</div><div class="part code-block-wrapper code-block-toolbar-handled" data-startline="69" data-endline="98" data-infoprefix-length="1" data-position="5541"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div>
        <pre class="click-event-handled"><code class="c hljs"><span class="token keyword">static</span> <span class="token keyword">inline</span> <span class="token keyword">float</span> <span class="token function">bf16_to_fp32</span><span class="token punctuation">(</span><span class="token class-name">bf16_t</span> h<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
    <span class="token keyword">union</span> <span class="token punctuation">{</span>
        <span class="token keyword">float</span> f<span class="token punctuation">;</span>
        <span class="token class-name">uint32_t</span> i<span class="token punctuation">;</span>
    <span class="token punctuation">}</span> u <span class="token operator">=</span> <span class="token punctuation">{</span><span class="token punctuation">.</span>i <span class="token operator">=</span> <span class="token punctuation">(</span><span class="token class-name">uint32_t</span><span class="token punctuation">)</span>h<span class="token punctuation">.</span>bits <span class="token operator">&lt;&lt;</span> <span class="token number">16</span><span class="token punctuation">}</span><span class="token punctuation">;</span> 
    <span class="token comment">// First, convert uint16_t to uint32_t and then right shift by 16 bits.</span>
    <span class="token keyword">return</span> u<span class="token punctuation">.</span>f<span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token keyword">static</span> <span class="token keyword">inline</span> <span class="token class-name">bf16_t</span> <span class="token function">fp32_to_bf16</span><span class="token punctuation">(</span><span class="token keyword">float</span> s<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
    <span class="token class-name">bf16_t</span> h<span class="token punctuation">;</span>
    <span class="token keyword">union</span> <span class="token punctuation">{</span>
        <span class="token keyword">float</span> f<span class="token punctuation">;</span>
        <span class="token class-name">uint32_t</span> i<span class="token punctuation">;</span>
    <span class="token punctuation">}</span> u <span class="token operator">=</span> <span class="token punctuation">{</span><span class="token punctuation">.</span>f <span class="token operator">=</span> s<span class="token punctuation">}</span><span class="token punctuation">;</span>
    <span class="token comment">// The following block handles converting NaN to QNaN.</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token punctuation">(</span>u<span class="token punctuation">.</span>i <span class="token operator">&amp;</span> <span class="token number">0x7fffffff</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0x7f800000</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> <span class="token comment">/* NaN */</span> <span class="token comment">// If the exponent is all 1s and the fraction is not 0, then it is NaN. If it is NaN, proceed to the next line.</span>
        h<span class="token punctuation">.</span>bits <span class="token operator">=</span> <span class="token punctuation">(</span>u<span class="token punctuation">.</span>i <span class="token operator">&gt;&gt;</span> <span class="token number">16</span><span class="token punctuation">)</span> <span class="token operator">|</span> <span class="token number">64</span><span class="token punctuation">;</span>         <span class="token comment">/* Force NaN to quiet (set the first bit of the Fraction to 1) */</span> <span class="token comment">// Right shift u.i by 16 bits and set the first bit of the Fraction to 1.</span>
        <span class="token keyword">return</span> h<span class="token punctuation">;</span>            <span class="token comment">// 64 converted to binary = 01000000                                                                                                                                    </span>
    <span class="token punctuation">}</span>
    <span class="token comment">// The following block handles non-NaN values.</span>
    h<span class="token punctuation">.</span>bits <span class="token operator">=</span> <span class="token punctuation">(</span>u<span class="token punctuation">.</span>i <span class="token operator">+</span> <span class="token punctuation">(</span><span class="token number">0x7fff</span> <span class="token operator">+</span> <span class="token punctuation">(</span><span class="token punctuation">(</span>u<span class="token punctuation">.</span>i <span class="token operator">&gt;&gt;</span> <span class="token number">0x10</span><span class="token punctuation">)</span> <span class="token operator">&amp;</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">&gt;&gt;</span> <span class="token number">0x10</span><span class="token punctuation">;</span> 
    <span class="token comment">//u.i &gt;&gt; 0x10，Right shift by 16 bits.</span>
    <span class="token comment">//(u.i &gt;&gt; 0x10) &amp; 1，Check if it is odd or even; odd is 1 and even is 0.</span>
    <span class="token keyword">return</span> h<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
      </div><h3 class="part" data-startline="100" data-endline="100" id="RISC-V-RV32I" data-id="RISC-V-RV32I"><a class="anchor hidden-xs" href="#RISC-V-RV32I" title="RISC-V-RV32I" smoothhashscroll="" data-size="14" data-position="6772"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="6772" data-size="14">RISC-V (RV32I)</span></h3><div class="part code-block-wrapper code-block-toolbar-handled" data-startline="101" data-endline="190" data-infoprefix-length="1" data-position="6788"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div>
        <pre class="click-event-handled"><code class="c hljs"><span class="token punctuation">.</span>data
num1<span class="token operator">:</span>    <span class="token punctuation">.</span>word <span class="token number">0x3F80</span><span class="token punctuation">,</span> <span class="token number">0xBF80</span><span class="token punctuation">,</span> <span class="token number">0x7F80</span>
num2<span class="token operator">:</span>    <span class="token punctuation">.</span>dword <span class="token number">0x407F8000</span><span class="token punctuation">,</span> <span class="token number">0x40C08000</span><span class="token punctuation">,</span> <span class="token number">0x7F800000</span>
iterate<span class="token operator">:</span> <span class="token punctuation">.</span>word <span class="token number">3</span>
str1<span class="token operator">:</span>    <span class="token punctuation">.</span>string <span class="token string">"\nThe fp32 value of bf16 number "</span>
str2<span class="token operator">:</span>    <span class="token punctuation">.</span>string <span class="token string">"\nThe bf16 value of fp32 number "</span>
str3<span class="token operator">:</span>    <span class="token punctuation">.</span>string <span class="token string">" is "</span>
<span class="token punctuation">.</span>text
main<span class="token operator">:</span>
    lw t6<span class="token punctuation">,</span>iterate
    la t5<span class="token punctuation">,</span>num1
test1<span class="token operator">:</span>
    lw a1<span class="token punctuation">,</span><span class="token number">0</span><span class="token punctuation">(</span>t5<span class="token punctuation">)</span>
    jal ra<span class="token punctuation">,</span> bf16_to_fp32
    jal ra<span class="token punctuation">,</span> printResult_fp32
    addi t6<span class="token punctuation">,</span>t6<span class="token punctuation">,</span><span class="token operator">-</span><span class="token number">1</span>
    addi t5<span class="token punctuation">,</span>t5<span class="token punctuation">,</span><span class="token number">4</span>
    bne zero<span class="token punctuation">,</span> t6<span class="token punctuation">,</span> test1

    lw t6<span class="token punctuation">,</span>iterate
    la t5<span class="token punctuation">,</span>num2
test2<span class="token operator">:</span>
    lw a1<span class="token punctuation">,</span><span class="token number">0</span><span class="token punctuation">(</span>t5<span class="token punctuation">)</span>
    jal ra<span class="token punctuation">,</span> fp32_to_bf16
    jal ra<span class="token punctuation">,</span> printResult_bf16
    addi t6<span class="token punctuation">,</span>t6<span class="token punctuation">,</span><span class="token operator">-</span><span class="token number">1</span>
    addi t5<span class="token punctuation">,</span>t5<span class="token punctuation">,</span><span class="token number">8</span>
    bne zero<span class="token punctuation">,</span> t6<span class="token punctuation">,</span> test2
    
    <span class="token macro property"><span class="token directive-hash">#</span> <span class="token expression">Exit the program</span></span>
    li a7<span class="token punctuation">,</span> <span class="token number">10</span>                  # System call code <span class="token keyword">for</span> exiting the program
    ecall                      # Make the exit system call
    
bf16_to_fp32<span class="token operator">:</span>
    slli t0<span class="token punctuation">,</span> a1<span class="token punctuation">,</span> <span class="token number">16</span>
    ret
    
fp32_to_bf16<span class="token operator">:</span>
    li t1<span class="token punctuation">,</span> <span class="token number">0x7FFFFFFF</span> #check nosign
    li t2<span class="token punctuation">,</span> <span class="token number">0x7F800000</span> #check is NaN 
    and t1<span class="token punctuation">,</span> a1<span class="token punctuation">,</span> t1
    srli t0<span class="token punctuation">,</span> a1<span class="token punctuation">,</span> <span class="token number">16</span>     #fp32 shift right logic <span class="token number">16</span> bits convert to bf16
    blt t2<span class="token punctuation">,</span> t1<span class="token punctuation">,</span> fp32_to_bf16_isNaN #
    andi t3<span class="token punctuation">,</span> t0<span class="token punctuation">,</span> <span class="token number">1</span>      #check bf16 fraction LSB is one<span class="token operator">?</span>
    bnez t3<span class="token punctuation">,</span> LSB_is_one #<span class="token keyword">goto</span> Round to Nearest<span class="token punctuation">,</span> ties to Even
    ret
LSB_is_one<span class="token operator">:</span>
    addi t0<span class="token punctuation">,</span> t0<span class="token punctuation">,</span> <span class="token number">1</span>      #When adding <span class="token number">1</span> to a bf16 value<span class="token punctuation">,</span> it is rounded to <span class="token function">even</span> <span class="token punctuation">(</span>Round to even<span class="token punctuation">)</span><span class="token punctuation">.</span>
    ret
fp32_to_bf16_isNaN<span class="token operator">:</span>
    ori t0<span class="token punctuation">,</span> t0<span class="token punctuation">,</span> <span class="token number">0x40</span>
    ret
    
printResult_fp32<span class="token operator">:</span>
    la a0<span class="token punctuation">,</span> str1 #set the system call use data
    li a7<span class="token punctuation">,</span> <span class="token number">4</span>    #set the system call number
    ecall
    
    mv a0<span class="token punctuation">,</span> a1 #set the system call use data
    li a7<span class="token punctuation">,</span> <span class="token number">1</span>  #set the system call number
    ecall
    
    la a0<span class="token punctuation">,</span> str3
    li a7<span class="token punctuation">,</span> <span class="token number">4</span>
    ecall
    
    mv a0<span class="token punctuation">,</span> t0
    li a7<span class="token punctuation">,</span> <span class="token number">1</span>
    ecall
    ret
    
printResult_bf16<span class="token operator">:</span>
    la a0<span class="token punctuation">,</span> str2 #set the system call use data
    li a7<span class="token punctuation">,</span> <span class="token number">4</span>    #set the system call number
    ecall
    
    mv a0<span class="token punctuation">,</span> a1 #set the system call use data
    li a7<span class="token punctuation">,</span> <span class="token number">1</span>  #set the system call number
    ecall
    
    la a0<span class="token punctuation">,</span> str3
    li a7<span class="token punctuation">,</span> <span class="token number">4</span>
    ecall
    
    mv a0<span class="token punctuation">,</span> t0
    li a7<span class="token punctuation">,</span> <span class="token number">1</span>
    ecall
    ret
</code></pre>
      </div><h2 class="part" data-startline="193" data-endline="193" id="LeetCode-2239-Find-Closest-Number-to-Zero" data-id="LeetCode-2239-Find-Closest-Number-to-Zero"><a class="anchor hidden-xs" href="#LeetCode-2239-Find-Closest-Number-to-Zero" title="LeetCode-2239-Find-Closest-Number-to-Zero" smoothhashscroll=""><span class="octicon octicon-link ph ph-link-simple-horizontal" data-size="41" data-position="8748"></span></a><a href="https://leetcode.com/problems/find-closest-number-to-zero/description/" target="_blank" rel="noopener ugc nofollow"><span data-position="8748" data-size="41">LeetCode 2239 Find Closest Number to Zero</span></a></h2><h3 class="part" data-startline="195" data-endline="195" id="Description" data-id="Description"><a class="anchor hidden-xs" href="#Description" title="Description" smoothhashscroll="" data-size="11" data-position="8868"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="8868" data-size="11">Description</span></h3><p class="part" data-startline="196" data-endline="196" data-position="8880" data-size="0"><span data-position="8880" data-size="23">Given an integer array </span><code data-position="8904" data-size="4">nums</code><span data-position="8909" data-size="9"> of size </span><code data-position="8919" data-size="1">n</code><span data-position="8922" data-size="46">, return the number with the value closest to </span><code data-position="8968" data-size="1">0</code><span data-position="8970" data-size="4"> in </span><code data-position="8975" data-size="4">nums</code><span data-position="8981" data-size="74">. If there are multiple answers, return the number with the largest value.</span></p><p class="part" data-startline="198" data-endline="198" data-position="9056" data-size="0"><span data-position="9056" data-size="12">Constraints:</span></p><ul class="part" data-startline="199" data-endline="201" data-position="9069" data-size="0">
<li class="" data-startline="199" data-endline="199" data-position="9074" data-size="0"><span data-position="9074" data-size="14">1 &lt;= n &lt;= 1000</span></li>
<li class="" data-startline="200" data-endline="201" data-position="9094" data-size="0"><span data-position="9095" data-size="22">-105 &lt;= nums[i] &lt;= 105</span></li>
</ul><p class="part" data-startline="202" data-endline="202" data-position="9118" data-size="0"><span data-position="9118" data-size="10">Example 1:</span></p><div class="code-block-wrapper part indent-code-block code-block-toolbar-handled" data-startline="204" data-endline="212" data-position="9130" data-size="349"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div><pre class="click-event-handled"><code class="code_block">​​​​Input: nums = [-4,-2,1,4,8]
​​​​Output: 1
​​​​Explanation:
​​​​    The distance from -4 to 0 is |-4| = 4.
​​​​    The distance from -2 to 0 is |-2| = 2.
​​​​    The distance from 1 to 0 is |1| = 1.
​​​​    The distance from 4 to 0 is |4| = 4.
​​​​    The distance from 8 to 0 is |8| = 8.
​​​​    Thus, the closest number to 0 in the array is 1.
</code></pre></div><p class="part in-view" data-startline="214" data-endline="214" data-position="9480" data-size="0"><span data-position="9480" data-size="10">Example 2:</span></p><div class="code-block-wrapper part indent-code-block code-block-toolbar-handled in-view" data-startline="216" data-endline="219" data-position="9492" data-size="151"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div><pre class="click-event-handled"><code class="code_block">​​​​Input: nums = [2,-1,1]
​​​​Output: 1
​​​​Explanation: 1 and -1 are both the closest numbers to 0, 
​​​​             so 1 being larger is returned.
</code></pre></div><h3 class="part in-view" data-startline="220" data-endline="220" id="Thinking8" data-id="Thinking"><a class="anchor hidden-xs" href="#Thinking8" title="Thinking8" smoothhashscroll="" data-size="8" data-position="9647"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="9647" data-size="8">Thinking</span></h3><p class="part in-view" data-startline="222" data-endline="222" data-position="9657" data-size="0"><span data-position="9657" data-size="677">My intuitive idea is to set a parameter called closest_num to represent the number closest to 0, with its default value set as the first element of the array. Using a for loop to scan through the array, if the absolute value of the current array element is smaller than the absolute value of closest_num, it means the current element is closer to 0, so I replace it with the new value of closest_num. Additionally, if the absolute value of the current array element is equal to the absolute value of closest_num, I need to check if the current element is greater than closest_num. If the current element is greater, it means it is larger, so I replace closest_num with the new value.</span></p><h3 class="part in-view" data-startline="223" data-endline="223" id="C-Program" data-id="C-Program"><a class="anchor hidden-xs" href="#C-Program" title="C-Program" smoothhashscroll="" data-size="9" data-position="10345"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="10345" data-size="9">C Program</span></h3><div class="part code-block-wrapper code-block-toolbar-handled in-view" data-startline="225" data-endline="239" data-infoprefix-length="1" data-position="10356"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div>
        <pre class="click-event-handled"><code class="c hljs"><span class="token keyword">int</span> <span class="token function">findClosestNumber</span><span class="token punctuation">(</span><span class="token keyword">int</span><span class="token operator">*</span> nums<span class="token punctuation">,</span> <span class="token keyword">int</span> numsSize<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">int</span> closest_num <span class="token operator">=</span> nums<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
    <span class="token keyword">for</span><span class="token punctuation">(</span><span class="token keyword">int</span> i<span class="token operator">=</span><span class="token number">0</span><span class="token punctuation">;</span>i<span class="token operator">&lt;</span>numsSize<span class="token punctuation">;</span>i<span class="token operator">++</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
        <span class="token keyword">if</span><span class="token punctuation">(</span><span class="token function">abs</span><span class="token punctuation">(</span>nums<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token operator">&lt;</span><span class="token function">abs</span><span class="token punctuation">(</span>closest_num<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
            closest_num <span class="token operator">=</span> nums<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">;</span>
        <span class="token punctuation">}</span><span class="token keyword">else</span> <span class="token keyword">if</span><span class="token punctuation">(</span><span class="token function">abs</span><span class="token punctuation">(</span>closest_num<span class="token punctuation">)</span><span class="token operator">==</span><span class="token function">abs</span><span class="token punctuation">(</span>nums<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">{</span>
            <span class="token keyword">if</span><span class="token punctuation">(</span>nums<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token operator">&gt;</span>closest_num<span class="token punctuation">)</span><span class="token punctuation">{</span>
                closest_num <span class="token operator">=</span> nums<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">;</span>
            <span class="token punctuation">}</span>
        <span class="token punctuation">}</span>
    <span class="token punctuation">}</span>
    <span class="token keyword">return</span> closest_num<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
      </div><h3 class="part" data-startline="243" data-endline="243" id="RISC-V-RV32I-version-1" data-id="RISC-V-RV32I-version-1"><a class="anchor hidden-xs" href="#RISC-V-RV32I-version-1" title="RISC-V-RV32I-version-1" smoothhashscroll="" data-size="24" data-position="10745"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="10745" data-size="24">RISC-V (RV32I) version 1</span></h3><div class="part code-block-wrapper code-block-toolbar-handled" data-startline="244" data-endline="322" data-infoprefix-length="7" data-position="10770"><div class="code-toolbar"><div class="flex flex-row gap-0.5 rounded-lg bg-element-bg-default px-[7px] py-[5px]"><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-edit-from-here-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-map-pin-simple" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><span class="ml-1 border-0 border-l border-solid border-background-subtle pr-1" aria-hidden="true"></span><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-link-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-link" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button><button class="rounded text-lg font-normal leading-sm flex bg-transparent text-text-default border border-solid border-transparent hocus:bg-element-bg-hover hocus:text-text-emphasize hover:border-element-bg-hover focus:shadow-[0_0_0_2px_#77777733] focus:border-element-border-hover disabled:bg-transparent disabled:hocus:bg-transparent disabled:hocus:border-transparent disabled:text-element-text-disabled disabled:hocus:text-element-text-disabled p-0.5 ui-code-block-copy-button ui-code-block-toolbar-type-code" data-state="closed"><i class="inline-flex ph ph-clipboard-text" aria-hidden="true" style="width: 20px; height: 20px; font-size: 20px; line-height: 20px;"></i></button></div></div>
        <pre class="click-event-handled"><code class="RISC-V hljs"><div class="wrapper"><div class="gutter linenumber"><span data-linenumber="1"></span>
<span data-linenumber="2"></span>
<span data-linenumber="3"></span>
<span data-linenumber="4"></span>
<span data-linenumber="5"></span>
<span data-linenumber="6"></span>
<span data-linenumber="7"></span>
<span data-linenumber="8"></span>
<span data-linenumber="9"></span>
<span data-linenumber="10"></span>
<span data-linenumber="11"></span>
<span data-linenumber="12"></span>
<span data-linenumber="13"></span>
<span data-linenumber="14"></span>
<span data-linenumber="15"></span>
<span data-linenumber="16"></span>
<span data-linenumber="17"></span>
<span data-linenumber="18"></span>
<span data-linenumber="19"></span>
<span data-linenumber="20"></span>
<span data-linenumber="21"></span>
<span data-linenumber="22"></span>
<span data-linenumber="23"></span>
<span data-linenumber="24"></span>
<span data-linenumber="25"></span>
<span data-linenumber="26"></span>
<span data-linenumber="27"></span>
<span data-linenumber="28"></span>
<span data-linenumber="29"></span>
<span data-linenumber="30"></span>
<span data-linenumber="31"></span>
<span data-linenumber="32"></span>
<span data-linenumber="33"></span>
<span data-linenumber="34"></span>
<span data-linenumber="35"></span>
<span data-linenumber="36"></span>
<span data-linenumber="37"></span>
<span data-linenumber="38"></span>
<span data-linenumber="39"></span>
<span data-linenumber="40"></span>
<span data-linenumber="41"></span>
<span data-linenumber="42"></span>
<span data-linenumber="43"></span>
<span data-linenumber="44"></span>
<span data-linenumber="45"></span>
<span data-linenumber="46"></span>
<span data-linenumber="47"></span>
<span data-linenumber="48"></span>
<span data-linenumber="49"></span>
<span data-linenumber="50"></span>
<span data-linenumber="51"></span>
<span data-linenumber="52"></span>
<span data-linenumber="53"></span>
<span data-linenumber="54"></span>
<span data-linenumber="55"></span>
<span data-linenumber="56"></span>
<span data-linenumber="57"></span>
<span data-linenumber="58"></span>
<span data-linenumber="59"></span>
<span data-linenumber="60"></span>
<span data-linenumber="61"></span>
<span data-linenumber="62"></span>
<span data-linenumber="63"></span>
<span data-linenumber="64"></span>
<span data-linenumber="65"></span>
<span data-linenumber="66"></span>
<span data-linenumber="67"></span>
<span data-linenumber="68"></span>
<span data-linenumber="69"></span>
<span data-linenumber="70"></span>
<span data-linenumber="71"></span>
<span data-linenumber="72"></span>
<span data-linenumber="73"></span>
<span data-linenumber="74"></span>
<span data-linenumber="75"></span>
<span data-linenumber="76"></span>
<span data-linenumber="77"></span></div><div class="code">    <span class="hljs-meta">.data</span>
<span class="hljs-symbol">nums:</span>
    <span class="hljs-meta">.word</span> -<span class="hljs-number">4</span>,-<span class="hljs-number">2</span>,<span class="hljs-number">1</span>,<span class="hljs-number">4</span>,<span class="hljs-number">8</span>
    <span class="hljs-meta">.word</span> <span class="hljs-number">0</span>,-<span class="hljs-number">2</span>,<span class="hljs-number">2</span>
    <span class="hljs-meta">.word</span> <span class="hljs-number">99</span>,-<span class="hljs-number">99</span>,<span class="hljs-number">100</span>,-<span class="hljs-number">100</span>,<span class="hljs-number">101</span>,-<span class="hljs-number">10</span>
<span class="hljs-symbol">numsSize:</span>
    <span class="hljs-meta">.word</span> <span class="hljs-number">5</span>
    <span class="hljs-meta">.word</span> <span class="hljs-number">3</span>
    <span class="hljs-meta">.word</span> <span class="hljs-number">6</span>
<span class="hljs-symbol">str:</span> .string <span class="hljs-string">"\n"</span>
<span class="hljs-meta">.text</span>
<span class="hljs-symbol">main:</span>
    li <span class="hljs-built_in">s1</span>, <span class="hljs-number">3</span>         <span class="hljs-comment">#number of test case</span>
    la <span class="hljs-built_in">s2</span>, numsSize  <span class="hljs-comment">#load address of numsSize</span>
    la <span class="hljs-built_in">s3</span>, nums      <span class="hljs-comment">#load address of nums</span>
<span class="hljs-symbol">test:</span>
    <span class="hljs-keyword">jal </span><span class="hljs-built_in">ra</span>, findNonMinOrMax <span class="hljs-comment">#goto findNonMinOrMax function</span>
    <span class="hljs-keyword">jal </span><span class="hljs-built_in">ra</span>, printResult     <span class="hljs-comment">#goto printResult function           </span>
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">s2</span>, <span class="hljs-built_in">s2</span>, <span class="hljs-number">4</span>          <span class="hljs-comment">#next numsSize</span>
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">s1</span>, <span class="hljs-built_in">s1</span>, -<span class="hljs-number">1</span>         
    <span class="hljs-keyword">bne </span><span class="hljs-built_in">s1</span>, <span class="hljs-built_in">zero</span>, test
    <span class="hljs-comment"># Exit the program</span>
    li <span class="hljs-built_in">a7</span>, <span class="hljs-number">10</span>      <span class="hljs-comment"># System call code for exiting the program</span>
    ecall          <span class="hljs-comment"># Make the exit system call</span>
<span class="hljs-symbol">
findNonMinOrMax:</span>
    <span class="hljs-keyword">lw </span><span class="hljs-built_in">t0</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">s2</span>)  <span class="hljs-comment">#int i</span>
    <span class="hljs-keyword">lw </span><span class="hljs-built_in">t1</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">s3</span>)  <span class="hljs-comment">#closest_num = nums[0]</span>
<span class="hljs-symbol">loop:</span>
    <span class="hljs-keyword">lw </span><span class="hljs-built_in">t2</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">s3</span>)  <span class="hljs-comment">#nums[i]</span>
    
    <span class="hljs-comment">#set abs(closest_num)</span>
    <span class="hljs-keyword">add </span><span class="hljs-built_in">a1</span>, <span class="hljs-built_in">zero</span>, <span class="hljs-built_in">t1</span> 
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">sp</span>, <span class="hljs-built_in">sp</span>, -<span class="hljs-number">4</span>
    <span class="hljs-keyword">sw </span><span class="hljs-built_in">ra</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">sp</span>)
    <span class="hljs-keyword">jal </span><span class="hljs-built_in">ra</span>, set_abs
    <span class="hljs-keyword">lw </span><span class="hljs-built_in">ra</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">sp</span>)
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">sp</span>, <span class="hljs-built_in">sp</span> <span class="hljs-number">4</span>
    <span class="hljs-keyword">add </span><span class="hljs-built_in">t3</span>, <span class="hljs-built_in">a1</span>, <span class="hljs-built_in">zero</span>

    <span class="hljs-comment">#set abs(nums[i])</span>
    <span class="hljs-keyword">add </span><span class="hljs-built_in">a1</span>, <span class="hljs-built_in">zero</span>, <span class="hljs-built_in">t2</span>
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">sp</span>, <span class="hljs-built_in">sp</span>, -<span class="hljs-number">4</span>
    <span class="hljs-keyword">sw </span><span class="hljs-built_in">ra</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">sp</span>)
    <span class="hljs-keyword">jal </span><span class="hljs-built_in">ra</span>, set_abs
    <span class="hljs-keyword">lw </span><span class="hljs-built_in">ra</span>, <span class="hljs-number">0</span>(<span class="hljs-built_in">sp</span>)
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">sp</span>, <span class="hljs-built_in">sp</span> <span class="hljs-number">4</span>
    <span class="hljs-keyword">add </span><span class="hljs-built_in">t4</span>, <span class="hljs-built_in">a1</span>, <span class="hljs-built_in">zero</span>
<span class="hljs-symbol">next_1:</span> 
    <span class="hljs-comment">#if(abs(nums[i])&lt;abs(closest_num)) then closest_num=nums[i]</span>
    <span class="hljs-keyword">bge </span><span class="hljs-built_in">t4</span>, <span class="hljs-built_in">t3</span>, next_3 
    <span class="hljs-keyword">add </span><span class="hljs-built_in">t1</span>, <span class="hljs-built_in">zero</span>, <span class="hljs-built_in">t2</span>
    <span class="hljs-keyword">j </span>loop_decrement
    <span class="hljs-comment">#if(abs(nums[i])==abs(closest_num)) then continue</span>
<span class="hljs-symbol">next_2:</span> <span class="hljs-keyword">bne </span><span class="hljs-built_in">t4</span>, <span class="hljs-built_in">t3</span>, loop_decrement
        <span class="hljs-comment">#if(nums[i]&gt;closest_num) then closest_num=nums[i]</span>
        <span class="hljs-keyword">ble </span><span class="hljs-built_in">t2</span>, <span class="hljs-built_in">t1</span>, loop_decrement 
        <span class="hljs-keyword">add </span><span class="hljs-built_in">t1</span>, <span class="hljs-built_in">zero</span>, <span class="hljs-built_in">t2</span>
<span class="hljs-symbol">loop_decrement:</span>
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">s3</span>, <span class="hljs-built_in">s3</span>, <span class="hljs-number">4</span>
    <span class="hljs-keyword">addi </span><span class="hljs-built_in">t0</span>, <span class="hljs-built_in">t0</span>, -<span class="hljs-number">1</span> 
    <span class="hljs-keyword">bne </span><span class="hljs-built_in">t0</span>,<span class="hljs-built_in">zero</span>, loop
    ret
<span class="hljs-symbol">set_abs:</span>
    <span class="hljs-keyword">bgez </span><span class="hljs-built_in">a1</span>, return
    neg <span class="hljs-built_in">a1</span>,<span class="hljs-built_in">a1</span>
<span class="hljs-symbol">return:</span>
    ret
<span class="hljs-symbol">printResult:</span>
    mv <span class="hljs-built_in">a0</span>, <span class="hljs-built_in">t1</span> <span class="hljs-comment">#set the system call use data</span>
    li <span class="hljs-built_in">a7</span>, <span class="hljs-number">1</span>  <span class="hljs-comment">#set the system call number</span>
    ecall
    
    la <span class="hljs-built_in">a0</span>, str <span class="hljs-comment">#set the system call use data</span>
    li <span class="hljs-built_in">a7</span>, <span class="hljs-number">4</span>  <span class="hljs-comment">#set the system call number</span>
    ecall
    ret
</div></div></code></pre>
      </div><p class="part" data-startline="324" data-endline="324" data-position="12592" data-size="0"><img src="https://hackmd.io/_uploads/Sy77nHsyJl.png" alt="1" width="50%" class="offline-handled error-handled" data-position="12592" data-size="53" loading="lazy"></p><h3 class="part" data-startline="329" data-endline="329" id="RISC-V-RV32I-version-2" data-id="RISC-V-RV32I-version-2"><a class="anchor hidden-xs" href="#RISC-V-RV32I-version-2" title="RISC-V-RV32I-version-2" smoothhashscroll="" data-size="24" data-position="12654"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="12654" data-size="24">RISC-V (RV32I) version 2</span></h3><div class="part code-block-wrapper" data-startline="330" data-endline="395" data-infoprefix-length="7" data-position="12679">
        <pre><code class="RISC-V hljs raw"><div class="wrapper"><div class="gutter linenumber"><span data-linenumber="1"></span>
<span data-linenumber="2"></span>
<span data-linenumber="3"></span>
<span data-linenumber="4"></span>
<span data-linenumber="5"></span>
<span data-linenumber="6"></span>
<span data-linenumber="7"></span>
<span data-linenumber="8"></span>
<span data-linenumber="9"></span>
<span data-linenumber="10"></span>
<span data-linenumber="11"></span>
<span data-linenumber="12"></span>
<span data-linenumber="13"></span>
<span data-linenumber="14"></span>
<span data-linenumber="15"></span>
<span data-linenumber="16"></span>
<span data-linenumber="17"></span>
<span data-linenumber="18"></span>
<span data-linenumber="19"></span>
<span data-linenumber="20"></span>
<span data-linenumber="21"></span>
<span data-linenumber="22"></span>
<span data-linenumber="23"></span>
<span data-linenumber="24"></span>
<span data-linenumber="25"></span>
<span data-linenumber="26"></span>
<span data-linenumber="27"></span>
<span data-linenumber="28"></span>
<span data-linenumber="29"></span>
<span data-linenumber="30"></span>
<span data-linenumber="31"></span>
<span data-linenumber="32"></span>
<span data-linenumber="33"></span>
<span data-linenumber="34"></span>
<span data-linenumber="35"></span>
<span data-linenumber="36"></span>
<span data-linenumber="37"></span>
<span data-linenumber="38"></span>
<span data-linenumber="39"></span>
<span data-linenumber="40"></span>
<span data-linenumber="41"></span>
<span data-linenumber="42"></span>
<span data-linenumber="43"></span>
<span data-linenumber="44"></span>
<span data-linenumber="45"></span>
<span data-linenumber="46"></span>
<span data-linenumber="47"></span>
<span data-linenumber="48"></span>
<span data-linenumber="49"></span>
<span data-linenumber="50"></span>
<span data-linenumber="51"></span>
<span data-linenumber="52"></span>
<span data-linenumber="53"></span>
<span data-linenumber="54"></span>
<span data-linenumber="55"></span>
<span data-linenumber="56"></span>
<span data-linenumber="57"></span>
<span data-linenumber="58"></span>
<span data-linenumber="59"></span>
<span data-linenumber="60"></span>
<span data-linenumber="61"></span>
<span data-linenumber="62"></span>
<span data-linenumber="63"></span>
<span data-linenumber="64"></span></div><div class="code">.data
nums:
    .word -4,-2,1,4,8
    .word 0,-2,2
    .word 99,-99,100,-100,101,-10
numsSize:
    .word 5
    .word 3
    .word 6
str: .string "\n"
.text
main:
    li s1, 3         #number of test case
    la s2, numsSize  #load address of numsSize
    la s3, nums      #load address of nums
test:
    jal ra, findNonMinOrMax #goto findNonMinOrMax function
    jal ra, printResult     #goto printResult function           
    addi s2, s2, 4          #next numsSize
    addi s1, s1, -1         
    bne s1, zero, test
    # Exit the program
    li a7, 10      # System call code for exiting the program
    ecall          # Make the exit system call

findNonMinOrMax:
    lw t0, 0(s2)  #int i
    lw t1, 0(s3)  #closest_num = nums[0]
loop:
    lw t2, 0(s3)  #nums[i]
    
    #set abs(closest_num)
    add t3, zero, t1 
    bgez t3, next_1
    neg t3,t3
next_1:
    #set abs(nums[i])
    add t4, zero, t2
    bgez t4, next_2
    neg t4,t4
next_2: 
    #if(abs(nums[i])&lt;abs(closest_num)) then closest_num=nums[i]
    bge t4, t3, next_3 
    add t1, zero, t2
    j loop_decrement
    #if(abs(nums[i])==abs(closest_num)) then continue
next_3: bne t4, t3, loop_decrement
        #if(nums[i]&gt;closest_num) then closest_num=nums[i]
        ble t2, t1, loop_decrement 
        add t1, zero, t2
loop_decrement:
    addi s3, s3, 4
    addi t0, t0, -1 
    bne t0,zero, loop
    ret
printResult:
    mv a0, t1 #set the system call use data
    li a7, 1  #set the system call number
    ecall
    
    la a0, str #set the system call use data
    li a7, 4  #set the system call number
    ecall
    ret
</div></div></code></pre>
      </div><p class="part" data-startline="397" data-endline="397" data-position="14287" data-size="0"><img src="https://hackmd.io/_uploads/Sy0O3Bj1yl.png" alt="2" width="50%" class="offline-handled error-handled" data-position="14287" data-size="53" loading="lazy"></p><h3 class="part" data-startline="401" data-endline="401" id="Difference" data-id="Difference"><a class="anchor hidden-xs" href="#Difference" title="Difference" smoothhashscroll="" data-size="10" data-position="14348"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="14348" data-size="10">Difference</span></h3><div class="part code-block-wrapper" data-startline="402" data-endline="486" data-infoprefix-length="5" data-position="14359">
        <pre><code class="diff hljs raw"><div class="wrapper"><div class="gutter linenumber"><span data-linenumber="1"></span>
<span data-linenumber="2"></span>
<span data-linenumber="3"></span>
<span data-linenumber="4"></span>
<span data-linenumber="5"></span>
<span data-linenumber="6"></span>
<span data-linenumber="7"></span>
<span data-linenumber="8"></span>
<span data-linenumber="9"></span>
<span data-linenumber="10"></span>
<span data-linenumber="11"></span>
<span data-linenumber="12"></span>
<span data-linenumber="13"></span>
<span data-linenumber="14"></span>
<span data-linenumber="15"></span>
<span data-linenumber="16"></span>
<span data-linenumber="17"></span>
<span data-linenumber="18"></span>
<span data-linenumber="19"></span>
<span data-linenumber="20"></span>
<span data-linenumber="21"></span>
<span data-linenumber="22"></span>
<span data-linenumber="23"></span>
<span data-linenumber="24"></span>
<span data-linenumber="25"></span>
<span data-linenumber="26"></span>
<span data-linenumber="27"></span>
<span data-linenumber="28"></span>
<span data-linenumber="29"></span>
<span data-linenumber="30"></span>
<span data-linenumber="31"></span>
<span data-linenumber="32"></span>
<span data-linenumber="33"></span>
<span data-linenumber="34"></span>
<span data-linenumber="35"></span>
<span data-linenumber="36"></span>
<span data-linenumber="37"></span>
<span data-linenumber="38"></span>
<span data-linenumber="39"></span>
<span data-linenumber="40"></span>
<span data-linenumber="41"></span>
<span data-linenumber="42"></span>
<span data-linenumber="43"></span>
<span data-linenumber="44"></span>
<span data-linenumber="45"></span>
<span data-linenumber="46"></span>
<span data-linenumber="47"></span>
<span data-linenumber="48"></span>
<span data-linenumber="49"></span>
<span data-linenumber="50"></span>
<span data-linenumber="51"></span>
<span data-linenumber="52"></span>
<span data-linenumber="53"></span>
<span data-linenumber="54"></span>
<span data-linenumber="55"></span>
<span data-linenumber="56"></span>
<span data-linenumber="57"></span>
<span data-linenumber="58"></span>
<span data-linenumber="59"></span>
<span data-linenumber="60"></span>
<span data-linenumber="61"></span>
<span data-linenumber="62"></span>
<span data-linenumber="63"></span>
<span data-linenumber="64"></span>
<span data-linenumber="65"></span>
<span data-linenumber="66"></span>
<span data-linenumber="67"></span>
<span data-linenumber="68"></span>
<span data-linenumber="69"></span>
<span data-linenumber="70"></span>
<span data-linenumber="71"></span>
<span data-linenumber="72"></span>
<span data-linenumber="73"></span>
<span data-linenumber="74"></span>
<span data-linenumber="75"></span>
<span data-linenumber="76"></span>
<span data-linenumber="77"></span>
<span data-linenumber="78"></span>
<span data-linenumber="79"></span>
<span data-linenumber="80"></span>
<span data-linenumber="81"></span>
<span data-linenumber="82"></span>
<span data-linenumber="83"></span></div><div class="code">.data
nums:
    .word -4,-2,1,4,8
    .word 0,-2,2
    .word 99,-99,100,-100,101,-10
numsSize:
    .word 5
    .word 3
    .word 6
str: .string "\n"
.text
main:
    li s1, 3         #number of test case
    la s2, numsSize  #load address of numsSize
    la s3, nums      #load address of nums
test:
    jal ra, findNonMinOrMax #goto findNonMinOrMax function
    jal ra, printResult     #goto printResult function           
    addi s2, s2, 4          #next numsSize
    addi s1, s1, -1         
    bne s1, zero, test
    # Exit the program
    li a7, 10      # System call code for exiting the program
    ecall          # Make the exit system call

findNonMinOrMax:
    lw t0, 0(s2)  #int i
    lw t1, 0(s3)  #closest_num = nums[0]
loop:
    lw t2, 0(s3)  #nums[i]
    
    #set abs(closest_num)
-   add a1, zero, t1 
-   addi sp, sp, -4
-   sw ra, 0(sp)
-   jal ra, set_abs
-   lw ra, 0(sp)
-   addi sp, sp 4
-   add t3, a1, zero
+   add t3, zero, t1 
+   bgez t3, next_1
+   neg t3,t3
next_1:
    #set abs(nums[i])
-   add a1, zero, t2
-   addi sp, sp, -4
-   sw ra, 0(sp)
-   jal ra, set_abs
-   lw ra, 0(sp)
-   addi sp, sp 4
-   add t4, a1, zero
+   add t4, zero, t2
+   bgez t4, next_2
+   neg t4,t4
next_2: 
    #if(abs(nums[i])&lt;abs(closest_num)) then closest_num=nums[i]
    bge t4, t3, next_3 
    add t1, zero, t2
    j loop_decrement
    #if(abs(nums[i])==abs(closest_num)) then continue
next_3: bne t4, t3, loop_decrement
        #if(nums[i]&gt;closest_num) then closest_num=nums[i]
        ble t2, t1, loop_decrement 
        add t1, zero, t2
loop_decrement:
    addi s3, s3, 4
    addi t0, t0, -1 
    bne t0,zero, loop
    ret
set_abs:
-   bgez a1, return
-    neg a1,a1
return:
-   ret
printResult:
    mv a0, t1 #set the system call use data
    li a7, 1  #set the system call number
    ecall
    
    la a0, str #set the system call use data
    li a7, 4  #set the system call number
    ecall
    ret
</div></div></code></pre>
      </div><p class="part" data-startline="487" data-endline="487" data-position="16293" data-size="0"><span data-position="16293" data-size="387">In the second version, I modified the first version where the process of setting the absolute value would jump to another set_abs function, and changed it to directly setting the absolute value within the findNonMinOrMax function. After this modification, we can also observe that the number of execution cycles decreased from 675 to 367, indicating a very effective optimization process.</span></p><h3 class="part" data-startline="488" data-endline="488" id="Analyze" data-id="Analyze"><a class="anchor hidden-xs" href="#Analyze" title="Analyze" smoothhashscroll="" data-size="7" data-position="16686"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="16686" data-size="7">Analyze</span></h3><hr><p class="part" data-startline="490" data-endline="490" data-position="16698" data-size="0"><span data-position="16698" data-size="638">After placing my second version of the assembly program into Ripes and successfully compiling it, I noticed that there were additional instructions that I hadn’t originally written. The reason for this is that my assembly program used some pseudo-instructions, which are created to make writing programs more convenient for programmers. In reality, the machine cannot execute pseudo-instructions. After being translated by the assembler, these pseudo-instructions are converted into equivalent assembly instructions. Additionally, I also noticed that the register names were converted from ABI names to the actual RISC-V register numbers.</span></p><h3 class="part" data-startline="491" data-endline="491" id="Pseudo-instruction" data-id="Pseudo-instruction"><a class="anchor hidden-xs" href="#Pseudo-instruction" title="Pseudo-instruction" smoothhashscroll="" data-size="18" data-position="17341"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="17341" data-size="18">Pseudo instruction</span></h3><div class="part code-block-wrapper" data-startline="492" data-endline="550" data-infoprefix-length="1" data-position="17360">
        <pre><code class="c hljs raw">00000000 &lt;main&gt;:
    0:         00300493        addi x9 x0 3
    4:         10000917        auipc x18 0x10000
    8:         03490913        addi x18 x18 52
    c:         10000997        auipc x19 0x10000
    10:        ff498993        addi x19 x19 -12

00000014 &lt;test&gt;:
    14:        01c000ef        jal x1 28 &lt;findNonMinOrMax&gt;
    18:        064000ef        jal x1 100 &lt;printResult&gt;
    1c:        00490913        addi x18 x18 4
    20:        fff48493        addi x9 x9 -1
    24:        fe0498e3        bne x9 x0 -16 &lt;test&gt;
    28:        00a00893        addi x17 x0 10
    2c:        00000073        ecall

00000030 &lt;findNonMinOrMax&gt;:
    30:        00092283        lw x5 0 x18
    34:        0009a303        lw x6 0 x19

00000038 &lt;loop&gt;:
    38:        0009a383        lw x7 0 x19
    3c:        00600e33        add x28 x0 x6
    40:        000e5463        bge x28 x0 8 &lt;next_1&gt;
    44:        41c00e33        sub x28 x0 x28

00000048 &lt;next_1&gt;:
    48:        00700eb3        add x29 x0 x7
    4c:        000ed463        bge x29 x0 8 &lt;next_2&gt;
    50:        41d00eb3        sub x29 x0 x29

00000054 &lt;next_2&gt;:
    54:        01ced663        bge x29 x28 12 &lt;next_3&gt;
    58:        00700333        add x6 x0 x7
    5c:        0100006f        jal x0 16 &lt;loop_decrement&gt;

00000060 &lt;next_3&gt;:
    60:        01ce9663        bne x29 x28 12 &lt;loop_decrement&gt;
    64:        00735463        bge x6 x7 8 &lt;loop_decrement&gt;
    68:        00700333        add x6 x0 x7

0000006c &lt;loop_decrement&gt;:
    6c:        00498993        addi x19 x19 4
    70:        fff28293        addi x5 x5 -1
    74:        fc0292e3        bne x5 x0 -60 &lt;loop&gt;
    78:        00008067        jalr x0 x1 0

0000007c &lt;printResult&gt;:
    7c:        00030513        addi x10 x6 0
    80:        00100893        addi x17 x0 1
    84:        00000073        ecall
    88:        10000517        auipc x10 0x10000
    8c:        fbc50513        addi x10 x10 -68
    90:        00400893        addi x17 x0 4
    94:        00000073        ecall
    98:        00008067        jalr x0 x1 0

</code></pre>
      </div><h3 class="part" data-startline="551" data-endline="551" id="5-stage-pipelined-processor" data-id="5-stage-pipelined-processor"><a class="anchor hidden-xs" href="#5-stage-pipelined-processor" title="5-stage-pipelined-processor" smoothhashscroll="" data-size="27" data-position="19425"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="19425" data-size="27">5-stage pipelined processor</span></h3><p class="part" data-startline="552" data-endline="552" data-position="19453" data-size="0"><span data-position="19453" data-size="143">I chose to use Ripes’ 5-stage processor, which includes a hazard detection unit and a forwarding unit. The architecture diagram is shown below:</span></p><p class="part" data-startline="554" data-endline="554" data-position="19598" data-size="0"><img src="https://hackmd.io/_uploads/SktJaBiJJg.png" alt="3" class="offline-handled error-handled" data-position="19598" data-size="47" loading="lazy"></p><p class="part" data-startline="556" data-endline="561" data-position="19647" data-size="0"><span data-position="19647" data-size="63">The so-called 5-stage pipeline refers to the following stages:：</span><br>
<span data-position="19715" data-size="25">1. IF (Instruction Fetch)</span><br>
<span data-position="19745" data-size="26">2. ID (Instruction Decode)</span><br>
<span data-position="19776" data-size="17">3. EX (Execution)</span><br>
<span data-position="19798" data-size="22">4. MEM (Memory Access)</span><br>
<span data-position="19825" data-size="18">5. WB (Write Back)</span></p><p class="part" data-startline="563" data-endline="563" data-position="19845" data-size="0"><span data-position="19845" data-size="316">The key to allowing instructions to be executed in concurrent in a pipeline lies in the rectangular bars that separate each stage, also known as pipeline registers. These registers store the data and control signals for each stage, ensuring that the correct data and control signals are available for the next stage.</span></p><p class="part" data-startline="565" data-endline="565" data-position="20163" data-size="0"><span data-position="20163" data-size="192">In the RISC-V ISA, machine code formats are divided into R, I, B, U, J, and S types. Below, I will introduce each instruction format in order and explain how they are executed in the pipeline.</span></p><h4 class="part" data-startline="567" data-endline="567" id="R-type-format-amp-data-flow" data-id="R-type-format-amp-data-flow"><a class="anchor hidden-xs" href="#R-type-format-amp-data-flow" title="R-type-format-amp-data-flow" smoothhashscroll="" data-size="25" data-position="20362"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="20362" data-size="25">R-type format &amp; data flow</span></h4><p class="part" data-startline="568" data-endline="568" data-position="20388" data-size="0"><span data-position="20388" data-size="211">In RISC-V assembly language, R-type instructions have two source registers and one destination register, which all follow this format. Examples include add, sub, and so on. The machine code format is as follows:</span></p><p class="part" data-startline="570" data-endline="594" data-position="20601" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{funct7}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs2}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs1}
\mid
\overbrace{
\underbrace{000}_\text{3 bits}
}^\text{funct3}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rd}
\mid
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="596" data-endline="596" data-position="21017" data-size="0"><span data-position="21017" data-size="91">Taking add x29, x0, x7 as an example for conversion into machine code, we can refer to the </span><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="21109" data-size="28">2024 RISC-V spec. (page 554)</span></a><span data-position="21210" data-size="370"> and know that for the add instruction, funct7 = 0000000, funct3 = 000, and opcode = 0110011. The source register rs1 is x0, which is converted into binary as 00000, and rs2 is x7, which is converted into binary as 00111. The destination register rd is x29, which is converted into binary as 11101. Finally, combining this information results in the 32-bit machine code:</span></p><p class="part" data-startline="598" data-endline="608" data-position="21582" data-size="0"><span class="mathjax raw">\begin{equation}\overbrace{0000000}^\text{funct7}
\mid
\overbrace{00111}^\text{rs2}
\mid
\overbrace{00000}^\text{rs1}
\mid
\overbrace{000}^\text{funct3}
\mid
\overbrace{11101}^\text{rd}
\mid
\overbrace{0110011}^\text{opcode}\end{equation}</span></p><p class="part" data-startline="610" data-endline="610" data-position="21822" data-size="0"><span data-position="21822" data-size="176">Finally, let’s use Ripes to see how the add instruction is executed in the pipeline. Using the code above as an example, here is an excerpt of the relevant portion of the code:</span></p><div class="part code-block-wrapper" data-startline="611" data-endline="616" data-infoprefix-length="1" data-position="21999">
        <pre><code class="c hljs raw">00000048 &lt;next_1&gt;:
    48:        00700eb3        add x29 x0 x7
    4c:        000ed463        bge x29 x0 8 &lt;next_2&gt;
    50:        41d00eb3        sub x29 x0 x29
</code></pre>
      </div><ol class="part" data-startline="617" data-endline="620" data-position="22171" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="617" data-endline="620" data-position="22174" data-size="0"><span data-position="22174" data-size="28">IF stage (Instruction Fetch)</span><br>
<img src="https://hackmd.io/_uploads/ryobaHs1Je.png" alt="4" width="50%" class="offline-handled error-handled" data-position="22203" data-size="53" loading="lazy"></li>
</ol><ul class="part" data-startline="621" data-endline="625" data-position="22259" data-size="0">
<li class="raw" data-startline="621" data-endline="621" data-position="22261" data-size="0"><span data-position="22261" data-size="170">The Program Counter outputs the instruction address 0x00000048, and then this address enters the adder, which automatically adds 4. The Program Counter stores 0x0000004c.</span></li>
<li class="raw" data-startline="622" data-endline="622" data-position="22434" data-size="0"><span data-position="22434" data-size="157">The Instruction Memory receives the input address 0x00000048 and retrieves the corresponding instruction from that address, with the output being 0x00700eb3.</span></li>
<li class="raw" data-startline="623" data-endline="623" data-position="22594" data-size="0"><span data-position="22594" data-size="195">The Compressed Decoder receives the instruction 0x00700eb3 and checks whether it is a compressed instruction. Since the instruction is not a compressed instruction, the output remains 0x00700eb3.</span></li>
<li class="raw" data-startline="624" data-endline="625" data-position="22792" data-size="0"><span data-position="22792" data-size="59">Finally, the relevant data is stored in the IF/ID register.</span></li>
</ul><ol start="2" class="part" data-startline="626" data-endline="629" data-position="22853" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="626" data-endline="629" data-position="22856" data-size="0"><span data-position="22856" data-size="30">ID stage (Instruction Decoder)</span><br>
<img src="https://hackmd.io/_uploads/ByPQpHo1kx.png" alt="5" width="50%" class="offline-handled error-handled" data-position="22887" data-size="53" loading="lazy"><span data-position="22940" data-size="1"> </span><img src="https://hackmd.io/_uploads/HJcQaHjyJg.png" alt="6" width="38%" class="offline-handled error-handled" data-position="22941" data-size="53" loading="lazy"></li>
</ol><ul class="part" data-startline="630" data-endline="639" data-position="22997" data-size="0">
<li class="raw" data-startline="630" data-endline="635" data-position="22999" data-size="0">
<p data-position="22997" data-size="0"><span data-position="22999" data-size="74">The Decoder receives the instruction 0x00700eb3 and performs the decoding.</span></p>
<ul data-position="23083" data-size="0">
<li class="raw" data-startline="632" data-endline="632" data-position="23086" data-size="0"><span data-position="23086" data-size="110">opcode (0110011), funct3 (000), and funct7 (0000000) are decoded, indicating that the instruction type is add.</span></li>
<li class="raw" data-startline="633" data-endline="633" data-position="23204" data-size="0"><span data-position="23204" data-size="64">R1 idx is decoded as 0x00, which corresponds to the x0 register.</span></li>
<li class="raw" data-startline="634" data-endline="634" data-position="23276" data-size="0"><span data-position="23276" data-size="64">R2 idx is decoded as 0x07, which corresponds to the x7 register.</span></li>
<li class="raw" data-startline="635" data-endline="635" data-position="23348" data-size="0"><span data-position="23348" data-size="65">Wr idx is decoded as 0x1d, which corresponds to the x29 register.</span></li>
</ul>
</li>
<li class="raw" data-startline="636" data-endline="636" data-position="23416" data-size="0">
<p data-position="23414" data-size="0"><span data-position="23416" data-size="249">The Immediate Generate Unit, based on the input type being R-type, finds that there is no available immediate field, so it outputs the default value 0xdeadbeef. For more on why the default is 0xdeadbeef, you can find discussions on the topic in the </span><a href="https://www.reddit.com/r/RISCV/comments/1fkkxm3/does_deadbeef_has_a_meaning_in_spike/?rdt=39734" target="_blank" rel="noopener ugc nofollow"><span data-position="23666" data-size="12">Reddit forum</span></a><span data-position="23777" data-size="1">.</span></p>
</li>
<li class="raw" data-startline="637" data-endline="637" data-position="23780" data-size="0">
<p data-position="23778" data-size="0"><span data-position="23780" data-size="137">The Register Files, based on the input R1 idx, find the corresponding x0 register and output the content of the x0 register (0x00000000).</span></p>
</li>
<li class="raw" data-startline="638" data-endline="638" data-position="23920" data-size="0">
<p data-position="23918" data-size="0"><span data-position="23920" data-size="137">The Register Files, based on the input R2 idx, find the corresponding x7 register and output the content of the x7 register (0xfffffffc).</span></p>
</li>
<li class="raw" data-startline="639" data-endline="639" data-position="24060" data-size="0">
<p data-position="24058" data-size="0"><span data-position="24060" data-size="59">Finally, the relevant data is stored in the ID/EX register.</span></p>
</li>
</ul><ol start="3" class="part" data-startline="640" data-endline="642" data-position="24120" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="640" data-endline="642" data-position="24123" data-size="0"><span data-position="24123" data-size="20">EX stage (Execution)</span><br>
<img src="https://hackmd.io/_uploads/H1VUarsyJe.png" alt="7" width="50%" class="offline-handled error-handled" data-position="24144" data-size="53" loading="lazy"></li>
</ol><ul class="part" data-startline="643" data-endline="647" data-position="24199" data-size="0">
<li class="raw" data-startline="643" data-endline="643" data-position="24201" data-size="0"><span data-position="24201" data-size="539">The value of rs1 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs1 (0x00000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the program counter value, only B-type, J-type, and auipc instructions will select the upper path. Therefore, the second-level 2x1 multiplexer passes the output of the first-level 3x1 multiplexer (0x00000000) to Op1.</span></li>
<li class="raw" data-startline="644" data-endline="644" data-position="24743" data-size="0"><span data-position="24743" data-size="503">The value of rs2 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs2 (0xfffffffc). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the immediate value, since R-type instructions operate using Rs1 and Rs2, the second-level 2x1 multiplexer passes the output of the first-level 3x1 multiplexer (0xfffffffc) to Op2.</span></li>
<li class="raw" data-startline="645" data-endline="645" data-position="25249" data-size="0"><span data-position="25249" data-size="85">Since the instruction is not a branch type, the Branch unit will not take any action.</span></li>
<li class="raw" data-startline="646" data-endline="646" data-position="25337" data-size="0"><span data-position="25337" data-size="133">The ALU performs addition based on the control signals, adding 0x00000000 to 0xfffffffc and calculating the result Res as 0xfffffffc.</span></li>
<li class="raw" data-startline="647" data-endline="647" data-position="25473" data-size="0"><span data-position="25473" data-size="60">Finally, the relevant data is stored in the EX/MEM register.</span></li>
</ul><ol start="4" class="part" data-startline="648" data-endline="650" data-position="25534" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="648" data-endline="650" data-position="25537" data-size="0"><span data-position="25537" data-size="25">MEM stage (Memory Access)</span><br>
<img src="https://hackmd.io/_uploads/S1Uw6Bok1g.png" alt="8" width="50%" class="offline-handled error-handled" data-position="25563" data-size="53" loading="lazy"></li>
</ol><ul class="part" data-startline="651" data-endline="655" data-position="25618" data-size="0">
<li class="raw" data-startline="651" data-endline="651" data-position="25620" data-size="0">
<p data-position="25618" data-size="0"><span data-position="25620" data-size="189">Although the value of rs2 is input to Data in and Res is input to Addr., under an R-type instruction, the control signal for Memory write is 0, indicating that no data is written to Memory.</span></p>
</li>
<li class="raw" data-startline="652" data-endline="654" data-position="25812" data-size="0">
<p data-position="25810" data-size="0"><span data-position="25812" data-size="292">Based on the address pointed to by Res, we can observe the state of the Memory. The address used is only up to 0x00000098, and addresses after that have not been used, meaning that 0xfffffffc has not yet been allocated any data. Therefore, the output at that address (Read Out) is 0x00000000.</span><br>
<img src="https://hackmd.io/_uploads/SJodpSiykx.png" alt="9" class="offline-handled error-handled" data-position="26105" data-size="47" loading="lazy"></p>
</li>
<li class="raw" data-startline="655" data-endline="655" data-position="26156" data-size="0">
<p data-position="26154" data-size="0"><span data-position="26156" data-size="60">Finally, the relevant data is stored in the MEM/WB register.</span></p>
</li>
</ul><ol start="5" class="part" data-startline="656" data-endline="659" data-position="26217" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="656" data-endline="659" data-position="26220" data-size="0"><span data-position="26220" data-size="21">WB stage (Write back)</span><br>
<img src="https://hackmd.io/_uploads/r1-9aHikyg.png" alt="10" width="30%" class="offline-handled error-handled" data-position="26242" data-size="54" loading="lazy"><span data-position="26296" data-size="1"> </span><img src="https://hackmd.io/_uploads/S1E5pBiJJl.png" alt="11" width="25%" class="offline-handled error-handled" data-position="26297" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="660" data-endline="661" data-position="26354" data-size="0">
<li class="raw" data-startline="660" data-endline="661" data-position="26356" data-size="0"><span data-position="26356" data-size="414">The 3x1 multiplexer is composed of the values from the next program counter, Res, and the value read from memory. Since the instruction type is R-type, the multiplexer selector will choose Res (0xfffffffc) to pass through. Res will then be input into Wr Data, while the rd value stored in the MEM/WB register (0x1c) will be input into Wr idx. Finally, 0xfffffffc will be written to the x29 register (ABI name: t4).</span></li>
</ul><p class="part" data-startline="662" data-endline="663" data-position="26772" data-size="0"><span data-position="26772" data-size="66">After completing all stages, the registers are updated as follows:</span><br>
<img src="https://hackmd.io/_uploads/Byhi6HskJx.png" alt="12" width="40%" class="offline-handled error-handled" data-position="26839" data-size="54" loading="lazy"></p><p class="part" data-startline="665" data-endline="665" data-position="26895" data-size="0"><span data-position="26895" data-size="71">We can see that 0xfffffffc was successfully written to the t4 register.</span></p><h4 class="part" data-startline="667" data-endline="667" id="I-type-format-amp-data-flow" data-id="I-type-format-amp-data-flow"><a class="anchor hidden-xs" href="#I-type-format-amp-data-flow" title="I-type-format-amp-data-flow" smoothhashscroll="" data-size="25" data-position="26973"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="26973" data-size="25">I-type format &amp; data flow</span></h4><p class="part" data-startline="668" data-endline="668" data-position="26999" data-size="0"><span data-position="26999" data-size="213">In RISC-V assembly language, I-type instructions mainly consist of load instructions and immediate instructions (except for lui and auipc), such as lw, addi, subi, and so on. The machine code format is as follows:</span></p><p class="part" data-startline="670" data-endline="690" data-position="27214" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
\underbrace{000000000000}_\text{12 bits}
}^\text{immediate}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs1}
\mid
\overbrace{
\underbrace{000}_\text{3 bits}
}^\text{funct3}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rd}
\mid
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="692" data-endline="692" data-position="27576" data-size="0"><span data-position="27576" data-size="90">Taking addi x9, x0, 3 as an example for conversion into machine code, we can refer to the </span><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="27667" data-size="28">2024 RISC-V spec. (page 554)</span></a><span data-position="27768" data-size="345"> and find that for the addi instruction, funct3 = 000 and opcode = 0010011. The source register rs1 is x0, which converts to binary as 00000, and the destination register rd is x9, which converts to binary as 01001. The immediate value 3 converts to binary as 000000000011. Finally, combining this information results in the 32-bit machine code:</span></p><p class="part" data-startline="694" data-endline="714" data-position="28115" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
000000000011
}^\text{immediate}
\mid
\overbrace{
00000
}^\text{rs1}
\mid
\overbrace{
000
}^\text{funct3}
\mid
\overbrace{
01001
}^\text{rd}
\mid
\overbrace{
0010011
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="717" data-endline="717" data-position="28342" data-size="0"><span data-position="28342" data-size="177">Finally, let’s use Ripes to see how the addi instruction is executed in the pipeline. Using the code above as an example, here is an excerpt of the relevant portion of the code:</span></p><div class="part code-block-wrapper" data-startline="718" data-endline="725" data-infoprefix-length="1" data-position="28520">
        <pre><code class="c hljs raw">00000000 &lt;main&gt;:
    0:        00300493        addi x9 x0 3
    4:        10000917        auipc x18 0x10000
    8:        03490913        addi x18 x18 52
    c:        10000997        auipc x19 0x10000
    10:       ff498993        addi x19 x19 -12
</code></pre>
      </div><ol class="part" data-startline="726" data-endline="728" data-position="28778" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="726" data-endline="728" data-position="28781" data-size="0"><span data-position="28781" data-size="28">IF stage (Instruction Fetch)</span><br>
<img src="https://hackmd.io/_uploads/B1C6aHokJg.png" alt="13" width="50%" class="offline-handled error-handled" data-position="28810" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="729" data-endline="732" data-position="28866" data-size="0">
<li class="raw" data-startline="729" data-endline="729" data-position="28868" data-size="0"><span data-position="28868" data-size="170">The Program Counter outputs the instruction address 0x00000000, and then this address enters the adder, which automatically adds 4. The Program Counter stores 0x00000004.</span></li>
<li class="raw" data-startline="730" data-endline="730" data-position="29041" data-size="0"><span data-position="29041" data-size="157">The Instruction Memory receives the input address 0x00000004 and retrieves the corresponding instruction from that address, with the output being 0x00300493.</span></li>
<li class="raw" data-startline="731" data-endline="731" data-position="29201" data-size="0"><span data-position="29201" data-size="195">The Compressed Decoder receives the instruction 0x00300493 and checks whether it is a compressed instruction. Since the instruction is not a compressed instruction, the output remains 0x00300493.</span></li>
<li class="raw" data-startline="732" data-endline="732" data-position="29399" data-size="0"><span data-position="29399" data-size="59">Finally, the relevant data is stored in the IF/ID register.</span></li>
</ul><ol start="2" class="part" data-startline="733" data-endline="735" data-position="29459" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="733" data-endline="735" data-position="29462" data-size="0"><span data-position="29462" data-size="30">ID stage (Instruction Decoder)</span><br>
<img src="https://hackmd.io/_uploads/HJvk0rjykg.png" alt="14" width="50%" class="offline-handled error-handled" data-position="29493" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="736" data-endline="744" data-position="29549" data-size="0">
<li class="raw" data-startline="736" data-endline="740" data-position="29551" data-size="0"><span data-position="29551" data-size="74">The Decoder receives the instruction 0x00300493 and performs the decoding.</span>
<ul data-position="29630" data-size="0">
<li class="raw" data-startline="737" data-endline="737" data-position="29632" data-size="0"><span data-position="29632" data-size="92">opcode (0010011) and funct3 (000) are decoded, indicating that the instruction type is addi.</span></li>
<li class="raw" data-startline="738" data-endline="738" data-position="29731" data-size="0"><span data-position="29731" data-size="64">R1 idx is decoded as 0x00, which corresponds to the x0 register.</span></li>
<li class="raw" data-startline="739" data-endline="739" data-position="29802" data-size="0"><span data-position="29802" data-size="233">R2 idx is decoded as 0x03, which corresponds to the x3 register (although I-type does not have an rs2 field, the corresponding position in the machine code format is the rightmost 5 bits of the immediate[11:0] field, which is 00011).</span></li>
<li class="raw" data-startline="740" data-endline="740" data-position="30042" data-size="0"><span data-position="30042" data-size="64">Wr idx is decoded as 0x09, which corresponds to the x9 register.</span></li>
</ul>
</li>
<li class="raw" data-startline="741" data-endline="741" data-position="30109" data-size="0"><span data-position="30109" data-size="162">The Immediate Generate Unit, based on the input type being I-type, reads immediate[11:0] and performs sign extension to 32 bits, with the output being 0x00000003.</span></li>
<li class="raw" data-startline="742" data-endline="742" data-position="30274" data-size="0"><span data-position="30274" data-size="137">The Register Files, based on the input R2 idx, find the corresponding x3 register and output the content of the x3 register (0x10000000).</span></li>
<li class="raw" data-startline="743" data-endline="744" data-position="30414" data-size="0"><span data-position="30414" data-size="59">Finally, the relevant data is stored in the ID/EX register.</span></li>
</ul><ol start="3" class="part" data-startline="745" data-endline="747" data-position="30475" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="745" data-endline="747" data-position="30478" data-size="0"><span data-position="30478" data-size="20">EX stage (Execution)</span><br>
<img src="https://hackmd.io/_uploads/S1W-CSo1Jx.png" alt="15" width="50%" class="offline-handled error-handled" data-position="30499" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="748" data-endline="752" data-position="30555" data-size="0">
<li class="raw" data-startline="748" data-endline="748" data-position="30557" data-size="0"><span data-position="30557" data-size="533">The value of rs1 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs1 (0x00000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the program counter value, since I-type instructions operate using rs1 and the immediate value, the second-level 2x1 multiplexer passes the output of the first-level 3x1 multiplexer (0x00000000) through to Op1.</span></li>
<li class="raw" data-startline="749" data-endline="749" data-position="31093" data-size="0"><span data-position="31093" data-size="502">The value of rs2 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs2 (0x10000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the immediate value, since I-type instructions operate using rs1 and the immediate value, the second-level 2x1 multiplexer passes the immediate output (0x00000003) through to Op2.</span></li>
<li class="raw" data-startline="750" data-endline="750" data-position="31598" data-size="0"><span data-position="31598" data-size="85">Since the instruction is not a branch type, the Branch unit will not take any action.</span></li>
<li class="raw" data-startline="751" data-endline="751" data-position="31686" data-size="0"><span data-position="31686" data-size="123">The ALU performs addition based on the control signals, adding 0x00000000 to 0x00000003, resulting in Res being 0x00000003.</span></li>
<li class="raw" data-startline="752" data-endline="752" data-position="31813" data-size="0"><span data-position="31813" data-size="60">Finally, the relevant data is stored in the EX/MEM register.</span></li>
</ul><ol start="4" class="part" data-startline="753" data-endline="755" data-position="31874" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="753" data-endline="755" data-position="31877" data-size="0"><span data-position="31877" data-size="25">MEM stage (Memory Access)</span><br>
<img src="https://hackmd.io/_uploads/HJfzABjkkx.png" alt="16" width="50%" class="offline-handled error-handled" data-position="31903" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="756" data-endline="760" data-position="31959" data-size="0">
<li class="raw" data-startline="756" data-endline="756" data-position="31961" data-size="0">
<p data-position="31959" data-size="0"><span data-position="31961" data-size="189">Although the value of rs2 is input to Data in and Res is input to Addr., under an S-type instruction, the control signal for memory write is 0, indicating that no data is written to Memory.</span></p>
</li>
<li class="raw" data-startline="757" data-endline="759" data-position="32153" data-size="0">
<p data-position="32151" data-size="0"><span data-position="32153" data-size="692">Based on the address pointed to by Res, we can observe the memory’s storage status. The address 0x00000003 corresponds to the address of Byte 3 at 0x00000000, while the other 3 bytes are taken from the address 0x00000004, meaning Byte 0, Byte 1, and Byte 2. Since RISC-V uses little-endian format, Byte 3 at 0x00000000 holds the least significant byte (LSB), and Byte 2 at 0x00000004 holds the most significant byte (MSB). Finally, the address outputs Read Out as 0x00091700. (Typically, memory accesses must follow alignment rules, where word accesses must be at addresses that are multiples of 4. However, this does not affect the pipeline as it doesn’t rely on the value read from memory.)</span><br>
<img src="https://hackmd.io/_uploads/rJimCroyJx.png" alt="17" class="offline-handled error-handled" data-position="32846" data-size="48" loading="lazy"></p>
</li>
<li class="raw" data-startline="760" data-endline="760" data-position="32898" data-size="0">
<p data-position="32896" data-size="0"><span data-position="32898" data-size="60">Finally, the relevant data is stored in the MEM/WB register.</span></p>
</li>
</ul><ol start="5" class="part" data-startline="761" data-endline="763" data-position="32959" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="761" data-endline="763" data-position="32962" data-size="0"><span data-position="32962" data-size="21">WB stage (Write back)</span><br>
<img src="https://hackmd.io/_uploads/HJZBRBs1kl.png" alt="18" width="30%" class="offline-handled error-handled" data-position="32984" data-size="54" loading="lazy"><span data-position="33038" data-size="1"> </span><img src="https://hackmd.io/_uploads/BkES0HskJe.png" alt="19" width="25%" class="offline-handled error-handled" data-position="33039" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="764" data-endline="765" data-position="33095" data-size="0">
<li class="raw" data-startline="764" data-endline="765" data-position="33097" data-size="0"><span data-position="33097" data-size="411">The 3x1 multiplexer is composed of the values from the next program counter, Res, and the value read from memory. Since the instruction type is I-type, the multiplexer selector will choose Res (0x00000003) to pass through. Res will then be input into Wr Data, and the rd value stored in the MEM/WB register (0x09) will be input into Wr idx. Finally, 0x00000003 will be written to the x9 register (ABI name: s1).</span></li>
</ul><p class="part" data-startline="766" data-endline="767" data-position="33510" data-size="0"><span data-position="33510" data-size="65">After completing all stages, the register updates are as follows:</span><br>
<img src="https://hackmd.io/_uploads/BkoLABi1Je.png" alt="20" width="40%" class="offline-handled error-handled" data-position="33576" data-size="54" loading="lazy"></p><p class="part" data-startline="769" data-endline="769" data-position="33632" data-size="0"><span data-position="33632" data-size="71">We can see that 0x00000003 was successfully written to the s1 register.</span></p><h4 class="part" data-startline="771" data-endline="771" id="S-type-format-amp-data-flow" data-id="S-type-format-amp-data-flow"><a class="anchor hidden-xs" href="#S-type-format-amp-data-flow" title="S-type-format-amp-data-flow" smoothhashscroll="" data-size="25" data-position="33710"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="33710" data-size="25">S-type format &amp; data flow</span></h4><p class="part" data-startline="772" data-endline="797" data-position="33736" data-size="0"><span data-position="33736" data-size="164">In RISC-V assembly language, S-type instructions are primarily related to Store Memory operations, such as sw, sb, and so on. The machine code format is as follows:</span><br>
<span class="mathjax raw">\begin{equation}
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{immediate[11:5]}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs2}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs1}
\mid
\overbrace{
\underbrace{000}_\text{3 bits}
}^\text{funct3}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{immediate[4:0]}
\mid
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="799" data-endline="799" data-position="34338" data-size="0"><span data-position="34338" data-size="89">Taking sw x5, 32(x7) as an example for conversion into machine code, we can refer to the </span><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="34428" data-size="28">2024 RISC-V spec. (page 554)</span></a><span data-position="34529" data-size="320"> and find that for the sw instruction, funct3 = 010 and opcode = 0100011. The source register rs1 is x7, which converts to binary as 00111, and rs2 is x5, which converts to binary as 00101. The immediate value 32 converts to binary as 000000100000. Finally, combining this information results in the 32-bit machine code:</span></p><p class="part" data-startline="801" data-endline="825" data-position="34851" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
0000001
}^\text{immediate[11:5]}
\mid
\overbrace{
00101
}^\text{rs2}
\mid
\overbrace{
00111
}^\text{rs1}
\mid
\overbrace{
010
}^\text{funct3}
\mid
\overbrace{
00000
}^\text{immediate[4:0]}
\mid
\overbrace{
0100011
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="827" data-endline="827" data-position="35126" data-size="0"><span data-position="35126" data-size="182">Finally, let’s use Ripes to see how the sw instruction is executed in the pipeline. Since the previous code did not include the sw instruction, a small example is created as follows:</span></p><div class="part code-block-wrapper" data-startline="828" data-endline="836" data-infoprefix-length="1" data-position="35309">
        <pre><code class="c hljs raw">00000000 &lt;main&gt;:
    0:        06400293        addi x5 x0 100
    4:        f9c00313        addi x6 x0 -100
    8:        fe628ce3        beq x5 x6 -8 &lt;main&gt;
    c:        00531313        slli x6 x6 5
    10:       0253a023        sw x5 32 x7

</code></pre>
      </div><ol class="part" data-startline="838" data-endline="840" data-position="35563" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="838" data-endline="840" data-position="35566" data-size="0"><span data-position="35566" data-size="28">IF stage (Instruction Fetch)</span><br>
<img src="https://hackmd.io/_uploads/Byv_0SiJJx.png" alt="21" width="50%" class="offline-handled error-handled" data-position="35595" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="841" data-endline="844" data-position="35651" data-size="0">
<li class="raw" data-startline="841" data-endline="841" data-position="35654" data-size="0"><span data-position="35654" data-size="170">The Program Counter outputs the instruction address 0x00000010, and then this address enters the adder, which automatically adds 4. The Program Counter stores 0x00000014.</span></li>
<li class="raw" data-startline="842" data-endline="842" data-position="35828" data-size="0"><span data-position="35828" data-size="157">The Instruction Memory receives the input address 0x00000010 and retrieves the corresponding instruction from that address, with the output being 0x0253a023.</span></li>
<li class="raw" data-startline="843" data-endline="843" data-position="35989" data-size="0"><span data-position="35989" data-size="195">The Compressed Decoder receives the instruction 0x0253a023 and checks whether it is a compressed instruction. Since the instruction is not a compressed instruction, the output remains 0x0253a023.</span></li>
<li class="raw" data-startline="844" data-endline="844" data-position="36188" data-size="0"><span data-position="36188" data-size="59">Finally, the relevant data is stored in the IF/ID register.</span></li>
</ul><ol start="2" class="part" data-startline="845" data-endline="848" data-position="36248" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="845" data-endline="848" data-position="36251" data-size="0"><span data-position="36251" data-size="30">ID stage (Instruction Decoder)</span><br>
<img src="https://hackmd.io/_uploads/SkKF0royyx.png" alt="22" width="50%" class="offline-handled error-handled" data-position="36282" data-size="54" loading="lazy"><img src="https://hackmd.io/_uploads/HkntRSo1kx.png" alt="23" width="36%" class="offline-handled error-handled" data-position="36336" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="849" data-endline="857" data-position="36393" data-size="0">
<li class="raw" data-startline="849" data-endline="853" data-position="36395" data-size="0"><span data-position="36395" data-size="74">The Decoder receives the instruction 0x0253a023 and performs the decoding.</span>
<ul data-position="36474" data-size="0">
<li class="raw" data-startline="850" data-endline="850" data-position="36476" data-size="0"><span data-position="36476" data-size="90">opcode (0100011) and funct3 (010) are decoded, indicating that the instruction type is sw.</span></li>
<li class="raw" data-startline="851" data-endline="851" data-position="36573" data-size="0"><span data-position="36573" data-size="64">R1 idx is decoded as 0x07, which corresponds to the x7 register.</span></li>
<li class="raw" data-startline="852" data-endline="852" data-position="36644" data-size="0"><span data-position="36644" data-size="64">R2 idx is decoded as 0x05, which corresponds to the x5 register.</span></li>
<li class="raw" data-startline="853" data-endline="853" data-position="36715" data-size="0"><span data-position="36715" data-size="207">Wr idx is decoded as 0x00, which corresponds to the x0 register (although S-type does not have an rd field, the corresponding position in the machine code format is the immediate[4:0] field, which is 00000).</span></li>
</ul>
</li>
<li class="raw" data-startline="854" data-endline="854" data-position="36925" data-size="0"><span data-position="36925" data-size="210">The Immediate Generate Unit, based on the input type being S-type, combines immediate[11:5] and immediate[4:0] into a 12-bit value, and then performs sign extension to 32 bits, with the output being 0x00000020.</span></li>
<li class="raw" data-startline="855" data-endline="855" data-position="37138" data-size="0"><span data-position="37138" data-size="137">The Register Files, based on the input R1 idx, find the corresponding x7 register and output the content of the x7 register (0x00000000).</span></li>
<li class="raw" data-startline="856" data-endline="856" data-position="37278" data-size="0"><span data-position="37278" data-size="137">The Register Files, based on the input R2 idx, find the corresponding x5 register and output the content of the x5 register (0x00000064).</span></li>
<li class="raw" data-startline="857" data-endline="857" data-position="37418" data-size="0"><span data-position="37418" data-size="59">Finally, the relevant data is stored in the ID/EX register.</span></li>
</ul><ol start="3" class="part" data-startline="858" data-endline="860" data-position="37478" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="858" data-endline="860" data-position="37481" data-size="0"><span data-position="37481" data-size="20">EX stage (Execution)</span><br>
<img src="https://hackmd.io/_uploads/Bkd2CSokke.png" alt="24" width="50%" class="offline-handled error-handled" data-position="37502" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="861" data-endline="865" data-position="37558" data-size="0">
<li class="raw" data-startline="861" data-endline="861" data-position="37560" data-size="0"><span data-position="37560" data-size="579">The value of rs1 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs1 (0x00000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the program counter value, because this is an S-type instruction that needs to calculate the memory address from rs1 and the immediate value, the second-level 2x1 multiplexer passes the output of the first-level 3x1 multiplexer (0x00000000) through to Op1.</span></li>
<li class="raw" data-startline="862" data-endline="862" data-position="38142" data-size="0"><span data-position="38142" data-size="553">The value of rs2 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs2 (0x00000064). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the program counter value, because this is an S-type instruction that needs to calculate the memory address from rs1 and the immediate value, the second-level 2x1 multiplexer passes the immediate value (0x00000020) through to Op2.</span></li>
<li class="raw" data-startline="863" data-endline="863" data-position="38698" data-size="0"><span data-position="38698" data-size="85">Since the instruction is not a branch type, the Branch unit will not take any action.</span></li>
<li class="raw" data-startline="864" data-endline="864" data-position="38786" data-size="0"><span data-position="38786" data-size="123">The ALU performs addition based on the control signals, adding 0x00000000 to 0x00000020, resulting in Res being 0x00000020.</span></li>
<li class="raw" data-startline="865" data-endline="865" data-position="38912" data-size="0"><span data-position="38912" data-size="60">Finally, the relevant data is stored in the EX/MEM register.</span></li>
</ul><ol start="4" class="part" data-startline="866" data-endline="868" data-position="38973" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="866" data-endline="868" data-position="38976" data-size="0"><span data-position="38976" data-size="25">MEM stage (Memory Access)</span><br>
<img src="https://hackmd.io/_uploads/ByjpABikye.png" alt="25" width="50%" class="offline-handled error-handled" data-position="39002" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="869" data-endline="875" data-position="39058" data-size="0">
<li class="raw" data-startline="869" data-endline="869" data-position="39060" data-size="0">
<p data-position="39058" data-size="0"><span data-position="39060" data-size="214">The value of rs2 (0x00000064) is input to Data in, and Res (0x00000020) is input to Addr.. In the case of an S-type instruction, the Memory write control signal is 1, indicating that data will be written to Memory.</span></p>
</li>
<li class="raw" data-startline="870" data-endline="872" data-position="39277" data-size="0">
<p data-position="39275" data-size="0"><span data-position="39277" data-size="244">Based on the address pointed to by Res, we can check the Memory’s current status. The address 0x00000020 has not yet been allocated data, and the write operation has not yet occurred at this point, so the address outputs Read Out as 0x00000000.</span><br>
<img src="https://hackmd.io/_uploads/SymkkUik1g.png" alt="26" class="offline-handled error-handled" data-position="39522" data-size="48" loading="lazy"></p>
</li>
<li class="raw" data-startline="873" data-endline="875" data-position="39574" data-size="0">
<p data-position="39572" data-size="0"><span data-position="39574" data-size="60">Finally, the relevant data is stored in the MEM/WB register.</span></p>
</li>
</ul><ol start="5" class="part" data-startline="876" data-endline="878" data-position="39637" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="876" data-endline="878" data-position="39640" data-size="0"><span data-position="39640" data-size="21">WB stage (Write back)</span><br>
<img src="https://hackmd.io/_uploads/rkDgkLiJ1x.png" alt="27" width="30%" class="offline-handled error-handled" data-position="39662" data-size="54" loading="lazy"><span data-position="39716" data-size="1"> </span><img src="https://hackmd.io/_uploads/rJqe1UikJl.png" alt="28" width="25%" class="offline-handled error-handled" data-position="39717" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="879" data-endline="880" data-position="39773" data-size="0">
<li class="raw" data-startline="879" data-endline="880" data-position="39775" data-size="0"><span data-position="39775" data-size="445">The 3x1 multiplexer is composed of the values from the next program counter, Res, and the value read from memory. Since the instruction type is S-type, the multiplexer selector will choose Res (0x00000020) to pass through. Res will then be input into Wr Data, and the rd value stored in the MEM/WB register (0x00) will be input into Wr idx. However, because the WrEn control signal is 0, the value cannot be successfully written to the register.</span></li>
</ul><p class="part" data-startline="881" data-endline="882" data-position="40222" data-size="0"><span data-position="40222" data-size="77">After the instruction completes all stages, the Memory is updated as follows:</span><br>
<img src="https://hackmd.io/_uploads/HkXfy8i1yg.png" alt="29" class="offline-handled error-handled" data-position="40300" data-size="48" loading="lazy"></p><p class="part" data-startline="884" data-endline="884" data-position="40350" data-size="0"><span data-position="40350" data-size="85">We can see that 0x00000064 was successfully written to the memory address 0x00000020.</span></p><h4 class="part" data-startline="885" data-endline="885" id="B-type-format-amp-data-flow" data-id="B-type-format-amp-data-flow"><a class="anchor hidden-xs" href="#B-type-format-amp-data-flow" title="B-type-format-amp-data-flow" smoothhashscroll="" data-size="25" data-position="40441"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="40441" data-size="25">B-type format &amp; data flow</span></h4><p class="part" data-startline="886" data-endline="886" data-position="40467" data-size="0"><span data-position="40467" data-size="158">In RISC-V assembly language, B-type instructions primarily involve branch operations, such as beq, bne, bge, and so on. The machine code format is as follows:</span></p><p class="part" data-startline="888" data-endline="921" data-position="40627" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
\underbrace{0}_\text{1 bit}
}^\text{immediate[12]}
\mid
\overbrace{
\underbrace{000000}_\text{6 bits}
}^\text{immediate[10:5]}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs2}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rs1}
\mid
\overbrace{
\underbrace{000}_\text{3 bits}
}^\text{funct3}
\mid
\overbrace{
\underbrace{0000}_\text{4 bits}
}^\text{immediate[4:1]}
\mid
\overbrace{
\underbrace{0}_\text{1 bit}
}^\text{immediate[11]}
\mid
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{opcode}
\end{equation}</span><br>
<span data-position="41197" data-size="349">We can see that in the machine code format, the position for immediate[0] is missing. The reason for this is that branch instructions are restricted to jump only to even addresses, so the actual immediate is 13 bits. After combining the machine code format into 12 bits, a 0 needs to be added on the right, which means that immediate[0] is always 0.</span></p><p class="part" data-startline="923" data-endline="926" data-position="41548" data-size="0"><span data-position="41548" data-size="53">Therefore, the jump range for branch instructions is:</span><br>
<span class="mathjax raw">\begin{equation}
-4096\sim4094 ~~ \text{bytes} 
\end{equation}</span></p><p class="part" data-startline="928" data-endline="928" data-position="41667" data-size="0"><span data-position="41667" data-size="91">Taking bne x5, x0, -60 as an example for conversion into machine code, we can refer to the </span><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="41759" data-size="28">2024 RISC-V spec. (page 554)</span></a><span data-position="41860" data-size="323"> and find that for the bne instruction, funct3 = 001 and opcode = 1100011. The source register rs1 is x5, which converts to binary as 00101, and rs2 is x0, which converts to binary as 00000. The immediate value -60 converts to binary as 1111111000100. Finally, combining this information results in the 32-bit machine code:</span></p><p class="part" data-startline="930" data-endline="962" data-position="42185" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
1
}^\text{imm[12]}
\mid
\overbrace{
111110
}^\text{imm[10:5]}
\mid
\overbrace{
00000
}^\text{rs2}
\mid
\overbrace{
00101
}^\text{rs1}
\mid
\overbrace{
001
}^\text{funct3}
\mid
\overbrace{
0010
}^\text{imm[4:1]}
\mid
\overbrace{
1
}^\text{imm[11]}
\mid
\overbrace{
1100011
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="964" data-endline="964" data-position="42518" data-size="0"><span data-position="42518" data-size="176">Finally, let’s use Ripes to see how the bne instruction is executed in the pipeline. Using the code above as an example, here is an excerpt of the relevant portion of the code:</span></p><div class="part code-block-wrapper" data-startline="965" data-endline="971" data-infoprefix-length="1" data-position="42695">
        <pre><code class="c hljs raw">0000006c &lt;loop_decrement&gt;:
    6c:        00498993        addi x19 x19 4
    70:        fff28293        addi x5 x5 -1
    74:        fc0292e3        bne x5 x0 -60 &lt;loop&gt;
    78:        00008067        jalr x0 x1 0
</code></pre>
      </div><ol class="part" data-startline="973" data-endline="975" data-position="42919" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="973" data-endline="975" data-position="42922" data-size="0"><span data-position="42922" data-size="28">IF stage (Instruction Fetch)</span><br>
<img src="https://hackmd.io/_uploads/B1dNJLsJyx.png" alt="30" width="50%" class="offline-handled error-handled" data-position="42951" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="976" data-endline="980" data-position="43007" data-size="0">
<li class="raw" data-startline="976" data-endline="976" data-position="43009" data-size="0"><span data-position="43009" data-size="170">The Program Counter outputs the instruction address 0x00000074, and then this address enters the adder, which automatically adds 4. The Program Counter stores 0x00000078.</span></li>
<li class="raw" data-startline="977" data-endline="977" data-position="43182" data-size="0"><span data-position="43182" data-size="157">The Instruction Memory receives the input address 0x00000074 and retrieves the corresponding instruction from that address, with the output being 0xfc0292e3.</span></li>
<li class="raw" data-startline="978" data-endline="978" data-position="43342" data-size="0"><span data-position="43342" data-size="195">The Compressed Decoder receives the instruction 0xfc0292e3 and checks whether it is a compressed instruction. Since the instruction is not a compressed instruction, the output remains 0xfc0292e3.</span></li>
<li class="raw" data-startline="979" data-endline="980" data-position="43540" data-size="0"><span data-position="43540" data-size="59">Finally, the relevant data is stored in the IF/ID register.</span></li>
</ul><ol start="2" class="part" data-startline="981" data-endline="985" data-position="43601" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="981" data-endline="985" data-position="43604" data-size="0"><span data-position="43604" data-size="30">ID stage (Instruction Decoder)</span><br>
<img src="https://hackmd.io/_uploads/By3Sk8jyyx.png" alt="31" width="50%" class="offline-handled error-handled" data-position="43635" data-size="54" loading="lazy"><span data-position="43689" data-size="1"> </span><img src="https://hackmd.io/_uploads/SykLkUsy1g.png" alt="32" width="37%" class="offline-handled error-handled" data-position="43690" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="986" data-endline="994" data-position="43748" data-size="0">
<li class="raw" data-startline="986" data-endline="990" data-position="43750" data-size="0"><span data-position="43750" data-size="74">The Decoder receives the instruction 0xfc0292e3 and performs the decoding.</span>
<ul data-position="43829" data-size="0">
<li class="raw" data-startline="987" data-endline="987" data-position="43831" data-size="0"><span data-position="43831" data-size="91">opcode (1100011) and funct3 (101) are decoded, indicating that the instruction type is bge.</span></li>
<li class="raw" data-startline="988" data-endline="988" data-position="43929" data-size="0"><span data-position="43929" data-size="64">R1 idx is decoded as 0x1d, which corresponds to the x5 register.</span></li>
<li class="raw" data-startline="989" data-endline="989" data-position="44000" data-size="0"><span data-position="44000" data-size="64">R2 idx is decoded as 0x1c, which corresponds to the x0 register.</span></li>
<li class="raw" data-startline="990" data-endline="990" data-position="44071" data-size="0"><span data-position="44071" data-size="244">Wr idx is decoded as 0x0c, which corresponds to the x5 register (although B-type does not have an rd field, the corresponding position in the machine code format is immediate[4:1] and immediate[11], combined into a 5-bit value, which is 00101).</span></li>
</ul>
</li>
<li class="raw" data-startline="991" data-endline="991" data-position="44318" data-size="0"><span data-position="44318" data-size="240">The Immediate Generate Unit, based on the input type being B-type, combines immediate[12], immediate[11], immediate[10:5], immediate[4:1], and 0 into a 13-bit value, then performs sign extension to 32 bits, with the output being 0xffffffc4.</span></li>
<li class="raw" data-startline="992" data-endline="992" data-position="44561" data-size="0"><span data-position="44561" data-size="137">The Register Files, based on the input R1 idx, find the corresponding x5 register and output the content of the x5 register (0x00000005).</span></li>
<li class="raw" data-startline="993" data-endline="993" data-position="44701" data-size="0"><span data-position="44701" data-size="137">The Register Files, based on the input R2 idx, find the corresponding x0 register and output the content of the x0 register (0x00000000).</span></li>
<li class="raw" data-startline="994" data-endline="994" data-position="44841" data-size="0"><span data-position="44841" data-size="59">Finally, the relevant data is stored in the ID/EX register.</span></li>
</ul><ol start="3" class="part" data-startline="995" data-endline="997" data-position="44901" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="995" data-endline="997" data-position="44904" data-size="0"><span data-position="44904" data-size="20">EX stage (Execution)</span><br>
<img src="https://hackmd.io/_uploads/Hk-dJIikJx.png" alt="33" width="50%" class="offline-handled error-handled" data-position="44925" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="998" data-endline="1012" data-position="44981" data-size="0">
<li class="raw" data-startline="998" data-endline="998" data-position="44983" data-size="0">
<p data-position="44981" data-size="0"><span data-position="44983" data-size="651">The value of rs1 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the previous instruction (addi x5, x5, -1) modified the x5 register, the forwarding function will be used here. Therefore, the first-level 3x1 multiplexer passes through the value from the previous instruction’s result (Res), generated in the EX stage, which is 0x00000004. In the second-level 2x1 multiplexer, composed of the output from the first-level 3x1 multiplexer and the program counter value, since the Branch Unit determines that a jump will occur, the second-level 2x1 multiplexer passes through the program counter value (0x00000074) into Op1.</span></p>
</li>
<li class="raw" data-startline="999" data-endline="999" data-position="45637" data-size="0">
<p data-position="45635" data-size="0"><span data-position="45637" data-size="495">The value of rs2 and the other two forwarding paths form the first-level 3x1 multiplexer, and since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs2 (0x00000000). In the second-level 2x1 multiplexer, composed of the output from the first-level 3x1 multiplexer and the program counter value, since the Branch Unit determines that a jump will occur, the second-level 2x1 multiplexer passes through the immediate value (0xffffffc4) into Op2.</span></p>
</li>
<li class="raw" data-startline="1000" data-endline="1000" data-position="46135" data-size="0">
<p data-position="46133" data-size="0"><span data-position="46135" data-size="65">The Branch Unit determines that the branch instruction will jump.</span></p>
</li>
<li class="raw" data-startline="1001" data-endline="1001" data-position="46203" data-size="0">
<p data-position="46201" data-size="0"><span data-position="46203" data-size="124">The ALU performs addition based on the control signals, adding 0x00000074 and 0xffffffc4, resulting in Res being 0x00000038.</span></p>
</li>
<li class="raw" data-startline="1002" data-endline="1004" data-position="46330" data-size="0">
<p data-position="46328" data-size="0"><span data-position="46330" data-size="106">After calculating the address, in the next cycle, the value of Res will be written to the Program Counter.</span><br>
<img src="https://hackmd.io/_uploads/Syc9y8oy1g.jpg" alt="34" class="offline-handled error-handled" data-position="46437" data-size="48" loading="lazy"></p>
</li>
<li class="raw" data-startline="1005" data-endline="1005" data-position="46489" data-size="0">
<p data-position="46487" data-size="0"><span data-position="46489" data-size="60">Finally, the relevant data is stored in the EX/MEM register.</span></p>
</li>
<li class="raw" data-startline="1006" data-endline="1012" data-position="46552" data-size="0">
<p data-position="46550" data-size="0"><span data-position="46552" data-size="164">We can see that when the branch instruction decides to jump, the two instructions immediately following the branch are cleared and replaced with nop (no operation).</span><br>
<img src="https://hackmd.io/_uploads/ryM2y8oJJx.png" alt="35" class="offline-handled error-handled" data-position="46717" data-size="48" loading="lazy"></p>
</li>
</ul><ol start="4" class="part" data-startline="1013" data-endline="1015" data-position="46771" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1013" data-endline="1015" data-position="46774" data-size="0"><span data-position="46774" data-size="25">MEM stage (Memory Access)</span><br>
<img src="https://hackmd.io/_uploads/ryegeUikye.png" alt="36" width="50%" class="offline-handled error-handled" data-position="46800" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1016" data-endline="1023" data-position="46856" data-size="0">
<li class="raw" data-startline="1016" data-endline="1016" data-position="46858" data-size="0">
<p data-position="46856" data-size="0"><span data-position="46858" data-size="218">The value of rs2 (0x00000000) is input into Data in, and Res (0x00000038) is input into Addr. However, under a B-type instruction, the Memory write control signal is 0, indicating that data cannot be written to Memory.</span></p>
</li>
<li class="raw" data-startline="1017" data-endline="1019" data-position="47079" data-size="0">
<p data-position="47077" data-size="0"><span data-position="47079" data-size="171">Based on the address pointed to by Res, we can check the memory status. The address 0x00000038 stores 0x0009a383, so the output from that address (Read Out) is 0x0009a383.</span><br>
<img src="https://hackmd.io/_uploads/rkSZeLjy1l.png" alt="37" class="offline-handled error-handled" data-position="47251" data-size="48" loading="lazy"></p>
</li>
<li class="raw" data-startline="1020" data-endline="1023" data-position="47303" data-size="0">
<p data-position="47301" data-size="0"><span data-position="47303" data-size="60">Finally, the relevant data is stored in the MEM/WB register.</span></p>
</li>
</ul><ol start="5" class="part" data-startline="1024" data-endline="1027" data-position="47367" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1024" data-endline="1027" data-position="47370" data-size="0"><span data-position="47370" data-size="21">WB stage (Write back)</span><br>
<img src="https://hackmd.io/_uploads/rkpzgIjJ1x.png" alt="38" width="30%" class="offline-handled error-handled" data-position="47392" data-size="54" loading="lazy"><span data-position="47446" data-size="1"> </span><img src="https://hackmd.io/_uploads/rJCMxUsyyg.png" alt="39" width="25%" class="offline-handled error-handled" data-position="47447" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1028" data-endline="1029" data-position="47504" data-size="0">
<li class="raw" data-startline="1028" data-endline="1029" data-position="47506" data-size="0"><span data-position="47506" data-size="445">The 3x1 multiplexer is composed of the values from the next program counter, Res, and the value read from memory. Since the instruction type is B-type, the multiplexer selector will choose Res (0x00000038) to pass through. Res will then be input into Wr Data, and the rd value stored in the MEM/WB register (0x05) will be input into Wr idx. However, because the WrEn control signal is 0, the value cannot be successfully written to the register.</span></li>
</ul><p class="part" data-startline="1030" data-endline="1031" data-position="47953" data-size="0"><span data-position="47953" data-size="65">After completing all stages, the register updates are as follows:</span><br>
<img src="https://hackmd.io/_uploads/HJ8VlLsyJe.png" alt="40" width="40%" class="offline-handled error-handled" data-position="48019" data-size="54" loading="lazy"></p><p class="part" data-startline="1033" data-endline="1033" data-position="48075" data-size="0"><span data-position="48075" data-size="239">Since the bne instruction is preceded by the addi x5, x5, -1 instruction, the x5 register will be modified from 0x00000005 to 0x00000004. The bne instruction does not change the value of the register, so the x5 register remains 0x00000004.</span></p><h4 class="part" data-startline="1037" data-endline="1037" id="U-type-format-amp-data-flow" data-id="U-type-format-amp-data-flow"><a class="anchor hidden-xs" href="#U-type-format-amp-data-flow" title="U-type-format-amp-data-flow" smoothhashscroll="" data-size="25" data-position="48323"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="48323" data-size="25">U-type format &amp; data flow</span></h4><p class="part" data-startline="1038" data-endline="1038" data-position="48349" data-size="0"><span data-position="48349" data-size="122">In RISC-V assembly language, there are only two U-type instructions: lui and auipc. The machine code format is as follows:</span></p><p class="part" data-startline="1040" data-endline="1052" data-position="48473" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
\underbrace{00000000000000000000}_\text{20 bits}
}^\text{immediate[31:12]}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rd}
\mid
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="1054" data-endline="1054" data-position="48723" data-size="0"><span data-position="48723" data-size="319">We can see that in the machine code format, the position for immediate[11:0] is missing. This is because the lui and auipc instructions operate on the upper 20 bits, meaning the actual immediate needs to be left-shifted by 12 bits. This indicates that immediate[11:0] consists of 12 zeros as a result of the left shift.</span></p><p class="part" data-startline="1056" data-endline="1056" data-position="49044" data-size="0"><span data-position="49044" data-size="93">Taking auipc x18, x10000 as an example for conversion into machine code, we can refer to the </span><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="49138" data-size="28">2024 RISC-V spec. (page 554)</span></a><span data-position="49239" data-size="262"> and find that the auipc opcode is 0010111. The destination register rd is x18, which converts to binary as 10010, and the immediate value 0x10000 converts to binary as 00010000000000000000. Finally, combining this information results in the 32-bit machine code:</span></p><p class="part" data-startline="1058" data-endline="1070" data-position="49503" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
00010000000000000000
}^\text{immediate[31:12]}
\mid
\overbrace{
10010
}^\text{rd}
\mid
\overbrace{
0010111
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="1072" data-endline="1072" data-position="49671" data-size="0"><span data-position="49671" data-size="178">Finally, let’s use Ripes to see how the auipc instruction is executed in the pipeline. Using the code above as an example, here is an excerpt of the relevant portion of the code:</span></p><div class="part code-block-wrapper" data-startline="1073" data-endline="1080" data-infoprefix-length="1" data-position="49850">
        <pre><code class="c hljs raw">00000000 &lt;main&gt;:
    0:        00300493        addi x9 x0 3
    4:        10000917        auipc x18 0x10000
    8:        03490913        addi x18 x18 52
    c:        10000997        auipc x19 0x10000
    10:       ff498993        addi x19 x19 -12
</code></pre>
      </div><ol class="part" data-startline="1082" data-endline="1084" data-position="50109" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1082" data-endline="1084" data-position="50112" data-size="0"><span data-position="50112" data-size="28">IF stage (Instruction Fetch)</span><br>
<img src="https://hackmd.io/_uploads/HkDQ-Ijkyx.png" alt="41" width="50%" class="offline-handled error-handled" data-position="50141" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1085" data-endline="1088" data-position="50197" data-size="0">
<li class="raw" data-startline="1085" data-endline="1085" data-position="50199" data-size="0"><span data-position="50199" data-size="170">The Program Counter outputs the instruction address 0x00000004, and then this address enters the adder, which automatically adds 4. The Program Counter stores 0x00000008.</span></li>
<li class="raw" data-startline="1086" data-endline="1086" data-position="50372" data-size="0"><span data-position="50372" data-size="157">The Instruction Memory receives the input address 0x00000004 and retrieves the corresponding instruction from that address, with the output being 0x10000917.</span></li>
<li class="raw" data-startline="1087" data-endline="1087" data-position="50532" data-size="0"><span data-position="50532" data-size="255">The Compressed Decoder receives the instruction 0x10000917 and checks whether it is a compressed instruction. Since the instruction is not a compressed instruction, the output remains 0x10000917.instruction，而該指令非 compression instruction，因此輸出仍為 0x10000917。</span></li>
<li class="raw" data-startline="1088" data-endline="1088" data-position="50790" data-size="0"><span data-position="50790" data-size="59">Finally, the relevant data is stored in the IF/ID register.</span></li>
</ul><ol start="2" class="part" data-startline="1089" data-endline="1092" data-position="50850" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1089" data-endline="1092" data-position="50853" data-size="0"><span data-position="50853" data-size="30">ID stage (Instruction Decoder)</span><br>
<img src="https://hackmd.io/_uploads/BJTVWLiyyx.png" alt="42" width="50%" class="offline-handled error-handled" data-position="50884" data-size="54" loading="lazy"><img src="https://hackmd.io/_uploads/HyfB-Lokkl.png" alt="43" width="37%" class="offline-handled error-handled" data-position="50938" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1093" data-endline="1101" data-position="50995" data-size="0">
<li class="raw" data-startline="1093" data-endline="1097" data-position="50997" data-size="0"><span data-position="50997" data-size="74">The Decoder receives the instruction 0x10000917 and performs the decoding.</span>
<ul data-position="51076" data-size="0">
<li class="raw" data-startline="1094" data-endline="1094" data-position="51078" data-size="0"><span data-position="51078" data-size="75">opcode (0010111) is decoded, indicating that the instruction type is auipc.</span></li>
<li class="raw" data-startline="1095" data-endline="1095" data-position="51160" data-size="0"><span data-position="51160" data-size="210">R1 idx is decoded as 0x00, which corresponds to the x0 register (although U-type does not have an rs1 field, the corresponding position in the machine code format is the immediate[20:16] field, which is 00000).</span></li>
<li class="raw" data-startline="1096" data-endline="1096" data-position="51377" data-size="0"><span data-position="51377" data-size="210">R2 idx is decoded as 0x00, which corresponds to the x0 register (although U-type does not have an rs2 field, the corresponding position in the machine code format is the immediate[25:21] field, which is 00000).</span></li>
<li class="raw" data-startline="1097" data-endline="1097" data-position="51594" data-size="0"><span data-position="51594" data-size="65">Wr idx is decoded as 0x12, which corresponds to the x18 register.</span></li>
</ul>
</li>
<li class="raw" data-startline="1098" data-endline="1098" data-position="51662" data-size="0"><span data-position="51662" data-size="141">The Immediate Generate Unit, based on the input type being U-type, left-shifts immediate[31:12] by 12 bits, with the output being 0x10000000.</span></li>
<li class="raw" data-startline="1099" data-endline="1099" data-position="51806" data-size="0"><span data-position="51806" data-size="137">The Register Files, based on the input R1 idx, find the corresponding x0 register and output the content of the x0 register (0x00000000).</span></li>
<li class="raw" data-startline="1100" data-endline="1101" data-position="51946" data-size="0"><span data-position="51946" data-size="137">The Register Files, based on the input R2 idx, find the corresponding x0 register and output the content of the x0 register (0x00000000).</span><br>
<span data-position="52085" data-size="60">-Finally, the relevant data is stored in the ID/EX register.</span></li>
</ul><ol start="3" class="part" data-startline="1102" data-endline="1104" data-position="52145" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1102" data-endline="1104" data-position="52148" data-size="0"><span data-position="52148" data-size="20">EX stage (Execution)</span><br>
<img src="https://hackmd.io/_uploads/HkEPZIjyJg.png" alt="44" width="50%" class="offline-handled error-handled" data-position="52169" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1105" data-endline="1109" data-position="52225" data-size="0">
<li class="raw" data-startline="1105" data-endline="1105" data-position="52227" data-size="0"><span data-position="52227" data-size="532">The value of rs1 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs1 (0x00000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the Program Counter value, since the auipc instruction adds the Program Counter value to the immediate constant, the second-level 2x1 multiplexer passes through the Program Counter value (0x00000004) into Op1.</span></li>
<li class="raw" data-startline="1106" data-endline="1106" data-position="52762" data-size="0"><span data-position="52762" data-size="526">The value of rs2 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs2 (0x00000064). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the Program Counter value, since the auipc instruction adds the Program Counter value to the immediate constant, the second-level 2x1 multiplexer passes through the immediate value (0x10000000) into Op2.</span></li>
<li class="raw" data-startline="1107" data-endline="1107" data-position="53291" data-size="0"><span data-position="53291" data-size="85">Since the instruction is not a branch type, the Branch unit will not take any action.</span></li>
<li class="raw" data-startline="1108" data-endline="1108" data-position="53379" data-size="0"><span data-position="53379" data-size="123">The ALU performs addition based on the control signals, adding 0x00000004 to 0x10000000, resulting in Res being 0x10000004.</span></li>
<li class="raw" data-startline="1109" data-endline="1109" data-position="53505" data-size="0"><span data-position="53505" data-size="60">Finally, the relevant data is stored in the EX/MEM register.</span></li>
</ul><ol start="4" class="part" data-startline="1110" data-endline="1112" data-position="53566" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1110" data-endline="1112" data-position="53569" data-size="0"><span data-position="53569" data-size="25">MEM stage (Memory Access)</span><br>
<img src="https://hackmd.io/_uploads/H1HObLiJ1g.png" alt="45" width="50%" class="offline-handled error-handled" data-position="53595" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1113" data-endline="1116" data-position="53651" data-size="0">
<li class="raw" data-startline="1113" data-endline="1113" data-position="53653" data-size="0"><span data-position="53653" data-size="218">The value of rs2 (0x00000000) is input into Data in, and Res (0x10000004) is input into Addr. However, under a U-type instruction, the Memory write control signal is 0, indicating that data cannot be written to Memory.</span></li>
<li class="raw" data-startline="1114" data-endline="1116" data-position="53874" data-size="0"><span data-position="53874" data-size="289">Based on the address pointed to by Res, we can check the memory status. The addresses used only go up to 0x00000098, and addresses beyond that have not been used, meaning that 0x10000004 has not yet been allocated any data. Therefore, the output from that address (Read Out) is 0x00000000.</span><br>
<img src="https://hackmd.io/_uploads/BkKFWLoJJg.png" alt="46" class="offline-handled error-handled" data-position="54164" data-size="48" loading="lazy"></li>
</ul><p class="part" data-startline="1117" data-endline="1117" data-position="54214" data-size="0"><span data-position="54214" data-size="60">Finally, the relevant data is stored in the MEM/WB register.</span></p><ol start="5" class="part" data-startline="1120" data-endline="1123" data-position="54277" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1120" data-endline="1123" data-position="54280" data-size="0"><span data-position="54280" data-size="21">WB stage (Write back)</span><br>
<img src="https://hackmd.io/_uploads/rks9-Likyg.png" alt="47" width="30%" class="offline-handled error-handled" data-position="54302" data-size="54" loading="lazy"><span data-position="54356" data-size="1"> </span><img src="https://hackmd.io/_uploads/SJlC5-Ui1kg.png" alt="48" width="25%" class="offline-handled error-handled" data-position="54357" data-size="55" loading="lazy"></li>
</ol><ul class="part" data-startline="1124" data-endline="1125" data-position="54415" data-size="0">
<li class="raw" data-startline="1124" data-endline="1125" data-position="54417" data-size="0"><span data-position="54417" data-size="412">The 3x1 multiplexer is composed of the values from the next program counter, Res, and the value read from memory. Since the instruction type is R-type, the multiplexer selector will choose Res (0x10000004) to pass through. Res will then be input into Wr Data, and the rd value stored in the MEM/WB register (0x12) will be input into Wr idx. Finally, 0x10000004 will be written to the x18 register (ABI name: s2).</span></li>
</ul><p class="part" data-startline="1126" data-endline="1127" data-position="54831" data-size="0"><span data-position="54831" data-size="65">After completing all stages, the register updates are as follows:</span><br>
<img src="https://hackmd.io/_uploads/r1N3WLskke.png" alt="49" width="40%" class="offline-handled error-handled" data-position="54897" data-size="54" loading="lazy"></p><p class="part" data-startline="1129" data-endline="1129" data-position="54953" data-size="0"><span data-position="54953" data-size="71">We can see that 0x10000004 was successfully written to the s2 register.</span></p><h4 class="part" data-startline="1132" data-endline="1132" id="J-type-format-amp-data-flow" data-id="J-type-format-amp-data-flow"><a class="anchor hidden-xs" href="#J-type-format-amp-data-flow" title="J-type-format-amp-data-flow" smoothhashscroll="" data-size="25" data-position="55032"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="55032" data-size="25">J-type format &amp; data flow</span></h4><p class="part" data-startline="1133" data-endline="1133" data-position="55058" data-size="0"><span data-position="55058" data-size="110">In RISC-V assembly language, there is only one J-type instruction: jal. The machine code format is as follows:</span></p><p class="part" data-startline="1135" data-endline="1159" data-position="55170" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
\underbrace{0}_\text{1 bit}
}^\text{immediate[20]}
\mid
\overbrace{
\underbrace{0000000000}_\text{10 bits}
}^\text{immediate[10:1]}
\mid
\overbrace{
\underbrace{0}_\text{1 bit}
}^\text{immediate[11]}
\mid
\overbrace{
\underbrace{00000000}_\text{8 bits}
}^\text{immediate[19:12]}
\mid
\overbrace{
\underbrace{00000}_\text{5 bits}
}^\text{rd}
\mid
\overbrace{
\underbrace{0000000}_\text{7 bits}
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="1161" data-endline="1161" data-position="55624" data-size="0"><span data-position="55624" data-size="86">Taking jal x1, 28 as an example for conversion into machine code, we can refer to the </span><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="55711" data-size="28">2024 RISC-V spec. (page 554)</span></a><span data-position="55812" data-size="254"> and find that the jal opcode is 1101111. The destination register rd is x1, which converts to binary as 00001, and the immediate value 28 converts to binary as 00000000000000011100. Finally, combining this information results in the 32-bit machine code:</span></p><p class="part" data-startline="1163" data-endline="1187" data-position="56068" data-size="0"><span class="mathjax raw">\begin{equation}
\overbrace{
0
}^\text{immediate[20]}
\mid
\overbrace{
0000011100
}^\text{immediate[10:1]}
\mid
\overbrace{
0
}^\text{immediate[11]}
\mid
\overbrace{
00000000
}^\text{immediate[19:12]}
\mid
\overbrace{
00001
}^\text{rd}
\mid
\overbrace{
1101111
}^\text{opcode}
\end{equation}</span></p><p class="part" data-startline="1189" data-endline="1189" data-position="56361" data-size="0"><span data-position="56361" data-size="176">Finally, let’s use Ripes to see how the jal instruction is executed in the pipeline. Using the code above as an example, here is an excerpt of the relevant portion of the code:</span></p><div class="part code-block-wrapper" data-startline="1190" data-endline="1199" data-infoprefix-length="1" data-position="56538">
        <pre><code class="c hljs raw">00000014 &lt;test&gt;:
    14:        01c000ef        jal x1 28 &lt;findNonMinOrMax&gt;
    18:        064000ef        jal x1 100 &lt;printResult&gt;
    1c:        00490913        addi x18 x18 4
    20:        fff48493        addi x9 x9 -1
    24:        fe0498e3        bne x9 x0 -16 &lt;test&gt;
    28:        00a00893        addi x17 x0 10
    2c:        00000073        ecall
</code></pre>
      </div><ol class="part" data-startline="1201" data-endline="1203" data-position="56906" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1201" data-endline="1203" data-position="56909" data-size="0"><span data-position="56909" data-size="28">IF stage (Instruction Fetch)</span><br>
<img src="https://hackmd.io/_uploads/HyHRbUoyyl.png" alt="50" width="50%" class="offline-handled error-handled" data-position="56938" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1204" data-endline="1207" data-position="56994" data-size="0">
<li class="raw" data-startline="1204" data-endline="1204" data-position="56996" data-size="0"><span data-position="56996" data-size="170">The Program Counter outputs the instruction address 0x00000014, and then this address enters the adder, which automatically adds 4. The Program Counter stores 0x00000018.</span></li>
<li class="raw" data-startline="1205" data-endline="1205" data-position="57169" data-size="0"><span data-position="57169" data-size="157">The Instruction Memory receives the input address 0x00000014 and retrieves the corresponding instruction from that address, with the output being 0x01c000ef.</span></li>
<li class="raw" data-startline="1206" data-endline="1206" data-position="57329" data-size="0"><span data-position="57329" data-size="195">The Compressed Decoder receives the instruction 0x01c000ef and checks whether it is a compressed instruction. Since the instruction is not a compressed instruction, the output remains 0x01c000ef.</span></li>
<li class="raw" data-startline="1207" data-endline="1207" data-position="57527" data-size="0"><span data-position="57527" data-size="59">Finally, the relevant data is stored in the IF/ID register.</span></li>
</ul><ol start="2" class="part" data-startline="1208" data-endline="1211" data-position="57587" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1208" data-endline="1211" data-position="57590" data-size="0"><span data-position="57590" data-size="30">ID stage (Instruction Decoder)</span><br>
<img src="https://hackmd.io/_uploads/SJZEfLs1yx.png" alt="51" width="50%" class="offline-handled error-handled" data-position="57621" data-size="54" loading="lazy"><img src="https://hackmd.io/_uploads/ByrNfUiJJl.png" alt="52" width="37%" class="offline-handled error-handled" data-position="57675" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1212" data-endline="1220" data-position="57732" data-size="0">
<li class="raw" data-startline="1212" data-endline="1216" data-position="57734" data-size="0"><span data-position="57734" data-size="74">The Decoder receives the instruction 0x01c000ef and performs the decoding.</span>
<ul data-position="57813" data-size="0">
<li class="raw" data-startline="1213" data-endline="1213" data-position="57815" data-size="0"><span data-position="57815" data-size="73">opcode (1101111) is decoded, indicating that the instruction type is jal.</span></li>
<li class="raw" data-startline="1214" data-endline="1214" data-position="57895" data-size="0"><span data-position="57895" data-size="229">R1 idx is decoded as 0x00, which corresponds to the x0 register (although J-type does not have an rs2 field, the corresponding position in the machine code format is the left 5 bits of the immediate[19:12] field, which is 00000).</span></li>
<li class="raw" data-startline="1215" data-endline="1215" data-position="58131" data-size="0"><span data-position="58131" data-size="279">R2 idx is decoded as 0x1c, which corresponds to the x28 register (although J-type does not have an rs1 field, the corresponding position in the machine code format is the right 4 bits of immediate[10:1] combined with the immediate[11] field, resulting in a 5-bit value of 11100).</span></li>
<li class="raw" data-startline="1216" data-endline="1216" data-position="58417" data-size="0"><span data-position="58417" data-size="64">Wr idx is decoded as 0x01, which corresponds to the x1 register.</span></li>
</ul>
</li>
<li class="raw" data-startline="1217" data-endline="1217" data-position="58485" data-size="0"><span data-position="58485" data-size="242">The Immediate Generate Unit, based on the input type being J-type, combines immediate[20], immediate[19:12], immediate[11], immediate[10:1], and 0 into a 21-bit value, then performs sign extension to 32 bits, with the output being 0x0000001c.</span></li>
<li class="raw" data-startline="1218" data-endline="1218" data-position="58730" data-size="0"><span data-position="58730" data-size="137">The Register Files, based on the input R1 idx, find the corresponding x0 register and output the content of the x0 register (0x00000000).</span></li>
<li class="raw" data-startline="1219" data-endline="1219" data-position="58870" data-size="0"><span data-position="58870" data-size="139">The Register Files, based on the input R2 idx, find the corresponding x28 register and output the content of the x28 register (0x00000000).</span></li>
<li class="raw" data-startline="1220" data-endline="1220" data-position="59012" data-size="0"><span data-position="59012" data-size="59">Finally, the relevant data is stored in the ID/EX register.</span></li>
</ul><ol start="3" class="part" data-startline="1221" data-endline="1223" data-position="59072" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1221" data-endline="1223" data-position="59075" data-size="0"><span data-position="59075" data-size="20">EX stage (Execution)</span><br>
<img src="https://hackmd.io/_uploads/H16rMUsJye.png" alt="53" width="50%" class="offline-handled error-handled" data-position="59096" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1224" data-endline="1234" data-position="59152" data-size="0">
<li class="raw" data-startline="1224" data-endline="1224" data-position="59154" data-size="0">
<p data-position="59152" data-size="0"><span data-position="59154" data-size="530">The value of rs1 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs1 (0x00000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the Program Counter value, since the jal instruction adds the Program Counter value to the immediate constant, the second-level 2x1 multiplexer passes through the Program Counter value (0x00000014) into Op1.</span></p>
</li>
<li class="raw" data-startline="1225" data-endline="1225" data-position="59687" data-size="0">
<p data-position="59685" data-size="0"><span data-position="59687" data-size="524">The value of rs2 and the other two forwarding paths form the first-level 3x1 multiplexer. Since the forwarding function is not used here, the first-level 3x1 multiplexer passes through the value of rs2 (0x00000000). In the second-level 2x1 multiplexer, which consists of the output from the first-level 3x1 multiplexer and the Program Counter value, since the jal instruction adds the Program Counter value to the immediate constant, the second-level 2x1 multiplexer passes through the immediate value (0x0000001c) into Op2.</span></p>
</li>
<li class="raw" data-startline="1226" data-endline="1226" data-position="60214" data-size="0">
<p data-position="60212" data-size="0"><span data-position="60214" data-size="124">The ALU performs addition based on the control signals, adding 0x00000014 and 0x0000001c, resulting in Res being 0x00000030.</span></p>
</li>
<li class="raw" data-startline="1227" data-endline="1229" data-position="60341" data-size="0">
<p data-position="60339" data-size="0"><span data-position="60341" data-size="106">After calculating the address, in the next cycle, the value of Res will be written to the Program Counter.</span><br>
<img src="https://hackmd.io/_uploads/H1Wsf8okkx.jpg" alt="54" class="offline-handled error-handled" data-position="60448" data-size="48" loading="lazy"></p>
</li>
<li class="raw" data-startline="1230" data-endline="1230" data-position="60500" data-size="0">
<p data-position="60498" data-size="0"><span data-position="60500" data-size="60">Finally, the relevant data is stored in the EX/MEM register.</span></p>
</li>
<li class="raw" data-startline="1231" data-endline="1234" data-position="60563" data-size="0">
<p data-position="60561" data-size="0"><span data-position="60563" data-size="148">We can see that when the jal instruction jumps, the two instructions immediately following the jal are cleared and replaced with nop (no operation).</span><br>
<img src="https://hackmd.io/_uploads/BJnhM8jJkg.png" alt="55" class="offline-handled error-handled" data-position="60712" data-size="48" loading="lazy"></p>
</li>
</ul><ol start="4" class="part" data-startline="1235" data-endline="1237" data-position="60763" data-size="0" style="padding-left: 2em;">
<li class="raw" data-startline="1235" data-endline="1237" data-position="60766" data-size="0"><span data-position="60766" data-size="25">MEM stage (Memory Access)</span><br>
<img src="https://hackmd.io/_uploads/SyuTfLjJ1e.png" alt="56" width="50%" class="offline-handled error-handled" data-position="60792" data-size="54" loading="lazy"></li>
</ol><ul class="part" data-startline="1238" data-endline="1241" data-position="60848" data-size="0">
<li class="raw" data-startline="1238" data-endline="1238" data-position="60850" data-size="0"><span data-position="60850" data-size="218">The value of rs2 (0x00000000) is input into Data in, and Res (0x00000030) is input into Addr. However, under a J-type instruction, the Memory write control signal is 0, indicating that data cannot be written to Memory.</span></li>
<li class="raw" data-startline="1239" data-endline="1241" data-position="61071" data-size="0"><span data-position="61071" data-size="171">Based on the address pointed to by Res, we can check the memory status. The address 0x00000030 stores 0x00092283, so the output from that address (Read Out) is 0x00092283.</span><br>
<img src="https://hackmd.io/_uploads/Hyv0GIiJkl.png" alt="57" class="offline-handled error-handled" data-position="61243" data-size="48" loading="lazy"></li>
</ul><p class="part" data-startline="1242" data-endline="1244" data-position="61293" data-size="0"><span data-position="61293" data-size="60">Finally, the relevant data is stored in the MEM/WB register.</span><br>
<span data-position="61354" data-size="24">5. WB stage (Write back)</span><br>
<img src="https://hackmd.io/_uploads/BywJmLs1kg.png" alt="58" width="30%" class="offline-handled error-handled" data-position="61379" data-size="54" loading="lazy"><span data-position="61433" data-size="1"> </span><img src="https://hackmd.io/_uploads/BJ5J7IjJJg.png" alt="59" width="25%" class="offline-handled error-handled" data-position="61434" data-size="54" loading="lazy"></p><ul class="part" data-startline="1247" data-endline="1250" data-position="61491" data-size="0">
<li class="raw" data-startline="1247" data-endline="1247" data-position="61493" data-size="0"><span data-position="61493" data-size="433">The 3x1 multiplexer is composed of the values from the next program counter, Res, and the value read from memory. Since the instruction type is J-type, the multiplexer selector will choose the program counter value (0x00000018) to pass through. Res will then be input into Wr Data, and the rd value stored in the MEM/WB register (0x01) will be input into Wr idx. Finally, 0x00000018 will be written to the x1 register (ABI name: ra).</span></li>
<li class="raw" data-startline="1248" data-endline="1250" data-position="61929" data-size="0"><span data-position="61929" data-size="65">After completing all stages, the register updates are as follows:</span><br>
<img src="https://hackmd.io/_uploads/Sk_-mIoJke.png" alt="60" width="50%" class="offline-handled error-handled" data-position="61995" data-size="54" loading="lazy"></li>
</ul><p class="part" data-startline="1251" data-endline="1251" data-position="62051" data-size="0"><span data-position="62051" data-size="71">We can see that 0x00000018 was successfully written to the ra register.</span></p><h4 class="part" data-startline="1252" data-endline="1252" id="forwarding" data-id="forwarding"><a class="anchor hidden-xs" href="#forwarding" title="forwarding" smoothhashscroll="" data-size="10" data-position="62128"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="62128" data-size="10">forwarding</span></h4><p class="part" data-startline="1253" data-endline="1253" data-position="62139" data-size="0"><span data-position="62139" data-size="497">In this architecture, executing instructions can lead to some issues, such as data hazards. A data hazard occurs when a later instruction requires data from an earlier instruction, but the data from the earlier instruction has not yet been written back to the register. As a result, the later instruction may access stale data, leading to incorrect results. If the data is incorrect, the final results of the executed instructions will not match the intended outcomes of the original instructions.</span></p><p class="part" data-startline="1255" data-endline="1255" data-position="62638" data-size="0"><span data-position="62638" data-size="195">The forwarding unit is one solution to address data hazards. The core idea is to send the freshly produced data directly to the instruction that needs it. Using the code from above as an example:</span></p><div class="part code-block-wrapper" data-startline="1256" data-endline="1263" data-infoprefix-length="1" data-position="62834">
        <pre><code class="c hljs raw">00000000 &lt;main&gt;:
    0:        00300493        addi x9 x0 3
    4:        10000917        auipc x18 0x10000
    8:        03490913        addi x18 x18 52
    c:        10000997        auipc x19 0x10000
    10:       ff498993        addi x19 x19 -12
</code></pre>
      </div><p class="part" data-startline="1264" data-endline="1266" data-position="63092" data-size="0"><span data-position="63092" data-size="174">After executing from the beginning to the 3rd cycle, the progress of each instruction in the pipeline and the contents stored in the registers are shown in the diagram below:</span><br>
<img src="https://hackmd.io/_uploads/S1CXm8jkke.png" alt="61" class="offline-handled error-handled" data-position="63267" data-size="48" loading="lazy"><br>
<img src="https://hackmd.io/_uploads/rkJrXLjkJg.png" alt="62" width="50%" class="offline-handled error-handled" data-position="63316" data-size="54" loading="lazy"></p><p class="part" data-startline="1268" data-endline="1269" data-position="63372" data-size="0"><span data-position="63372" data-size="448">We can see that the instruction in the EX stage (auipc x18, 0x10000) needs to write back to the x18 register, while the instruction in the ID stage (addi x18, 52) also needs to read from the x18 register. However, it has already retrieved incorrect data in the ID stage (x18 = 0x00000000). Next, we observe the progress of each instruction in the pipeline and the contents stored in the registers after the 4th cycle, as shown in the diagram below:</span><br>
<img src="https://hackmd.io/_uploads/BJjUQUikJe.jpg" alt="63" class="offline-handled error-handled" data-position="63821" data-size="48" loading="lazy"></p><p class="part" data-startline="1271" data-endline="1271" data-position="63871" data-size="0"><span data-position="63871" data-size="560">Due to the forwarding unit, it (marked in yellow) pulls the data generated by the instruction (auipc x18, 0x10000) from the EX stage back to the EX stage. This data, along with Reg1 (marked in red) and another forwarding path (marked in blue), is fed into a 3x1 multiplexer. After detecting a data hazard between the instructions (auipc x18, 0x10000) and (addi x18, 52), the multiplexer is configured to allow the yellow forwarding path to pass through, enabling the instruction (addi x18, 52) to read the correct data from the x18 register (x18 = 0x10000004).</span></p><p class="part" data-startline="1273" data-endline="1274" data-position="64433" data-size="0"><span data-position="64433" data-size="266">Finally, we manually calculate addi x18, x18, 52 (0x10000004 + 52 = 0x10000004 + 0x34), and the execution result sets x18 to 0x10000038. We check the result after the addi x18, x18, 52 instruction passes through the WB stage and confirm that this is indeed the case:</span><br>
<img src="https://hackmd.io/_uploads/rJ3PXUiyye.png" alt="64" width="50%" class="offline-handled error-handled" data-position="64700" data-size="54" loading="lazy"></p><h4 class="part" data-startline="1277" data-endline="1277" id="Hazard-detection" data-id="Hazard-detection"><a class="anchor hidden-xs" href="#Hazard-detection" title="Hazard-detection" smoothhashscroll="" data-size="16" data-position="64762"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="64762" data-size="16">Hazard detection</span></h4><p class="part" data-startline="1278" data-endline="1278" data-position="64779" data-size="0"><span data-position="64779" data-size="762">Although forwarding provides a way to resolve data hazards, there is a specific type of data hazard called Load-use data hazard that cannot be resolved simply through forwarding. A Load-use data hazard occurs when the destination register of a Load instruction is the same as the source register of the subsequent instruction. This is because the Load instruction needs to read data through the MEM stage before it can forward the required data. However, by the time the Load instruction reads the data from the MEM stage, the subsequent instruction has already computed its result in the EX stage, making it impossible to use the correct data. If the subsequent instruction could be stalled for one cycle, it would have time to use the forwarded data correctly.</span></p><p class="part" data-startline="1280" data-endline="1280" data-position="65543" data-size="0"><span data-position="65543" data-size="252">The solution is to add a hazard detection unit that, when the destination register of the Load instruction matches the source register of the following instruction, stalls the subsequent instruction for 1 cycle, thus resolving the Load-use data hazard.</span></p><p class="part" data-startline="1282" data-endline="1282" data-position="65797" data-size="0"><span data-position="65797" data-size="23">Here is a code example:</span></p><div class="part code-block-wrapper" data-startline="1284" data-endline="1290" data-infoprefix-length="1" data-position="65822">
        <pre><code class="c hljs raw">00000000 &lt;main&gt;:
    0:        06428293        addi x5 x5 100
    4:        0002a303        lw x6 0 x5
    8:        00638333        add x6 x7 x6
    c:        00000513        addi x10 x0 0
</code></pre>
      </div><p class="part" data-startline="1292" data-endline="1294" data-position="66022" data-size="0"><span data-position="66022" data-size="174">After executing from the beginning to the 3rd cycle, the progress of each instruction in the pipeline and the contents stored in the registers are shown in the diagram below:</span><br>
<img src="https://hackmd.io/_uploads/BJ8F7Lokyg.png" alt="65" class="offline-handled error-handled" data-position="66197" data-size="48" loading="lazy"><br>
<img src="https://hackmd.io/_uploads/Sk09XUokke.png" alt="66" width="50%" class="offline-handled error-handled" data-position="66246" data-size="54" loading="lazy"></p><p class="part" data-startline="1296" data-endline="1297" data-position="66302" data-size="0"><span data-position="66302" data-size="435">We can see that the instruction in the EX stage (lw x6, 0(x5)) needs to write back to the x6 register, while the instruction in the ID stage (add x6, x7, x6) also needs to read from the x6 register. This will be detected as a Load-use data hazard by the hazard detection unit. Next, we observe the progress of each instruction in the pipeline and the contents stored in the registers after the 4th cycle, as shown in the diagram below:</span><br>
<img src="https://hackmd.io/_uploads/Hy16XIo1yg.png" alt="67" class="offline-handled error-handled" data-position="66738" data-size="48" loading="lazy"></p><p class="part" data-startline="1299" data-endline="1300" data-position="66788" data-size="0"><span data-position="66788" data-size="467">The instruction originally in the ID stage (add x6, x7, x6) should have moved to the EX stage in the next cycle. However, because the hazard detection unit detected a Load-use data hazard, it stalled both the ID stage and the IF stage instructions for one cycle and cleared the EX stage to a NOP instruction. Next, we observe the progress of each instruction in the pipeline and the contents stored in the registers after the 5th cycle, as shown in the diagram below:</span><br>
<img src="https://hackmd.io/_uploads/rkSCQIjJyx.jpg" alt="68" class="offline-handled error-handled" data-position="67256" data-size="48" loading="lazy"></p><p class="part" data-startline="1302" data-endline="1303" data-position="67306" data-size="0"><span data-position="67306" data-size="562">We can see that after stalling for 1 cycle, the instruction (add x6, x7, x6) can utilize the memory forwarding path (marked in yellow). The data transmitted by this forwarding path uses the value of the x5 register (0x00000064) as the memory address to store it in the x6 register. From the diagram below, we can see that the value stored at address 0x00000064 is 0x00000000, so the value returned by the forwarding path is 0x00000000. Finally, we manually calculate add x6, x7, x6 (0x00000000 + 0x00000000 = 0x00000000), resulting in x6 being set to 0x00000000.</span><br>
<img src="https://hackmd.io/_uploads/BkJeNIiJ1g.png" alt="69" class="offline-handled error-handled" data-position="67869" data-size="48" loading="lazy"></p><p class="part" data-startline="1305" data-endline="1306" data-position="67919" data-size="0"><span data-position="67919" data-size="130">We check the result after the instruction add x6, x7, x6 passes through the WB stage, and we confirm that this is indeed the case:</span><br>
<img src="https://hackmd.io/_uploads/S1Og4Liy1e.png" alt="70" width="50%" class="offline-handled error-handled" data-position="68050" data-size="54" loading="lazy"></p><div class="alert alert-danger part" data-startline="1308" data-endline="1309" data-position="68106" data-size="87">
<p data-position="68116" data-size="0"><span data-position="68116" data-size="73">Change the permissions of the above diagrams, making them visible to all!</span></p>
</div><div class="alert alert-success part" data-startline="1312" data-endline="1313" data-position="68195" data-size="70">
<p data-position="68206" data-size="0"><span data-position="68206" data-size="55">I change the permissions of the above diagrams, thanks.</span></p>
</div><h2 class="part" data-startline="1315" data-endline="1315" id="Resource" data-id="Resource"><a class="anchor hidden-xs" href="#Resource" title="Resource" smoothhashscroll="" data-size="8" data-position="68269"><span class="octicon octicon-link ph ph-link-simple-horizontal"></span></a><span data-position="68269" data-size="8">Resource</span></h2><p class="part" data-startline="1316" data-endline="1316" data-position="68278" data-size="0"><a href="https://drive.google.com/file/d/1uviu1nH-tScFfgrovvFCrj7Omv8tFtkp/view" target="_blank" rel="noopener ugc nofollow"><span data-position="68279" data-size="63">The RISC-V Instruction Set Manual Volume I (Version 2024/04/11)</span></a></p></div>