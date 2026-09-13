---
theme: default
title: "Building C++20 Modules: The Rest of the Story"
info: |
  ## Building C++20 Modules: The Rest of the Story
  Presented at CppCon 2026
colorSchema: light
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
mdc: true
fonts:
  mono: Source Code Pro
layout: image
image: /title.png
backgroundSize: cover
---

---
layout: full
class: talk-slide
---

<img src="/talk1.png" class="talk-img" v-click-hide="1" alt="Talk" />
<img src="/talk2.png" class="talk-img" v-click="1" alt="Talk" />

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Question

<div class="q-grid">
  <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 0ms">main.cpp</div>
  <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 0ms"></div>
  <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-DDEBUG</div>
  <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-Iinclude</div>
  <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 90ms">render.cpp</div>
  <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 90ms"></div>
  <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-DENABLE_VULKAN</div>
  <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-Irender</div>
  <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 180ms">math.cpp</div>
  <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 180ms"></div>
  <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-DUSE_SIMD</div>
  <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-Imath</div>
  <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 270ms">util.cpp</div>
  <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 270ms"></div>
  <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 270ms">-DUSE_STD_FORMAT</div>
  <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 270ms">-Iutil</div>
</div>

<hr class="q-rule-long" v-click.fade-in="3" style="transition-delay: 0ms">

<div class="q-cmds font-mono">
  <div class="q-cmd" v-click.fade-in="3" style="transition-delay: 0ms"><span class="c-driver">c++</span><span class="c-opt">-c</span><span class="c-src">main.cpp</span><span class="c-opt">-o</span><span class="c-obj">main.o</span><span class="c-def">-DDEBUG</span><span class="c-inc">-Iinclude</span></div>
  <div class="q-cmd" v-click.fade-in="3" style="transition-delay: 90ms"><span class="c-driver">c++</span><span class="c-opt">-c</span><span class="c-src">render.cpp</span><span class="c-opt">-o</span><span class="c-obj">render.o</span><span class="c-def">-DENABLE_VULKAN</span><span class="c-inc">-Irender</span></div>
  <div class="q-cmd" v-click.fade-in="3" style="transition-delay: 180ms"><span class="c-driver">c++</span><span class="c-opt">-c</span><span class="c-src">math.cpp</span><span class="c-opt">-o</span><span class="c-obj">math.o</span><span class="c-def">-DUSE_SIMD</span><span class="c-inc">-Imath</span></div>
  <div class="q-cmd" v-click.fade-in="3" style="transition-delay: 270ms"><span class="c-driver">c++</span><span class="c-opt">-c</span><span class="c-src">util.cpp</span><span class="c-opt">-o</span><span class="c-obj">util.o</span><span class="c-def">-DUSE_STD_FORMAT</span><span class="c-inc">-Iutil</span></div>
  <hr class="q-rule-short" v-click.fade-in="3" style="transition-delay: 320ms">
  <div class="q-cmd-link" v-click.fade-in="3" style="transition-delay: 360ms">c++ main.o render.o math.o util.o -o prog</div>
</div>

<!--
I have a few quick audience participation questions, just to get an idea of where the background knowledge is on building code in general. !! Given a list of files !! And given a list of associated flags. Please raise your hand if you feel you could compile a program, given only this information, without a build system. We'll say that this is regular C++17-style code, no modules or anything else; and we're trying to produce an executable, not a DLL or anything else complicated.
!!
An acceptable answer might look something like this. We compile each translation unit into a an object file, and then link the object files into a final program. Note that we can accomplish this without ever looking at the contents of the files. We should be able to derive the rules necessary for building the code without concern for the contents of the files.
-->

---
layout: default
class: text-center
---

# Question

<div class="q-question">
  <div class="q-brace-wrap" v-click.fade-in="3" style="transition-delay: 0ms">
    <span class="q-brace-label font-mono">module<br>interface<br>units</span>
    <svg class="q-brace-svg" viewBox="0 0 20 100" preserveAspectRatio="none" aria-hidden="true">
      <path d="M 17 1 C 7 1 13 47 1 50 C 13 53 7 99 17 99" fill="none" stroke="#555555" stroke-width="2" vector-effect="non-scaling-stroke" />
    </svg>
  </div>
  <div class="q-grid">
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 0ms">main.cpp</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 0ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-DDEBUG</div>
    <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-Iinclude</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 90ms">render.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 90ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-DENABLE_VULKAN</div>
    <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-Irender</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 180ms">math.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 180ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-DUSE_SIMD</div>
    <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-Imath</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 270ms">util.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 270ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 270ms">-DUSE_STD_FORMAT</div>
    <div class="q-inc font-mono" v-click.right.fade-in="2" style="transition-delay: 270ms">-Iutil</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Question

<div class="q-question">
  <div class="q-grid">
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 0ms">main.cpp</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 0ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-DDEBUG</div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-std=c++20</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 90ms">render.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 90ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-DENABLE_VULKAN</div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-std=c++20</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 180ms">math.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 180ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-DUSE_SIMD</div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-std=c++23</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 270ms">util.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 270ms"></div>
    <div class="q-def font-mono" v-click.right.fade-in="2" style="transition-delay: 270ms">-DUSE_STD_FORMAT</div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 270ms">-std=c++26</div>
  </div>
</div>

<div class="q-readings">
  <img class="q-reading" src="/reading1.png" v-click.fade-in="3" style="transition-delay: 0ms" alt="reading 1">
  <img class="q-reading" src="/reading2.png" v-click.fade-in="4" style="transition-delay: 0ms" alt="reading 2">
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Topics

<ul class="topics-list">
  <li v-click="1">How to build C++20 modules</li>
  <li v-click="2">Specific command examples and their purpose</li>
  <li v-click="3">The general consensus method</li>
  <li v-click="4">Unsolved problems</li>
  <li v-click="5">Misconceptions and Pitfalls</li>
</ul>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Before Modules

<div class="tu">
  <div class="tu-label font-mono">Translation Unit</div>
  <div class="tu-file">
    <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
      <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
      <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
    </svg>
    <div class="tu-code font-mono">
      <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;vector&gt;</span></div>
      <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;string&gt;</span></div>
      <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;iostream&gt;</span></div>
      <div>&nbsp;</div>
      <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
      <div>}</div>
      <div>&nbsp;</div>
      <div><span class="tok-type">double</span> <span class="tok-fn">mul</span>(<span class="tok-type">double</span> a, <span class="tok-type">double</span> b) {</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a * b;</div>
      <div>}</div>
      <div>&nbsp;</div>
      <div><span class="tok-type">int</span> <span class="tok-fn">clamp</span>(<span class="tok-type">int</span> v, <span class="tok-type">int</span> lo, <span class="tok-type">int</span> hi) {</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> v &lt; lo ? lo : v;</div>
      <div>}</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# After Modules

<div class="mod-groups">
  <div class="mod-group">
    <div class="mod-group-label mod-label-multiline font-mono">Translation<br>Unit which<br>is not a<br>Module Unit</div>
    <div class="mod-files">
      <div class="mod-file">
        <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
          <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
        </svg>
        <div class="mod-code font-mono">
          <div><span class="tok-pre">#include</span> <span class="tok-str">"math.h"</span></div>
          <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;iostream&gt;</span></div>
          <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;vector&gt;</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">std::vector</span>&lt;<span class="tok-type">int</span>&gt; v = <span class="tok-fn">range</span>(<span class="tok-num">5</span>);</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> total = <span class="tok-fn">sum</span>(v);</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">std::cout</span> &lt;&lt; total &lt;&lt; <span class="tok-str">"\n"</span>;</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">std::cout</span> &lt;&lt; <span class="tok-fn">greet</span>() &lt;&lt; <span class="tok-str">"\n"</span>;</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">for</span> (<span class="tok-type">int</span> x : v) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">std::cout</span> &lt;&lt; x &lt;&lt; <span class="tok-str">"\n"</span>;</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;}</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-num">0</span>;</div>
          <div>}</div>
        </div>
      </div>
    </div>
  </div>
  <div class="mod-group">
    <div class="mod-group-label font-mono">Header<br>Unit</div>
    <div class="mod-files">
      <div class="mod-file">
        <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
          <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
        </svg>
        <div class="mod-code font-mono">
          <div><span class="tok-pre">#ifndef</span> <span class="tok-type">MATH_H</span></div>
          <div><span class="tok-pre">#define</span> <span class="tok-type">MATH_H</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;string&gt;</span></div>
          <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;vector&gt;</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">square</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">cube</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">min</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">max</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
          <div><span class="tok-type">std::string</span> <span class="tok-fn">greet</span>();</div>
          <div><span class="tok-type">std::vector</span>&lt;<span class="tok-type">int</span>&gt; <span class="tok-fn">range</span>(<span class="tok-type">int</span> n);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sum</span>(<span class="tok-kw">const</span> <span class="tok-type">std::vector</span>&lt;<span class="tok-type">int</span>&gt;&amp; v);</div>
        </div>
        <svg class="x-mark" v-click="1" viewBox="0 0 100 130" aria-hidden="true">
          <line class="x-stroke" x1="10" y1="10" x2="90" y2="120" />
          <line class="x-stroke" x1="90" y1="10" x2="10" y2="120" />
        </svg>
      </div>
    </div>
  </div>
  <div class="mod-group">
    <div class="mod-group-label mod-label-named font-mono">Named Module Units<span class="ul ul-red" v-click="[2,3]"></span><span class="ul ul-blue" v-click="[3,4]"></span><span class="ul ul-purple" v-click="4"></span></div>
    <div class="mod-files">
      <div class="mod-file">
        <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
          <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
        </svg>
        <div class="mod-code font-mono">
          <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
          <div>&nbsp;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a * b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">div</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
        </div>
      </div>
      <div class="mod-file">
        <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
          <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
        </svg>
        <div class="mod-code font-mono">
          <div><span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a * b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">helper</span>(<span class="tok-type">int</span> x);</div>
        </div>
      </div>
      <div class="mod-file">
        <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
          <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
        </svg>
        <div class="mod-code font-mono">
          <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>:<span class="tok-type">core</span>;</div>
          <div>&nbsp;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">square</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">cube</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">min</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">max</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">abs</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">gcd</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">lcm</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">pow</span>(<span class="tok-type">int</span> b, <span class="tok-type">int</span> e);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">clamp</span>(<span class="tok-type">int</span> v, <span class="tok-type">int</span> lo, <span class="tok-type">int</span> hi);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">sign</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">negate</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">inc</span>(<span class="tok-type">int</span> x);</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">dec</span>(<span class="tok-type">int</span> x);</div>
        </div>
      </div>
      <div class="mod-file">
        <svg class="tu-outline" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
          <path d="M152 2 V48 H198" fill="none" stroke="#c8c8c8" stroke-width="2" stroke-linejoin="round" />
        </svg>
        <div class="mod-code font-mono">
          <div><span class="tok-kw">module</span> <span class="tok-type">math</span>:<span class="tok-type">core</span>;</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">square</span>(<span class="tok-type">int</span> x) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> x * x;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">cube</span>(<span class="tok-type">int</span> x) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> x * x * x;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">min</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a &lt; b ? a : b;</div>
          <div>}</div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">max</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b);</div>
        </div>
      </div>
    </div>
  </div>
