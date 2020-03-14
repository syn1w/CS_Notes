# Static Analysis

NJU 《static program analysis》

## 1. Intro

## 2. Rice's theorem

Any **non-trivial** property of the behavior of programs in a r.e. language is **undecidable**.



Perfect static analysis is not exist.

- Sound - false positive
- Complete - false negative

![sound&complete](../../imgs/sa/1_1.png)

Mostly compromising completeness: **Sound** but not **fully-precise** static analysis.

Ensure(or get close to soundness), while making good trade-offs between analysis precision and analysis speed.



## 4. Abstraction and over-approximation

- Concrete -> Abstraction

- over-approximation
  - Transfer functions: Transfer functions define how to evaluate different program statements on abstract values.
  - Control flows

