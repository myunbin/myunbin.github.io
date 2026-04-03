---
title: "Peep into Peephole Optimization"
date: 2026-04-03
draft: false
---

*내가 속한 연구실의 석사 과정생은 1-2주마다 글감을 궁리하고 이를 지면에 옮기는 과제가 주어진다. 논리적인 글을 작성하는 것에 익숙하지 않아 아직은 형편없는 실력이지만, 형편없음을 드러내는 부끄러움보다 점진적으로 발전하는 모습을 보여주고픈 마음이 앞서기에 원문을 옮길 예정이다.*

*~~마음이 바뀌면 지울지도 모른다. 아무도 안 볼 것 같기도 하고.~~*

### Abstract

This article discusses peephole optimizations through refinement, motivated by the *TransOpt* project. The compositionality of refinement explains why local rewrites preserve correctness, as each transformation reduces the set of possible behaviors without introducing new ones. Building on this perspective, the relationship between local correctness and optimization is further considered, raising the question of how true global optimality can be achieved beyond local rewrites.

### 본문
*TransOpt* highlights a gap between mature compilers and newer ones. Mature compilers such as LLVM provide a rich set of peephole optimizations, while newer compilers like Cranelift still lack many of them. Transplanting peephole optimizations may seem straightforward, but it raises important questions: how can we ensure that these optimizations remain correct across different compilers? Why do small, local rewrites preserve the correctness of the entire program, and what potential issues can arise?

Refinement describes transforming a system into a more specific version by reducing its set of possible behaviors without introducing new ones. In compilers, this process is closely tied to undefined behavior. While undefined behavior is often seen as a programming error, it serves a different purpose in compiler design: it provides freedom for optimization by allowing the compiler to disregard certain executions. In systems like LLVM, undefined behavior is not merely an error condition but a mechanism that propagates through the program, enabling aggressive and effective optimizations.

The correctness of peephole optimizations follows from the compositionality of refinement. Let the initial program be $f$. Suppose a small code fragment $o$ is transformed into a refined version $o'$, such that $o \rightarrow o'$, meaning that $o'$ preserves all defined behaviors of $o$ while possibly restricting its undefined ones. The program can be decomposed as $f = r(o)$, where $r$ represents the surrounding context. Since refinement is compositional, if $x \rightarrow y$, then $g(x) \rightarrow g(y)$ holds for any function $g$. Therefore, $r(o) \rightarrow r(o')$, and from $f = r(o)$ we obtain $f \rightarrow r(o')$. In this sense, replacing $o$ with $o'$ refines the entire program by reducing its set of possible behaviors without introducing new ones, which explains why local rewrites can be safely applied in practice.

So far, the discussion has focused on "peephole rewrites" rather than peephole optimization as a whole. Individual rewrites may yield local improvements but do not guarantee global optimality. Optimization is not achieved by simply accumulating locally "good" transformations, and such transformations can lead to de-optimization. This raises the question of how to achieve global optimization beyond local rewrites, under what conditions optimizations become trapped in local optima, and how to avoid them.