</div>

<div class="p1838-link" v-click="5">
  <a href="https://wg21.link/P1838" target="_blank" rel="noopener">P1838: Modules User-Facing Lexicon and File Extensions</a>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center std-slide
---

# Module Declarations

<div class="std-quote">
  <div class="std-head">
    <span>10.1 Module units and purviews</span>
    <span class="std-tag">[module.unit]</span>
  </div>
  <div class="std-body font-mono">
    <div>module-declaration:</div>
    <div class="std-prod" :class="{ 'std-colored': $clicks >= 2 }"><span class="std-term std-export">export-keyword<sub>opt</sub></span> <span class="std-term std-module">module-keyword</span> <span class="std-term std-name">module-name</span> <span class="std-term std-partition">module-partition<sub>opt</sub></span> <span class="std-term std-attr" v-click-hide="1">attribute-specifier-seq<sub>opt</sub></span> ;</div>
    <div class="std-example" v-click="2"><span class="ex-export">export</span> <span class="ex-module">module</span> <span class="ex-name">math</span><span class="ex-partition">:utils</span>;</div>
  </div>
  <div class="std-notes">
    <div class="std-note" v-click="[3,4]">A <strong><em>module interface unit</em></strong> is a module unit whose <em>module-declaration</em> starts with <span class="note-export">export-keyword</span>; any other module unit is a <strong><em>module implementation unit</em></strong>.</div>
    <div class="std-note" v-click="4">A <strong><em>module partition</em></strong> is a module unit whose <em>module-declaration</em> contains a <span class="note-partition">module-partition</span>.</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center ax-slide
---

# Module Units

<div class="axes">
  <div class="ax-vline"></div>
  <div class="ax-hrow">
    <div class="ax-label ax-purple ax-notapartition" v-click="2">Not A Partition</div>
    <div class="ax-hline"></div>
    <div class="ax-label ax-purple ax-partition" v-click="2">Partition</div>
  </div>
  <div class="ax-label ax-red ax-interface" v-click="1">Interface</div>
  <div class="ax-label ax-red ax-implementation" v-click="1">Implementation</div>
  <div class="qex qex-tl">
    <div class="qex-desc" v-click="5">Primary Module Interface Unit</div>
    <div class="qex-code font-mono" v-click="3"><span class="ex-export">export</span> <span class="ex-module">module</span> <span class="ex-name">math</span>;</div>
  </div>
  <div class="qex qex-tr">
    <div class="qex-desc-wrap">
      <div class="qex-desc" v-click="[4,6]">Interface unit which is a partition</div>
      <div class="qex-desc" v-click="6">Partition Interface Unit</div>
    </div>
    <div class="qex-code font-mono" v-click="3"><span class="ex-export">export</span> <span class="ex-module">module</span> <span class="ex-name">math</span><span class="ex-partition">:utils</span>;</div>
  </div>
  <div class="qex qex-bl">
    <div class="qex-desc-wrap">
      <div class="qex-desc" v-click="[4,6]">Implementation unit which is not a partition</div>
      <div class="qex-desc" v-click="6">Module Implementation Unit</div>
    </div>
    <div class="qex-code font-mono" v-click="3"><span class="ex-module">module</span> <span class="ex-name">math</span>;</div>
  </div>
  <div class="qex qex-br">
    <div class="qex-desc-wrap">
      <div class="qex-desc" v-click="[4,6]">Implementation unit which is a partition</div>
      <div class="qex-desc" v-click="6">Partition Implementation Unit</div>
    </div>
    <div class="qex-code font-mono" v-click="3"><span class="ex-module">module</span> <span class="ex-name">math</span><span class="ex-partition">:utils.impl</span>;</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center ax-slide
---

# Module Units

<div class="import-word" v-click="1">import</div>

<div class="axes">
  <div class="ax-vline"></div>
  <div class="ax-hrow">
    <div class="ax-label ax-purple ax-notapartition">Not A Partition</div>
    <div class="ax-hline"></div>
    <div class="ax-label ax-purple ax-partition">Partition</div>
  </div>
  <div class="ax-label ax-red ax-interface">Interface</div>
  <div class="ax-label ax-red ax-implementation">Implementation</div>
  <div class="qex qex-tl">
    <div class="qex-code font-mono"><span class="ex-export">export</span> <span class="ex-module">module</span> <span class="ex-name">math</span>;</div>
    <div class="qex-bullets">
      <div class="qex-bullet bullet-green" v-click="2"><span class="dot dot-green"></span>Importable</div>
      <div class="qex-bullet bullet-orange" v-click="3"><span class="dot dot-orange"></span>Necessarily reachable</div>
    </div>
  </div>
  <div class="qex qex-tr">
    <div class="qex-code font-mono"><span class="ex-export">export</span> <span class="ex-module">module</span> <span class="ex-name">math</span><span class="ex-partition">:utils</span>;</div>
    <div class="qex-bullets">
      <div class="qex-bullet bullet-green" v-click="2"><span class="dot dot-green"></span>Importable within module</div>
      <div class="qex-bullet bullet-orange" v-click="3"><span class="dot dot-orange"></span>Necessarily reachable</div>
    </div>
  </div>
  <div class="qex qex-bl">
    <div class="qex-code font-mono"><span class="ex-module">module</span> <span class="ex-name">math</span>;</div>
    <div class="qex-bullets">
      <div class="qex-bullet bullet-green" v-click="2"><span class="dot dot-green"></span>Not importable</div>
      <div class="qex-bullet bullet-orange" v-click="3"><span class="dot dot-orange"></span>Not reachable</div>
    </div>
  </div>
  <div class="qex qex-br">
    <div class="qex-code font-mono"><span class="ex-module">module</span> <span class="ex-name">math</span><span class="ex-partition">:utils.impl</span>;</div>
    <div class="qex-bullets">
      <div class="qex-bullet bullet-green" v-click="2"><span class="dot dot-green"></span>Importable within module</div>
      <div class="qex-bullet bullet-orange" v-click="3"><span class="dot dot-orange"></span>Implementation-defined reachability</div>
    </div>
  </div>
  <div class="import-border" v-click="4">
    <span class="ib-edge ib-top"></span>
    <span class="ib-edge ib-right"></span>
    <span class="ib-edge ib-bot-right"></span>
    <span class="ib-edge ib-mid-v"></span>
    <span class="ib-edge ib-mid-h"></span>
    <span class="ib-edge ib-left"></span>
  </div>
  <div class="export-border" v-click="5">
    <span class="ib-edge ib-bl-top"></span>
    <span class="ib-edge ib-bl-right"></span>
    <span class="ib-edge ib-bl-bottom"></span>
    <span class="ib-edge ib-bl-left"></span>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Let's Build a PMIU

<div class="pm-pair">
  <div class="pm-build font-mono" :class="{ 'pm-shrunk': $clicks >= 1, 'pm-templated': $clicks >= 6 }">
    <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
    <div>&nbsp;</div>
    <div><span class="tok-kw">export</span> <span class="pm-type-int tok-type">int</span><span class="pm-type-auto tok-kw">auto</span> <span class="tok-fn">add</span>(<span class="pm-type-int tok-type">int</span><span class="pm-type-auto tok-kw">auto</span> a, <span class="pm-type-int tok-type">int</span><span class="pm-type-auto tok-kw">auto</span> b) {</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
    <div>}</div>
  </div>
  <div class="pm-consumer font-mono" v-click="4">
    <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;<span class="qm qm-left" v-click="5" style="transition-delay: 0.25s">?</span><span class="qm qm-mid" v-click="5" style="transition-delay: 0.05s">?</span><span class="qm qm-right" v-click="5" style="transition-delay: 0.25s">?</span></div>
    <div>&nbsp;</div>
    <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
    <div>}</div>
  </div>
</div>

<div class="pm-files">
  <div class="pm-artifact" v-click="2">
    <div class="pm-artifact-file">
      <svg class="pm-artifact-icon" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="pm-artifact-body pm-source font-mono">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
        <div>}</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a * b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="pm-artifact-label font-mono">math.cppm</div>
  </div>
  <div class="pm-artifact" v-click="3">
    <div class="pm-artifact-file">
      <svg class="pm-artifact-icon" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="pm-artifact-body pm-binary font-mono">
        <div>01001000 01100101</div>
        <div>01101100 01101100</div>
        <div>01101111 00100000</div>
        <div>01110111 01101111</div>
        <div>01110010 01101100</div>
        <div>01100100 00100001</div>
        <div>10110011 01011010</div>
        <div>11001010 00110101</div>
        <div>01101001 10101100</div>
        <div>00111100 11000011</div>
      </div>
    </div>
    <div class="pm-artifact-label font-mono">math.o</div>
  </div>
  <div class="pm-artifact" v-click="7">
    <div class="pm-artifact-file">
      <svg class="pm-artifact-icon" viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="pm-artifact-body pm-bmi font-mono">?</div>
    </div>
    <div class="pm-artifact-label font-mono">math.bmi</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center bmi-slide
---

# Built Module Interface

<div class="bmi-row">
  <div class="bmi-col">
    <div class="bmi-topwrap" v-click="1">
      <div class="bmi-top font-mono">Clang</div>
      <div class="bmi-file">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <img class="bmi-logo" src="/clang.png" alt="Clang">
      </div>
    </div>
    <div class="bmi-fmt font-mono" v-click="2">PCM</div>
    <div class="bmi-sub" v-click="3">LLVM Bitcode</div>
    <div class="bmi-writer font-mono" v-click="4">
      <div class="bmi-writer-line">clang::ASTWriter</div>
      <div class="bmi-writer-arrow">↓</div>
      <div class="bmi-writer-line">llvm::BitstreamWriter</div>
    </div>
  </div>
  <div class="bmi-col">
    <div class="bmi-topwrap" v-click="1">
      <div class="bmi-top font-mono">GCC</div>
      <div class="bmi-file">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <img class="bmi-logo" src="/gcc.svg" alt="GCC">
      </div>
    </div>
    <div class="bmi-fmt font-mono" v-click="2">GCM</div>
    <div class="bmi-sub" v-click="3">ELF / ELRoND</div>
    <div class="bmi-writer font-mono" v-click="4">
      <div class="bmi-writer-line">trees_out</div>
      <div class="bmi-writer-arrow">↓</div>
      <div class="bmi-writer-line">elf_out</div>
    </div>
  </div>
  <div class="bmi-col">
    <div class="bmi-topwrap" v-click="1">
      <div class="bmi-top font-mono">MSVC</div>
      <div class="bmi-file">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <img class="bmi-logo" src="/msvc.svg" alt="MSVC">
      </div>
    </div>
    <div class="bmi-fmt font-mono" v-click="2">IFC</div>
    <div class="bmi-sub" v-click="3">IFC (file format)</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center graph-slide
---

# Build & Consume a PMIU

<div class="build-graph">
  <svg class="graph-lines" viewBox="0 0 868 416" preserveAspectRatio="none" aria-hidden="true">
    <defs>
      <marker id="garr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
      </marker>
    </defs>
    <line v-click="2" class="gline" x1="166" y1="140" x2="240" y2="140" marker-end="url(#garr)" />
    <line v-click="3" class="gline" x1="348" y1="124" x2="467" y2="62" marker-end="url(#garr)" />
    <line v-click="3" class="gline" x1="354" y1="150" x2="468" y2="176" marker-end="url(#garr)" />
    <line v-click="4" class="gline" x1="468" y1="228" x2="310" y2="320" marker-end="url(#garr)" />
    <line v-click="4" class="gline" x1="161" y1="350" x2="240" y2="350" marker-end="url(#garr)" />
    <line v-click="5" class="gline" x1="356" y1="350" x2="467" y2="350" marker-end="url(#garr)" />
    <line v-click="6" class="gline" x1="536" y1="80" x2="670" y2="192" marker-end="url(#garr)" />
    <line v-click="6" class="gline" x1="536" y1="330" x2="670" y2="216" marker-end="url(#garr)" />
    <line v-click="7" class="gline" x1="768" y1="205" x2="792" y2="205" marker-end="url(#garr)" />
  </svg>

  <div class="gnode gfile" style="left:120px; top:140px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.cppm</div>
  </div>

  <div class="gnode gproc" style="left:300px; top:140px" v-click="2">
    <div class="gproc-label">Compiler</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:62px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>01001000 01100101</div>
        <div>01101100 01101100</div>
        <div>01101111 00100000</div>
        <div>01110111 01101111</div>
        <div>01110010 01101100</div>
        <div>01100100 00100001</div>
        <div>10110011 01011010</div>
        <div>11001010 00110101</div>
      </div>
    </div>
    <div class="glabel">math.o</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:210px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>Module</div>
        <div>&nbsp;&nbsp;Function: add</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Return: +</div>
        <div>&nbsp;&nbsp;Function: sub</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
      </div>
    </div>
    <div class="glabel">math.bmi</div>
  </div>

  <div class="gnode gfile" style="left:120px; top:350px" v-click="4">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> x = <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> x;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">main.cpp</div>
  </div>

  <div class="gnode gproc" style="left:300px; top:350px" v-click="4">
    <div class="gproc-label">Compiler</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:350px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>01101001 10101100</div>
        <div>00111100 11000011</div>
        <div>00001111 01011010</div>
        <div>11110000 10101010</div>
        <div>01010101 00110011</div>
        <div>11001100 00011110</div>
        <div>10011001 01100110</div>
        <div>00110101 11100001</div>
      </div>
    </div>
    <div class="glabel">main.o</div>
  </div>

  <div class="gnode gproc" style="left:720px; top:205px" v-click="6">
    <div class="gproc-label">Linker</div>
  </div>

  <div class="gnode gexec" style="left:830px; top:205px" v-click="7">
    <div class="gexec-label">prog</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center cmd-slide
---

# Let's Build a PMIU

<div class="clang-cmds font-mono">
  <div class="clang-cmd" v-click="1"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.o</span> <span class="cc-bmi">-fmodule-output=math.bmi</span></div>
  <div class="clang-cmd" v-click="2"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-bmi">-fmodule-file=math=math.bmi</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-driver">clang++</span> <span class="cc-obj">math.o</span> <span class="cc-obj">main.o</span> <span class="cc-opt">-o</span> <span class="cc-exe">prog</span></div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Let's Build Two PMIUs

<div class="q-question">
  <div class="q-grid q-grid-3">
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 0ms">main.cpp</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 0ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-std=c++20</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 90ms">math.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 90ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-std=c++20</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 180ms">util.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 180ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-std=c++20</div>
  </div>
</div>

<div class="scenarios">
  <div class="scenario" v-click="3">
    <div class="sc-graph" style="width:110px">
      <svg class="sc-lines" viewBox="0 0 110 240" preserveAspectRatio="none" aria-hidden="true">
        <defs>
          <marker id="scarr1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
          </marker>
        </defs>
        <line class="gline" x1="55" y1="70" x2="55" y2="99" marker-end="url(#scarr1)" />
        <line class="gline" x1="55" y1="141" x2="55" y2="170" marker-end="url(#scarr1)" />
      </svg>
      <div class="sc-node font-mono" style="left:55px; top:49px">util.cppm</div>
      <div class="sc-node font-mono" style="left:55px; top:120px">math.cppm</div>
      <div class="sc-node font-mono" style="left:55px; top:191px">main.cpp</div>
    </div>
  </div>
  <div class="scenario" v-click="4">
    <div class="sc-graph" style="width:232px">
      <svg class="sc-lines" viewBox="0 0 232 240" preserveAspectRatio="none" aria-hidden="true">
        <defs>
          <marker id="scarr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
          </marker>
        </defs>
        <line class="gline" x1="57" y1="106" x2="92" y2="135" marker-end="url(#scarr2)" />
        <line class="gline" x1="175" y1="106" x2="140" y2="135" marker-end="url(#scarr2)" />
      </svg>
      <div class="sc-node font-mono" style="left:57px; top:85px">util.cppm</div>
      <div class="sc-node font-mono" style="left:175px; top:85px">math.cppm</div>
      <div class="sc-node font-mono" style="left:116px; top:156px">main.cpp</div>
    </div>
  </div>
  <div class="scenario" v-click="5">
    <div class="sc-graph" style="width:150px">
      <svg class="sc-lines" viewBox="0 0 150 240" preserveAspectRatio="none" aria-hidden="true">
        <defs>
          <marker id="scarr3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
          </marker>
        </defs>
        <line class="gline" x1="80" y1="70" x2="66" y2="99" marker-end="url(#scarr3)" />
        <line class="gline" x1="126" y1="70" x2="126" y2="170" marker-end="url(#scarr3)" />
        <line class="gline" x1="70" y1="141" x2="88" y2="170" marker-end="url(#scarr3)" />
      </svg>
      <div class="sc-node font-mono" style="left:95px; top:49px">util.cppm</div>
      <div class="sc-node font-mono" style="left:55px; top:120px">math.cppm</div>
      <div class="sc-node font-mono" style="left:95px; top:191px">main.cpp</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center graph-slide scanner-slide
---

# The Scanner

<div class="build-graph">
  <svg class="graph-lines" viewBox="0 0 868 416" preserveAspectRatio="none" aria-hidden="true">
    <defs>
      <marker id="sarr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
      </marker>
    </defs>
    <line v-click="2" class="gline" x1="157" y1="70" x2="354" y2="70" marker-end="url(#sarr)" />
    <line v-click="2" class="gline" x1="157" y1="200" x2="354" y2="200" marker-end="url(#sarr)" />
    <line v-click="2" class="gline" x1="157" y1="330" x2="354" y2="330" marker-end="url(#sarr)" />
    <line v-click="3" class="gline" x1="466" y1="70" x2="643" y2="70" marker-end="url(#sarr)" />
    <line v-click="3" class="gline" x1="466" y1="200" x2="643" y2="200" marker-end="url(#sarr)" />
    <line v-click="3" class="gline" x1="466" y1="330" x2="643" y2="330" marker-end="url(#sarr)" />
  </svg>

  <div class="gnode gfile" style="left:110px; top:70px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">import</span> <span class="tok-type">util</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">main.cpp</div>
  </div>
  <div class="gnode gproc" style="left:410px; top:70px" v-click="2">
    <div class="gproc-label">Scanner</div>
  </div>
  <div class="gnode gfile gjson" style="left:710px; top:70px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>{</div>
        <div>&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
        <div>&nbsp;<span class="tok-str">"rules"</span>: [{</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>:</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"main.o"</span>,</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"requires"</span>: [</div>
        <div>&nbsp;&nbsp;&nbsp;{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"math"</span>},</div>
        <div>&nbsp;&nbsp;&nbsp;{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>}</div>
        <div>&nbsp;&nbsp;]</div>
        <div>&nbsp;}]</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">main.ddi.json</div>
  </div>

  <div class="gnode gfile" style="left:110px; top:200px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">import</span> <span class="tok-type">util</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.cppm</div>
  </div>
  <div class="gnode gproc" style="left:410px; top:200px" v-click="2">
    <div class="gproc-label">Scanner</div>
  </div>
  <div class="gnode gfile gjson" style="left:710px; top:200px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>{</div>
        <div>&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
        <div>&nbsp;<span class="tok-str">"rules"</span>: [{</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>:</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"math.o"</span>,</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"provides"</span>: [{</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"math"</span>,</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"math.cppm"</span></div>
        <div>&nbsp;&nbsp;}],</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"requires"</span>: [{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>}]</div>
        <div>&nbsp;}]</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.ddi.json</div>
  </div>

  <div class="gnode gfile" style="left:110px; top:330px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">util</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">min</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a &lt; b ? a : b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">util.cppm</div>
  </div>
  <div class="gnode gproc" style="left:410px; top:330px" v-click="2">
    <div class="gproc-label">Scanner</div>
  </div>
  <div class="gnode gfile gjson" style="left:710px; top:330px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>{</div>
        <div>&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
        <div>&nbsp;<span class="tok-str">"rules"</span>: [{</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>:</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"util.o"</span>,</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"provides"</span>: [{</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>,</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"util.cppm"</span></div>
        <div>&nbsp;&nbsp;}]</div>
        <div>&nbsp;}]</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">util.ddi.json</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# The Scanner Format

<div class="scanner-format font-mono" :class="{ 'sf-shifted': $clicks >= 1 }">
  <div>{</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"rules"</span>: [{</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>: <span class="tok-str">"math.o"</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"provides"</span>: [{</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"is-interface"</span>: <span class="tok-kw">true</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"math"</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"math.cppm"</span></div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;}],</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"requires"</span>: [{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>}]</div>
  <div>&nbsp;&nbsp;}],</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"version"</span>: 1</div>
  <div>}</div>
</div>

<div class="sf-right" :class="{ 'sf-shown': $clicks >= 1 }">
  <div class="sf-row sf-compilers">
    <div class="sf-cell">
      <img class="sf-logo" src="/clang.png" alt="Clang">
      <svg class="x-mark" v-click="2" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
        <line class="x-stroke" x1="12" y1="12" x2="88" y2="88" />
        <line class="x-stroke" x1="88" y1="12" x2="12" y2="88" />
      </svg>
    </div>
    <div class="sf-cell">
      <img class="sf-logo" src="/gcc.svg" alt="GCC">
      <svg class="x-mark" v-click="2" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
        <line class="x-stroke" x1="12" y1="12" x2="88" y2="88" />
        <line class="x-stroke" x1="88" y1="12" x2="12" y2="88" />
      </svg>
    </div>
    <div class="sf-cell">
      <img class="sf-logo" src="/msvc.svg" alt="MSVC">
      <svg class="x-mark" v-click="2" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
        <line class="x-stroke" x1="12" y1="12" x2="88" y2="88" />
        <line class="x-stroke" x1="88" y1="12" x2="12" y2="88" />
      </svg>
    </div>
  </div>
  <div class="sf-row sf-buildsystems">
    <div class="sf-cell">
      <div class="sf-word font-mono" v-click="3">Ninja</div>
      <svg class="x-mark" v-click="4" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
        <line class="x-stroke" x1="12" y1="12" x2="88" y2="88" />
        <line class="x-stroke" x1="88" y1="12" x2="12" y2="88" />
      </svg>
    </div>
    <div class="sf-cell">
      <div class="sf-word font-mono" v-click="3">Make</div>
      <svg class="x-mark" v-click="4" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
        <line class="x-stroke" x1="12" y1="12" x2="88" y2="88" />
        <line class="x-stroke" x1="88" y1="12" x2="12" y2="88" />
      </svg>
    </div>
    <div class="sf-cell">
      <div class="sf-word font-mono" v-click="3">MSBuild</div>
      <svg class="x-mark" v-click="4" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
        <line class="x-stroke" x1="12" y1="12" x2="88" y2="88" />
        <line class="x-stroke" x1="88" y1="12" x2="12" y2="88" />
      </svg>
    </div>
  </div>
</div>

<div class="p1689-link">
  <a href="http://wg21.link/P1689" target="_blank" rel="noopener">P1689: Format for describing dependencies of source files</a>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center graph-slide collator-slide
---

# The Collator

<div class="build-graph">
  <svg class="graph-lines" viewBox="0 0 868 416" preserveAspectRatio="none" aria-hidden="true">
    <defs>
      <marker id="carr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
      </marker>
    </defs>
    <line v-click="2" class="gline" x1="102" y1="80" x2="144" y2="80" marker-end="url(#carr)" />
    <line v-click="2" class="gline" x1="102" y1="200" x2="144" y2="200" marker-end="url(#carr)" />
    <line v-click="2" class="gline" x1="102" y1="320" x2="144" y2="320" marker-end="url(#carr)" />
    <line v-click="3" class="gline" x1="256" y1="80" x2="293" y2="80" marker-end="url(#carr)" />
    <line v-click="3" class="gline" x1="256" y1="200" x2="293" y2="200" marker-end="url(#carr)" />
    <line v-click="3" class="gline" x1="256" y1="320" x2="293" y2="320" marker-end="url(#carr)" />
    <line v-click="4" class="gline" x1="426" y1="80" x2="524" y2="184" marker-end="url(#carr)" />
    <line v-click="4" class="gline" x1="426" y1="200" x2="524" y2="200" marker-end="url(#carr)" />
    <line v-click="4" class="gline" x1="426" y1="320" x2="524" y2="216" marker-end="url(#carr)" />
    <line v-click="5" class="gline" x1="582" y1="180" x2="586" y2="111" marker-end="url(#carr)" />
    <line v-click="5" class="gline" x1="599" y1="180" x2="650" y2="131" marker-end="url(#carr)" />
    <line v-click="5" class="gline" x1="632" y1="180" x2="753" y2="131" marker-end="url(#carr)" />
    <line v-click="5" class="gline" x1="632" y1="220" x2="753" y2="269" marker-end="url(#carr)" />
    <line v-click="5" class="gline" x1="599" y1="220" x2="650" y2="269" marker-end="url(#carr)" />
    <line v-click="5" class="gline" x1="582" y1="220" x2="586" y2="289" marker-end="url(#carr)" />
  </svg>

  <div class="gnode gfile" style="left:55px; top:80px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">import</span> <span class="tok-type">util</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">main.cpp</div>
  </div>
  <div class="gnode gproc" style="left:200px; top:80px" v-click="2">
    <div class="gproc-label">Scanner</div>
  </div>
  <div class="gnode gfile gjson" style="left:360px; top:80px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>{</div>
        <div>&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
        <div>&nbsp;<span class="tok-str">"rules"</span>: [{</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>:</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"main.o"</span>,</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"requires"</span>: [</div>
        <div>&nbsp;&nbsp;&nbsp;{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"math"</span>},</div>
        <div>&nbsp;&nbsp;&nbsp;{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>}</div>
        <div>&nbsp;&nbsp;]</div>
        <div>&nbsp;}]</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">main.ddi.json</div>
  </div>

  <div class="gnode gfile" style="left:55px; top:200px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">import</span> <span class="tok-type">util</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.cppm</div>
  </div>
  <div class="gnode gproc" style="left:200px; top:200px" v-click="2">
    <div class="gproc-label">Scanner</div>
  </div>
  <div class="gnode gfile gjson" style="left:360px; top:200px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>{</div>
        <div>&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
        <div>&nbsp;<span class="tok-str">"rules"</span>: [{</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>:</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"math.o"</span>,</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"provides"</span>: [{</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"math"</span>,</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"math.cppm"</span></div>
        <div>&nbsp;&nbsp;}],</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"requires"</span>: [{<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>}]</div>
        <div>&nbsp;}]</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.ddi.json</div>
  </div>

  <div class="gnode gfile" style="left:55px; top:320px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">util</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">min</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a &lt; b ? a : b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">util.cppm</div>
  </div>
  <div class="gnode gproc" style="left:200px; top:320px" v-click="2">
    <div class="gproc-label">Scanner</div>
  </div>
  <div class="gnode gfile gjson" style="left:360px; top:320px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>{</div>
        <div>&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
        <div>&nbsp;<span class="tok-str">"rules"</span>: [{</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"primary-output"</span>:</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"util.o"</span>,</div>
        <div>&nbsp;&nbsp;<span class="tok-str">"provides"</span>: [{</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"util"</span>,</div>
        <div>&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"util.cppm"</span></div>
        <div>&nbsp;&nbsp;}]</div>
        <div>&nbsp;}]</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">util.ddi.json</div>
  </div>

  <div class="gnode gproc" style="left:580px; top:200px" v-click="4">
    <div class="gproc-label">Collator</div>
  </div>

  <div class="gnode gfile gjson gout-top" style="left:590px; top:55px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">-fmodule-file</span>=<span class="tok-type">math</span>=<span class="tok-str">math.bmi</span></div>
        <div><span class="tok-kw">-fmodule-file</span>=<span class="tok-type">util</span>=<span class="tok-str">util.bmi</span></div>
      </div>
    </div>
    <div class="glabel">main.modmap</div>
  </div>
  <div class="gnode gfile gjson gout-top" style="left:700px; top:75px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">-fmodule-file</span>=<span class="tok-type">util</span>=<span class="tok-str">util.bmi</span></div>
      </div>
    </div>
    <div class="glabel">math.modmap</div>
  </div>
  <div class="gnode gfile gjson gout-top" style="left:810px; top:110px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body"></div>
    </div>
    <div class="glabel">util.modmap</div>
  </div>

  <div class="gnode gfile gjson" style="left:590px; top:345px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">ninja_dyndep_version</span> = 1</div>
        <div><span class="tok-kw">build</span> <span class="tok-type">main.o</span>: dyndep |</div>
        <div>&nbsp;&nbsp;<span class="tok-str">math.bmi</span> <span class="tok-str">util.bmi</span></div>
      </div>
    </div>
    <div class="glabel">main.dd</div>
  </div>
  <div class="gnode gfile gjson" style="left:700px; top:325px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">ninja_dyndep_version</span> = 1</div>
        <div><span class="tok-kw">build</span> <span class="tok-type">math.o</span>: dyndep |</div>
        <div>&nbsp;&nbsp;<span class="tok-str">util.bmi</span></div>
      </div>
    </div>
    <div class="glabel">math.dd</div>
  </div>
  <div class="gnode gfile gjson" style="left:810px; top:290px" v-click="5">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">ninja_dyndep_version</span> = 1</div>
        <div><span class="tok-kw">build</span> <span class="tok-type">util.o</span>: dyndep |</div>
      </div>
    </div>
    <div class="glabel">util.dd</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center modmap-slide
---

# Module Maps

<div class="modmap-grid">
  <div class="modmap-card" v-click="1">
    <div class="modmap-box font-mono">
      <div><span class="tok-kw">-fmodule-file</span>=<span class="tok-type">math</span>=<span class="tok-str">math.bmi</span></div>
      <div><span class="tok-kw">-fmodule-file</span>=<span class="tok-type">util</span>=<span class="tok-str">util.bmi</span></div>
    </div>
    <div class="modmap-label font-mono">main.modmap</div>
  </div>
  <div class="modmap-card" v-click="1">
    <div class="modmap-box font-mono">
      <div><span class="tok-kw">-fmodule-file</span>=<span class="tok-type">util</span>=<span class="tok-str">util.bmi</span></div>
    </div>
    <div class="modmap-label font-mono">math.modmap</div>
  </div>
  <div class="modmap-card" v-click="1">
    <div class="modmap-box font-mono"></div>
    <div class="modmap-label font-mono">util.modmap</div>
  </div>
</div>

<div class="modmap-cmds font-mono">
  <div class="clang-cmd" v-click="2"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@main.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-bmi">-fmodule-file=math=math.bmi</span> <span class="cc-bmi">-fmodule-file=util=util.bmi</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
</div>

<div class="p1184-link">
  <a href="https://wg21.link/P1184" target="_blank" rel="noopener">P1184: A Module Mapper</a>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# DynDeps

<div class="modmap-grid">
  <div class="modmap-card" v-click="1">
    <div class="dynep-box font-mono">
      <div><span class="tok-kw">ninja_dyndep_version</span> = 1</div>
      <div><span class="tok-kw">build</span> <span class="tok-type">main.o</span>: dyndep |</div>
      <div>&nbsp;&nbsp;<span class="tok-str">math.bmi</span> <span class="tok-str">util.bmi</span></div>
    </div>
    <div class="modmap-label font-mono">main.dd</div>
  </div>
  <div class="modmap-card" v-click="1">
    <div class="dynep-box font-mono">
      <div><span class="tok-kw">ninja_dyndep_version</span> = 1</div>
      <div><span class="tok-kw">build</span> <span class="tok-type">math.o</span>: dyndep |</div>
      <div>&nbsp;&nbsp;<span class="tok-str">util.bmi</span></div>
    </div>
    <div class="modmap-label font-mono">math.dd</div>
  </div>
  <div class="modmap-card" v-click="1">
    <div class="dynep-box font-mono">
      <div><span class="tok-kw">ninja_dyndep_version</span> = 1</div>
      <div><span class="tok-kw">build</span> <span class="tok-type">util.o</span>: dyndep |</div>
    </div>
    <div class="modmap-label font-mono">util.dd</div>
  </div>
</div>

<div class="dynep-bottom">
  <div class="dynep-graph" :class="{ 'dynep-graph-left': $clicks >= 3 }" v-click="2">
    <div class="sc-graph" style="width:150px">
      <svg class="sc-lines" viewBox="0 0 150 240" preserveAspectRatio="none" aria-hidden="true">
        <defs>
          <marker id="dyneparr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
          </marker>
        </defs>
        <line class="gline" x1="80" y1="70" x2="66" y2="99" marker-end="url(#dyneparr)" />
        <line class="gline" x1="126" y1="70" x2="126" y2="170" marker-end="url(#dyneparr)" />
        <line class="gline" x1="70" y1="141" x2="88" y2="170" marker-end="url(#dyneparr)" />
      </svg>
      <div class="sc-node font-mono" style="left:95px; top:49px">util.cppm</div>
      <div class="sc-node font-mono" style="left:55px; top:120px">math.cppm</div>
      <div class="sc-node font-mono" style="left:95px; top:191px">main.cpp</div>
    </div>
  </div>
  <div class="letter" v-click="3">
    <div class="letter-greeting">Dear Programmer,</div>
    <div class="letter-body">Please build the modules in this order:</div>
    <div class="letter-steps">
      <div><span class="letter-num">1.</span> <span class="font-mono tok-type">util.cppm</span></div>
      <div><span class="letter-num">2.</span> <span class="font-mono tok-type">math.cppm</span></div>
      <div><span class="letter-num">3.</span> <span class="font-mono tok-type">main.cpp</span></div>
    </div>
    <div class="letter-signature">
      <div>Sincerely,</div>
      <div class="letter-sig-name">The Collator</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# A Quick Note About Scanners

<div class="scanner-cmds font-mono">
  <div class="clang-cmd" v-click="1"><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-driver">g++</span> <span class="cc-opt">-fdeps-format=p1689r5</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-driver">cl.exe</span> <span class="cc-opt">-scanDependencies</span></div>
</div>

<div class="scanner-code font-mono" v-click="2">
  <div><span class="tok-pre">#ifdef</span> <span class="tok-type">__clang__</span></div>
  <div><span class="tok-kw">import</span> <span class="tok-type">clang-specific-module</span>;</div>
  <div><span class="tok-pre">#elif</span> <span class="tok-type">__GNUC__</span> &gt; <span class="tok-num">15</span></div>
  <div><span class="tok-kw">import</span> <span class="tok-type">modern-gcc-module</span>;</div>
  <div><span class="tok-pre">#else</span></div>
  <div><span class="tok-kw">import</span> <span class="tok-type">fallback-module</span>;</div>
  <div><span class="tok-pre">#endif</span></div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center build2-slide
---

# Let's Build Two PMIUs

<div class="clang-cmds build2 font-mono">
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.ddi.json</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.ddi.json</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.ddi.json</span></div>
  <div class="clang-cmd" v-click="2"><span class="cc-step"><span class="cc-num">2</span></span><span class="cc-driver">collate</span> <span class="cc-obj">main.ddi.json</span> <span class="cc-obj">math.ddi.json</span> <span class="cc-obj">util.ddi.json</span></div>
  <div class="clang-cmd" v-click="[3,4]"><span class="cc-step"></span><span class="cc-driver">cmake</span> <span class="cc-opt">-E</span> <span class="cc-opt">cmake_ninja_dyndep</span></div>
  <div class="clang-cmd" v-click="4"><span class="cc-step"><span class="cc-q" v-click="[4,6]">?</span><span class="cc-ans" v-click="6">4</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@math.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.o</span> <span class="cc-bmi">-fmodule-output=math.bmi</span></div>
  <div class="clang-cmd" v-click="4"><span class="cc-step"><span class="cc-q" v-click="[4,6]">?</span><span class="cc-ans" v-click="6">3</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@util.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.o</span> <span class="cc-bmi">-fmodule-output=util.bmi</span></div>
  <div class="clang-cmd" v-click="4"><span class="cc-step"><span class="cc-q" v-click="[4,6]">?</span><span class="cc-ans" v-click="6">5</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@main.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
  <div class="clang-cmd" v-click="7"><span class="cc-step"><span class="cc-num">6</span></span><span class="cc-driver">clang++</span> <span class="cc-obj">math.o</span> <span class="cc-obj">util.o</span> <span class="cc-obj">main.o</span> <span class="cc-opt">-o</span> <span class="cc-exe">prog</span></div>
</div>

<div class="letter-pop" v-click="[5,6]">
  <div class="letter-greeting">Dear Programmer,</div>
  <div class="letter-body">Please build the modules in this order:</div>
  <div class="letter-steps">
    <div><span class="letter-num">1.</span> <span class="font-mono tok-type">util.cppm</span></div>
    <div><span class="letter-num">2.</span> <span class="font-mono tok-type">math.cppm</span></div>
    <div><span class="letter-num">3.</span> <span class="font-mono tok-type">main.cpp</span></div>
  </div>
  <div class="letter-signature">
    <div>Sincerely,</div>
    <div class="letter-sig-name">The Collator</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center blog-slide
---

# Let's Build Two PMIUs, The Remix

<div class="q-question">
  <div class="q-grid q-grid-3">
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 0ms">main.cpp</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 0ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-std=c++20</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 90ms">math.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 90ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-std=c++23</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 180ms">util.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 180ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-std=c++20</div>
  </div>
</div>

<div class="blog-ref" v-click="3">
  <img class="blog-img" src="/blog.png" alt="BMI Compatibility blog post">
</div>

<div class="pblog-link" v-click="3">
  <a href="https://blog.vito.nyc/posts/bmi-compatibility/" target="_blank" rel="noopener">https://blog.vito.nyc/posts/bmi-compatibility/</a>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# The Problem

<div class="dialect-frame font-mono">
  <div class="dialect-half" v-click="1">
    <div class="dialect-badge">-std=c++23</div>
    <div class="dialect-lines">
      <div><span class="tok-pre">#include</span> <span class="tok-type">&lt;print&gt;</span></div>
      <div><span class="tok-pre">#include</span> <span class="tok-str">"math.hpp"</span></div>
      <div>&nbsp;</div>
      <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> result = <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
    </div>
  </div>
  <div class="dialect-sep"></div>
  <div class="dialect-half dialect-bottom" v-click="2">
    <div class="dialect-badge">-std=c++20</div>
    <div class="dialect-lines">
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-fn">std::println</span>(<span class="tok-str">"result = {}"</span>, result);</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-num">0</span>;</div>
      <div>}</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center bmi-slide
---

# BMI Only

<div class="bmi-graphs">
  <div class="bmi-graph">
    <svg class="graph-lines" viewBox="0 0 868 200" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="bmiarr1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="1" class="gline" x1="274" y1="100" x2="366" y2="100" marker-end="url(#bmiarr1)" />
      <line v-click="1" class="gline" x1="482" y1="95" x2="598" y2="58" marker-end="url(#bmiarr1)" />
      <line v-click="1" class="gline" x1="482" y1="105" x2="598" y2="142" marker-end="url(#bmiarr1)" />
    </svg>
    <div class="gnode gfile" style="left:244px; top:100px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">math.cppm</div>
    </div>
    <div class="gnode gstd" style="left:424px; top:58px" v-click="1">-std=c++23</div>
    <div class="gnode gproc" style="left:424px; top:100px" v-click="1">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gfile" style="left:629px; top:52px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>01001000 01100101</div>
          <div>01101100 01101100</div>
          <div>01101111 00100000</div>
          <div>01110111 01101111</div>
          <div>01110010 01101100</div>
          <div>01100100 00100001</div>
        </div>
      </div>
      <div class="glabel">math.o</div>
    </div>
    <div class="gnode gfile" style="left:629px; top:148px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>Module</div>
          <div>&nbsp;&nbsp;Function: add</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;Return: +</div>
        </div>
      </div>
      <div class="glabel">math.bmi</div>
    </div>
  </div>

  <div class="bmi-graph">
    <svg class="graph-lines" viewBox="0 0 868 200" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="bmiarr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="2" class="gline" x1="274" y1="100" x2="366" y2="100" marker-end="url(#bmiarr2)" />
      <line v-click="2" class="gline" x1="482" y1="100" x2="598" y2="100" marker-end="url(#bmiarr2)" />
    </svg>
    <div class="gnode gfile" style="left:244px; top:100px" v-click="2">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
          <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">math.cppm</div>
    </div>
    <div class="gnode gstd" style="left:424px; top:58px" v-click="2">-std=c++20</div>
    <div class="gnode gproc" style="left:424px; top:100px" v-click="2">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gfile" style="left:629px; top:100px" v-click="2">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>Module</div>
          <div>&nbsp;&nbsp;Function: add</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
          <div>&nbsp;&nbsp;&nbsp;&nbsp;Return: +</div>
        </div>
      </div>
      <div class="glabel">math.bmi</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center header-slide
---

# Consider the Header

<div class="header-graphs">
  <div class="header-graph">
    <svg class="graph-lines" viewBox="0 0 868 72" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="hdrarr1" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="1" class="gline" x1="213" y1="14" x2="436" y2="34" marker-end="url(#hdrarr1)" />
      <line v-click="1" class="gline" x1="213" y1="58" x2="436" y2="38" marker-end="url(#hdrarr1)" />
      <line v-click="1" class="gline" x1="532" y1="36" x2="652" y2="36" marker-end="url(#hdrarr1)" />
    </svg>
    <div class="gnode gfile" style="left:190px; top:6px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
        </div>
      </div>
      <div class="glabel">math.hpp</div>
    </div>
    <div class="gnode gfile" style="left:190px; top:74px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-pre">#include</span> <span class="tok-str">"math.hpp"</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
          <div>&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">main.cpp</div>
    </div>
    <div class="gnode gproc" style="left:486px; top:36px" v-click="1">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gstd" style="left:486px; top:8px" v-click="1">-std=c++20</div>
    <div class="gnode gfile" style="left:676px; top:36px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>01001000 01100101</div>
          <div>01101100 01101100</div>
          <div>01101111 00100000</div>
        </div>
      </div>
      <div class="glabel">main.o</div>
    </div>
  </div>

  <div class="header-graph">
    <svg class="graph-lines" viewBox="0 0 868 72" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="hdrarr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="1" class="gline" x1="380" y1="14" x2="436" y2="34" marker-end="url(#hdrarr2)" />
      <line v-click="1" class="gline" x1="380" y1="58" x2="436" y2="38" marker-end="url(#hdrarr2)" />
      <line v-click="1" class="gline" x1="532" y1="36" x2="652" y2="36" marker-end="url(#hdrarr2)" />
    </svg>
    <div class="gnode gfile" style="left:360px; top:6px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
        </div>
      </div>
      <div class="glabel">math.hpp</div>
    </div>
    <div class="gnode gfile" style="left:360px; top:74px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-pre">#include</span> <span class="tok-str">"math.hpp"</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">run</span>() {</div>
          <div>&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">sub</span>(<span class="tok-num">5</span>, <span class="tok-num">3</span>);</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">app.cpp</div>
    </div>
    <div class="gnode gproc" style="left:486px; top:36px" v-click="1">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gstd" style="left:486px; top:8px" v-click="1">-std=c++20</div>
    <div class="gnode gfile" style="left:676px; top:36px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>01001000 01100101</div>
          <div>01101100 01101100</div>
          <div>01101111 00100000</div>
        </div>
      </div>
      <div class="glabel">app.o</div>
    </div>
  </div>

  <div class="header-graph">
    <svg class="graph-lines" viewBox="0 0 868 72" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="hdrarr3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="1" class="gline" x1="213" y1="14" x2="436" y2="34" marker-end="url(#hdrarr3)" />
      <line v-click="1" class="gline" x1="213" y1="58" x2="436" y2="38" marker-end="url(#hdrarr3)" />
      <line v-click="1" class="gline" x1="532" y1="36" x2="652" y2="36" marker-end="url(#hdrarr3)" />
    </svg>
    <div class="gnode gfile" style="left:190px; top:6px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
        </div>
      </div>
      <div class="glabel">math.hpp</div>
    </div>
    <div class="gnode gfile" style="left:190px; top:74px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-pre">#include</span> <span class="tok-str">"math.hpp"</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">tick</span>() {</div>
          <div>&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">mul</span>(<span class="tok-num">4</span>, <span class="tok-num">5</span>);</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">engine.cpp</div>
    </div>
    <div class="gnode gproc" style="left:486px; top:36px" v-click="1">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gstd" style="left:486px; top:8px" v-click="1">-std=c++23</div>
    <div class="gnode gfile" style="left:676px; top:36px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>01001000 01100101</div>
          <div>01101100 01101100</div>
          <div>01101111 00100000</div>
        </div>
      </div>
      <div class="glabel">engine.o</div>
    </div>
  </div>

  <div class="header-graph">
    <svg class="graph-lines" viewBox="0 0 868 72" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="hdrarr4" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="1" class="gline" x1="380" y1="14" x2="436" y2="34" marker-end="url(#hdrarr4)" />
      <line v-click="1" class="gline" x1="380" y1="58" x2="436" y2="38" marker-end="url(#hdrarr4)" />
      <line v-click="1" class="gline" x1="532" y1="36" x2="652" y2="36" marker-end="url(#hdrarr4)" />
    </svg>
    <div class="gnode gfile" style="left:360px; top:6px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
        </div>
      </div>
      <div class="glabel">math.hpp</div>
    </div>
    <div class="gnode gfile" style="left:360px; top:74px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-pre">#include</span> <span class="tok-str">"math.hpp"</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">draw</span>() {</div>
          <div>&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">7</span>, <span class="tok-num">8</span>);</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">render.cpp</div>
    </div>
    <div class="gnode gproc" style="left:486px; top:36px" v-click="1">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gstd" style="left:486px; top:8px" v-click="1">-std=c++20</div>
    <div class="gnode gfile" style="left:676px; top:36px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>01001000 01100101</div>
          <div>01101100 01101100</div>
          <div>01101111 00100000</div>
        </div>
      </div>
      <div class="glabel">render.o</div>
    </div>
  </div>

  <div class="header-graph">
    <svg class="graph-lines" viewBox="0 0 868 72" preserveAspectRatio="none" aria-hidden="true">
      <defs>
        <marker id="hdrarr5" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
        </marker>
      </defs>
      <line v-click="1" class="gline" x1="213" y1="14" x2="436" y2="34" marker-end="url(#hdrarr5)" />
      <line v-click="1" class="gline" x1="213" y1="58" x2="436" y2="38" marker-end="url(#hdrarr5)" />
      <line v-click="1" class="gline" x1="532" y1="36" x2="652" y2="36" marker-end="url(#hdrarr5)" />
    </svg>
    <div class="gnode gfile" style="left:190px; top:6px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
        </div>
      </div>
      <div class="glabel">math.hpp</div>
    </div>
    <div class="gnode gfile" style="left:190px; top:74px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div><span class="tok-pre">#include</span> <span class="tok-str">"math.hpp"</span></div>
          <div>&nbsp;</div>
          <div><span class="tok-type">int</span> <span class="tok-fn">play</span>() {</div>
          <div>&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">sub</span>(<span class="tok-num">9</span>, <span class="tok-num">1</span>);</div>
          <div>}</div>
        </div>
      </div>
      <div class="glabel">audio.cpp</div>
    </div>
    <div class="gnode gproc" style="left:486px; top:36px" v-click="1">
      <div class="gproc-label">Compiler</div>
    </div>
    <div class="gnode gstd" style="left:486px; top:8px" v-click="1">-std=c++23</div>
    <div class="gnode gfile" style="left:676px; top:36px" v-click="1">
      <div class="gfile-box">
        <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
          <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
          <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        </svg>
        <div class="gfile-body">
          <div>01001000 01100101</div>
          <div>01101100 01101100</div>
          <div>01101111 00100000</div>
        </div>
      </div>
      <div class="glabel">audio.o</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center better-slide
---

# A Better Collator

<div class="better-bottom">
  <div class="better-graph" :class="{ 'better-graph-left': $clicks >= 1 }">
    <div class="sc-graph" style="width:150px">
      <svg class="sc-lines" viewBox="0 0 150 240" preserveAspectRatio="none" aria-hidden="true">
        <defs>
          <marker id="betterarr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
          </marker>
        </defs>
        <line class="gline" x1="80" y1="70" x2="66" y2="99" marker-end="url(#betterarr)" />
        <line class="gline" x1="126" y1="70" x2="126" y2="170" marker-end="url(#betterarr)" />
        <line class="gline" x1="70" y1="141" x2="88" y2="170" marker-end="url(#betterarr)" />
      </svg>
      <div class="sc-node font-mono" style="left:95px; top:49px">util.cppm</div>
      <div class="sc-node font-mono" style="left:55px; top:120px">math.cppm</div>
      <div class="sc-node font-mono" style="left:95px; top:191px">main.cpp</div>
      <div class="better-label" style="left:165px; top:49px" v-click="1">-std=c++20</div>
      <div class="better-label" style="right:155px; top:120px" v-click="1"><span v-click="[1,2]">-std=c++20</span><span v-click="2">-std=c++23</span></div>
      <div class="better-label" style="left:165px; top:191px" v-click="1">-std=c++20</div>
    </div>
  </div>
  <div class="better-letter" :class="{ 'better-letter-expanded': $clicks >= 3 }">
    <div class="letter-greeting">Dear Programmer,</div>
    <div class="letter-body"><span v-click-hide="3">Please build the modules in this order:</span><span v-click="3">Please consider the following dependency information:</span></div>
    <div class="letter-steps">
      <div class="letter-steps-set" v-click-hide="3">
        <div><span class="letter-num">1.</span> <span class="font-mono tok-type">util.cppm</span></div>
        <div><span class="letter-num">2.</span> <span class="font-mono tok-type">math.cppm</span></div>
        <div><span class="letter-num">3.</span> <span class="font-mono tok-type">main.cpp</span></div>
      </div>
      <div class="letter-steps-set" v-click="3">
        <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">util.cppm</span> has no dependencies</div>
        <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">math.cppm</span> depends on <span class="font-mono tok-type">util</span></div>
        <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">main.cpp</span> depends on <span class="font-mono tok-type">math</span> and <span class="font-mono tok-type">util</span></div>
      </div>
    </div>
    <div class="letter-signature">
      <div>Sincerely,</div>
      <div class="letter-sig-name">The Collator</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center build2-slide
---

# Let's Build Two PMIUs, The Remix

<div class="clang-cmds build2 font-mono">
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.ddi.json</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.ddi.json</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.ddi.json</span></div>
  <div class="clang-cmd" v-click="2"><span class="cc-step"><span class="cc-num">2</span></span><span class="cc-driver">collate</span> <span class="cc-obj">main.ddi.json</span> <span class="cc-obj">math.ddi.json</span> <span class="cc-obj">util.ddi.json</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-ans" v-click="5">3</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">--precompile</span> <span class="cc-resp">@util23.modmap</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">util23.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-red" v-click="5">4</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">--precompile</span> <span class="cc-resp">@math20.modmap</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">math20.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-red" v-click="5">3</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@util20.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.o</span> <span class="cc-bmi">-fmodule-output=util20.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-ans" v-click="5">4</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++23</span> <span class="cc-resp">@math23.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.o</span> <span class="cc-bmi">-fmodule-output=math23.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-red" v-click="5">5</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@main20.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
  <div class="clang-cmd" v-click="6"><span class="cc-step"><span class="cc-num">6</span></span><span class="cc-driver">clang++</span> <span class="cc-obj">math.o</span> <span class="cc-obj">util.o</span> <span class="cc-obj">main.o</span> <span class="cc-opt">-o</span> <span class="cc-exe">prog</span></div>
</div>

<div class="letter-pop" v-click="[4,5]">
  <div class="letter-greeting">Dear Programmer,</div>
  <div class="letter-body">Please consider the following dependency information</div>
  <div class="letter-steps">
    <div class="letter-steps-set">
      <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">util.cppm</span> has no dependencies</div>
      <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">math.cppm</span> depends on <span class="font-mono tok-type">util</span></div>
      <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">main.cpp</span> depends on <span class="font-mono tok-type">math</span> and <span class="font-mono tok-type">util</span></div>
    </div>
  </div>
  <div class="letter-signature">
    <div>Sincerely,</div>
    <div class="letter-sig-name">The Collator</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Unsolved Problem in Computer Science #1

<div class="note-body">
  <div class="clang-cmds build2 font-mono">
    <div class="clang-cmd" v-click="1"><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.ddi.json</span></div>
    <div class="clang-cmd" v-click="1"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">--precompile</span> <span class="cc-resp">@util23.modmap</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">util23.bmi</span></div>
    <div class="clang-cmd" v-click="1"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@util20.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.o</span> <span class="cc-bmi">-fmodule-output=util20.bmi</span></div>
  </div>

  <div class="scanner-code font-mono" v-click="2">
    <div><span class="tok-pre">#if</span> <span class="tok-type">__cplusplus</span> &gt;= <span class="tok-num">202302L</span></div>
    <div><span class="tok-kw">import</span> <span class="tok-type">std</span>;</div>
    <div><span class="tok-pre">#endif</span></div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Unsolved Problem in Computer Science #2

<div class="note-body">
  <div class="flag-cols font-mono" v-click="1">
    <div class="flag-col">
      <div>-O2</div>
      <div>-fno-exceptions</div>
      <div>-g</div>
      <div>-fchar8_t</div>
      <div>-fPIC</div>
      <div>-fshort-enums</div>
      <div>-gdwarf-5</div>
    </div>
    <div class="flag-col">
      <div>-fno-rtti</div>
      <div>-fomit-frame-pointer</div>
      <div>-funsigned-char</div>
      <div>-pipe</div>
      <div>-fno-math-errno</div>
      <div>-march=native</div>
      <div>-ffast-math</div>
    </div>
    <div class="flag-col">
      <div>-pthread</div>
      <div>-fms-extensions</div>
      <div>-fsanitize=address</div>
      <div>-fdelayed-template-parsing</div>
      <div>-fstack-protector-strong</div>
      <div>-fvisibility=hidden</div>
      <div>-fno-threadsafe-statics</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# Let's Build Two PMIUs, The Finale

<div class="q-question">
  <div class="q-grid q-grid-3">
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 0ms">main.cpp</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 0ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 0ms">-std=c++20</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 90ms">math.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 90ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 90ms">-std=c++23 -DUSE_SIMD</div>
    <div class="q-file font-mono" v-click.down.fade-in="1" style="transition-delay: 180ms">util.cppm</div>
    <div class="q-rule" v-click.right.fade-in="2" style="transition-delay: 180ms"></div>
    <div class="q-std font-mono" v-click.right.fade-in="2" style="transition-delay: 180ms">-std=c++20 -DUSE_STD_FORMAT</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center build2-slide finale-slide
---

# Let's Build Two PMIUs, The Finale

<div class="clang-cmds build2 font-mono">
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.ddi.json</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">-DUSE_SIMD</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.ddi.json</span></div>
  <div class="clang-cmd" v-click="1"><span class="cc-step"><span class="cc-num">1</span></span><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-DUSE_STD_FORMAT</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.ddi.json</span></div>
  <div class="clang-cmd" v-click="2"><span class="cc-step"><span class="cc-num">2</span></span><span class="cc-driver">collate</span> <span class="cc-obj">main.ddi.json</span> <span class="cc-obj">math.ddi.json</span> <span class="cc-obj">util.ddi.json</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-ans" v-click="5">3</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">-DUSE_STD_FORMAT</span> <span class="cc-opt">--precompile</span> <span class="cc-resp">@util23.modmap</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">util23.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-red" v-click="5">4</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-DUSE_SIMD</span> <span class="cc-opt">--precompile</span> <span class="cc-resp">@math20.modmap</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">math20.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-red" v-click="5">3</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-DUSE_STD_FORMAT</span> <span class="cc-resp">@util20.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">util.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">util.o</span> <span class="cc-bmi">-fmodule-output=util20.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-ans" v-click="5">4</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">-DUSE_SIMD</span> <span class="cc-resp">@math23.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.o</span> <span class="cc-bmi">-fmodule-output=math23.bmi</span></div>
  <div class="clang-cmd" v-click="3"><span class="cc-step"><span class="cc-q" v-click="[3,5]">?</span><span class="cc-red" v-click="5">5</span></span><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-resp">@main20.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
  <div class="clang-cmd" v-click="6"><span class="cc-step"><span class="cc-num">6</span></span><span class="cc-driver">clang++</span> <span class="cc-obj">math.o</span> <span class="cc-obj">util.o</span> <span class="cc-obj">main.o</span> <span class="cc-opt">-o</span> <span class="cc-exe">prog</span></div>
</div>

<div class="letter-pop" v-click="[4,5]">
  <div class="letter-greeting">Dear Programmer,</div>
  <div class="letter-body">Please consider the following dependency information</div>
  <div class="letter-steps">
    <div class="letter-steps-set">
      <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">util.cppm</span> has no dependencies</div>
      <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">math.cppm</span> depends on <span class="font-mono tok-type">util</span></div>
      <div><span class="letter-bullet">•</span> <span class="font-mono tok-type">main.cpp</span> depends on <span class="font-mono tok-type">math</span> and <span class="font-mono tok-type">util</span></div>
    </div>
  </div>
  <div class="letter-signature">
    <div>Sincerely,</div>
    <div class="letter-sig-name">The Collator</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Unsolved Problem in Computer Science #3

<div class="note-body">
  <div class="clang-cmds build2 font-mono">
    <div class="clang-cmd" v-click="1"><span class="cc-driver">clang-scan-deps</span> <span class="cc-opt">-format=p1689</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">-DMATH_EXPORT</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.ddi.json</span></div>
    <div class="clang-cmd" v-click="1"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-DMATH_EXPORT</span> <span class="cc-opt">--precompile</span> <span class="cc-resp">@math20.modmap</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">math20.bmi</span></div>
    <div class="clang-cmd" v-click="1"><span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++23</span> <span class="cc-opt">-DMATH_EXPORT</span> <span class="cc-resp">@math23.modmap</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.o</span> <span class="cc-bmi">-fmodule-output=math23.bmi</span></div>
  </div>

  <div class="scanner-code font-mono" v-click="2">
    <div><span class="tok-pre">#ifdef</span> <span class="tok-type">MATH_EXPORT</span></div>
    <div><span class="tok-pre">#define</span> <span class="tok-type">MATH_API</span> <span class="tok-kw">__declspec</span>( <span class="tok-type">dllexport</span> )</div>
    <div><span class="tok-pre">#else</span></div>
    <div><span class="tok-pre">#define</span> <span class="tok-type">MATH_API</span> <span class="tok-kw">__declspec</span>( <span class="tok-type">dllimport</span> )</div>
    <div><span class="tok-pre">#endif</span></div>
    <div>&nbsp;</div>
    <div><span class="tok-type">MATH_API</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
    <div>&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
    <div>}</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center graph-slide
---

# Module Libraries

<div class="build-graph">
  <svg class="graph-lines" viewBox="0 0 868 416" preserveAspectRatio="none" aria-hidden="true">
    <defs>
      <marker id="libarr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
      </marker>
    </defs>
    <line v-click="3" class="gline" x1="166" y1="140" x2="240" y2="140" marker-end="url(#libarr)" />
    <line v-click="4" class="gline" x1="348" y1="124" x2="467" y2="62" marker-end="url(#libarr)" />
    <line v-click="4" class="gline" x1="354" y1="150" x2="468" y2="176" marker-end="url(#libarr)" />
    <line v-click="5" class="gline" x1="468" y1="228" x2="310" y2="320" marker-end="url(#libarr)" />
    <line v-click="5" class="gline" x1="161" y1="350" x2="240" y2="350" marker-end="url(#libarr)" />
    <line v-click="6" class="gline" x1="356" y1="350" x2="467" y2="350" marker-end="url(#libarr)" />
    <line v-click="7" class="gline" x1="536" y1="80" x2="625" y2="192" marker-end="url(#libarr)" />
    <line v-click="7" class="gline" x1="536" y1="330" x2="625" y2="216" marker-end="url(#libarr)" />
    <line v-click="8" class="gline" x1="755" y1="205" x2="780" y2="205" marker-end="url(#libarr)" />
  </svg>

  <div class="gnode gfile" style="left:120px; top:140px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.cppm</div>
  </div>

  <div class="gnode gproc" style="left:300px; top:140px" v-click="3">
    <div class="gproc-label">Compiler</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:62px" v-click="4">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>01001000 01100101</div>
        <div>01101100 01101100</div>
        <div>01101111 00100000</div>
        <div>01110111 01101111</div>
        <div>01110010 01101100</div>
        <div>01100100 00100001</div>
        <div>10110011 01011010</div>
        <div>11001010 00110101</div>
      </div>
    </div>
    <div class="glabel">math.cppm.o</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:210px" v-click="4">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>Module</div>
        <div>&nbsp;&nbsp;Function: add</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Return: +</div>
        <div>&nbsp;&nbsp;Function: sub</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
      </div>
    </div>
    <div class="glabel">math.bmi</div>
  </div>

  <div class="gnode gfile" style="left:120px; top:350px" v-click="2">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">mul</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a * b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.cpp</div>
  </div>

  <div class="gnode gproc" style="left:300px; top:350px" v-click="5">
    <div class="gproc-label">Compiler</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:350px" v-click="6">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>01101001 10101100</div>
        <div>00111100 11000011</div>
        <div>00001111 01011010</div>
        <div>11110000 10101010</div>
        <div>01010101 00110011</div>
        <div>11001100 00011110</div>
        <div>10011001 01100110</div>
        <div>00110101 11100001</div>
      </div>
    </div>
    <div class="glabel">math.cpp.o</div>
  </div>

  <div class="gnode gproc" style="left:690px; top:205px" v-click="7">
    <div class="gproc-label">Archiver</div>
  </div>

  <div class="gnode gexec" style="left:830px; top:205px" v-click="8">
    <div class="gexec-label">math.a</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center metadata-slide
---

# Module Manifest

<div class="scanner-format font-mono">
  <div>{</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"version"</span>: 1,</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"revision"</span>: 1,</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"modules"</span>: [{</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"math"</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"math.cppm"</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"local-arguments"</span>: {</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"definitions"</span>: [{</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"name"</span>: <span class="tok-str">"USE_SIMD"</span></div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}]</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;}</div>
  <div>&nbsp;&nbsp;}]</div>
  <div>}</div>
</div>

<div class="p3286-link">
  <a href="https://wg21.link/P3286" target="_blank" rel="noopener">P3286: Module Metadata Format for Distribution with Pre-Built Libraries</a>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center
---

# CPS Manifest

<div class="scanner-format font-mono">
  <div>{</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"name"</span>: <span class="tok-str">"math"</span>,</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"cps_version"</span>: <span class="tok-str">"0.15.0"</span>,</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"cps_path"</span>: <span class="tok-str">"@prefix@/lib/cps/math"</span>,</div>
  <div>&nbsp;&nbsp;<span class="tok-str">"components"</span>: {</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"math"</span>: {</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"type"</span>: <span class="tok-str">"archive"</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"location"</span>: <span class="tok-str">"@prefix@/lib/math.a"</span>,</div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"cpp_module_metadata"</span>: <span class="tok-str">"@prefix@/lib/cps/math.modules.json"</span></div>
  <div>&nbsp;&nbsp;&nbsp;&nbsp;}</div>
  <div>&nbsp;&nbsp;}</div>
  <div>}</div>
</div>

<div class="pcps-link">
  <a href="https://cps-org.github.io/cps/index.html" target="_blank" rel="noopener">https://cps-org.github.io/cps/index.html</a>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# What about the stdlib?

<div class="note-body note-body-grid">
  <div class="scanner-code font-mono stdlib-cmds" v-click-hide="1">
    <div><span class="tok-prompt">$</span> <span class="cc-driver">g++</span> <span class="cc-opt">--print-file-name=libstdc++.modules.json</span></div>
    <div class="print-result">/usr/lib/libstdc++.modules.json</div>
    <div class="print-blank"></div>
    <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">--print-file-name=libc++.modules.json</span></div>
    <div class="print-result">/usr/lib/libc++.modules.json</div>
  </div>

  <div class="scanner-format font-mono stdlib-json" v-click="[1, 2]">
    <div>{</div>
    <div>&nbsp;&nbsp;<span class="tok-str">"version"</span>: 1,</div>
    <div>&nbsp;&nbsp;<span class="tok-str">"revision"</span>: 1,</div>
    <div>&nbsp;&nbsp;<span class="tok-str">"modules"</span>: [</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;{</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"std"</span>,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"../include/c++/16/bits/std.cc"</span>,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"is-std-library"</span>: <span class="tok-kw">true</span></div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;},</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;{</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"logical-name"</span>: <span class="tok-str">"std.compat"</span>,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"source-path"</span>: <span class="tok-str">"../include/c++/16/bits/std.compat.cc"</span>,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"is-std-library"</span>: <span class="tok-kw">true</span></div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;}</div>
    <div>&nbsp;&nbsp;]</div>
    <div>}</div>
  </div>

  <div class="scanner-format font-mono stdlib-msvc" v-click="2">
    <div>{</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"version"</span>: 1,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"revision"</span>: 0,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"library"</span>: <span class="tok-str">"microsoft/STL"</span>,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"module-sources"</span>: [</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"std.ixx"</span>,</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-str">"std.compat.ixx"</span></div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;]</div>
    <div>}</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: section
---

# Bonus Slides

Oddities, Gotchas, and Footguns

<!--
Speaker notes go here
-->

---
layout: default
class: text-center graph-slide
---

# Interface Only Libraries

<div class="build-graph">
  <svg class="graph-lines" viewBox="0 0 868 416" preserveAspectRatio="none" aria-hidden="true">
    <defs>
      <marker id="libarr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b7280" />
      </marker>
    </defs>
    <line v-click="2" class="gline" x1="166" y1="208" x2="240" y2="208" marker-end="url(#libarr2)" />
    <line v-click="3" class="gline" x1="348" y1="192" x2="467" y2="128" marker-end="url(#libarr2)" />
    <line v-click="3" class="gline" x1="354" y1="218" x2="468" y2="254" marker-end="url(#libarr2)" />
    <line v-click="4" class="gline" x1="536" y1="146" x2="625" y2="195" marker-end="url(#libarr2)" />
    <line v-click="5" class="gline" x1="755" y1="208" x2="780" y2="208" marker-end="url(#libarr2)" />
  </svg>

  <div class="gnode gfile" style="left:120px; top:208px" v-click="1">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">sub</span>(<span class="tok-type">int</span> a,</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="glabel">math.cppm</div>
  </div>

  <div class="gnode gproc" style="left:300px; top:208px" v-click="2">
    <div class="gproc-label">Compiler</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:128px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>01001000 01100101</div>
        <div>01101100 01101100</div>
        <div>01101111 00100000</div>
        <div>01110111 01101111</div>
        <div>01110010 01101100</div>
        <div>01100100 00100001</div>
        <div>10110011 01011010</div>
        <div>11001010 00110101</div>
      </div>
    </div>
    <div class="glabel">math.cppm.o</div>
  </div>

  <div class="gnode gfile" style="left:505px; top:288px" v-click="3">
    <div class="gfile-box">
      <svg viewBox="0 0 200 260" preserveAspectRatio="none" aria-hidden="true">
        <path d="M2 2 H152 L198 48 V258 H2 Z" fill="#ffffff" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
        <path d="M152 2 V48 H198" fill="none" stroke="#9aa3ad" stroke-width="2" stroke-linejoin="round" vector-effect="non-scaling-stroke" />
      </svg>
      <div class="gfile-body">
        <div>Module</div>
        <div>&nbsp;&nbsp;Function: add</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Return: +</div>
        <div>&nbsp;&nbsp;Function: sub</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: a</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;Param: b</div>
      </div>
    </div>
    <div class="glabel">math.bmi</div>
  </div>

  <div class="gnode gproc" style="left:690px; top:208px" v-click="4">
    <div class="gproc-label">Archiver</div>
  </div>

  <div class="gnode gexec" style="left:830px; top:208px" v-click="5">
    <div class="gexec-label">math.a</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Interface Only Libraries

<div class="only-cols">
  <div class="scanner-code font-mono">
    <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
    <div>&nbsp;</div>
    <div><span class="tok-kw">export</span> <span class="tok-kw">template</span> &lt;<span class="tok-kw">typename</span> <span class="tok-type">T</span>&gt;</div>
    <div><span class="tok-type">T</span> <span class="tok-fn">add</span>(<span class="tok-type">T</span> a, <span class="tok-type">T</span> b) {</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
    <div>}</div>
    <div>&nbsp;</div>
    <div><span class="tok-kw">export</span> <span class="tok-kw">template</span> &lt;<span class="tok-kw">typename</span> <span class="tok-type">T</span>&gt;</div>
    <div><span class="tok-type">T</span> <span class="tok-fn">sub</span>(<span class="tok-type">T</span> a, <span class="tok-type">T</span> b) {</div>
    <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a - b;</div>
    <div>}</div>
  </div>

  <div class="scanner-code font-mono" v-click="1">
    <div><span class="tok-prompt">$</span> <span class="cc-driver">nm</span> <span class="cc-src">math.o</span></div>
    <div>0000000000000000 T <span class="tok-fn">_ZGIW4math</span></div>
    <div class="only-note">&nbsp;&nbsp;initializer for module math</div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Interface Only Libraries

<div class="only-cols">
  <div class="only-stack">
    <div class="scanner-code font-mono">
      <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
      <div>&nbsp;</div>
      <div><span class="tok-kw">export</span> <span class="tok-kw">template</span> &lt;<span class="tok-kw">typename</span> <span class="tok-type">T</span>&gt;</div>
      <div><span class="tok-type">T</span> <span class="tok-fn">add</span>(<span class="tok-type">T</span> a, <span class="tok-type">T</span> b) {</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
      <div>}</div>
      <div v-click="1">&nbsp;</div>
      <div v-click="1"><span class="tok-type">int</span> cppcon = <span class="tok-num">26</span>;</div>
    </div>
    <div class="scanner-code font-mono">
      <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;</div>
      <div>&nbsp;</div>
      <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
      <div>}</div>
    </div>
  </div>

  <div class="only-stack">
    <div class="scanner-code font-mono only-cmd">
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">--precompile</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-bmi">math.bmi</span></div>
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-fmodule-file=</span><span class="cc-bmi">math=math.bmi</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-exe">prog</span></div>
    </div>
    <div class="scanner-code font-mono only-err" v-click="2">
      <div>/usr/bin/ld: undefined reference to `<span class="cc-red">initializer for module math</span>'</div>
      <div>clang++: <span class="cc-red">error</span>: linker command failed with exit code 1</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: image
image: /bugzilla.png
backgroundSize: contain
---

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Partition Implementation Units

<div class="part-cols">
  <div class="part-stack">
    <div class="part-item">
      <div class="part-label font-mono">math.cppm</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div><span class="tok-kw">export</span> <span class="tok-kw">import</span> :<span class="tok-type">part</span>;</div>
      </div>
    </div>
    <div class="part-item">
      <div class="part-label font-mono">math-part.cppm</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>:<span class="tok-type">part</span>;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span>, <span class="tok-type">int</span>);</div>
      </div>
    </div>
    <div class="part-item">
      <div class="part-label font-mono">math-part.cpp</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">module</span> <span class="tok-type">math</span>:<span class="tok-type">part</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">add</span>(<span class="tok-type">int</span> a, <span class="tok-type">int</span> b) {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> a + b;</div>
        <div>}</div>
      </div>
    </div>
    <div class="part-item">
      <div class="part-label font-mono">main.cpp</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-fn">add</span>(<span class="tok-num">1</span>, <span class="tok-num">2</span>);</div>
        <div>}</div>
      </div>
    </div>
  </div>

  <div class="part-stack">
    <div class="scanner-code font-mono part-cmd">
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-c</span> <span class="cc-src">math-part.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math-part.cppm.o</span> <span class="cc-opt">&#8209;fmodule&#8209;output=</span><span class="cc-bmi">math-part.cppm.bmi</span></div>
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">&#8209;fmodule&#8209;file=</span><span class="cc-bmi">math:part=math-part.cppm.bmi</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.cppm.o</span> <span class="cc-opt">&#8209;fmodule&#8209;output=</span><span class="cc-bmi">math.cppm.bmi</span></div>
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-x c++-module</span> <span class="cc-opt">-c</span> <span class="cc-src">math-part.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">math-part.cpp.o</span> <span class="cc-opt">&#8209;fmodule&#8209;output=</span><span class="cc-bmi">math-part.cpp.bmi</span></div>
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">&#8209;fmodule&#8209;file=</span><span class="cc-bmi">math=math.cppm.bmi</span> <span class="cc-opt">&#8209;fmodule&#8209;file=</span><span class="cc-bmi">math:part=math-part.cppm.bmi</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
    </div>
    <div class="scanner-code font-mono part-err" v-click="1">
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-obj">math.cppm.o</span> <span class="cc-obj">math-part.cppm.o</span> <span class="cc-obj">math-part.cpp.o</span> <span class="cc-obj">main.o</span> <span class="cc-opt">-o</span> <span class="cc-exe">prog</span></div>
      <div>/usr/bin/ld: multiple definition of `<span class="cc-red">initializer for module math:part</span>'</div>
      <div>clang++: <span class="cc-red">error</span>: linker command failed with exit code 1</div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: default
class: text-center note-slide
---

# Partition Implementation Units

<div class="part-cols">
  <div class="part-stack">
    <div class="part-item">
      <div class="part-label font-mono">math.cppm</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">export</span> <span class="tok-kw">module</span> <span class="tok-type">math</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">import</span> :<span class="tok-type">detail</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">export</span> <span class="tok-type">point</span> <span class="tok-fn">origin</span>();</div>
      </div>
    </div>
    <div class="part-item">
      <div class="part-label font-mono">math-detail.cpp</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">module</span> <span class="tok-type">math</span>:<span class="tok-type">detail</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-kw">struct</span> <span class="tok-type">point</span> { <span class="tok-type">int</span> x, y; };</div>
      </div>
    </div>
    <div class="part-item">
      <div class="part-label font-mono">main.cpp</div>
      <div class="scanner-code font-mono part-src">
        <div><span class="tok-kw">import</span> <span class="tok-type">math</span>;</div>
        <div>&nbsp;</div>
        <div><span class="tok-type">int</span> <span class="tok-fn">main</span>() {</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">auto</span> p = <span class="tok-fn">origin</span>();</div>
        <div>&nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">return</span> <span class="tok-num">0</span>;</div>
        <div>}</div>
      </div>
    </div>
  </div>

  <div class="part-stack">
    <div class="scanner-code font-mono part-cmd">
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">-x c++-module</span> <span class="cc-opt">-c</span> <span class="cc-src">math-detail.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">math-detail.cpp.o</span> <span class="cc-opt">&#8209;fmodule&#8209;output=</span><span class="cc-bmi">math-detail.cpp.bmi</span></div>
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">&#8209;fmodule&#8209;file=</span><span class="cc-bmi">math:detail=math-detail.cpp.bmi</span> <span class="cc-opt">-c</span> <span class="cc-src">math.cppm</span> <span class="cc-opt">-o</span> <span class="cc-obj">math.cppm.o</span> <span class="cc-opt">&#8209;fmodule&#8209;output=</span><span class="cc-bmi">math.cppm.bmi</span></div>
      <div><span class="tok-prompt">$</span> <span class="cc-driver">clang++</span> <span class="cc-opt">-std=c++20</span> <span class="cc-opt">&#8209;fmodule&#8209;file=</span><span class="cc-bmi">math=math.cppm.bmi</span> <span class="cc-opt">&#8209;fmodule&#8209;file=</span><span class="cc-bmi">math:detail=math-detail.cpp.bmi</span> <span class="cc-opt">-c</span> <span class="cc-src">main.cpp</span> <span class="cc-opt">-o</span> <span class="cc-obj">main.o</span></div>
    </div>
    <div class="scanner-code font-mono part-err" v-click="1">
      <div>main.cpp:4:10: <span class="cc-red">error</span>: definition of '<span class="cc-red">point</span>' must be imported from module 'math' before it is required</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;4 | &nbsp;&nbsp;&nbsp;&nbsp;<span class="tok-kw">auto</span> p = <span class="tok-fn">origin</span>();</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cc-ans">^</span></div>
      <div>math-detail.cpp:3:8: note: definition here is <span class="cc-red">not reachable</span></div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;3 | <span class="tok-kw">struct</span> <span class="tok-type">point</span> { <span class="tok-type">int</span> x, y; };</div>
      <div>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cc-ans">^</span></div>
    </div>
  </div>
</div>

<!--
Speaker notes go here
-->

---
layout: image
image: /bug1.png
backgroundSize: contain
---

<!--
Speaker notes go here
-->

---
layout: image
image: /bug2.png
backgroundSize: contain
---

<!--
Speaker notes go here
-->

---
layout: statement
---

# Thank You

<div class="thank-question">Questions?</div>

<!--
Speaker notes go here
-->
