# Answers to exam questions (in human language)

An English study guide covering all 45 questions, with plain-language explanations, analogies, worked examples, self-check exercises, and an exam-focus summary for every topic.

**Language:** [Українська версія](Exam-Answers-UA.md) · English

## How to study with this outline

Rereading can make a topic feel familiar without making it retrievable under exam conditions. Each question therefore follows a short learning cycle:

1. **Retrieve before reading (1–2 min).** Read the learning outcome and, without looking at the explanation, say what you already know. An error here is useful because it tells you where to focus.
2. **Study in chunks (10–15 min).** After each subsection, pause and state its main idea in one sentence.
3. **Work through the examples.** Predict the next step before revealing it. Then change a number or condition and check whether the method still works.
4. **Use active recall (3–5 min).** Close the main text and answer the recall questions. Attempt the mini-practice before expanding its answer; the hidden answer is for feedback, not for peeking.
5. **Teach it aloud (1 min).** Give a compact explanation: definition → key idea → your own example → common mistake. If you get stuck, revisit only that weak point.
6. **Review with spacing.** Return after roughly 1, 3, 7, and 14 days. Begin each review with retrieval, not another full reread.

Rate each attempt: **red** — cannot explain without the text; **yellow** — can explain but make mistakes in the example; **green** — can explain and solve a new example. Review red and yellow topics more often. Interleave related questions—for example 7–8–14 (graphs), 20–27 (distributed and cloud systems), and 37–43 (machine learning)—so that you practise choosing a method instead of recognizing it only in one familiar context.

---

## Contents

- [1. Set operations, Cartesian product. Cardinality and comparison of sets](#q1)
- [2. Binary relations: main classes and operations](#q2)
- [3. Algebraic systems and Boolean algebra: Boolean functions, normal forms, functional completeness](#q3)
- [4. Predicates and predicate calculus](#q4)
- [5. Permutations, arrangements, and combinations. Enumeration methods](#q5)
- [6. Inclusion–exclusion, recurrence relations, and generating functions](#q6)
- [7. Graphs: vertices and edges, adjacency and incidence, connectivity, paths and cycles](#q7)
- [8. Eulerian and Hamiltonian graphs. Trees, planar graphs, coloring, networks and flows](#q8)
- [9. Formal languages and automata. Grammars, finite automata, and pushdown automata](#q9)
- [10. Data structures: stack, queue, heap, tree, graph, and hash table](#q10)
- [11. The concept and properties of algorithms. Recursive functions, Turing machines, and Markov algorithms](#q11)
- [12. Sorting algorithms and QuickSort](#q12)
- [13. Dynamic programming and greedy algorithms](#q13)
- [14. Graph algorithms](#q14)
- [15. Programming languages: procedural and problem-oriented. Syntax and semantics](#q15)
- [16. Language processors: compilers and interpreters. Stages of translation](#q16)
- [17. Programming methods and OOP. Modularity, classes and objects, encapsulation, inheritance, and polymorphism](#q17)
- [18. Structured, functional, and logic programming](#q18)
- [19. Software specification, verification, and testing](#q19)
- [20. Distributed computing: transparency, openness, flexibility, and extensibility](#q20)
- [21. MapReduce: the Map, shuffle, and Reduce stages](#q21)
- [22. Hadoop: principles, HDFS, YARN, MapReduce, non-relational data, and use cases](#q22)
- [23. Distributed information-processing environments, data warehousing, and interaction models](#q23)
- [24. High-load systems and high-performance computing. HPC architectures, parallelism, and multithreading](#q24)
- [25. Service-oriented architecture (SOA): request–response and publish–subscribe](#q25)
- [26. Software agents and multi-agent systems. Distributed applications with SOAP and REST](#q26)
- [27. Distributed computing infrastructure and cloud systems (IaaS/PaaS/SaaS). Cloud deployment](#q27)
- [28. Mathematical modeling: principles, parameter estimation, adequacy, validation, and verification](#q28)
- [29. Simulation models: discrete events, Monte Carlo, Petri nets, system dynamics, and multi-agent simulation](#q29)
- [30. Systems analysis, complex systems, decomposition, aggregation, and perturbation methods](#q30)
- [31. System optimization: linear and nonlinear programming, optimality, constraints, and search methods](#q31)
- [32. Discrete and Boolean optimization, Gomory cuts, branch and bound, and metaheuristics](#q32)
- [33. Multi-objective optimization, Pareto-optimal solutions, minimax, and criterion aggregation](#q33)
- [34. Decision theory, utility, preferences, AHP, uncertainty, and risk](#q34)
- [35. Decision-making under conflict: game theory, collective choice, and Bayesian networks](#q35)
- [36. Knowledge-based systems, knowledge representation and inference, semantic networks, frames, rules, and ontologies](#q36)
- [37. Artificial neural networks, activation, backpropagation, recurrent networks, Hopfield networks, and Hamming networks](#q37)
- [38. Self-organizing and competitive-learning networks, SOM, stochastic adaptation, and CNNs](#q38)
- [39. Machine learning, empirical risk, overfitting, and the bias–variance trade-off](#q39)
- [40. Supervised learning, cross-validation, regularization, SVMs, and kernel methods](#q40)
- [41. Unsupervised learning, clustering, PCA, reinforcement learning, and Q-learning](#q41)
- [42. Fuzzy systems, membership functions, fuzzy inference, defuzzification, and fuzzy neural networks](#q42)
- [43. Fuzzy clustering: Fuzzy C-Means and Gustafson–Kessel](#q43)
- [44. Intelligent distributed information systems, information retrieval, the Semantic Web, and agents](#q44)
- [45. Applications: Data Mining, BI, image processing, computer vision, NLP, and decision support](#q45)

---

<a id="q1"></a>

## 1. Set operations, Cartesian product. Cardinality and comparison of sets

> **Learning outcome.** After this chapter, you will be able to perform operations on sets, construct the Cartesian product, and compare cardinalities using mappings. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Think of a set as a bag of objects: there are no duplicates, and their order does not matter. The set {1, 2, 3} is therefore the same as {3, 2, 1}. Your Telegram contacts form one set and your Instagram contacts form another. Set theory answers simple questions: who is in both lists, who is in at least one, and who is in exactly one.

We say “x belongs to A” when x is in the set and “x does not belong to A” otherwise. The empty set is a set with no elements. We say that A is a subset of B when every element of A is also in B; for example, “classmates who are your friends” is a subset of “all classmates.”

### Basic operations

Let A = {1, 2, 3}, B = {3, 4}, and the "universe" of the problem be numbers from 1 to 5. Then:

| Operation | Meaning | Result |
|---|---|---|
| Union | everything that is in at least one of the sets | {1, 2, 3, 4} |
| Intersection | only what is in both at the same time | {3} |
| Difference A \\ B | what is in A but not in B | {1, 2} |
| Symmetric difference | what is in exactly one of the two sets | {1, 2, 4} |
| Complement of A | everything from the "universe" that is not in A | {4, 5} |

A symmetric difference means “in one set or the other, but not both”: contacts who appear in exactly one of the two apps.

It is also important to know the **power set**—the set of all subsets. A set with n elements has exactly 2ⁿ subsets because each element independently has two possibilities: include it or exclude it. For example, {1, 2} has four subsets: ∅, {1}, {2}, and {1, 2}. The empty set and the set itself always belong to the power set.

### Laws of operations

Set operations behave much like ordinary arithmetic: union and intersection are commutative and associative. The most useful laws are **De Morgan's laws**: the complement of a union equals the intersection of the complements, and vice versa. In everyday language, “it is not true that I went to the cinema OR the theatre” means “I did not go to the cinema AND I did not go to the theatre.”

A quick check: A ∪ B = {1, 2, 3, 4}, so its complement is {5}. The complement of A is {4, 5}, the complement of B is {1, 2, 5}, and their intersection is again {5}. Both sides agree.

### Cartesian product

The Cartesian product A × B is the set of all ordered pairs whose first component comes from A and second component comes from B. Here order matters: (1, 2) is not the same as (2, 1). A chessboard is a familiar example because each square is identified by a file and rank; 8 choices for each component produce 8 · 8 = 64 squares. In general, |A × B| = |A|·|B| for finite sets.

If A = {1, 2} and B = {x, y}, then A × B contains (1,x), (1,y), (2,x), and (2,y). In general, B × A is a different set because the components are reversed. The idea extends to tuples of any length; the Cartesian plane is the set of ordered pairs of real numbers.

### Cardinality and comparison of sets

Cardinality means “how many elements a set contains.” For finite sets, we simply count. Infinite sets are compared using mappings:

- **injection** — distinct inputs have distinct images;
- **surjection** — every element of the target set is reached;
- **bijection** — both properties hold, giving a one-to-one correspondence.

Two sets have the same cardinality, or are equinumerous, if a bijection exists between them. Imagine seating guests without counting: if every guest gets one chair and no chair remains empty, the two sets have equal size.

**A countable set** is one that can be numbered with natural numbers. A surprise: integers are countable, although it "seems" that there are twice as many as natural numbers. We number them like this: 0, 1, −1, 2, −2, 3, −3 and so on — each integer gets its own number. Even fractions (rational numbers) are countable.

Real numbers, however, are uncountable. Their cardinality is called the **continuum**, and uncountability can be proved by **Cantor's diagonal argument**: suppose all real numbers in [0, 1] appear in an infinite list. Construct a new number by changing the first digit of the first number, the second digit of the second number, and so on. The constructed number differs from every listed number in at least one digit, so it is missing from the supposedly complete list—a contradiction.

Two theorems worth remembering in words:

- **Cantor's Theorem**: a power set always has strictly greater cardinality than the original set - therefore there is no "greatest" infinity, the hierarchy of cardinalities is infinite.
- **Cantor–Bernstein theorem**: if A "fits" into B without gluing and B "fits" into A, then they are equivalent. It is convenient to prove equality with two "inequalities".


### Reinforcement

**Additional worked example.** Let A = {1, 2, 3}, B = {3, 4}. Then A ∪ B = {1, 2, 3, 4}, A ∩ B = {3}, A \ B = {1, 2}, and A × B contains 3 · 2 = 6 ordered pairs. For example, (1, 3) belongs to A × B, but (3, 1) may not belong to it, because the order of the components is important.

**Transfer example.** In a database, the set of students has 120 elements and the set of courses has 8. The set of all possible student-course pairs has 120 · 8 = 960 elements, although the number of actually registered pairs may be smaller.

**Active recall.** Close the explanation above and answer without peeking: How does an element of a set differ from one of its subsets? Why do injections A → B and B → A imply that A and B have the same cardinality?

**Mini practice — check yourself.** For U = {1, 2, 3, 4, 5}, A = {1, 3, 5}, B = {2, 3, 4}, find the complement of A in U and the symmetric difference of A and B.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** U \ A = {2, 4}; A △ B = {1, 2, 4, 5}. Element 3 is removed from the symmetric difference because it is common.

</details>

**Exam focus:**

- Set difference and the Cartesian product are NOT commutative: A minus B is not the same as B minus A, and the pairs in the product depend on the order.
- An empty set is a subset of any set, and its power set is not empty: it contains one element, the empty set itself.
- Don't confuse "belongs" (the element lies in the bag) and "is a subset" (the entire small bag fits into the big one): 1 belongs to {1, 2}, but {1} is a subset of {1, 2}.
- Rational numbers are countable, although "dense"; uncountability begins with reals.

---

<a id="q2"></a>

## 2. Binary relations: main classes and operations

> **Learning outcome.** After this chapter, you will be able to recognize properties of binary relations, distinguish equivalence from order, and perform inversion and composition. First, try to explain the topic in your own words, and then test yourself with the block at the end.

A relation is simply a formal way of writing the relationship between objects: "x is a friend of y", "x divides y", "x is less than y". Technically, you take all the pairs (x, y) for which the relationship is fulfilled, add them into a set - and this is the relationship. That is, the binary relation between A and B is any subset of the direct product of A by B. For example, "less than" on the set {1, 2, 3} is three pairs: (1,2), (1,3), (2,3).

It is convenient to draw the relationship: either with a matrix (one in the cell, if there is a pair), or with arrows between the points - then it is a directed graph. This is not just a picture: half of the problems are solved by looking at such a picture.

### Properties of relations

There are "character traits" in relationships, and you need to be able to recognize them:

| Property | Intuition | Example |
|---|---|---|
| Reflexivity | everyone is connected with himself | "equal to", "not more than" |
| Irreflexivity | no one is related to himself | "strictly less", "to be a father" |
| Symmetry | communication is always two-way | "to be a relative" |
| Antisymmetry | two-way communication is possible only "with oneself" | "no more", divisibility |
| Transitivity | communication is transmitted through the chain | "ancestor", "lesser" |
| Connectivity | any two elements are comparable | "no more" on the numbers |

Let's analyze "x divides y" on the set {1, 2, 3, 4, 6}. Each number divides itself — there is reflexivity. If a divides b, and b divides a, then it is the same number - there is an antisymmetry. The chain also works: 2 divides 4... if a divides b, and b divides c, then a divides c — there is transitivity. But there is no connection: 4 and 6 are not connected in any way - neither divides the other.

### Main classes

- **Equivalence relation** is reflexive, symmetric and transitive together. This is "generalized equality": the same remainder when dividing, similarity of triangles, "studying in the same group." The main effect: equivalence cuts a set into **classes** — groups of mutually "identical" elements that either coincide or do not intersect at all. Example: the numbers from 0 to 6 fall into exactly three groups — {0, 3, 6}, {1, 4}, and {2, 5}.
- **Partial order** — reflexive, antisymmetric and transitive. This is "generalized less-than-or-equal to": inclusion of sets, divisibility. "Partial" - because not all pairs are comparable (like 4 and 6 in the example above). Such orders are drawn with a **Hasse diagram**: smaller elements at the bottom, only "immediate neighbors", without loops and unnecessary arrows. For divisibility by {1, 2, 3, 6}, a rhombus is obtained: 1 at the bottom, 2 and 3 above it, and 6 at the top.
- **Linear order** is a partial order where all are comparable: the usual "no more" on numbers.
- **Strict order** is irreflexive and transitive, like the usual "strictly less".
- **Function** is a relation where each x corresponds to exactly one y. It can be injective (does not glue), surjective (covers everything) and bijective (both).

### Operations on relations

Since the relation is a set of pairs, all the usual operations are applicable to it: union, intersection, difference. But there are also their own, specific ones.

**Composition** means following two relations in sequence: first one relation, then the other. For example, if R means "is a parent of," then composing R with itself gives "is a grandparent of": x is a parent of y, and y is a parent of z. Numerically, if R contains (1,2) and (2,3), and S contains (2,5) and (3,6), then following R and then S gives (1,5) and (2,6). Composition is generally non-commutative, so the order matters.

**Inversion** - simply reverse all the arrows: inverted to "father" - "child", inverted to "less" - "bigger".

**Closure** is the minimal "completion" of a relation to the desired property: reflexive closure adds all loops, symmetric closure adds all back arrows, and transitive closure adds all "chain reaches". For example, if R contains (1,2) and (2,3), then the transitive closure will add (1,3), because from 1 you can get to 3 through 2. Computed by Warshall's algorithm in cubic time: a triple loop that asks "is it possible to get from i to j via an intermediate vertex k."


### Reinforcement

**Additional worked example.** On the set {1, 2, 3}, the relation "have the same parity" contains, in particular, (1, 1), (1, 3), (3, 1). It is reflexive, symmetric, and transitive, so it is an equivalence and divides the set into the classes {1, 3} and {2}.

**Transfer example.** The relation "file lies inside a directory" is asymmetric. Its transitive closure gives all ancestors: if a file is in directory A, and A is in B, then the file lies indirectly in B.

**Active recall.** Close the explanation above and answer without peeking: What are the three properties required for equivalence? Why is "≤" antisymmetric even though pairs can exist in both directions?

**Mini practice — check yourself.** For R = {(1, 2), (2, 3)} write R⁻¹ and the smallest transitive closure of R.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** R⁻¹ = {(2, 1), (3, 2)}; (1, 3) should be added to the transitive closure.

</details>

**Exam focus:**

- "Not symmetric" is not the same as "antisymmetric": the relationship can be neither one nor the other, and "equal" is both symmetric and antisymmetric.
- Irreflexivity is not merely the absence of reflexivity: non-reflexive means that at least one required loop is missing, while irreflexive means that no element is related to itself.
- Equivalence classes cannot partially overlap: they are either identical or disjoint.
- The composition is non-commutative: "friend's father" and "father's friend" are different people.

---

<a id="q3"></a>

## 3. Algebraic systems and Boolean algebra: Boolean functions, normal forms, functional completeness

> **Learning outcome.** After this chapter, you will be able to construct truth tables and normal forms and explain the functional completeness of a set of Boolean operations. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Boolean algebra is a kind of arithmetic in which values are limited to 0 (false) and 1 (true), and the main operations are AND, OR, and NOT. Digital circuits and every `if` condition in code rely on this logic. Normal forms are standard patterns into which logical formulas can be rewritten, while functional completeness asks which sets of operations can express every Boolean function.

### Algebraic systems

The general framework is as follows: you take a carrier set, a set of operations on it and a set of relations — this is an algebraic system. If there are only operations, it is algebra: a group (one "good" operation with a neutral element and inverses), a ring, a field, Boolean algebra. If there is only a relation, it is a relational model, and there are relational databases on it: a relation is a table, a tuple is a row, and the operations of relational algebra are selection of rows (selection), selection of columns (projection), gluing of tables (join).

### Boolean functions

A Boolean function takes n bits and returns one bit. There are 2^(2ⁿ) Boolean functions of n variables: there are 2ⁿ possible input combinations, and the function may independently output either 0 or 1 for each one. For two variables, this gives 2⁴ = 16 functions.

Main operations are NOT (negation), AND (conjunction), OR (disjunction), implication ("if …, then …"), equivalence ("if and only if"), XOR (addition modulo 2—true when exactly one input is true), and NAND and NOR—the negations of AND and OR. Each operation is described by a **truth table**, which lists its value for every input combination:

| x | y | x AND y | x OR y | x → y | XOR |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | **1** | 0 |
| 0 | 1 | 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 | 0 |

The trickiest case is implication with a false antecedent: in classical logic, a false antecedent makes the implication true. Thus an implication is false only when its antecedent is true and its consequent is false. Two formulas are equivalent if they have the same value on every truth-table row. The key identity is x → y ≡ ¬x ∨ y.

### Normal forms

- **DNF** — a formula of the form "(...and...) OR (...and...)": disjunction of conjunctions.
- **PDNF** (perfect DNF) is built mechanically from the truth table: you take those rows where the function is equal to 1, for each you write the conjunction of all variables (with negation where the variable in the row is zero) and connect everything through OR.
- **PCNF** — a mirror construction on zero lines: a conjunction of disjunctions.
- **Zhegalkin polynomial** — a representation using XOR and conjunction without negation; for each function it is unique, for example the negation of x is "1 XOR x".
- Formulas are shortened by minimization: Karnaugh maps or the Quine–McCluskey method.

Consider the PDNF of implication. Its truth table has output 1 in three rows: (0,0), (0,1), and (1,1). Write one conjunction for each row: "not x and not y," "not x and y," and "x and y." Joining them with OR gives the PDNF. Simplifying it produces the familiar formula "not x or y."

### Completeness

A set of functions is **functionally complete** if every Boolean function can be expressed using functions from that set. The set {NOT, AND, OR} is complete; in fact, {NOT, AND} and {NOT, OR} are also complete. Even NAND alone is complete, so any Boolean circuit can be built entirely from NAND gates.

How do we check completeness? **Post's theorem** identifies five closed classes: functions that preserve 0, functions that preserve 1, self-dual functions, monotone functions, and affine (linear) functions. A set of Boolean functions is functionally complete if and only if, for each of these five classes, the set contains at least one function outside that class.

Why is NAND complete by itself? NAND preserves neither 0 nor 1 and is neither monotone, self-dual, nor affine. Thus this single function lies outside all five of Post's closed classes.


### Reinforcement

**Additional worked example.** For f(x, y) = x XOR y, the 1-rows are (0, 1) and (1, 0). Therefore its PDNF is (¬x ∧ y) ∨ (x ∧ ¬y). It explicitly lists every input combination for which the result is 1.

**Transfer example.** In an access-control scheme, the rule "admin OR (owner AND not locked)" is a Boolean function. Its truth table reveals a case that is easy to overlook in code: an admin passes the check even when the resource is locked because the first condition is sufficient.

**Active recall.** Close the explanation above and give the answer without peeking: Which truth-table rows are used to build PDNF and PCNF? Why are AND and OR operations alone not enough for a complete basis?

**Mini practice — check yourself.** Write a shorter formula for ¬(x → y) and check it on four sets of values.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Since x → y ≡ ¬x ∨ y, the negation is x ∧ ¬y. It is true only for x = 1, y = 0.

</details>

**Exam focus:**

- PDNF is built from rows where the function equals 1; PCNF is built from rows where it equals 0. This distinction is a common source of mistakes.
- For the identically false function there are no 1-rows, and for the identically true function there are no 0-rows; depending on convention, the corresponding canonical form is represented as an empty disjunction or conjunction.
- The set with only AND and OR is NOT complete: both operations are monotonic, negation cannot be expressed with them.
- The Zhegalkin polynomial is unique for each function — it is convenient to check linearity.
- An implication is false in only one case: true condition, false conclusion.

---

<a id="q4"></a>

## 4. Predicates and predicate calculus

> **Learning outcome.** After this chapter, you will be able to translate sentences in the language of predicates, negate quantifiers correctly, and see the dependence of variables on the order of quantifiers. First, try to explain the topic in your own words, and then test yourself with the block at the end.


A proposition is a complete statement that is either true or false, such as "2 is prime." A predicate is a statement with variables, such as "x is prime." Before x is assigned a value, this expression does not yet have a truth value; substituting 7 makes it true, while substituting 8 makes it false. Formally, a predicate maps elements of a domain to truth values. Its number of arguments is called its arity: "x is prime" is unary, while "x is less than y" is binary.

### Quantifiers

Quantifiers are a way to close holes for the entire set at once, without iterating over each value individually:

- **universal quantifier** — "for all x, P holds": in essence, a big AND over the entire set;
- **existential quantifier** — "there is at least one x for which P": big OR.

This is easy to see on a finite set. Take {2, 3, 4} and the predicate "x is even." "Every x is even" is false because 3 is a counterexample. "There exists an even x" is true because 2 is one such value.

A variable covered by a quantifier is called bound; without a quantifier - free. A formula without free variables is closed: it is already a complete statement with a specific truth value.

### Main rules for working with quantifiers

The most important rule is the **negation of quantifiers** (an analogue of De Morgan's laws): ¬∀x P(x) ≡ ∃x ¬P(x), and ¬∃x P(x) ≡ ∀x ¬P(x). In words, "not everyone passed" means "someone did not pass," while "nobody passed" means "everyone did not pass." The negation moves inward and the quantifier changes to the other kind.

The second key idea is the **order of quantifiers**. Consecutive quantifiers of the same kind can be exchanged, but exchanging universal and existential quantifiers can change the meaning. Consider "y is greater than x" over the integers:

- "for every x there is a y greater than it" is true: add one to any integer, and every x has a suitable y;
- "there is a y greater than every x" is false: the largest integer does not exist.

In the first statement, y may depend on x; in the second, one fixed y must work for every x.

Third, the universal quantifier distributes over AND, and the existential quantifier distributes over OR, but the reverse pairings do not generally work. For example, "there exists an even number, and there exists an odd number" does not imply that one number is both even and odd.

### Predicate calculus (first-order logic)

Predicate calculus is a formal "rule game" for mechanical proof. It contains an alphabet (variables, predicate and functional symbols, relations, quantifiers), rules for constructing formulas, axioms and two rules of derivation: **modus ponens** (from "A" and "if A, then B" we deduce "B") and a rule of generalization (from the proven "A(x)" we deduce "A is true for all x"). A typical quantifier axiom in words: if something is true for all, then it is true for any particular element.

For a formula to have a meaning, it needs an **interpretation**: a domain of discourse and meanings for its non-logical symbols. A formula that is true in every interpretation is logically valid; one that is true in at least one interpretation is satisfiable.

Two major results must be distinguished:

- **Gödel's completeness theorem**: a formula is derived according to the rules if and only if it is logically valid. That is, mechanical proof and "truth everywhere" are the same thing: syntax coincides with semantics.
- **Church's theorem:** first-order logical validity is undecidable—there is no algorithm that always determines, in finite time, whether an arbitrary first-order formula is logically valid. In contrast, propositional formulas can always be decided by a finite truth table.

Practical application: formulas lead to prenex form (all quantifiers are brought forward), then get rid of existence quantifiers through Skolem functions (instead of "for every x there exists y" write "y is a function of x") - and this is what the resolution method works on, that is, automatic theorem proving and the Prolog language.


### Reinforcement

**Additional worked example.** The statement "every student has read some book" is written as ∀s ∃b Read(s, b). The book may be different for each student. The formula ∃b ∀s Read(s, b) is stronger: it requires one shared book for all.

**Transfer example.** The negation of the requirement "all requests succeeded" is not "all failed" but "there is at least one failed request". This is the kind of counterexample the test is looking for.

**Active recall.** Close the explanation above and answer without peeking: How do ∀ and ∃ change under negation? Give your own example where permuting different quantifiers changes the truth.

**Mini practice — check yourself.** Negate the formula ∀x (P(x) → ∃y Q(x, y)), without leaving a negation directly before a quantifier or implication.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** ∃x (P(x) ∧ ∀y ¬Q(x, y)). There is an x with property P for which no y satisfies Q(x, y).

</details>

**Exam focus:**

- Under negation, the quantifier CHANGES to the other kind: "not for every x" means "there exists an x for which not," whereas "there does not exist an x" means "for every x, not."
- You cannot change the order of quantifiers with different names - keep an example with a "larger number" ready.
- Do not confuse completeness (Gödel: derivability coincides with logical validity) with decidability (Church: no decision algorithm exists for first-order validity); they are different properties.
- "First order" - because quantifiers are attached only to elements, not to predicates or sets.
- The two statements "there is an x with property P" and "there is an x with property Q" do not guarantee that the same x satisfies both properties.

---

<a id="q5"></a>

## 5. Permutations, arrangements, and combinations. Enumeration methods

> **Learning outcome.** After this chapter, you will be able to choose the correct calculation formula based on two features: whether the order is important and whether repetitions are allowed. First, try to explain the topic in your own words, and then test yourself with the block at the end.


Combinatorics studies the question "How many ways?" It counts ways to arrange books on a shelf, choose three friends for a trip, or create a PIN. Before choosing a formula, ask two questions: does the **order** matter, and are **repetitions** allowed? The answers usually determine the correct method.

First, two basic rules on which everything else rests. **Rule of sum**: if you have to choose something "either here or there", the number of options is added. There are three soups and four salads on the menu - you can choose one dish in seven ways. **Rule of product**: if the selection is made sequentially, step by step, the quantities are multiplied. Three shirts and four pants make twelve sets, because every shirt goes with any pair of pants.

We also need one notation: n! ("n factorial") is the product of the integers from 1 through n. For example, 5! = 120. By convention, 0! = 1; this makes the standard formulas work consistently at their boundary cases.

Now the three main characters:

- A **permutation** arranges all n distinct objects in order. There are n! permutations. Three books A, B, and C can be placed on a shelf in six ways: ABC, ACB, BAC, BCA, CAB, and CBA, which is exactly 3! = 6. There are three choices for the first position, two for the second, and one for the third, so we multiply 3 · 2 · 1.
- **Arrangement** — we choose k items from n, and the order is important. A classic example: out of five runners, give out gold and silver medals. "Ivan is gold, Peter is silver" and vice versa - these are different results. 5·4 = 20 ways: five candidates for gold, then four for silver.
- A **combination** chooses k objects from n when order does not matter: only which group is selected matters, not the roles or positions. The formula is:

  **C(n,k) = n! / (k!·(n−k)!)**

  It is read "n choose k" and is called a binomial coefficient. In other words, we count all ordered selections and divide by the k! ways to reorder the selected items, because those reorderings represent the same combination.

Let's count on our fingers: how many ways are there to choose two people out of five for the committee? C(5,2) = 120 / (2·6) = 10. You can check by counting: for people a, b, c, d, e, the pairs are ab, ac, ad, ae, bc, bd, be, cd, ce, de. Exactly ten. And there would be twenty arrangements - twice as many, because each pair is counted there in two orders. This is, by the way, a useful checking identity: arrangements = combinations · k!.

If repetitions are allowed, the picture changes:

- **Arrangements with repetition**: each of the k positions independently has n choices, so there are nᵏ arrangements. A four-digit PIN has 10⁴ = 10,000 possibilities.
- **Permutations with repetitions**: when some objects are identical, divide n! by the factorial of each multiplicity. For example, the letters of MAMA can be arranged in 4! / (2!·2!) = 6 distinct ways because the two M's are identical and the two A's are identical.
- **Combinations with repetitions** are counted by the **stars-and-bars** method: distributing k identical candies among n pockets can be represented by k stars and n−1 bars, and we count the resulting strings using combinations. The result is C(n+k−1, k). For example, choosing three cakes from two types gives four possibilities: 3+0, 2+1, 1+2, or 0+3.

Binomial coefficients have several properties that are worth knowing by heart. **Symmetry**: choosing k people to take is the same as choosing n−k not to take, so C(5,2) = C(5,3). **Pascal's rule**: let's look at a specific person - he is either included in the sample or not; from here, each coefficient is equal to the sum of the two adjacent ones from above, and the famous Pascal triangle grows:

```
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
1 5 10 10 5 1   ← therefore C(5,2)=10
```

The sum of the entries in row n of the triangle is 2ⁿ. This is no coincidence: 2ⁿ is the number of all subsets of an n-element set because each element is independently included or excluded. These coefficients also appear in binomial expansions: in (a+b)³ = a³ + 3a²b + 3ab² + b³, the coefficients 1, 3, 3, 1 form a row of Pascal's triangle. This is the **binomial theorem**.

In addition to direct formulas, there are other enumeration methods. The **bijective method** replaces a difficult counting problem with a one-to-one correspondence to a simpler set; because the correspondence is bijective, the two sets have the same cardinality. This is the idea behind stars and bars. **Inclusion-exclusion, recurrences, and generating functions** are separate major topics (question 6). **Burnside's lemma** counts objects "up to symmetry"—for example, necklaces considered identical under rotation. The **pigeonhole principle** does not count objects directly but proves existence: if more objects are placed into fewer boxes, some box contains at least two objects. Thus, among thirteen people, at least two were born in the same month.


### Reinforcement

**Additional worked example.** The chairman, secretary and treasurer are chosen from 7 candidates. The roles are different, so the order is important: A(7, 3) = 7 · 6 · 5 = 210. If we simply choose a commission of three people, we have C(7, 3) = 35.

**Transfer example.** A four-digit PIN has 10⁴ = 10,000 options, because repetitions are allowed and each position is important. If all the digits must be different, there are 10 · 9 · 8 · 7 = 5040 options.

**Active recall.** Close the explanation above and answer without peeking: What two questions should be asked before choosing a formula? Why is 0! defined to equal 1?

**Mini practice — check yourself.** In how many ways can you choose 2 fillings out of 6 without repetition, if the order does not matter? And if the first filling is the main one, the second one is additional?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Without roles: C(6, 2) = 15. With roles: A(6, 2) = 6 · 5 = 30.

</details>

**Exam focus:**

- The main trap is to confuse arrangements and combinations. The saving question: "if you swap the chosen ones, is this a different result?" Yes means an arrangement; no means a combination.
- 0! = 1, so "select zero elements" or "select all" is always one way.
- Arrangements = combinations · k! — quick check of your answer.
- "How many subsets are there in a set of n elements?" — the answer is 2ⁿ, not n!.

---

<a id="q6"></a>

## 6. Inclusion–exclusion, recurrence relations, and generating functions

> **Learning outcome.** After this chapter, you will be able to apply inclusion–exclusion, form recurrence relations, and read the coefficients of the generating function. First, try to explain the topic in your own words, and then test yourself with the block at the end.


These are three powerful tools for calculating when the direct formulas from the previous question won't work.

### Inclusion–exclusion

The idea is easiest to see in people. Let's count how many students in the group know English or German. If you simply add "English speakers" and "German speakers", then those who know both languages ​​will get into the sum twice. Therefore, they must be subtracted once. This is the formula for two sets:

**|A or B| = |A| + |B| − |A and B|**

For three sets, the logic continues: we add all three individually, subtract all three pairwise intersections, and then add back the triple intersection. An element in all three sets was added three times and subtracted three times, so without the last term it would not be counted at all. In general, intersections of an odd number of sets are added and intersections of an even number of sets are subtracted, so that every element is counted exactly once.

Analyzed example: how many numbers from 1 to 100 are divisible by 2 or 3? 50 numbers are divisible by 2, 33 by 3. But numbers that are multiples of six (16 of them) were included in both lists. Answer: 50 + 33 − 16 = 67. Without subtracting the number 6, 12, 18 and so on would be counted twice.

One classic application is counting **derangements**: permutations in which no element remains in its original position. Imagine that a secretary randomly places n letters into n addressed envelopes: in how many arrangements does nobody receive their own letter? Inclusion–exclusion gives Dₙ = n!·Σₖ₌₀ⁿ(−1)ᵏ/k!, so the probability Dₙ/n! approaches 1/e ≈ 0.37. For three elements there are exactly two derangements, 231 and 312. Euler's totient function φ(n), which counts the integers from 1 to n that are coprime to n, and the number of surjections can also be derived using inclusion–exclusion.

### Recursive methods

Recurrence is when the answer for n is built from the answers for smaller values, like a ladder: to stand on step n, one must already stand on the previous ones. The most famous example is the **Fibonacci numbers**: each number is the sum of the two previous ones, starting with 0 and 1, then 1, 2, 3, 5, 8, 13... In order for the sequence to be determined unambiguously, in addition to the rule itself, **initial conditions** are needed - the first terms from which everything unfolds.

For linear recurrences with constant coefficients (when the new term is simply the sum of the previous ones with numerical multipliers), there is a standard recipe - **characteristic equation**. Idea: we guess that the solution looks like "something to the power of n", substitute and see which bases of the power are suitable. I will show by example: aₙ = 5aₙ₋₁ − 6aₙ₋₂, first terms 1 and 4.

1. Write the characteristic equation: x² − 5x + 6 = 0 (powers have been replaced by x², x and unit).
2. Roots: 2 and 3, they are different - lucky.
3. So, the answer is a mixture of two and three to the n power: aₙ = A·2ⁿ + B·3ⁿ, where A and B are still unknown.
4. Substitute the first terms: for n=0 we have A + B = 1, for n=1 — 2A + 3B = 4. Hence B = 2, A = −1.
5. Answer: aₙ = −2ⁿ + 2·3ⁿ. Check a₂, the third listed term because indexing starts at 0: the formula gives −4 + 18 = 14, and the recurrence gives 5·4 − 6·1 = 14.

Two nuances. If the characteristic equation has a repeated root r, one power is not enough: the corresponding solution has the form (A + Bn)rⁿ, which provides enough freedom to fit both initial conditions. If the recurrence has an extra nonhomogeneous term, such as “+3” at each step, its solution is the general homogeneous solution plus one particular solution. Separately, the **Master Theorem** estimates the running time of divide-and-conquer recurrences such as T(n) = aT(n/b) + f(n).

### Generating functions

A generating function is a "suitcase" for an infinite sequence of numbers: all terms are packed into one expression

**G(x) = a₀ + a₁x + a₂x² + ...**

where x acts as a position marker: the coefficient of x⁵ is a₅, the term with index 5 and the sixth term when counting from a₀. We usually treat the series formally, so convergence is not the immediate concern. Operations on sequences then become ordinary algebra: multiplying by x shifts the sequence, while multiplying two generating functions forms the convolution of their coefficients. That is why generating functions are useful for coin-change problems.

Mini-example: how many ways to collect 4 hryvnias with coins of 1 and 2 hryvnias each? For hryvnia coins, we write a polynomial, where the degree is how many hryvnias were collected with them: 1 + x + x² + x³ + x⁴. For two coins — 1 + x² + x⁴. We multiply and look at the coefficient at x⁴: it is equal to 3. We check with our hands: 1+1+1+1, 1+1+2, 2+2 are really three ways.

Recurrences can also be solved with generating functions: multiply the recurrence relation by xⁿ, sum over n, solve the resulting algebraic equation for the generating function, decompose it into partial fractions, and read off the coefficients. This is one way to derive the explicit formula for Fibonacci numbers. Another important example is the Catalan numbers (1, 1, 2, 5, 14...), which count balanced parenthesis sequences and binary trees: for example, three pairs of parentheses can be arranged correctly in exactly five ways.


### Reinforcement

**Additional worked example.** In a group of 30 students, 18 know Python, 14 know Java, and 7 know both. At least one language is known by 18 + 14 − 7 = 25 students, so 30 − 25 = 5 know neither. We subtract the intersection because it was counted twice when adding.

**Transfer example.** The number of binary strings of length n without consecutive 1 bits satisfies aₙ = aₙ₋₁ + aₙ₋₂: a valid string ends either in 0 or in 01. From the base cases a₁ = 2 and a₂ = 3, we get 2, 3, 5, 8, ...

**Active recall.** Close the explanation above and answer without peeking: When does an inclusion–exclusion intersection add back? What must be specified together with the recurrence relation?

**Mini practice — check yourself.** Among the numbers from 1 to 100, how many are divisible by 2 or 5?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** 50 is divisible by 2, 20 by 5, 10 by 10, so 50 + 20 − 10 = 60.

</details>

**Exam focus:**

- In inclusion-exclusion, signs strictly alternate; forgetting the last term is a typical mistake.
- “Not divisible by any” means taking the complement of the union: N − |A₁ ∪ ··· ∪ Aₖ|.
- With a repeated root r of the characteristic equation, use the term (A + Bn)rⁿ.
- A favorite question: what does the probability of a derangement approach? It approaches 1/e, about 0.37.
- About the convergence of the generating function, we can honestly say: "we consider the series formally."

---

<a id="q7"></a>

## 7. Graphs: vertices and edges, adjacency and incidence, connectivity, paths and cycles

> **Learning outcome.** After this chapter, you will be able to read basic graph terminology, calculate vertex degrees, and identify paths, cycles, and connected components. First, try to explain the topic in your own words, and then test yourself with the block at the end.

A graph is simply "dots and lines": cities and roads between them, people and friendships on social networks, pages and links. Points are called **vertices**, lines are called **edges**. The whole theory of graphs is about how to answer practical questions according to such a scheme: is it possible to get there, what is the shortest path, what will break if one road is removed.

Formally, a graph is a pair: a set of vertices and a set of edges, where each edge connects two vertices. If the roads are two-way, the graph is **undirected**; if the edges are unidirectional arrows, the graph is **directed** (digraph). There are also more exotic variants: **multigraph** allows several edges between the same pair of cities, **pseudograph** - loops (the road from the city to it), and in the **weighted** graph each road is assigned a number - length or cost.

Two terms that are constantly confused, even though they are simple. **Adjacency** is the relationship between two vertices: they are adjacent if they are connected by an edge, i.e. "neighbors". **Incidence** is the relationship between a vertex and an edge: an edge is incident to its two ends. It is easy to remember: adjacency is "vertex with vertex", incidence is "vertex with edge". Edges are also adjacent to each other - when they have a common end.

**The degree of a vertex** is the number of edges attached to it (the loop is counted twice). And here is the first beautiful fact — the **handshake lemma**: if you sum the degrees of all vertices, you get exactly twice the number of edges. Why? Each edge is like a handshake: it adds one to the degree of both endpoints, i.e., two to the sum. An important consequence: the number of vertices of odd degree is always even. Therefore, for example, a graph with degrees 3, 3 and 1 does not exist - the sum of 7 is odd.

Let's check a concrete example. Vertices: 1, 2, 3, 4; edges: 1–2, 1–3, 2–3, 3–4. The vertex degrees are 2, 2, 3, and 1. Their sum is 8, exactly twice the number of edges. There are two vertices of odd degree, as the handshake lemma requires.

In a digraph, each vertex has an **indegree**, the number of incoming edges, and an **outdegree**, the number of outgoing edges.

How is the graph stored in the computer's memory? Two main ways. **Adjacency matrix** is an n by n table, where the intersection of row i and column j has a unit if there is an edge. Plus: check "is there a road between i and j?" you can instantly. Minus: memory needs n², even if there are very few edges. **Adjacency lists** — for each vertex, it is simply a list of its neighbors; memory is needed in proportion to the number of vertices plus edges, and this is the standard for "sparse" graphs where there are few edges. A rule of thumb: a dense graph is a matrix, a sparse graph is a list. The matrix of an undirected graph is symmetric about the diagonal, and the sum of the units in the row is the degree of the vertex.

Now about walking along the graph. There is a whole ladder of concepts, from the loosest to the strictest:

| Concept | What is forbidden to repeat |
|---|---|
| Walk | nothing |
| Trail | edges |
| Simple path | vertices |

A **closed walk** returns to its starting vertex; a **cycle** is a closed path with no repeated vertices except the first and last vertex. **Distance** between two vertices is the length of the shortest path between them, measured by the number of edges or the sum of their weights. In our example, 1→2→3→4 is a simple path of length three, 1→2→3→1 is a cycle, and the distance from 1 to 4 is two via vertex 3.

**Connectivity** asks whether the graph is all in one piece. An undirected graph is connected if every vertex can be reached from every other vertex. Otherwise, it splits into **connected components**: for example, a graph with edges 1–2 and 3–4 has two components. A digraph is **strongly connected** if every vertex can reach every other vertex while respecting edge directions. It is **weakly connected** if the underlying undirected graph is connected. A digraph with the single edge 1→2 is weakly but not strongly connected. Strongly connected components can be found with Kosaraju's or Tarjan's algorithm, both based on depth-first search.

Two more notions describe vulnerable parts of a network. A **bridge** is an edge whose removal increases the number of connected components. An **articulation point**, or **cut vertex**, is a vertex with the analogous property. In the path 1–2–3, both edges are bridges and vertex 2 is an articulation point: removing it disconnects vertices 1 and 3.

Finally, there are special graphs that are worth recognizing: **complete** (each pair of vertices is connected; n(n−1)/2 edges), **bipartite** (vertices are divided into two groups, edges only between groups — like "students and subjects"), **regular** (all degrees are the same), as well as cycle and path as separate graphs. And the notion of **isomorphism**: two graphs are isomorphic if they are "the same up to the renaming of the vertices" - even if they are drawn completely differently.


### Reinforcement

**Additional worked example.** In a graph with edges A–B, B–C, C–A, C–D, the degrees are equal to 2, 2, 3, 1. Their sum is 8 = 2 · 4 edges; there are two odd vertices, which is consistent with the handshake lemma.

**Transfer example.** A friendship network is modeled as an undirected graph, while social-media follows form a directed graph: if A follows B, that does not imply a reverse edge. Their connectivity properties therefore differ.

**Active recall.** Close the explanation above and answer without peeking: How does adjacency differ from incidence? Why can't the number of vertices of odd degree be odd?

**Mini practice — check yourself.** Is there a simple undirected graph with degrees 3, 2, 2, 1? Check only the necessary condition of the lemma.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** The sum of degrees is 8, which is even, so the lemma does not prohibit such a graph. The edges AB, AC, AD, and BC provide an example. The parity check is necessary but not sufficient by itself.

</details>

**Exam focus:**

- Adjacency — between two vertices, incidence — between a vertex and an edge. Do not confuse.
- Trap “is there a graph with degrees 3, 3, 1?” — no: the sum of degrees is odd, while the handshake lemma says it must be even.
- Remember the progression walk → trail → simple path: a trail does not repeat edges, and a simple path does not repeat vertices.
- For a digraph, be sure to distinguish strong connectivity from weak connectivity.
- For a sparse graph, the correct answer about storage is adjacency lists, not a matrix.

---

<a id="q8"></a>

## 8. Eulerian and Hamiltonian graphs. Trees, planar graphs, coloring, networks and flows

> **Learning outcome.** After this chapter, you will be able to distinguish between Euler and Hamiltonian problems and apply criteria for trees, planarity, coloring, and maximum flow. First, try to explain the topic in your own words, and then test yourself with the block at the end.


This question is a parade of the most beautiful plots of graph theory. Let's go through each one.

### Eulerian graphs: "with one stroke"

Do you remember the children's problem “draw a figure without lifting the pencil and without tracing a line twice”? This asks for an **Eulerian cycle**: a traversal that uses every edge exactly once and returns to its starting vertex. An undirected graph has an Eulerian cycle if and only if every vertex with nonzero degree belongs to one connected component and every vertex has even degree. The intuition is that every arrival at a vertex must be paired with a departure, so incident edges are used in pairs. An open **Eulerian path** instead exists when exactly two vertices have odd degree; they become its endpoints.

Graph theory is traditionally traced to Euler's 1736 solution of the seven bridges of Königsberg problem. The four land areas become vertices and the seven bridges become edges; all four vertices have odd degree, so no Eulerian path exists. An Eulerian path or cycle can be found with Hierholzer's algorithm in O(V + E) time. Fleury's algorithm is useful pedagogically but is generally slower. A familiar “house” drawing with exactly two odd vertices can be drawn in one stroke, but the stroke must finish at a different vertex from where it began.

### Hamiltonian cycles: "visit every city"

A **Hamiltonian cycle** is a simple cycle that visits **every vertex** exactly once; it need not use every edge. Thus, **Eulerian problems concern edges, while Hamiltonian problems concern vertices**. Deciding whether a graph has a Hamiltonian cycle is NP-complete, and no polynomial-time algorithm is known. There are useful sufficient conditions. **Dirac's theorem** states that a graph with n ≥ 3 vertices is Hamiltonian if every vertex has degree at least n/2. **Ore's theorem** requires deg(u) + deg(v) ≥ n for every pair of nonadjacent vertices u and v. These conditions are not necessary: the cycle C₆ is Hamiltonian even though every vertex has degree 2 < 3.

### Trees: a network without excess

A **tree** is a connected graph without cycles. For a graph with n vertices, several equivalent characterizations are useful: it is connected and has exactly n − 1 edges; there is exactly one simple path between every pair of vertices; and it is minimally connected, meaning that removing any edge disconnects it. Adding an edge between two existing vertices creates exactly one cycle. A collection of disjoint trees is a **forest**. A **spanning tree** of a connected graph is a tree subgraph that contains every vertex. Cayley's formula says that there are nⁿ⁻² labeled trees on n vertices: three for n = 3 and sixteen for n = 4.

### Planar graphs: a scheme without crossing wires

A **planar graph** can be drawn in the plane so that edges intersect only at shared endpoints. For a connected planar embedding, the famous **Euler formula** is:

**V − E + F = 2**

where V is the number of vertices, E the number of edges, and F the number of faces, including the unbounded outer face. For a cube, 8 − 12 + 6 = 2. It follows that a simple planar graph with V ≥ 3 has at most 3V − 6 edges. Therefore, K₅ is nonplanar: it has 10 edges, but the bound allows at most 9. Another famous nonplanar graph is K₃,₃, the “three houses and three utilities” graph. **Kuratowski's theorem** states that a finite graph is planar if and only if it contains no subdivision of K₅ or K₃,₃ as a subgraph.

### Coloring: we make a schedule

A **proper vertex coloring** assigns colors so that adjacent vertices receive different colors. The minimum number of colors required is the **chromatic number**. For scheduling, vertices can represent events and an edge can mean that two events share a student and cannot occupy the same time slot; the chromatic number is then the minimum number of slots. Similar models allocate radio frequencies and compiler registers. A graph is bipartite if and only if it contains no odd cycle, so it is 2-colorable; it requires exactly two colors if it has at least one edge. The complete graph Kₙ requires n colors, an even cycle requires two, and an odd cycle requires three. The **four-color theorem** states that every planar graph can be colored with at most four colors. Finding the chromatic number is NP-hard, while greedy coloring uses at most Δ + 1 colors, where Δ is the maximum degree.

### Networks and flows: how much water will pass through the pipes

A **flow network** is a directed graph with a **source**, a **sink**, and a capacity on every edge. A **flow** assigns an amount to each edge subject to two rules: an edge's flow cannot exceed its capacity, and at every intermediate vertex total inflow equals total outflow. The goal is to maximize the flow from the source to the sink.

The **max-flow min-cut theorem** states that the maximum flow value equals the minimum capacity of an s–t cut. Such a cut partitions the vertices into two sets, with the source on one side and the sink on the other; its capacity is the sum of the capacities of edges directed across the cut from the source side. The **Ford–Fulkerson method** repeatedly finds an augmenting path in the residual network, sends additional flow along it, and updates the residual capacities. The Edmonds–Karp algorithm chooses each augmenting path with breadth-first search and runs in O(VE²) time.

A small example: pipes s→a on 3, s→b on 2, a→t on 2, b→t on 3, a→b on 1. Let's start 2 by the path s→a→t, 2 more by the path s→b→t, and 1 more by the bypass path s→a→b→t. A total of 5. We check the neck: the pipes coming out of the source together pass 3 + 2 = 5 — it coincided, it is no longer possible. Flows are also used unexpectedly: for example, to find pairings in bipartite graphs (who to "pair" with whom in assignment tasks).


### Reinforcement

**Additional worked example.** A street-cleaning route must cover edges, so it is modeled as an Eulerian path problem. A courier who must visit each address once cares about vertices, so that is a Hamiltonian path problem. The similar names describe different tasks.

**Transfer example.** In a connected planar graph with V = 6 and E = 9, Euler's formula gives F = 2 − V + E = 5 faces. The outer region is also a face, otherwise it would be 4.

**Active recall.** Close the explanation above and answer without peeking: Which object passes exactly once in the Euler and Hamiltonian cycles? Why does any tree with n vertices have n − 1 edges?

**Mini practice — check yourself.** A connected graph has exactly two vertices of odd degree. What is guaranteed to exist: an Euler cycle, an open Euler path, or neither?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** There is an open Euler path starting at one odd vertex and ending at another; there is no cycle.

</details>

**Exam focus:**

- Classics: Euler — edges, Hamilton — vertices; for the first there is a simple criterion, for the second only sufficient conditions, because the problem is NP-complete.
- Eulerian cycle: zero odd-degree vertices; open Eulerian path: exactly two. All vertices of nonzero degree must belong to one connected component.
- Tree: edges are one less than vertices; any added edge creates exactly one cycle.
- Limitation on the number of edges of a planar graph is a necessary but not sufficient condition: "three houses and three wells" satisfy it, but the graph is non-planar.
- Four colors — only for planar graphs; in general, the chromatic number can be arbitrarily large.
- In Ford–Fulkerson, the flow is compared with the minimum cut among all possible cuts, not with a single weakest edge.

---

<a id="q9"></a>

## 9. Formal languages and automata. Grammars, finite automata, and pushdown automata

> **Learning outcome.** After this chapter, you will be able to explain the relationship between alphabet, language, grammar, and automata, and relate language classes to the required memory. First, try to explain the topic in your own words, and then test yourself with the block at the end.


Imagine that you invent your own "language": you take a set of symbols and say which words from them are considered correct. That's what formal language is - just a set of "allowed" words built according to clear rules. Like a list of correct passwords: a password either fits or it doesn't, no "well almost".

A few basic words, without which there is no way. **Alphabet** is a set of symbols from which we build words, for example, just two letters a and b, or zeros and ones. **Word** is any finite sequence of these characters, say abba. There is also an empty word — a zero-length word that contains nothing at all. A **Language** is any set of words, for example "all words where the letter a occurs an even number of times". Unlike Ukrainian or English, which are full of ambiguities, the formal language is set absolutely strictly. Syntax (the rules of HOW to write constructions) and semantics (WHAT these constructions mean) are distinguished here.

### Grammar is a recipe for generating words

If a language is a set of valid words, then a **grammar** is a system for generating those words. A grammar has four components: **terminals**, the symbols that may occur in completed words; **nonterminals**, auxiliary symbols used during derivation; **production rules**, which specify permitted replacements; and a **start symbol**, from which every derivation begins.

This is best seen in an example. Let the start symbol S have two production rules: S → aSb and S → ε, where ε is the empty word. A derivation can proceed as follows:

```
S → aSb → aaSbb → aabb
```

Apply the first rule twice, then replace S with ε to obtain aabb. The grammar can also derive ab, aaabbb, and so on. Its language consists of all words with n copies of a followed by n copies of b.

### Chomsky's hierarchy — who is capable of what

Grammars are of different "power", and Chomsky divided them into four floors. The main focus: each floor corresponds to its own "machine", which knows how to recognize exactly such languages.

| Type | Language class | Recognized by |
|---|---|---|
| 3 | Regular | Finite automata |
| 2 | Context-free | Nondeterministic pushdown automata |
| 1 | Context-sensitive | Linear-bounded automata |
| 0 | Recursively enumerable (unrestricted) | Turing machines |

Each class is contained in the next: every regular language is context-free, every context-free language is context-sensitive, and every context-sensitive language is recursively enumerable.

### Finite automaton

This is the simplest machine: it has a finite set of **states**, an initial state, and a set of **accepting states**. The current state represents all the memory the automaton has about the input read so far. It processes the word symbol by symbol, following its transition function. After the entire word has been read, the word is accepted if the automaton is in an accepting state and rejected otherwise.

Example: an automaton that accepts words containing an even number of a symbols needs two states: “even,” which is both initial and accepting, and “odd.” Reading a switches between them, while reading b leaves the current state unchanged. For abaa the states are even → odd → odd → even → odd. The automaton finishes in a non-accepting state, correctly rejecting the word because it contains three a symbols.

Automata are deterministic (exactly one transition from each state to each symbol) and nondeterministic (there can be several transitions, the word is accepted if there is at least one successful path). An important fact: a non-deterministic automaton is NOT more powerful - any can be rebuilt into a deterministic one, it's just that there can be significantly more states. Automata live everywhere: lexical analyzers of compilers, search by pattern (grep, regular expressions).

And what can't the machine do? Count to infinity. He does not recognize the language "n letters a, then n letters b": in order to compare numbers, you need to remember an arbitrarily large n, and he has only a handful of states in his memory. This is formally proved by the **pumping lemma**; it works only in one direction — it proves that the language is NOT regular, but it doesn't prove anything in the opposite direction.

### Pushdown automaton (PDA)

A pushdown automaton extends a finite automaton with a **stack** of unbounded depth. A nondeterministic PDA recognizes exactly the context-free languages, while deterministic PDAs recognize only the deterministic context-free languages. To recognize {aⁿbⁿ | n ≥ 0}, a PDA pushes one marker for every a and pops one for every b. It accepts only after consuming the entire input and returning the stack to its bottom marker. Unlike finite automata, nondeterministic pushdown automata are strictly more expressive than deterministic ones.

Context-free grammars describe much of the syntax of programming languages, commonly using BNF notation, and parse trees are fundamental to parsers. A grammar is ambiguous if some word has two distinct parse trees; the classic example is the **dangling else**. A PDA also has limits: the language {aⁿbⁿcⁿ | n ≥ 0} is not context-free and cannot be recognized by a single-stack PDA.


### Reinforcement

**Additional worked example.** The language of binary strings with an even number of 1 bits is regular. Two states are enough: even and odd. Each 1 switches the state, while 0 leaves it unchanged. The accepting state is even.

**Transfer example.** Correctly nested parentheses require remembering an unbounded nesting depth, so finitely many states are not enough. A pushdown automaton pushes '(' onto its stack and pops one for ')'; an empty stack at the end indicates balanced parentheses.

**Active recall.** Close the explanation above and answer without peeking: How does generative grammar differ from recognition automaton? What additional memory does the PDA have?

**Mini practice — check yourself.** Construct a sequence of states of the parity automaton for the word 10110, starting from the even state.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** even →1→ odd →0→ odd →1→ even →1→ odd →0→ odd. There are three 1 bits, so the word is not accepted.

</details>

**Exam focus:**

- The language {aⁿbⁿ | n ≥ 0} is not regular but is context-free; {aⁿbⁿcⁿ | n ≥ 0} is not context-free but is context-sensitive. Do not confuse the levels.
- Is nondeterminism more powerful? For finite automata, no; for pushdown automata, yes.
- The pumping lemma proves only IRREGULARITY, it cannot be used to prove regularity.
- Know the correspondence between grammar types and automata: regular — finite automata; context-free — nondeterministic pushdown automata; type 0 — Turing machines.

---

<a id="q10"></a>

## 10. Data structures: stack, queue, heap, tree, graph, and hash table

> **Learning outcome.** After this chapter, you will be able to select a data structure according to the required operations and explain its typical complexity and limitations. First, try to explain the topic in your own words, and then test yourself with the block at the end.


A data structure is a way to arrange data in memory so that exactly the operations you need work quickly. There is no universal "best" structure: each is a compromise, and the question always sounds like "what do we do with data most often?".

### Stack: a stack of plates

The stack operates on a last-in-first-out (LIFO) basis. Imagine a stack of plates: you put them on top, you also take them from the top, you can't reach the bottom one. Operations: put on top (push), remove the top (pop), look at the top (top) - all in a constant time, that is, equally fast even on ten elements, even on a million.

Example: we put 3, then 7, then 1. We remove it - it turns out 1, then 7: in the reverse order of insertion. A stack is a stack of function calls, an undo operation, a depth-first traversal of a graph, and a classic problem about the balance of parentheses: we put the opening parenthesis on the stack, we remove the closing parenthesis and check whether it is a pair. The line has been read, and the stack is empty - the parentheses are balanced.

### Queue: a line at the store

A queue is **first in, first out** (FIFO), like a supermarket line: elements are added at the tail and removed from the head, both in constant time with an appropriate implementation. Enqueuing 3, 7, and 1 returns them in the order 3, 7, 1, whereas a stack returns 1, 7, 3. A queue can be implemented with a circular buffer or a linked list. Related structures include a **deque**, which supports insertion and removal at both ends, and a **priority queue**, often implemented with a heap, which removes the highest-priority element rather than the oldest. Queues are used in breadth-first search, task schedulers, and buffers.

### Heap: a tree with the minimum at the top

A **binary heap** is a complete binary tree that satisfies the heap-order property: in a min-heap, every parent is no greater than its children. The minimum is therefore at the root and can be inspected in O(1) time. A binary heap is usually stored compactly in an array without pointers. With zero-based indexing, the children of element i are at positions 2i + 1 and 2i + 2.

For insertion, place the new element at the end and repeatedly exchange it with its parent while the heap property is violated. Insertion and removal of the minimum both take O(log n), while inspecting the minimum takes O(1). A heap can be built from an existing array in O(n), faster than inserting all elements one at a time. Heaps implement priority queues used by Dijkstra's and Prim's algorithms, heapsort, and top-k tasks.

### Tree and binary search tree

A tree is a hierarchy: root, parents and children, leaves below. The most useful variant is a **binary search tree (BST)**: at each node, everything to the left is smaller than it, everything to the right is larger. The search turns into a series of "left or right?" and costs as many steps as the height of the tree.

Example: insert 5, 3, 8, 2, 4. Five is the root, 3 goes to the left, 8 to the right, 2 to the left of the three, 4 to the right of it. We are looking for 4: three comparisons - and we found it. But there is a trap: if you insert already sorted numbers, the tree is drawn into a "stick", and the search degrades to viewing the entire list. Therefore, in reality they use balanced trees that themselves keep a small height: AVL, red-black (within std::map and TreeMap), B-trees in databases. And remember the in-order traversal (left — node — right): for BST, it outputs elements in sorted order.

### A graph as a structure

The graph is stored in two ways. **Adjacency matrix** is a "vertex-to-vertex" table, where one means an edge: checking "whether there is an edge" is instantaneous, but the square of the number of vertices is required in memory. **Adjacency lists** — for each vertex, it's just a list of its neighbors: memory proportional to vertices plus edges, a standard for sparse graphs where there are few edges.

### Hash table: a locker with a calculated cell number

The idea is brilliantly simple: an array of m "bins", and the bin number for the key is instantly calculated by a hash function - usually as the remainder of dividing the hash by m. An example with five baskets: key 12 gives a remainder of 2 — we put it in basket 2; key 7 gives the same remainder 2 — and this is a **collision**, two keys claiming the same cell.

Two common collision-resolution methods are **separate chaining**, where each bucket stores multiple entries so keys 12 and 7 can coexist, and **open addressing**, which probes for another available position using linear probing, quadratic probing, or double hashing. When the load factor becomes too high, the table is enlarged and all entries are rehashed. A resize is expensive, but its cost is amortized across many operations.

Search, insertion, and deletion take O(1) expected time under suitable hashing and load control, but O(n) in the worst case. Hash tables implement dictionaries such as dict and HashMap, as well as sets and caches. Unlike a hash table, a balanced search tree preserves key order and supports efficient range queries such as “return all keys from 10 to 20.”


### Reinforcement

**Additional worked example.** The Undo button requires a stack: the most recently performed action is undone first. For the print queue - FIFO queue. For constant selection of the closest task in terms of priority — min-heap.

**Transfer example.** Users' dictionary by email is naturally stored in a hash table: average search O(1). But for outputting email in sorted order, a balanced tree with O(log n) per operation is better.

**Active recall.** Close the explanation above and answer without peeking: Which structure supports LIFO and which supports FIFO? Why isn't a heap a fully sorted array?

**Mini practice — check yourself.** Required operations: often add events, always remove the event with the smallest timestamp. What structure to choose and what is the complexity of insertion and removal?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Min-heap; inserting and removing a minimum is O(log n), viewing a minimum is O(1).

</details>

**Exam focus:**

- Stack — plates (LIFO), queue — store (FIFO). They often give a push/pop sequence and ask what will come back.
- Building a heap from an array is O(n), not O(n log n). A classic trap.
- Search in BST is fast only in a balanced tree; a degenerate tree is actually a list.
- The O(1) time of a hash table is expected or average-case performance; the worst case is O(n).
- An in-order traversal of a BST produces a sorted sequence.

---

<a id="q11"></a>

## 11. The concept and properties of algorithms. Recursive functions, Turing machines, and Markov algorithms

> **Learning outcome.** After this chapter, you will be able to name properties of an algorithm and explain why different formal models of computation describe the same computability limit. First, try to explain the topic in your own words, and then test yourself with the block at the end.

An algorithm is a precise step-by-step procedure for obtaining a result from input data, written so that an executor can follow it without inventing any missing steps. In the 20th century, mathematicians asked a deeper question: what can be computed by an algorithm at all? To answer it, they developed several formal models of computation—and remarkably, all of them turned out to have the same computational power.

### Properties of the algorithm

Any algorithm has five core properties. **Discreteness**: execution proceeds in separate steps. **Definiteness**: each step is unambiguous, so two executors following the same instructions obtain the same result. **Finiteness**: execution completes after finitely many steps. **Generality**: the algorithm solves a whole class of instances—it sorts any valid input array, not just one particular array. **Effectiveness**: every instruction is elementary enough for the chosen executor to perform.

We also analyze an algorithm's **complexity**: time complexity measures the number of operations, while space complexity measures the memory used. Asymptotic notation describes growth for large inputs: O gives an upper bound, Ω gives a lower bound, and Θ gives a tight bound, up to constant factors.

### Three formalizations of one concept

#### Recursive functions (Gödel, Kleene)

The idea: to collect ALL computable numerical functions from a handful of the simplest bricks. Bricks are a function that always returns zero; "next number" function (add one); and projection functions that simply select one of their arguments. Bricks are glued in three ways: substituting functions one into another; **primitive recursion** — we determine the value at zero separately, and the value at "n plus one" — through the already known value at n; and **minimization** — "find the smallest number on which the condition is satisfied" (such a search may never end, so the function does not become defined everywhere).

Example — addition through primitive recursion:

```
add(0, y)   = y
add(n+1, y) = add(n, y) + 1
```

That is, "add zero" is to do nothing, and "add n+1" is to add n and one more. Let's count add(2,3): we reduce to add(1,3) plus one, then to add(0,3) plus two, which is 3 plus 2, i.e. 5. Multiplication is also determined through addition, factorial through multiplication.

Functions constructed without unbounded minimization are called primitive recursive functions, and they are total. Adding unbounded minimization yields the partial recursive functions—the class of partial computable functions. A famous counterexample is the Ackermann function: it is total and computable, but it cannot be defined using primitive recursion alone.

#### Turing machine

It is a "mechanical calculator": an infinite tape of cells, a head that reads and writes one character at a time, and a finite set of states. The entire program is a table of commands of the form: "if you are in such a state and see such a symbol, write this down, move to the left or right and go to such a state."

An example is to add one to a binary number, starting from the least significant bit: if you see 1, write 0 and move further to the left (carry); if you see 0 or it's empty, write 1 and stop. The number 101 (five) turns into 110 (six) by two table commands.

Three big ideas surrounding Turing machines. The first is the **Church–Turing thesis**: everything that can be calculated by any algorithm can be calculated by a Turing machine. This is not a theorem, but an accepted hypothesis - it cannot be proved, because "algorithm" is an informal concept. The second is the **universal machine**: there is one machine that receives the description of any other machine along with its input and simulates its operation; it is a prototype of an ordinary computer that executes programs. The third is the **halting problem**: there is no algorithm that can tell, given an arbitrary program and input, whether it will ever terminate. Proved by the opposite: assume that such a solver exists, build a program that does the opposite, and run it on itself — you will get a contradiction.

#### Normal Markov algorithms

Here, computation means rewriting strings. A normal Markov algorithm is an ordered list of substitution, or rewriting, rules, some of which are terminating rules. At each step, take the FIRST applicable rule and replace the LEFTmost matching occurrence. Stop when a terminating rule is applied or when no rule applies.

Example: suppose the only rule is “replace `a` with `b`.” The string `aba` becomes `bba`, then `bbb`; the rule is no longer applicable, so the algorithm stops. Markov's normalization principle states that every effectively computable function can be implemented by a suitable normal algorithm.

### The main conclusion

Recursive functions, Turing machines, Markov algorithms (and Church's lambda calculus) are all equivalent in power: what one model can do, all can do. Independently invented formalizations converged at one point — and this is the strongest argument that we have correctly grasped the concept of "computable".


### Reinforcement

**Additional worked example.** The instruction “repeat until it is good enough” is not a valid algorithm without a precise criterion: definiteness and possibly termination are violated. “Repeat while error > 0.001 and fewer than 1000 steps have run” provides a testable stopping rule.

**Transfer example.** A Turing machine has very primitive steps, but can simulate a high-level program. The Church–Turing thesis asserts not speed, but the coincidence of intuitively calculable functions with this formal model.

**Active recall.** Close the explanation above and answer without peeking: What properties distinguish an algorithm from a vague advice? How does the Church-Turing thesis differ from a theorem?

**Mini practice — check yourself.** Evaluate the command "go over the natural numbers until you find a counterexample" as an algorithm for the case when there is no counterexample.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Each step is defined, but if no counterexample exists, the process never terminates. It is a semidecision procedure, not a decision algorithm that always returns an answer.

</details>

**Exam focus:**

- The Church–Turing thesis is NOT a theorem, it cannot be proved.
- The halting problem is not merely “very difficult”; it is undecidable—no algorithm can solve every instance.
- Primitive recursive functions form a proper subset of the total computable functions; the Ackermann function is a standard counterexample.
- In Markov, the order of the rules and the replacement of the leftmost entry are critically important - a typical question "why such a result".
- "Recursive functions" from the theory of computability are not the same as recursion in programming.

---

<a id="q12"></a>

## 12. Sorting algorithms and QuickSort

> **Learning outcome.** After this chapter, you will be able to compare sorting by time, memory, and stability, and perform QuickSort partitioning step by step. First, try to explain the topic in your own words, and then test yourself with the block at the end.


Sorting is the arrangement of elements, say numbers, in ascending order. Simple methods are similar to dealing cards in a hand, but require about n² operations: on a million elements, that's a trillion operations - unsustainable. Smart divide-and-conquer methods take about n·log n steps, and the logarithm grows ridiculously slowly: it's only about twenty for a million. That is, instead of a trillion — some twenty million operations.

### Simple sorting

Quadratic complexity means: double the array — you wait four times longer. For small arrays, this is quite normal.

- **Bubble sort**: repeatedly scan the array and swap adjacent elements that are out of order; large elements gradually “bubble” to the end. It is stable, meaning that equal elements retain their original relative order.
- **Insertion sort**: like sorting cards in your hand, take the next element and insert it into the correct position in the already sorted left part. Its best case is linear, and it is efficient on small or nearly sorted arrays; it is stable.
- **Selection sort**: repeatedly find the minimum remaining element and put it into the next position. It remains quadratic even for an already sorted array, but performs only O(n) swaps.

An example of insertions for the array 5, 2, 4, 1: we take 2 and insert before 5 — we have 2, 5; we take 4 and insert between them — 2, 4, 5; take 1 and put it at the beginning — 1, 2, 4, 5. Done.

### Effective sorting

- **Merge sort**: divide the array in half, recursively sort each half, and then merge the two sorted halves in one pass—compare their first unmerged elements and take the smaller one. Example: when merging [2, 5] and [1, 4], take 1, then 2, then 4, then 5. It guarantees O(n log n) time and is stable, but array implementations require O(n) auxiliary memory. It is especially useful for linked lists and external sorting.
- **Heap sort**: build a max-heap (see question 10), then repeatedly remove the maximum and place it at the end. It guarantees O(n log n) time, sorts in place using O(1) auxiliary storage apart from the input array, and is unstable.

An important theorem is the **lower bound**: no sort that learns about elements only by comparison can be faster than

```
Ω(n log n)
```

i.e. "not less than the order of n·log n steps". Why: the algorithm must distinguish all n! possible orders of elements, and its decision tree with this number of leaves cannot be lower than approximately n·log n.

The comparison lower bound does not apply when an algorithm can inspect the structure of the keys. **Counting sort** counts how often each value occurs and works when integer keys come from a suitably small range; **radix sort** processes keys digit by digit; **bucket sort** distributes elements among buckets. Under suitable bounds on the key range, number of digits, or bucket distribution, these algorithms can run in linear time.

### Quick sort (QuickSort, Hoare, 1960)

A three-step plan. First, select a **pivot**. Second, **partition** the elements into those smaller than, equal to, and greater than the pivot. In schemes such as Lomuto partitioning, the pivot itself is placed in its final sorted position. Third, recursively sort the parts on either side; a part with at most one element is already sorted and forms the base case.

For example, consider the array 3, 8, 2, 5, 1 and choose the last element as the pivot each time. With pivot 1, there are no smaller elements, so the partition is extremely uneven: 1 comes first and everything else remains to its right. Next, sort 3, 8, 2, 5 using pivot 5: 3 and 2 go to the left, 8 goes to the right, and 5 reaches its final position. The part 3, 2 with pivot 2 becomes 2, 3; the one-element part containing 8 is already sorted. The result is 1, 2, 3, 5, 8.

Main properties:

- Its average running time is O(n log n), and its small constant factors and cache-friendly access pattern often make it fast for in-memory arrays.
- The worst case is O(n²), caused by systematically poor pivot choices. For example, if the array is already sorted and the first element is always chosen as the pivot, one partition is empty every time. Randomized pivot selection or the **median of three**—the median of the first, middle, and last elements—reduces this risk.
- It usually sorts in place and is unstable. The recursion stack uses O(log n) space on average but can grow to O(n) in the worst case.
- Practical improvements include switching to insertion sort for small subarrays and using three-way partitioning for arrays with many duplicate keys. **Introsort**, used by many implementations of `std::sort`, begins with quicksort and switches to heap sort if recursion becomes too deep, guaranteeing O(n log n) worst-case time.
- A nice bonus — **QuickSelect**: you can find the k-th largest element in an average linear time if, after splitting, you recursively go only to the part where it lies.

### Comparison table

| Algorithm | Average | Worst case | Auxiliary memory | Stable |
|---|---|---|---|---|
| Bubble sort | O(n²) | O(n²) | O(1) | Yes |
| Insertion sort | O(n²) | O(n²) | O(1) | Yes |
| Selection sort | O(n²) | O(n²) | O(1) | No |
| Merge sort | O(n log n) | O(n log n) | O(n) | Yes |
| Heap sort | O(n log n) | O(n log n) | O(1) | No |
| **QuickSort** | **O(n log n)** | **O(n²)** | O(log n) average; O(n) worst | No |


### Reinforcement

**Additional worked example.** For [4, 2, 5, 1, 3] with pivot 3, after partitioning we obtain groups [2, 1], [3], [4, 5]. We recursively sort only the outer groups; result [1, 2, 3, 4, 5]. The specific order within the groups depends on the partition scheme.

**Transfer example.** Sorting a grade book first by name and then stably by grade preserves alphabetical order among students with equal grades. An unstable sorting algorithm can destroy that order.

**Active recall.** Close the explanation above and answer without peeking: What does sort stability mean? Which choice makes QuickSort degrade to O(n²)?

**Mini practice — check yourself.** If the first element is always a pivot, what recurrence will QuickSort get on an already sorted array of length n?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** T(n) = T(n − 1) + O(n), because one part is empty, and the other has n − 1 elements; the sum of works gives O(n²).

</details>

**Exam focus:**

- The worst case of QuickSort is quadratic; an already sorted array combined with a naive pivot choice is a standard example.
- QuickSort is unstable; you need stability - take merge sort.
- The Ω(n log n) lower bound applies only to comparison sorting; counting sort and radix sort use additional structure in the keys and can avoid it.
- Insertion sort runs in O(n + k), where k is the number of inversions, so it approaches linear time on nearly sorted data and works well on small QuickSort partitions.
- Building a heap is O(n), but the whole heapsort is O(n log n). Do not confuse.

---

<a id="q13"></a>

## 13. Dynamic programming and greedy algorithms

> **Learning outcome.** After this chapter, you will be able to recognize optimal substructure and repeated subproblems and distinguish proven greedy selection from local heuristics. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Imagine that you are solving a big problem, which breaks down into a bunch of small ones, and at the same time, the same small problems come up again and again. Dynamic programming (DP) is "intelligent laziness": solve each small problem exactly once, write the answer in a notebook, and then just peek. The greedy algorithm is a different strategy: at every turn, grab what looks best right now and never look back. Sometimes greed produces a perfect result, and sometimes it fails miserably - and then saves the DP.

### Dynamic programming

Two things are necessary for DP to make any sense at all:

- **Optimal substructure** (Bellman's principle): the best solution of a large problem consists of the best solutions of its pieces. Vital intuition: if the shortest route from Kyiv to Odesa passes through Uman, then its section "Kyiv-Uman" must also be the shortest - otherwise the entire route could be improved.
- **Overlapping subtasks**: The same small task occurs multiple times. This is what distinguishes DP from divide and conquer (as in merge sort), where chunks are independent and non-repeating.

It can be implemented in two ways:

- **Top-down** (top-down, memoization): we write a normal recursion, but before the calculation we look in the cache - if the answer is already there, we return it instantly.
- **Bottom-up** (tabulation): without recursion, we fill the table in a loop, from the smallest subproblems to the target one.

The recipe for constructing a DP solution is always the same: come up with a **state** (a set of parameters that describes the subproblem), write a **recurrence relation** (how the answer to the state consists of smaller ones), specify the **base cases**, and determine the order in which to calculate everything.

**The classic example is Fibonacci.** The formula is simple:

`F(n) = F(n-1) + F(n-2)` - each number is the sum of the previous two.

Naive recursion works by `O(2^n)` — time doubles with each increase in n, because the same numbers are counted over and over. With the table — linearly, `O(n)`:

| n | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| F(n) | 0 | 1 | 1 | 2 | 3 | 5 | 8 |

Five additions—and it is done. A naive recursion for F(6) makes 25 total function invocations, including the initial call, while F(2) is calculated five times.

**0/1 knapsack.** Each item has a weight and a value, the knapsack has a fixed capacity, and each item must be taken whole or left behind. For every pair “first i items, capacity w,” the table stores the better of two choices: exclude item i, or include it when it fits and add its value to the best result for the remaining capacity. Consider capacity 5 and items A (weight 2, value 3), B (3, 4), and C (4, 5):

| items \ capacity | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| no items | 0 | 0 | 0 | 0 | 0 | 0 |
| A| 0 | 0 | 3 | 3 | 3 | 3 |
| A, B | 0 | 0 | 3 | 4 | 4 | **7** |
| A, B, C | 0 | 0 | 3 | 4 | 5 | **7** |

The answer is 7: take A and B, whose total weight is 2 + 3 = 5. By contrast, the greedy rule “take the most valuable item first” chooses C, with value 5, after which nothing else fits. Dynamic programming wins 7 to 5. Its running time is `O(n·W)`, which is **pseudopolynomial** because it depends on the numeric value of W rather than only on the length of W's representation.

Other well-known DP problems include the longest common subsequence of two strings, edit distance (the minimum number of insertions, deletions, and substitutions needed to transform one string into another), coin change with a minimum number of coins, and the Floyd–Warshall algorithm from question 14. Bellman–Ford can also be understood as repeated dynamic-programming relaxation over the maximum number of edges in a path.

### Greedy algorithms

A greedy algorithm makes a locally best choice at each step and never revisits it. It honestly works when the **greedy choice property** holds: the local choice is guaranteed to enter some globally optimal solution, i.e., the greedy "future-proof" move. Formally, this is substantiated by the theory of matroids (the Rado–Edmonds theorem), but an idea is enough for the exam.

Examples where greed works perfectly:

- **Selection of applications**: in order to collect the maximum number of applications that do not overlap in time, we sort them by completion time and take each compatible one. Example: applications A(1–4), B(3–5), C(0–6), D(5–7), E(6–8). We take A (ends the earliest), B and C intersect with it - skip, D starts after 4 - take, E intersects with D - skip. The result is {A, D}, which is the maximum.
- **Huffman codes**: frequent symbols receive short codes, while rare symbols receive longer ones.
- **Fractional knapsack** (items can be divided): take items in decreasing order of value per unit of weight. This greedy rule is optimal here, unlike in 0/1 knapsack.
- Kruskal's, Prim's, and Dijkstra's algorithms from question 14 are also greedy algorithms.

Many greedy algorithms run in `O(n log n)` because they sort first and then make one pass, but the exact complexity depends on the algorithm and the data structures it uses.

**Brief comparison:** DP iterates over all subproblems and guarantees optimality always (if the recurrence is correct), but costs more. Greedy makes one choice per step, works fast, but is correct only when the greedy choice property is proven.


### Reinforcement

**Additional worked example.** For coins {1, 3, 4} and a target sum of 6, the greedy choice takes 4 + 1 + 1, or three coins. Dynamic programming finds 3 + 3, or two coins. This is a counterexample to applying a greedy rule without proving it correct.

**Transfer example.** For shortest paths in a DAG, process vertices in topological order so that every predecessor distance is available before computing the current vertex's distance. This is a bottom-up dynamic-programming formulation.

**Active recall.** Close the explanation above and answer without peeking: What two characteristics indicate dynamic programming? What is missing from the phrase "take the best now" to become a valid algorithm?

**Mini practice — check yourself.** Find the minimum number of coins {1, 3, 4} for sums 0…6, using the previous answers in sequence.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** dp = [0, 1, 2, 1, 1, 2, 2]. For example, dp[6] = 1 + min(dp[5], dp[3], dp[2]) = 2.

</details>

**Exam focus:**

- The “greedy is always correct, only faster” trap is false. The 0/1 knapsack example above is a ready-made counterexample: greedy obtains 5, while the optimum is 7.
- DP is not "divide and conquer": there the subtasks do not overlap, in DP they overlap, this is the whole point of caching.
- `O(n·W)` for 0/1 knapsack is pseudopolynomial because W is represented using only `O(log W)` input bits, while the running time depends on the numeric value W.
- Differentiate between top-down (recursion plus cache) and bottom-up (table with a loop) - they often ask.
- Greedy coin change works for canonical systems such as {1, 2, 5, 10}, but not for every denomination set. For {1, 3, 4} and target 6, greedy returns 4 + 1 + 1 (three coins), whereas the optimum is 3 + 3 (two coins).

---

<a id="q14"></a>

## 14. Graph algorithms

> **Learning outcome.** After this chapter, you will be able to choose BFS, DFS, Dijkstra, Bellman–Ford, or Floyd–Warshall depending on the properties of the graph and the desired result. First, try to explain the topic in your own words, and then test yourself with the block at the end.

A graph is a kind of “map”: circles represent vertices—cities, people, or pages—and lines represent edges—roads, relationships, or links. Graph algorithms answer natural questions: how can we visit the vertices, find a shortest path, connect every city as cheaply as possible, or send the maximum flow from point A to point B? We write n for the number of vertices and m for the number of edges. A graph can be **undirected**, with two-way edges, or **directed**, with arrows; it is **weighted** when each edge has a value such as a road length.

You can store a graph in memory in two ways: **adjacency matrix** (an n by n table, where the cell contains a unit or weight, if there is an edge; quickly check "if there is an edge", but it eats up memory quadratically) or **adjacency lists** (for each vertex - a list of its neighbors; memory is needed in proportion to n + m, and this is how they do it in practice).

### Graph traversals

**Depth-first search (DFS)** follows edges as deeply as possible and backtracks at a dead end. It is implemented with recursion or an explicit stack (“last in, first out”). Applications include finding connected components, detecting cycles, topological sorting, and finding bridges and articulation points—edges and vertices whose removal disconnects an undirected graph. In a directed graph, DFS detects a cycle when it finds a back edge to an ancestor still on the recursion stack; in an undirected graph, the edge back to the current vertex's parent must be ignored.

**Breadth-first search (BFS)** traverses the graph in layers: first all neighbors of the start, then their unvisited neighbors, and so on. It uses a queue (“first in, first out”). In an unweighted graph, BFS finds shortest paths measured by the number of edges. For example, in a graph with edges A–B, A–C, B–D, C–D, and D–E, a search from A reaches B and C at distance 1, D at distance 2, and E at distance 3.

Both traversals work according to `O(n + m)` — we inspect each vertex and each edge a constant number of times.

**Topological sorting** orders the vertices of a directed acyclic graph (DAG) so that every edge points from an earlier vertex to a later one: dependencies come before the tasks that depend on them. Such an ordering exists exactly when the directed graph is acyclic. Kahn's algorithm repeatedly removes vertices of in-degree zero; DFS can produce the order by reversing the finishing order. Both methods run in `O(n + m)`.

### Euler and Hamilton cycles

An **Eulerian cycle** traverses every EDGE exactly once and returns to its starting vertex; the bridges of Königsberg are the classic example. In a connected graph, such a cycle exists exactly when every vertex has even degree. An open Eulerian trail exists exactly when there are two odd-degree vertices; with zero odd-degree vertices, there is an Eulerian cycle. In the square A–B–C–D–A, all degrees are 2, so a cycle exists. Adding diagonal A–C makes A and C odd, so there is no Eulerian cycle, but there is an Eulerian trail from A to C.

A **Hamiltonian cycle** visits every VERTEX exactly once before returning to the start. Despite the similar name, deciding whether one exists is NP-complete, and there is no comparably simple criterion. There are useful sufficient conditions: Dirac's theorem states that for a simple graph with n ≥ 3 vertices, minimum degree at least n/2 guarantees a Hamiltonian cycle; Ore's theorem gives a related condition for every pair of nonadjacent vertices.

### Shortest paths in weighted graphs

- **Dijkstra's algorithm**: finds paths from one source when all edge weights are nonnegative; the weights need not be integers. It greedily takes the nearest unprocessed vertex and performs **relaxation**, asking for each neighbor whether going through the current vertex gives a shorter path. With a binary heap, it runs in `O((n + m) log n)`.
- **Bellman–Ford**: finds paths from one source and allows negative edge weights. It relaxes every edge n−1 times in `O(n·m)`. If a distance still improves on an additional pass, a negative cycle is reachable from the source; shortest distances are undefined for vertices reachable from that cycle.
- **Floyd–Warshall**: distances between all pairs of vertices at once; this is DP - we go through all possible "intermediate" vertices and ask if it is shorter through them. Time `O(n³)`.

Dijkstra's example: edges A→B with a weight of 4, A→C with a weight of 1, C→B with a weight of 2, B→D with a weight of 5. At first, B is supposed to be 4 (a straight edge), but after processing C (closest, distance 1), we see: through C to B, we get 1+2=3 — we improve. As a result, to B — 3 (via A, C, B), not 4. This is "improved assessment through the neighbor" and is relaxation.

### Minimum spanning tree (MST)

For a connected, undirected, weighted graph, a minimum spanning tree (MST) connects ALL vertices using n−1 edges without cycles and has the minimum possible total edge weight. The intuition is to connect all cities by cable as cheaply as possible. Two greedy algorithms find an MST:

- **Kruskal**: sorts all edges by weight and adds each one, if it does not close the loop (this is quickly checked by the structure of DSU - the "group system", which knows how to merge groups of vertices and ask if vertices are already in the same group); `O(m log m)`.
- **Prim**: grows one tree from a starting vertex, repeatedly adding the lightest edge that connects the tree to a vertex outside it; with a binary heap, it runs in `O(m log n)`.

Kruskal's example: vertices A, B, C, D; edges AB=1, CD=2, BC=3, AC=4, BD=5. We take AB, then CD, then BC (it connects two separate groups), discard AC and BD - they would close the loops. Sum of MST = 6.

**How many spanning trees does a graph have?** This is answered by the Kirchhoff theorem through the determinant of a special matrix, and for a complete graph (where all pairs of vertices are connected) there is a simple Cayley formula:

`n^(n−2)` is the number of spanning trees of a complete graph on n vertices.

For a triangle (n=3), it is 3 in the first power = 3 trees: we throw out any one of the three edges.

### Maximum flow

Imagine a network of pipes with a **source**, a **sink**, and a capacity on each pipe. The question is how much flow can be sent from source to sink. The max-flow min-cut theorem states that the maximum flow equals the total capacity of a minimum cut—the minimum-capacity set of edges whose removal separates source from sink. The Ford–Fulkerson method repeatedly finds an augmenting path with residual capacity and sends additional flow along it; reverse residual edges allow earlier choices to be adjusted. The Edmonds–Karp variant finds augmenting paths with BFS and runs in `O(n·m²)`. Applications include maximum matchings in bipartite graphs, scheduling, and connectivity analysis.


### Reinforcement

**Additional worked example.** In a maze where each step costs the same, BFS finds the shortest route in layers. If transitions have different nonnegative weights, Dijkstra's algorithm is needed; regular BFS already minimizes the edges, not the sum of the weights.

**Transfer example.** For dependencies A→C, B→C, and C→D, Kahn's algorithm may first take A or B, then whichever of A or B remains, followed by C and D. A directed cycle would eventually leave vertices with no zero-in-degree choice.

**Active recall.** Close the explanation above and answer without peeking: When does BFS guarantee the shortest path? Why does Dijkstra not work with a negative edge?

**Mini practice — check yourself.** There are edges S→A with weight 5, S→B with weight 2, B→A with weight 1. What distance to A will Dijkstra find and through which vertex?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** A distance of 3 through B: S→B→A costs 2 + 1, which is better than a direct edge of weight 5.

</details>

**Exam focus:**

- The main trap: the Eulerian cycle is about EDGES (there is a simple criterion of parity of degrees), the Hamiltonian cycle is about VERTICES (an NP-complete problem). Dirac's and Ore's theorems are only sufficient conditions, not criteria.
- Dijkstra does NOT work with negative weights—use Bellman–Ford instead. When shortest paths are needed for every pair, use Floyd–Warshall.
- BFS gives the shortest paths only in the unweighted graph, DFS does not give the shortest paths at all.
- Complexities to memorize: DFS/BFS `O(n+m)`; Dijkstra with a heap `O((n+m) log n)`; Bellman–Ford `O(n·m)`; Floyd–Warshall `O(n³)`; Kruskal `O(m log m)`.
- Topological sorting exists only for graphs without cycles and is usually not unique.
- Cayley's formula is exactly for a complete graph; for an arbitrary graph, calculate according to Kirchhoff's theorem.

---

<a id="q15"></a>

## 15. Programming languages: procedural and problem-oriented. Syntax and semantics

> **Learning outcome.** After this chapter, you will be able to distinguish syntax from semantics and explain how paradigm and language level affect the way a problem is described. First, try to explain the topic in your own words, and then test yourself with the block at the end.

**The essence of the question.** A programming language lets a person describe computations for a computer. Like a natural language, it has rules for valid expressions—its syntax—and rules for what those expressions mean—its semantics. Languages also differ in purpose: some are general-purpose, while others focus on a particular domain. Think of a kitchen knife and a pizza cutter: the first handles many tasks, while the second is specialized for one task.

**Key concepts.**

Any programming language consists of three parts, and the analogy with Ukrainian works perfectly here:

| Constituent | What is this | Analogy |
|---|---|---|
| Alphabet | Allowed characters: letters, numbers, brackets, semicolon | Letters of the alphabet |
| Syntax | Rules for forming valid constructions from symbols | Grammar and spelling |
| Semantics | What a correctly written construction means | Content of the sentence |

Languages are also classified by level. Machine code consists of binary instructions executed directly by a processor. Assembly language is a symbolic, architecture-specific representation that an assembler translates into machine code; it is not itself “the same machine code.” Code written for one instruction-set architecture generally cannot run directly on another. High-level languages such as Pascal, C, and Python provide abstractions over hardware. A compiler or interpreter, discussed in question 16, translates or executes the program for a target platform.

**Procedural languages.** These are generally imperative languages: a program is a sequence of commands that change its state step by step. State includes the current values of variables. A program is divided into procedures and functions—named blocks of code that can be called repeatedly. Its basic tools include variables and assignment, branching, loops, parameter passing, and recursion. Classic examples include Fortran, Pascal, C, and Ada. Object-oriented languages such as C++, Java, and C# also support imperative and procedural techniques; they are discussed in question 17.

The key idea is that procedural code describes HOW to obtain a result. To compute a sum, for example, initialize an accumulator, iterate through the values, and add each value to it.

**Problem-oriented languages.** Also called domain-specific languages when they target a narrow domain, these languages often use a declarative style: describe WHAT result is required, and let the system decide how to obtain it. Examples include SQL for database queries, regular expressions for text patterns, and HTML and LaTeX for document markup. MATLAB and R focus on numerical and statistical computing, while Prolog is a logic-programming language. HTML and LaTeX are markup languages rather than programming languages in the strict sense.

**An example of the "what" vs. "how" difference.** Challenge: Find students with a score above 90.

- SQL: `SELECT name FROM students WHERE grade > 90;` - one sentence, "give me the names where the score is greater than 90". How exactly to go through the rows is not our business.
- C: write a loop over the array, check the condition for each student, and collect the matching students in a list.

Problem-oriented languages are concise and reduce accidental complexity within their domain, but they are limited outside it: SQL, for example, is not a practical language for implementing an entire video game.

**Syntax in more detail.** Syntax defines only the form of the record, and is specified by formal grammars—sets of rules that can be used to "generate" correct strings. The classical notation is the Backus–Naur form (BNF). For example:

```
<digit> ::= 0 | 1 | ... | 9
<number> ::= <digit> | <number><digit>
```

Read the rules as follows: “a digit is 0, 1, and so on through 9,” and “a number is either one digit or a number followed by another digit.” The second rule is recursive, so it can generate a number of any length. To derive 473, first derive 4 as a digit and therefore a number; append digit 7 to obtain 47, then append digit 3 to obtain 473. We cannot derive 4a7 because `a` is not a digit. Programming-language syntax is commonly specified with context-free grammars (type 2 in Chomsky's hierarchy), while tokens such as identifiers and numbers are commonly described by regular languages (type 3). Additional static-semantic constraints are not generally context-free.

**Semantics in more detail.** Semantics tells what a construct means and how it behaves at runtime. There are several formal approaches, and it is enough to understand them at the level of an idea:

- operational semantics: the content of the program is a sequence of machine states during execution, "what happens step by step";
- denotational: each construction is assigned a mathematical object — a function over states;
- axiomatic semantics (Hoare logic): assertions of the form “if the precondition is true before the command, then the postcondition will be true afterward.” For example, if x = 3 before “increase x by 1,” then x = 4 afterward. This approach is used to prove program correctness;
- static semantics: constraints that grammar alone cannot express, such as “the variable is declared before use” and “the types are compatible.” A compiler or another static-analysis tool can check them.

Key example: a construct can be syntactically valid but semantically invalid. An expression such as `x = "text" / 2` fits the grammar of many languages, but dividing text by a number is not a defined operation in those languages.


### Reinforcement

**Additional worked example.** The statement `x = 3 +` is syntactically incomplete, so the parser rejects it. The statement `x = y / 0` can be syntactically valid but cause a run-time error in a language where division by zero is invalid. Syntax analysis catches the first problem; semantic analysis or the run time catches the second.

**Transfer example.** SQL describes the required result with a declarative statement such as `SELECT`, not the exact sequence of disk-page reads. It is therefore a problem-oriented, declarative language; the query optimizer chooses the execution plan.

**Active recall.** Close the explanation above and answer without peeking: What does the grammar of a language check, and what does its semantics ask? Name one problem that is natural for declarative description.

**Mini practice — check yourself.** Classify errors: missing parenthesis; access to an undeclared variable; division by zero at runtime.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Accordingly: syntactic; static semantic; dynamic semantic error.

</details>

**Exam focus:**

- Syntax — form, semantics — content. The trap question is "can a syntactically correct program be incorrect?" Yes — semantically: a type error or just a wrong algorithm.
- "Procedural" and "problem-oriented" are not opposites of the same order: the first is universal and imperative, the second is highly specialized and often declarative.
- HTML is a problem-oriented markup language, but not a programming language in the strict sense.
- A useful correspondence: programming-language syntax is commonly described by context-free grammars, while lexemes are commonly described by regular languages; static-semantic constraints require additional checks.

---

<a id="q16"></a>

## 16. Language processors: compilers and interpreters. Stages of translation

> **Learning outcome.** After this chapter, you will be able to trace a program through the stages of translation and compare a compiler, an interpreter, and a JIT. First, try to explain the topic in your own words, and then test yourself with the block at the end.

**The essence of the question.** A processor executes machine instructions, while programmers usually write in high-level languages. A language processor bridges that gap. A compiler resembles a translator who prepares an entire book in advance: translation takes time first, but the processor can then run the generated program efficiently. An interpreter resembles a simultaneous translator: it processes and executes the program as it goes. Execution can begin immediately, but work may be repeated on later runs.

**Key concepts.**

- A translator converts a program from one language into an equivalent program in another language.
- A compiler translates a high-level program into machine code or object code. Object files are usually linked with libraries before execution; compilation and execution are separate steps.
- An assembler translates assembly-language mnemonics into machine code.
- A cross-compiler runs on one platform but generates code for another—for example, compiling microcontroller firmware on a laptop.
- An interpreter parses and executes a program as it runs, usually statement by statement, rather than first producing a standalone executable (for example, classic BASIC or an interactive Python session).
- Hybrid systems first compile source code into bytecode or another intermediate representation for a virtual machine. Java on the JVM and C# on .NET commonly use this approach. A just-in-time (JIT) compiler can translate frequently executed sections into native machine code while the program runs.

**Comparison in one table:**

| Criterion | Compiler | Interpreter |
|---|---|---|
| When code runs | After translation and usually linking | As the source or intermediate code is processed |
| Execution speed | Usually higher after compilation | Usually lower without JIT compilation |
| When errors are detected | Many errors are reported before execution | An error may appear only when execution reaches the relevant statement |
| Interactivity | Usually lower | Usually higher; statements can be run one at a time |
| Typical output | Object code or an executable file | Usually no standalone executable |

**Translation stages on a live example.** Let's trace what the compiler does with the line `pos = init + rate * 60`:

1. **Lexical analysis.** The scanner splits the character stream into lexemes (tokens): identifiers, numbers, operators, and other minimal meaningful units. Whitespace and comments are discarded, and identifiers are recorded in the symbol table. The example becomes the sequence: identifier `pos`, assignment, identifier `init`, plus, identifier `rate`, multiplication, number `60`. This stage is based on regular languages, regular expressions, and finite automata.
2. **Syntax analysis.** The parser checks whether the token sequence conforms to the language grammar and builds a syntax tree that shows how operations are nested. Because multiplication has higher precedence than addition, the tree represents `rate * 60` first, then addition of `init`, and finally assignment to `pos`. This stage is based on context-free grammars and pushdown automata.
3. **Semantic analysis.** The compiler checks meaning-related rules: whether every name is declared, whether operand types are compatible, and whether required conversions are allowed. For example, if `rate` is a floating-point value and the language permits the conversion, the integer `60` may be converted to `60.0`.
4. **Intermediate-code generation.** The tree is translated into a simpler, largely platform-independent representation in which each instruction performs one action: `t1 = rate * 60.0`, `t2 = init + t1`, and `pos = t2`.
5. **Optimization.** Semantics-preserving transformations improve speed or reduce code size. Examples include constant folding (`2 * 3` becomes `6` at compile time), eliminating repeated calculations and dead code, and moving loop-invariant calculations outside the loop.
6. **Code generation.** The intermediate representation is converted into instructions for a particular processor, including decisions about register allocation. The linker then combines object files and libraries into an executable file.

The symbol table and error-handling machinery support all translation phases; they are not separate sequential phases.

**Two methods of parsing.** Parsing is top-down and bottom-up:

- Top-down (LL family): the tree is built from the root to the leaves. The parser predicts which production to apply by examining lookahead tokens. The simplest form is recursive descent, with a procedure for each nonterminal. Left-recursive rules cause naive recursive-descent parsers to loop and must be transformed.
- Bottom-up (LR family): the tree grows from the leaves to the root. The parser alternates between **shift** (push the next token onto the stack) and **reduce** (replace a recognized sequence by the corresponding nonterminal). LR parsers handle a broader class of grammars than LL parsers and are commonly generated by tools such as yacc or Bison. For common programming languages, both approaches work in linear time—the parsing time grows proportionally to the length of the program.


### Reinforcement

**Additional worked example.** For `total = price + 2;`, the lexer generates the tokens IDENT, '=', IDENT, '+', NUMBER, ';'. The parser assembles the assignment tree, the semantic analysis checks the types, and the code generator creates the target machine instructions.

**Transfer example.** A JIT starts from an intermediate representation, gathers an execution profile, and compiles hot sections to optimized machine code. A long-running server process can therefore become faster after warm-up.

**Active recall.** Close the explanation above and answer without peeking: Which stage sees the individual tokens and which stage sees the structure of the expression? How does bytecode differ from the machine code of a specific CPU?

**Mini practice — check yourself.** At what stage is it most natural to discover that a string is being added to a number without type conversion allowed?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** During semantic analysis after constructing the syntax tree, when the operand types are already known.

</details>

**Exam focus:**

- "Is Java Compiled or Interpreted?" Hybrid: compilation to bytecode plus virtual machine with JIT.
- The order of phases is constantly asked: lexical → syntactic → semantic → intermediate code → optimization → code generation.
- Correspondence of levels: lexical analysis — regular languages and finite automata; syntax — context-free grammars and pushdown automata.
- LR parsers handle a broader class of grammars than LL parsers; left-recursive rules must be eliminated or transformed for LL/top-down parsing.

---

<a id="q17"></a>

## 17. Programming methods and OOP. Modularity, classes and objects, encapsulation, inheritance, and polymorphism

> **Learning outcome.** After this chapter, you will be able to model the domain with classes and explain encapsulation, inheritance, composition, and polymorphism at the contract level. First, try to explain the topic in your own words, and then test yourself with the block at the end.


**The essence of the question.** OOP organizes a program around objects that combine state with behavior. A class is like a vehicle blueprint; an object is a particular vehicle built from it. Encapsulation is like a closed hood: a driver uses the steering wheel and pedals instead of manipulating the engine directly. Inheritance lets a specialized type, such as `Truck`, reuse and extend a more general type, such as `Vehicle`. Polymorphism lets the same operation, such as `move()`, invoke behavior appropriate to the actual object.

**Paradigms in general.** A paradigm is a "philosophy" of building programs. Two major families are imperative (a program is a sequence of commands that change state; this includes procedural, structured, and object-oriented programming) and declarative (describing what to calculate, not how; this includes functional and logic programming, discussed in question 18).

**Modularity.** This is the principle of dividing a system into cohesive modules with clear public interfaces and hidden implementations. Good decomposition has two complementary qualities:

- high cohesion: everything inside the module is about one thing; the "work with dates" module does not print reports;
- weak coupling: modules are minimally dependent on each other, so replacing one does not break the others.

Plus the principle of information hiding (Parnas, 1972): the module hides solutions that can change so that the change remains local.

**Classes and objects.** A class is a description of structure and behavior: fields (variables within an object) plus methods (functions that work with them) in one unit. An object is an instance of a class, a concrete "thing" in memory. The object has three properties:

1. state — current field values;
2. behavior — methods that can be called;
3. identity — uniqueness, independent of state: two bank accounts with the same balance of 100 hryvnias are still different accounts.

A constructor is a special initialization method that runs when an object is created.

**Three (plus one) pillars of OOP.**

1. **Encapsulation** — data and methods live together in the class, and direct access to internal state is limited by modifiers: `private` (visible only inside the class), `protected` (also visible to derived classes), and `public` (visible to everyone). This allows the class to protect its invariants. If an account's balance cannot be negative, outside code cannot set it to minus five hundred directly; it must call a withdrawal method that validates the amount.
2. **Inheritance** — a new class is based on an existing class, inherits its available behavior, and can add or override behavior. It models an **is-a** relationship: a cat is an animal, and a truck is a vehicle. Do not confuse it with composition, which models a **has-a** relationship: a car has an engine, but an engine is not a kind of car. As a design rule of thumb, prefer composition when inheritance does not represent a genuine substitutable is-a relationship.
3. **Polymorphism** — one interface, different implementations. Common forms include:

   - **subtype polymorphism:** a virtual method call is resolved at runtime using the object's actual type. A variable declared as `Shape` may refer to a `Circle`, so calling `area()` uses the circle implementation;
   - **parametric polymorphism:** generic types, such as `List<T>`, allow the same code to work with many types;
   - **overloading:** several methods have the same name but different parameter lists, such as printing a number or a string.

4. **Abstraction** — the optional fourth pillar—highlights essential behavior while hiding irrelevant detail. Abstract classes and interfaces are common tools for expressing abstractions and contracts.

**Example of how everything works together.** There is an abstract class `Shape` with an `area()` method. `Circle` (area = πr²) and `Square` (area = a²) inherit from it. A `totalArea` function accepts a list of shapes and calls `area()` on each one without knowing its concrete type. A circle with radius 1 contributes approximately 3.14, and a square with side 2 contributes 4, for a total of approximately 7.14. A new `Triangle` implementation can be added without changing `totalArea`: this demonstrates the Open/Closed Principle.

**SOLID principles** (in words, one sentence at a time):

- Single Responsibility: in the class there is one responsibility — one reason for change.
- Open/Closed: software entities should be open for extension but closed for modification; new behavior should usually be added without changing stable existing code.
- Liskov Substitution: the descendant should work wherever the ancestor is expected without breaking anything.
- Interface Segregation: clients should depend on focused interfaces rather than one broad interface they do not fully use.
- Dependency Inversion: depend on abstractions rather than concrete implementations.


### Reinforcement

**Additional worked example.** The `Shape.area()` interface allows a list to contain Circle and Rectangle. Calling `area()` polymorphically selects the desired implementation without `if` by type; the client depends on the contract, not the details.

**Transfer example.** Car has Engine, so composition is more natural than inheritance: Car is not a type of Engine. The "is" versus "has" test helps to avoid building false hierarchies.

**Active recall.** Close the explanation above and answer without peeking: What exactly does encapsulation hide? When is composition safer than inheritance?

**Mini practice — check yourself.** The `BankAccount` class allows callers to assign a negative value directly to `balance`. Which principle is violated, and how should the API change?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Encapsulation is broken, so the class cannot protect its invariant. Make the field private and change it only through `deposit()` and `withdraw()` methods that validate the amount and available balance.

</details>

**Exam focus:**

- A class and an object are not the same thing: a class is a description (type), an object is an instance in memory; there can be any number of objects of the same class.
- Classic pitfall: overloading (the same name with different parameter lists, resolved at compile time) versus overriding (a derived class supplies a new implementation of an inherited method, resolved at runtime).
- Inheritance models an **is-a** relationship; composition models a **has-a** relationship. A car has an engine, so that relationship is composition.
- "Three pillars": encapsulation, inheritance, polymorphism; abstraction is sometimes called the fourth.

---

<a id="q18"></a>

## 18. Structured, functional, and logic programming

> **Learning outcome.** After this chapter, you will be able to compare structured, functional, and logic programming and select ideas from each style for a specific task. First, try to explain the topic in your own words, and then test yourself with the block at the end.

**The point of the question.** These are three different philosophies of writing programs. Structured programming builds a program from clear sequence, selection, and iteration blocks rather than chaotic jumps. Functional programming composes functions and emphasizes pure functions: the same arguments produce the same result without side effects. Logic programming records facts and rules, accepts a query, and uses inference to derive answers.

**Structured programming.** The methodology of Dijkstra and Wirth (1960-70s): the program is built from a hierarchy of only three basic structures, without the goto operator — an unconditional jump "to a label" that turns the logic into "spaghetti code." Three designs:

1. sequence — do one thing, then another;
2. branching — if the condition is true, make one branch, otherwise another;
3. iteration — repeat a block while a condition holds or until a stopping condition is met.

The theoretical basis is the Bohm–Iacopini theorem (1966): any algorithm can be implemented by a combination of only these three constructions. That is, goto is not needed in principle. Each block has one input and one output — the program can be read from top to bottom and even formally proved to be correct. Dijkstra's famous article was called "Go To Statement Considered Harmful" (1968). This also includes top-down design: we break down a large task into subtasks, those into smaller ones, down to elementary steps; units of breakdown — procedures and functions.

Example: the sum of numbers from 1 to n in three constructions — first a sequence (reset the sum to zero, set the counter to 1), then a loop (until the counter has exceeded n — add it to the sum and increase it), and a branch to check that n is not negative.

**Functional programming.** This declarative paradigm treats a program as a composition of functions, where the result of one can be passed to the next. Its theoretical basis is Alonzo Church's lambda calculus (1930s). Languages include Haskell, F#, Lisp, and Erlang; C# (LINQ), Java, and Python also support functional techniques.

Key ideas in words:

- **Pure function**: the result depends only on the arguments, and the function does not change anything in the surrounding world—it does not write to files or modify global variables. Sine is a pure function; printing to the screen is not, because it has a side effect.
- **Immutability of data**: values ​​are not modified; instead of "change the list", a new version of it is created. It sounds wasteful, but tricky data structures share common memory with older versions.
- **Higher-order functions** — functions that accept other functions as arguments. The trinity to know: map (apply the function to each element: "double each" from the list 1, 2, 3 gives 2, 4, 6), filter (leave the elements that pass the test: "only even" from 1, 2, 3, 4 gives 2, 4) and fold/reduce (collapse the list to one value). Example of fold: fold the list by adding 1, 2, 3, starting from zero: add 1 to 0 — it became 1; add 2 to 1 - it became 3; add 3 to 3 — it became 6. The result is the sum of the list.
- **Recursion and higher-order functions instead of mutable loops:** pure functional code commonly expresses iteration through recursion, `map`, `fold`, and related combinators.
- **Tail recursion:** the recursive call is the function's final operation. An implementation that guarantees tail-call optimization can execute it without growing the call stack.
- **Lazy evaluation**: an expression is evaluated only when its value is needed. This makes it possible to represent an infinite sequence, such as all natural numbers, and take only its first five elements.
- **Currying and partial application**: currying transforms a function of several arguments into a chain of one-argument functions; partial application fixes some arguments to create a new function—for example, deriving `addOne` from `add` by fixing one argument to 1.

The main practical benefit: without shared mutable state, threads do not fight for data — parallelism and testing are dramatically simplified.

**Logic programming.** This declarative paradigm is based on predicate logic: a program is a set of facts and rules, and computation is the logical derivation of an answer. Its best-known language is Prolog (1972). Three basic constructs are:

- fact: "Tom is Bob's father" is an unconditional truth;
- rule: "X is the grandfather of Z if X is the father of Y and Y is the father of Z";
- query: "For which Z is Tom the grandfather?"

**How the system derives an answer.** Given the facts "Tom is Bob's father" and "Bob is Anna's father," plus the grandfather rule, the query asks whose grandfather Tom is. The system first finds Tom's child, Bob, and then Bob's child, Anna. It therefore concludes that Tom is Anna's grandfather. If a candidate fails at the second step, backtracking returns to an earlier choice and tries another candidate; this is the normal search mechanism, not an error.

Two more concepts in words: unification is two-way matching that finds variable substitutions which make two terms identical, unlike one-way assignment. **Negation as failure** means that a goal is treated as false when it cannot be proven from the knowledge base; this relies on a closed-world assumption and differs from proving that the goal is false. Kowalski's useful formula is: algorithm = logic + control (logic states what is true; control determines how to search). Applications include expert systems, knowledge bases, natural-language processing, and combinatorial problems.


### Reinforcement

**Additional worked example.** A structured sum algorithm iterates over an array and changes an accumulator. The functional expression `sum(map(square, xs))` composes pure transformations. A logic-programming solution specifies facts and a rule, such as `grandparent(X,Z) :- parent(X,Y), parent(Y,Z)`.

**Transfer example.** A pure `tax(income)` function with the same argument always produces the same result and does not change the world. It is easy to test and safe to run in parallel.

**Active recall.** Close the explanation above and answer without peeking: What three control constructs are sufficient for structured programming? Why does immutability facilitate concurrency?

**Mini practice — check yourself.** Identify the style: "Specify the relationships that must hold, and let the inference engine search for values that satisfy them."

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** This is logic programming, which is a declarative style: facts, rules, and a query are specified, not the order of search steps.

</details>

**Exam focus:**

- The Bohm–Iacopini theorem is precisely about the sufficiency of three constructions; goto is not needed in principle.
- A pure function satisfies two conditions at the same time: the same input gives the same output, and there are no side effects. A random-number generator and printing to the screen are not pure.
- Classification trap: functional and logic programming are declarative; structured programming is imperative.
- In Prolog, the sign ":-" is read as "if", and variables are written with a capital letter - the opposite of the usual languages.

---

<a id="q19"></a>

## 19. Software specification, verification, and testing

> **Learning outcome.** After this chapter, you will be able to distinguish between specification, verification, validation, and testing, and construct checks from equivalence classes and boundary values. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Imagine building a house. A specification is a blueprint: a detailed description of what should come out. Verification is a check that the builders are building exactly according to the drawing. Validation is a check that the house itself suits the customer (because the drawing could also be unsuccessful). And testing is when you walk around a finished house and try: whether the door opens, whether the faucet is running. Important: this way you will find individual problems, but you will never prove that they no longer exist.

### Specification

A specification is a precise description of *what* a program is supposed to do, without answering the question of *how* it will do it. This is a benchmark: the program is considered correct relative to the specification, not "in general". Specifications come in different levels of rigor: informal (plain text - terms of reference, list of requirements), semi-formal (schemas with rules, such as UML diagrams or use cases), and formal - mathematically rigorous descriptions (Z-notation, VDM, TLA+, contracts).

One of the best-known formal techniques is the **Hoare triple**, written in three parts:

```
{P} S {Q}
```

It means: if precondition `P` holds before statement `S` executes and `S` terminates, then postcondition `Q` holds afterward. For example, `{x = 5} x := x + 1 {x = 6}` states that starting with `x = 5` and executing the assignment produces `x = 6`. A verification proof must establish that the triple follows from the rules of Hoare logic.

Note the phrase "if `S` terminates." A Hoare triple by itself expresses **partial correctness**: if `S` terminates, its result satisfies `Q`. **Total correctness** additionally requires a proof that `S` terminates.

Design by contract (Bertrand Meyer, Eiffel language) grew out of this idea: each method has a contract—preconditions that the caller must satisfy, postconditions that the method guarantees, and class invariants that must hold before and after each public operation.

### Verification and validation are not the same thing

Verification answers the question "are we building the product correctly?" — that is, does the code match the specification. Validation — "are we building the right product?" — whether it meets the real needs of the user. This is a classic formulation by Barry Boehm, and this difference is often asked in the exam.

### How to verify

**Deductive verification** — mathematical proof of correctness according to the rules of Hoare logic. Loops are especially important. To prove a loop, you need a *loop invariant*: a statement that is true before the loop and remains true after each iteration. Suppose `s = 0` and `i = 0`, and each iteration increments `i` and then adds it to `s`. The invariant is: "`s` equals the sum of the integers from 1 through `i`." It is true initially, and each iteration preserves it. When the loop terminates with `i = n`, the invariant implies that `s` is the sum from 1 through `n`.

Termination of the loop is proved with a *variant function*—a non-negative value that strictly decreases on every iteration and therefore cannot decrease indefinitely. In the example, `n - i` decreases from `n` to zero. Tools for deductive verification include Coq, Isabelle, Dafny, and Frama-C.

**Model checking** exhaustively explores the state space of a finite-state model and checks properties expressed in temporal logic. The two main classes are safety ("something bad never happens"—for example, two threads are never in a critical section simultaneously) and liveness ("something good eventually happens"—for example, every accepted request is eventually processed). Tools include SPIN, NuSMV, and TLA+. The main challenge is state-space explosion: the number of states grows rapidly with the number of components.

**Static analysis** — code analysis without running it: linters, data flow analysis, abstract interpretation.

### Testing

Testing — experimental verification of the program on a finite set of tests. Dijkstra's key thesis: *testing can show the presence of errors, but not their absence*. A test is an input plus an expected output; the source of the "right answer" is called an oracle.

How tests are classified:

- **By access to the code.** In black-box testing, we derive tests from the specification without inspecting the implementation. Two useful techniques are equivalence partitioning (divide inputs into classes expected to behave alike and select representatives) and boundary-value analysis (test the edges). Suppose a function accepts an age from 0 to 120. The classes are negative values (invalid), 0–120 (valid), and values above 120 (invalid). Six boundary-focused tests are `-1`, `0`, `1`, `119`, `120`, and `121`. In white-box testing, we inspect the code structure and measure whether statements and branches are executed.
- **By level:** unit (an individual function or class), integration (interactions between modules), system (the complete system), and acceptance (validation by the customer). Regression testing reruns checks after changes to make sure existing behavior has not regressed; smoke testing quickly checks whether the basic system starts and works.
- **By method:** manual and automated (JUnit, NUnit, pytest; "prepare data — perform action — check result" pattern).

A few more useful techniques are TDD (write a failing test, write the minimum code needed to pass it, then refactor), mutation testing (make small intentional code changes and check whether the tests detect them), fuzzing (feed the program automatically generated or malformed inputs), and property-based testing (check properties such as "reversing a list twice returns the original list" on many generated inputs).

And a useful chain of concepts: human error creates a defect (bug) in the code, and a defect at runtime manifests itself as a failure. Finding and eliminating a defect is debugging.


### Reinforcement

**Additional worked example.** For the age field 18…65, the equivalence classes are below 18, acceptable range, above 65. Boundary-value tests: 17, 18, 65, 66. Four values provide more information than dozens of random middle values.

**Transfer example.** The specification may require ascending sorting. The test with [2, 1, 2] checks for order and duplicates, but does not prove correct for all inputs; formal verification tries to prove the invariant in general.

**Active recall.** Close the explanation above and answer without peeking: How is "building the product right" different from "building the right product"? Why are successful tests not a proof of the absence of errors?

**Mini practice — check yourself.** For a division function, give one normal test, one boundary test, and one negative test.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Assuming 32-bit signed integers: normal—`6 / 3 = 2`; boundary—`INT_MAX / 1 = INT_MAX`; negative—division by zero must produce the error specified by the contract.

</details>

**Exam focus:**

- Verification and validation are different things: the first is conformity to the specification, the second is to the user's needs.
- "Does the testing prove that there are no errors?" — no, quote Dijkstra.
- A Hoare triple expresses partial correctness; total correctness also requires proving termination, often with a variant function.
- Do not confuse an invariant (remains true) with a variant (strictly decreases and helps prove termination).

---

<a id="q20"></a>

## 20. Distributed computing: transparency, openness, flexibility, and extensibility

> **Learning outcome.** After this chapter, you will be able to explain the goals of a distributed system and analyze the trade-offs of transparency, fault tolerance, scalability, and consistency. First, try to explain the topic in your own words, and then test yourself with the block at the end.


A distributed system is when many separate computers work together in such a way that it appears to the user as if it were one computer. Like a network of bank branches: you go to any branch and see "one bank", although inside there are hundreds of servers in different cities that communicate with each other by messages.

### What is it and why?

Andrew Tanenbaum's classic definition says that a distributed system is a collection of independent computers that appears to its users as a single coherent system. Its nodes do not share one physical memory or a perfectly synchronized global clock; they coordinate by exchanging messages over a network.

Why is this being built: to share resources (files, printers, data), to scale "wide" — to add more machines when more capacity is needed; to be reliable through replication — storing multiple copies of data on different nodes; to serve users worldwide; and to save - a cluster of cheap standard servers instead of one super expensive one.

### Four main tasks

**Transparency** hides selected distribution details from users and programs. Eight commonly taught forms are: access (local and remote resources use similar operations), location (a resource's physical location is hidden), migration (a resource can move), relocation (it can move while in use), replication (one logical resource may have several replicas), concurrency (many clients can share a resource safely), failure (some failures and recovery are hidden), and persistence (whether a resource is in memory or durable storage is hidden). For example, if two users try to buy the final ticket, concurrency transparency helps ensure that exactly one purchase succeeds. Complete transparency is unattainable and not always desirable: network latency and partial failures cannot be hidden completely.

**Openness** — the system is built on published standard interfaces and protocols (REST/HTTP, gRPC, interface description languages). This provides interoperability (components from different manufacturers understand each other) and portability (a component can be run in another environment), as well as the ability to replace parts without rebuilding everything.

**Flexibility** — the system's ability to change and evolve: modular architecture, loose coupling, separation of policy ("what to do") and mechanism ("how to do"). Practical tools: middleware — an intermediate "layer" between the OS and applications, message queues, microservices.

**Extensibility** is the ease with which new components or functions can be added through stable interfaces. **Scalability** is the ability to maintain acceptable service as the number of users, nodes, regions, or administrative domains grows. Scalability techniques include asynchronous communication, caching, replication, sharding (partitioning data across servers), and decentralization. Vertical scaling means using a more powerful server; horizontal scaling means adding more servers.

### Fundamental limitations

A defining difficulty is *partial failure*: one node or network link may fail while the rest of the system continues. Another node often cannot immediately distinguish a crash from a slow response or a network partition.

The second problem is the absence of a perfectly shared global time: physical clocks on different machines cannot simply be compared as if they were identical. Lamport logical clocks assign counters to events: a local event increments the local counter, and receiving a message sets the counter to one more than the maximum of the local and received values. If event `a` causally precedes event `b`, then `L(a) < L(b)`, although the converse is not necessarily true. Vector clocks can additionally identify concurrent events, where neither causally precedes the other.

**CAP theorem** (Eric Brewer): during a network partition, a distributed data system cannot guarantee both linearizable consistency (operations behave as if there were one up-to-date copy) and availability (every request to a non-failing node receives a non-error response). If two data centers lose contact after a write, the second must either reject or delay an operation to preserve consistency, or respond using its local, potentially stale state to preserve availability. Eventual consistency is one common availability-oriented design: replicas are allowed to differ temporarily but converge later.

**FLP theorem:** in a fully asynchronous message-passing system, even one possible crash failure makes it impossible for a deterministic consensus algorithm to guarantee termination in every admissible execution. Paxos and Raft preserve safety but obtain practical liveness by relying on periods of synchrony and timeouts; they do not invalidate FLP. For distributed transactions, two-phase commit first asks participants to prepare and then tells them to commit or abort.

**Byzantine Generals problem:** in the classic unauthenticated Byzantine consensus model, tolerating `f` Byzantine faults requires at least `3f + 1` replicas. Thus tolerating one Byzantine node requires at least four replicas; different assumptions, such as authenticated messages, can change the bound.

And finally, eight false assumptions of distributed computing (Peter Deutsch) that beginners mistakenly believe: that the network is reliable, latency is zero, bandwidth is infinite, the network is secure, the topology is immutable, there is one administrator, data transfer is free, and the network is homogeneous. All this is not true.


### Reinforcement

**Additional worked example.** Three service replicas behind a load balancer provide location transparency: a client uses one service name without knowing which node handles the request. Keeping session state only in one replica makes routing and failover depend on that node. Stateless replicas or shared session storage preserve transparent routing, whereas sticky sessions are only a limited workaround.

**Transfer example.** During a network partition, a system cannot guarantee both immediately consistent responses and availability in every partition. A practical architecture must define its behavior explicitly rather than hide this trade-off.

**Active recall.** Close the explanation above and answer without peeking: What types of transparency does the user see? Why is a partial failure more difficult than a complete local application crash?

**Mini practice — check yourself.** One of the three nodes is not responding, but two are healthy. Name two techniques that will allow you to continue the service.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** A load balancer with health checks can stop routing traffic to the unhealthy node. Replication with an appropriate quorum can continue serving requests from the two healthy replicas. Timeouts prevent clients from waiting indefinitely.

</details>

**Exam focus:**

- CAP is not "choose two of three forever": the trade-off between consistency and availability arises while a network partition exists; partition tolerance is a required design concern when partitions cannot be ruled out.
- Know the common forms of transparency—access, location, migration, relocation, replication, concurrency, failure, and persistence—and remember that complete transparency is unattainable.
- Do not confuse: interoperability (different manufacturers work together) and portability (the same component in a different environment); vertical and horizontal scaling.
- FLP concerns guaranteed termination of deterministic consensus in a fully asynchronous system with a possible crash; Paxos and Raft make progress under additional timing assumptions while preserving safety.

---

<a id="q21"></a>

## 21. MapReduce: the Map, shuffle, and Reduce stages

> **Learning outcome.** After this chapter, you will be able to decompose a batch task into Map, shuffle/group, and Reduce and evaluate what can be done locally before transmission over the network. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Imagine counting votes in elections. Instead of taking all the ballots to the capital, each precinct counts its own — this is Map, parallel preliminary processing on the ground. Then the results for each candidate are collected together and added — this is Reduce, a convolution. MapReduce is the same scheme, only for a computer cluster: you write two small functions, and all "logistics" - distribution, parallelism, failures - are taken over by the framework.

### The essence

MapReduce is a programming model and framework for distributed batch processing on clusters of commodity hardware. Google introduced it in a 2004 paper by Jeffrey Dean and Sanjay Ghemawat. The model is inspired by the map and reduce operations from functional programming, and records are processed as key–value pairs. Formally, the signatures are as follows:

```
map:    (k1, v1)        → list(k2, v2)
reduce: (k2, list(v2))  → list(k3, v3)
```

In other words: map generates a list of intermediate pairs from one input pair; reduce takes the key and *all* the values ​​bound to it and collapses them into the result.

### How to complete the task, step by step

1. **Split.** The input is divided into logical input splits. Each split is assigned to a map task, which creates data parallelism by applying the same operation to different portions at the same time.
2. **Map.** Each map task applies the map function to every input record in its split. Map tasks are independent and can run in parallel. The framework tries to run a task near the data it reads: move computation to the data instead of moving large datasets over the network.
3. **Combine** (optional). A combiner performs local aggregation on mapper output to reduce network traffic. Because it may run zero, one, or several times, it is safe only when it cannot change the final result; summing counts is safe, while averaging averages is not.
4. **Shuffle and sort.** Intermediate pairs are partitioned among reducers, commonly by a hash of the key, and then transferred, sorted, and grouped by key. All values for the same key are sent to the same reducer. This is usually the most network-intensive stage.
5. **Reduce.** For each unique key, the reduce function receives that key and its grouped values, produces output records, and writes them to the distributed file system.

### A canonical example is WordCount

Counting word frequencies. For each word in the text, Map outputs a "word, unit" pair. Reduce takes a word and a list of units — and simply sums them up.

Let's trace on small data. Two documents: the first - "cat dog cat", the second - "dog cat"; two mappers and two reducers.

| Stage | What is happening |
|---|---|
| Map | Mapper1 outputs (cat,1), (dog,1), (cat,1); Mapper2 — (dog,1), (cat,1) |
| Combine on Mapper1 | (cat,2), (dog,1) — traffic decreased from 3 pairs to 2 |
| Shuffle | all "cat" go to Reducer0: cat → [2, 1]; all "dog" - to Reducer1: dog → [1, 1] |
| Reduce | Reducer0 issues (cat, 3); Reducer1 — (dog, 2) |

Answer: "cat" - three times, "dog" - twice. The key point: all "cats" gathered at one reducer — this is the guarantee of shuffle.

### Fault tolerance

The coordinator monitors tasks and reruns failed task attempts on other workers. It may also launch duplicate attempts for unusually slow tasks and accept the first successful framework-managed output. Because retries and speculative execution can run a task more than once, user functions should be deterministic and avoid externally visible side effects.

### Properties and limitations

MapReduce can scale horizontally, but speedup is limited by shuffle traffic, serialization, data skew, stragglers, and sequential work. WordCount examines O(N) input records; shuffle sorting is comparison-based within each partition and is often summarized as O(N log N) in the worst case, although the exact cost depends on partitioning and implementation.

The main limitation of the model is that it is *batch* processing with high latency, not a good fit for interactive queries. It is especially inefficient for iterative algorithms such as PageRank, k-means, and gradient descent because Hadoop MapReduce materializes intermediate results between jobs. This motivated systems such as Apache Spark, which can cache reusable intermediate data in memory and avoid forcing every stage boundary through HDFS, while still spilling to disk when necessary.

Typical applications: inverted search engine index (word → document list), log analysis, distributed pattern search, sorting giant arrays, statistics, ETL (extract-transform-load data).


### Reinforcement

**Additional worked example.** For words `cat dog cat` Map outputs (cat,1), (dog,1), (cat,1). Shuffle groups cat→[1,1], dog→[1], Reduce sums and gets cat→2, dog→1. The key determines which records will be processed by the same reducer.

**Transfer example.** Combiner can locally convert a thousand pairs (cat,1) to one (cat,1000) even before the network. This is safe for sum, but not for plain average without passing the (sum, count) pair.

**Active recall.** Close the explanation above and answer without peeking: What is shuffle responsible for? When can a combiner change the correct result?

**Mini practice — check yourself.** Design a Map key and value to calculate the total amount of sales by city.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Map issues `(city, amount)` for each sale; shuffle collects sums of one city; Reduce adds amounts. If necessary, the combiner does a local sum.

</details>

**Exam focus:**

- Reduce doesn't get a single pair, but a key and a list of *all* its values - they were collected by shuffle.
- A combiner is optional and may run zero, one, or several times, so using it must not change the result. Sum is safe; a plain arithmetic mean is not unless the combiner passes a `(sum, count)` pair.
- Know map and reduce signatures by heart.
- Favorite follow-up question: why MapReduce is bad for iterations and how Spark answered (memory instead of disk).
- Data locality: calculations go to the data, not the other way around.

---

<a id="q22"></a>

## 22. Hadoop: principles, HDFS, YARN, MapReduce, non-relational data, and use cases

> **Learning outcome.** After this chapter, you will be able to describe the roles of HDFS, YARN, and MapReduce and explain how data locality and replication support large-scale batch computing. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Hadoop is an "operating system" for large data warehouses from hundreds of low-cost computers. HDFS - Warehouse Shelves: Where and how boxes of data blocks are stored, each in triplicate in case a shelf falls. YARN is a dispatcher that distributes jobs and resources to employees. And MapReduce is the teams themselves that process data according to the scheme from the previous question.

### What is this

Apache Hadoop is an open platform (in Java) for distributed storage and batch processing of big data on clusters of cheap standard servers. In fact, it is an open implementation of Google's ideas: the GFS file system has become HDFS, and Google MapReduce has become Hadoop MapReduce.

### Basic principles

- **Horizontal scaling:** capacity is increased by adding cheap nodes, not by buying a more expensive server.
- **Fault tolerance due to replication:** node failures are the norm, not the exception; data is copied, tasks are restarted.
- **Locality of data:** calculations go to the node where the block of data lies - minimum network traffic.
- **Schema-on-read:** the structure is applied to the data when reading, not when writing (in relational databases, it's the opposite), so you can store "raw" semi-structured data as is.
- **WORM** (write once, read many): the file is written once and read many times; arbitrary editing is not supported.

### Three core components

**HDFS** is a distributed file system. Files are divided into large blocks; 128 MiB is a common default, but the block size is configurable. The default replication factor is commonly 3 and is also configurable. Rack-aware placement distributes replicas across nodes and racks; the exact placement depends on the cluster topology and Hadoop version. The NameNode stores *metadata* — which files exist, which blocks form them, and where their replicas are located. The blocks themselves live on DataNodes, which regularly send the NameNode heartbeats and block reports.

For example, a 300 MiB file with a 128 MiB block size is divided into blocks of 128, 128, and 44 MiB. With replication factor 3, the replicas contain 900 MiB of logical block data, excluding filesystem overhead. When reading, the client asks the NameNode where the blocks are, receives a list of DataNodes, and reads *directly from them* — the data does not flow through the NameNode. If a DataNode stops sending heartbeats, the client can read a healthy replica and the NameNode schedules replication to restore the configured factor. HDFS is efficient for streaming large files, but millions of small files consume substantial NameNode metadata memory, and HDFS is not designed for low-latency random access.

**YARN** is a cluster resource manager (appeared in Hadoop 2). Three roles: ResourceManager — global resource planner; NodeManager is an agent on each node that manages containers (a container is an allocated processor and memory quota for a task); ApplicationMaster is a separate "foreman" of each application, which negotiates resources and manages its tasks. The main achievement of YARN: it separated resource management from the calculation model, so MapReduce, Spark, Flink and others live on the same cluster at the same time.

**Hadoop MapReduce** is a batch computing engine on top of YARN. Intermediate results are written to disk: this provides fault tolerance, but also high latency.

### Work with non-relational data

Hadoop can store unstructured and semi-structured data as well as binary formats. Avro is row-oriented, whereas Parquet and ORC are columnar formats. Column pruning and predicate pushdown can prevent query engines from reading irrelevant data, especially with Parquet and ORC.

A whole ecosystem has grown up around the core: HBase is a distributed wide-column database built on HDFS (an analogue of Google Bigtable) for quick access by key; Hive is a SQL query system that can use execution engines such as MapReduce, Tez, or Spark; Pig is a data-flow language for ETL; Sqoop exchanges data with relational databases; Flume and Kafka ingest streaming data; ZooKeeper provides distributed coordination; and Oozie schedules workflows.

### Examples of use

Analysis of web server logs and user click sequences; ETL and corporate data warehouses; indexing and search (actually, Hadoop was born from the Nutch search project); recommendation systems, anti-fraud and risk analysis in banks; data lake — "data lake", a single repository of raw data in any format for further analytics and machine learning.


### Reinforcement

**Additional worked example.** A 300 MiB file with a 128 MiB block size becomes three blocks: 128, 128, and 44 MiB. With a replication factor of 3, the cluster stores three replicas of each block on different DataNodes.

**Transfer example.** The scheduler tries to run a map task on a node that already has the required HDFS block. It is cheaper to move small code to data than to drag hundreds of megabytes over the network.

**Active recall.** Close the explanation above and answer without peeking: What does the NameNode store and what does the DataNode store? What problem does YARN solve?

**Mini practice — check yourself.** A DataNode with a block crashes while reading. What happens when there is a healthy replica?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** The client reads the block from another healthy replica. The NameNode later schedules replication to restore the configured replication factor.

</details>

**Exam focus:**

- Trap: "NameNode stores data?" No, only metadata; blocks lie on DataNodes, and data is read directly from them.
- Common defaults are a 128 MiB block size and a replication factor of 3, but both settings are configurable; rack-aware placement spreads replicas across failure domains.
- "Is Hadoop a database?" No, it is a storage and processing platform; HBase provides a database on top of HDFS, while Hive provides a SQL query layer.
- Schema-on-read (Hadoop) vs. schema-on-write (relational DBMS) is a frequent comparative question.
- YARN separates resource management from processing engines, allowing MapReduce and other distributed applications to share a cluster.
- HDFS does not like small files and cannot edit files arbitrarily.

---

<a id="q23"></a>

## 23. Distributed information-processing environments, data warehousing, and interaction models

> **Learning outcome.** After this chapter, you will be able to compare operational and analytical environments, read a data-warehouse schema, and distinguish between centralized coordination and distributed coordination. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Imagine a large restaurant kitchen: many cooks prepare different parts of the order, and the customer sees one finished dish. So is a distributed system: it is a bunch of separate computers connected by a network, which work together in such a way that they look like one big computer to the user. This, by the way, is Tanenbaum's classic definition. Nodes have no shared memory — they can only communicate by forwarding messages across the network, like chefs echoing across the hall.

In order for this "magic of unity" to work, a special layer - middleware - is placed between the operating system and programs. It is such a "translator and dispatcher" that hides from the programmer all the network details: where the data is physically located, how many copies there are, which node is currently alive.

The main virtue of a distributed system is transparency: the user simply does not notice that the system is distributed. It is distinguished by what exactly is hidden: access transparency (no difference in data formats is visible), location (you do not know where the file is physically located), migration (the resource moved to another server - no one noticed), replication (there are several copies, but it seems that there is only one), parallelism (the resource is used by others at the same time) and failures (some node died, but the service lives on).

But you have to pay for beauty, and here are the main pain points:

- **There is no common clock.** You cannot ask for one perfectly synchronized physical time for every node. Logical clocks therefore track event ordering instead of wall-clock time. With a Lamport clock, a process increments its counter for an event; a receiver sets its counter to one more than the maximum of its local value and the received timestamp. This preserves the rule that if event A happened before event B, then A has a smaller timestamp, although the converse is not guaranteed. Vector clocks keep one counter per process and can distinguish causal ordering from concurrent events, but they do not prove real-world causation.
- **CAP theorem (Brewer).** During a network partition, a distributed system cannot guarantee both linearizable consistency and availability for every request. Partition tolerance means continuing to operate despite lost or delayed messages, so a partition forces a trade-off between consistency and availability.

For a balance operation, a bank may favor consistency and reject a request rather than return an incorrect value. A social feed may favor availability and return slightly stale data. These are design choices made for particular operations, not universal rules for every bank or social network.

- **Partial failures.** Some nodes may fail while others continue running, so the system must distinguish a crashed node from a slow or unreachable one. The FLP result states that, in a completely asynchronous system, no deterministic consensus algorithm can guarantee termination if even one process may crash. Paxos and Raft preserve safety under these conditions and obtain practical liveness by assuming periods of sufficient synchrony and using timeouts; timeouts do not invalidate FLP.

### Problem-oriented environments

These are distributed environments, "sharpened" for a specific class of tasks, and not universal. Examples: grid-systems connecting computers of many organizations for scientific calculations (BOINC); OLTP transaction processing environments - many short record operations of the "withdraw money from the account" type; OLAP analytical environments — complex queries for reports; streaming (Kafka, Flink), where data is processed "on the fly" as it arrives.

### Data Warehousing

According to Inmon, a data warehouse is a subject-oriented, integrated, time-variant, and nonvolatile collection of data that supports decision-making.

Data commonly reaches the warehouse through ETL: it is extracted from operational sources, transformed by cleaning and integrating formats and meanings, and then loaded into the warehouse.

In a multidimensional model, **facts** contain measurements such as sales amount, while **dimensions** describe analysis axes such as time, product, and city. In a star schema, a central fact table is surrounded by denormalized dimension tables; in a snowflake schema, dimension tables are normalized into related subtables. OLAP operations include roll-up, which aggregates from cities to countries; drill-down, which reveals detail from years to months; slice, which fixes one dimension value; dice, which selects a subcube using several conditions; and pivot, which changes the report's orientation.

A small example. Sales: Kyiv 100 UAH on March 1, Kyiv 50 on March 2, Lviv 70 on March 1. Roll-up by time to month: Kyiv — 150, Lviv — 70. Slice "city = Kyiv" leaves only the first two entries.

Do not confuse OLTP with OLAP. OLTP handles many short operational transactions and usually emphasizes current state and normalized schemas. OLAP handles scan- and aggregation-heavy analytical queries, often over historical warehouse data organized in dimensional schemas.

### Interaction models

1. **Client-server.** The server provides a service and the client consumes it, commonly through request–response protocols such as HTTP. A single unreplicated server can be a bottleneck and single point of failure, but replication and load balancing can remove that limitation.
2. **Central coordinator.** A coordinator directs participants. In two-phase commit (2PC), it first asks whether every participant can commit and then announces commit only if all vote yes; otherwise it announces rollback. Traditional 2PC can block after certain coordinator failures until the coordinator or its state is recovered.
3. **Distributed coordination.** Nodes collectively maintain an agreed state. In Raft, nodes elect a leader that replicates log entries; an entry becomes committed after it is stored on a majority of the cluster, subject to Raft's term rules.

Under the usual Byzantine-fault model, tolerating `f` Byzantine nodes requires at least `3f + 1` nodes, so tolerating one requires four. Other decentralized techniques include distributed hash tables, consistent hashing, and gossip protocols. Some systems built with these techniques provide eventual consistency, but it is not an automatic property of the techniques themselves.


### Reinforcement

**Additional worked example.** In the star schema, the fact `Sales(amount, date_id, product_id)` is joined to the Date and Product dimensions. The query "sales by category by month" aggregates facts across these two dimensions.

**Transfer example.** A central coordinator simplifies the distribution of tasks, but creates a bottleneck and the risk of a single point of failure. Distributed consensus is more complex, but survives the loss of individual nodes in the presence of a quorum.

**Active recall.** Close the explanation above and answer without peeking: How is an OLTP workload different from an OLAP one? What is the role of fact tables and dimension tables?

**Mini practice — check yourself.** The operational database receives thousands of short records and the analyst runs a difficult grouping over the years. Where to perform the second request and why?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** In a separate storage/OLAP system: It is optimized for scans and aggregations and heavy query will not interfere with OLTP transactions.

</details>

**Exam focus:**

- CAP is not simply "pick any two." During a network partition, the relevant choice is between consistency and availability.
- 2PC is an atomic-commit protocol, not a universal consensus algorithm; it can block if the coordinator fails at a critical point.
- OLTP supports operational transactions, whereas OLAP supports analytics; star schemas use denormalized dimensions and snowflake schemas normalize them.
- FLP concerns guaranteed termination of deterministic consensus in a fully asynchronous system with a possible crash failure. Practical protocols rely on additional timing assumptions or randomization; timeouts alone do not disprove FLP.

---

<a id="q24"></a>

## 24. High-load systems and high-performance computing. HPC architectures, parallelism, and multithreading

> **Learning outcome.** After this chapter, you will be able to distinguish between high-load and HPC, determine levels of parallelism, and find races, deadlocks, and speedup limits. First, try to explain the topic in your own words, and then test yourself with the block at the end.


First, let's separate two concepts that are constantly confusing. A highly loaded system (highload) is a supermarket with a million customers: a lot of small independent requests, and the main thing is to quickly serve everyone. High-performance computing (HPC) is one giant scientific task (weather forecasting, climate modeling) that is chopped up and calculated on thousands of processors simultaneously. They have one thing in common: parallelism, that is, the execution of many things at the same time.

High-load systems are measured by throughput, such as requests per second, latency, and availability. A p99 latency of 200 ms means that 99% of measured requests completed within 200 ms; percentiles reveal slow tails that an average can hide. Typical techniques include horizontal scaling, load balancing, caching, sharding, and queues.

In the HPC world, FLOPS is measured - the number of floating-point operations per second; according to this indicator, the TOP500 rating is made, and the modern leaders have already crossed the exaflop milestone (a billion billion operations per second).

### Flynn's classification

Computers are divided by how many instruction streams and how many data streams they process simultaneously:

| Class | Meaning | Example |
|---|---|---|
| SISD | one command - one data | ordinary single-processor PC |
| SIMD | one command over a bunch of data at once | vector instructions, GPU |
| MISD | many commands on the same data | exotic, almost does not happen |
| MIMD | many commands, many data | multicores, clusters |

### Architectures of HPC systems

- **Shared Memory (SMP).** All processors see the same memory as a board that everyone reads from and writes to. It happens that access is equally fast from everywhere (UMA), and it happens that "your" memory is closer than "someone else's" (NUMA). The headache is that copies of data in caches of different cores do not diverge (coherence of caches).
- **Distributed memory (MPP).** Thousands of nodes, each with its own memory, communicating only by messages over a high-speed network.
- **Clusters.** Conventional servers plus fast networking are the de facto standard of modern HPC. In fact, it is a hybrid: memory is distributed between nodes, shared within a node.
- **Heterogeneous systems.** Processor plus accelerators, most often GPU (programmed via CUDA). Most of the TOP500 leaders are just like that.

### How much parallelism gives: Amdahl's law

The intuition is simple: if a part of the work cannot be parallelized in principle, it will become a bottleneck. Formally, if the share of sequential code is equal to s, then the acceleration on p processors:

```
S(p) = 1 / (s + (1 - s)/p),   and never greater than 1/s
```

In other words, the speedup is limited by the reciprocal of the serial fraction. For example, if 10% of the code is serial (`s = 0.1`), four processors give a speedup of about 3.08 rather than 4. The upper limit is 10 even with arbitrarily many processors. Gustafson's law considers a scaled problem whose parallel workload grows with the available processors. Parallel efficiency is speedup divided by the number of processors: here it is approximately 0.77.

### Levels of parallelism

From smallest to largest:

1. **Bit** - the processor adds 64-bit numbers in one cycle simply because it has wide "iron".
2. **Instruction level** — the processor overlaps or executes instructions using techniques such as pipelining, superscalar execution, and branch prediction.
3. **Data level** — one operation on the entire array at once: vectorization, GPU.
4. **Thread level** — multicore, Hyper-Threading.
5. **Process level** — multiple processes communicating through the Message Passing Interface (MPI).
6. **Task level** — independent tasks on the cluster that are scattered by the scheduler (SLURM).

### Multi-threaded programming

A thread is a "light" worker within a process. Key fact: threads of the same process share common memory (code, heap, global variables), but each has its own stack and registers. Therefore, it is cheaper to switch between threads than between processes — but shared memory creates a headache.

The main danger is the race condition: the result depends on who happens to be the first. Classic: Two threads do `counter++`, which is actually three steps - read, add, write. Both read zero, both added one, both wrote one — expected 2, got 1, one update lost. A section of code with shared data is called a critical section: there should be only one thread at a time.

Remedies are synchronization primitives: mutex (the lock — whoever captured it works, the rest waits), semaphore (permission counter, invented by Dijkstra), conditional variables (wait for an event), barriers (all threads wait for each other at the collection point), atomic operations like "replace the value if it's still old."

The second danger is deadlock: threads wait forever for one another, like two courteous drivers on a narrow bridge. It can occur only when all four Coffman conditions hold at once: mutual exclusion, hold and wait, no preemption, and circular wait. Breaking any one of them prevents deadlock; a common technique is to acquire locks in a consistent global order.

Toolkit: for shared memory — POSIX threads and OpenMP (placed directives above the loop — and it is parallel), for distributed — MPI (explicit sending and receiving of messages); they are often combined, also adding CUDA for GPUs. Typical patterns: thread pool, producer-consumer, master-worker, fork-join.


### Reinforcement

**Additional worked example.** Two threads execute `counter++` from value 0. Both can read 0 and write 1, so the result is 1 instead of 2. An atomic operation or mutex protects a critical section.

**Transfer example.** According to Amdahl's law, if 10% of the program is serial, even an infinite number of cores gives a speedup of no more than 1/0.1 = 10. Optimizing only the parallel part will not remove this ceiling.

**Active recall.** Close the explanation above and answer without peeking: How does the high-load goal differ from the HPC goal? What are data race and deadlock?

**Mini practice — check yourself.** If 80% of the work is perfectly parallelizable on four cores and 20% remains serial, estimate the speedup using Amdahl's law.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** S = 1 / (0.2 + 0.8/4) = 1/0.4 = 2.5, not 4.

</details>

**Exam focus:**

- Highload is not HPC: many independent requests against one large task; RPS/latency vs. FLOPS metrics.
- In Amdahl's law, `s` is the fraction of serial code; the speedup limit is `1/s`.
- GPU is data level parallelism (one command over many data), not MIMD.
- Threads share a heap, but each has its own stack - a classic trap question.
- Deadlock requires all four Coffman conditions at once.

---

<a id="q25"></a>

## 25. Service-oriented architecture (SOA): request–response and publish–subscribe

> **Learning outcome.** After this chapter, you will be able to explain SOA principles and choose request–response or publish–subscribe based on connectivity and response time requirements. First, try to explain the topic in your own words, and then test yourself with the block at the end.


The idea of ​​SOA is simple: instead of one giant monolith, we build the program as a team of independent "specialists" services. One can accept payment, the second can calculate delivery, the third can search for goods. Everyone communicates with others over the network according to clear "rules of the game" - a contract. Formally: service-oriented architecture is a style of building distributed systems, where functionality is provided in the form of loosely coupled services — autonomous components with a well-defined interface, accessible via a network using standard protocols.

Every service has three things: a contract (a formal description—what operations it can do, what it accepts, and what it returns; for example, WSDL for SOAP or OpenAPI for REST), an endpoint (a network address to knock on), and a hidden implementation—the consumer doesn't care what's inside.

Key principles explained in human terms:

- **Weak coupling** — the client depends only on the contract: the service can be rewritten from Java to Python, and no one will notice.
- **Autonomy** — the service is the owner of its own logic and data.
- **Reusable** — one service serves many consumers.
- **Composability** — services can be combined into business processes. In orchestration, one coordinator manages the interactions; in choreography, each participant reacts according to an agreed protocol without a central coordinator.
- **Stateless** — it is desirable that the service does not remember the session between calls: such services are easy to clone and scale.
- **Discovery** — services are registered in the "directory" where they can be found.

Hence the classic "SOA triangle" that is often asked to be drawn: the service provider publishes its contract in the registry; the consumer finds the service in the register; then the consumer contacts the supplier directly and calls him. Three verbs: publish — find — bind/invoke.

### Request-Response Template

In request–response, a client sends a request and expects one correlated response. The interaction is logically one-to-one and may be implemented synchronously, with the caller waiting, or asynchronously, with a future, callback, or reply queue. Typical implementations include RPC, SOAP over HTTP, REST-style HTTP APIs, and gRPC.

A step-by-step example: the mobile application asks for the weather - "give the weather for Kyiv", the server calculates and answers "21 degrees". If the server is down, the client immediately receives an error and has to repeat himself.

Synchronous request–response is simple and predictable, but both parties must normally be available at the same time, and long dependency chains can cause cascading failures. An asynchronous variant can attach a correlation identifier and reply address or queue while preserving the one-request/one-response relationship.

### Publish-subscribe template

It's like a newsletter: the author publishes, all subscribers receive, and the author does not even know who they are. Between the participants there is an intermediary - a message broker (a separate server that receives, stores and distributes messages). Publishers send messages to topics (named channels), subscribers receive the topics to which they are subscribed. Communication is one to many.

Publish–subscribe can provide spatial decoupling because publishers need not know subscriber addresses, synchronization decoupling because publishers need not wait for subscribers, and — when durable storage and subscriptions are configured — temporal decoupling because participants need not be online simultaneously.

Common delivery semantics are **at most once**, where loss is possible; **at least once**, where unacknowledged messages are redelivered and duplicates are possible; and **exactly once** within a defined system boundary, which requires additional coordination. At-least-once consumers should be idempotent. Well-known messaging technologies include Apache Kafka, RabbitMQ, and MQTT, a protocol widely used for the Internet of Things.

Step-by-step example (online store): the order service publishes the event "order #42 created" to an orders topic. Payment, warehouse, and email subscribers can react independently. If the broker retains the event and the email service uses a durable subscription or stored offset, it can consume the missed event after restarting. With direct request–response calls, the order service would instead have to invoke each service and handle its failures.

A quick comparison:

| Criterion | Request–response | Publish–subscribe |
|---|---|---|
| Interaction | One request, one correlated response | One publication, zero or more subscribers |
| Typical execution | Synchronous or asynchronous | Usually asynchronous |
| Temporal coupling | Usually present for synchronous calls | Can be reduced with durable messaging |
| Typical transport | HTTP/RPC or messaging | Message broker/topic |

Related concepts: ESB is the central "bus" of integration in classical corporate SOA; microservices are the modern evolution of SOA without a heavy bus, based on the principle of "smart endpoints, simple channels".


### Reinforcement

**Additional worked example.** Checking stock before payment naturally requires request–response because the order needs a specific answer. The `OrderPaid` event suits publish–subscribe: warehouse, email, and analytics consumers react independently.

**Transfer example.** If the event consumer is temporarily unavailable, the broker can save the message and deliver it later. This weakens the time bound, but requires idempotency due to possible redelivery.

**Active recall.** Close the explanation above and answer without peeking: What forms of coupling does pub/sub reduce? Why is the service contract more important than its internal implementation?

**Mini practice — check yourself.** The new recommendation service should respond to purchases without changing the payment service. Which template to choose?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Publish–subscribe: the payment service publishes a purchase event, and the recommendation service becomes another subscriber without introducing a direct dependency on the publisher.

</details>

**Exam focus:**

- SOA is a paradigm, a style, not a specific technology; SOAP is just one implementation.
- SOA triangle: publish — find — bind. Often asked to draw.
- Publish–subscribe does not guarantee immediate delivery. Delivery timing, durability, ordering, and retry behavior depend on the broker and subscription configuration; eventual consistency is a separate architectural choice.
- Trap: "is the request-response asynchronous?" — yes, through the ID number and the response queue, but the connection is still one-to-one.
- "At least once" allows duplicates, so the handler must be idempotent (repetition doesn't hurt).

---

<a id="q26"></a>

## 26. Software agents and multi-agent systems. Distributed applications with SOAP and REST

> **Learning outcome.** After this chapter, you will be able to characterize a software agent and compare SOAP and REST by contract, message format, state, and typical application. First, try to explain the topic in your own words, and then test yourself with the block at the end.


### Software agents

A software agent is a program that perceives its environment, acts autonomously to pursue goals, and affects the environment through actuators or action interfaces. A robot vacuum is a useful analogy: its sensors detect dirt, and its controller decides when and where to clean.

Classical properties of an agent (according to Wooldridge and Jennings):

- **Autonomy** — acts without direct human intervention and controls its own internal state and actions.
- **Reactivity** — reacts to changes in the environment in a timely manner.
- **Proactiveness** — not only reacts, but also takes initiative for the sake of the goal.
- **Sociality** — communicates with other agents in special communication languages ​​(KQML, FIPA-ACL), where each message is classified by intent: "report a fact", "ask to do", "suggest".

How does an agent differ from an ordinary object in programming? Unlike a passive object, whose method runs when invoked, an agent has its own control flow and may decide whether and how to respond to a request.

Agent architectures are commonly classified as reactive, deliberative, or hybrid. A reactive agent follows stimulus-response rules without an explicit world model: it is fast but short-sighted. A deliberative agent reasons about an explicit model; the best-known example is BDI, in which beliefs represent what the agent knows, desires represent goals, and intentions represent plans to which it has committed. A hybrid architecture combines fast reactive behavior with higher-level planning.

An example of BDI on a smart home thermostat: belief — "it's 17 degrees in the room, frost outside the window"; desire - "to keep 21 degrees and save energy"; the intention is to "turn on the heating for half an hour, then remeasure."

### Multi-agent systems

A multi-agent system is a team of agents that coordinate to solve a problem, like a team on a construction site. Agents may coordinate through negotiation and auctions: in an English auction the price rises, in a Dutch auction it falls, and in a sealed-bid second-price (Vickrey) auction the highest bidder wins but pays the second-highest bid. In the Contract Net Protocol, a manager announces a task, potential contractors submit bids, the manager awards the task, and the selected contractor performs it and reports the result.

Example: the logistics manager announces "deliver the cargo to Lviv." Truck A offers "8 hours, 5000 UAH" and truck B offers "10 hours, 4000 UAH." If the manager minimizes delivery time, it awards the contract to A; truck A then performs the delivery and reports completion.

In multi-agent service-oriented systems, agents act as consumers and providers of services: the agent finds the required service by itself, and thanks to semantic descriptions (OWL-S), it can "understand" what the service does, without human intervention, and make a chain of several services.

### Web services: SOAP and REST

A web service is a system for program-to-program communication over a network, without a human in the middle. The two main "languages" of such communication are SOAP and REST. In short: SOAP is a formal bureaucrat with a bunch of stamps, REST is a simple and lightweight style on top of plain HTTP.

**SOAP** is an XML message exchange protocol. Each message is an envelope containing an optional Header (metadata such as security and addressing information) and a Body (the message payload); errors are represented by a Fault element. SOAP service contracts are commonly described with WSDL, an XML document that specifies operations and data types, although SOAP itself does not require WSDL. A family of WS-* standards adds features such as message-level security, reliable messaging, and transaction coordination. SOAP messages may be transported over HTTP, SMTP, or messaging middleware. This flexibility and the WS-* ecosystem make SOAP common in enterprise B2B systems, although its XML messages and supporting standards can be verbose and complex.

**REST** is not a protocol, but an architectural style (described by Roy Fielding in a 2000 thesis) with a set of limitations: client-server; the server does not remember the session (each request is self-sufficient); responses can be cached; uniform rules for all resources. The main idea: everything is a resource (user, order, book), each resource has an address (URI, for example `/users/42`), and we work with it using the usual HTTP methods:

| Method | Action | Feature |
|---|---|---|
| GET | retrieve a representation | safe and idempotent |
| PUT | create or replace the resource at the target URI | idempotent |
| POST | submit data for processing, often creating a subordinate resource | not idempotent by default |
| DELETE | delete | idempotent but not safe |

Idempotence means that repeating the same request has the same intended effect on server state as making it once. Retrying PUT is therefore safe from repeated state changes; retrying POST may repeat the operation unless the API adds an idempotency mechanism.

An example of a REST script with the "book" resource: we create a book via POST - the server responds "created" and gives the address `/books/7`; we read it via GET; update the name via PUT (even if you repeat it three times, the status is the same); delete via DELETE — and the next GET honestly says "not found" (code 404).

Comparison of two approaches:

| Criterion | SOAP | REST |
|---|---|---|
| Nature | protocol | architectural style |
| Format | XML only | usually JSON, but any |
| Contract | WSDL (commonly used; not required by SOAP itself) | OpenAPI (optional) |
| Security | WS-Security in the message itself | HTTPS, OAuth 2.0, JWT |
| Typical spheres | banking, B2B, transactions | web and mobile APIs, microservices |


### Reinforcement

**Additional worked example.** The delivery agent accepts new orders, plans the route himself, reacts to traffic jams and coordinates the transfer with the warehouse agents. It illustrates autonomy, proactivity, reactivity and sociability.

**Transfer example.** The REST request `GET /orders/42` reads a resource and should be safe and idempotent. A SOAP message represents an operation in an XML envelope and often relies on a strict WSDL contract.

**Active recall.** Close the explanation above and answer without peeking: What four properties does an intelligent agent have? Why doesn't REST just mean "JSON over HTTP"?

**Mini practice — check yourself.** The client retried `PUT /profile/7` with the same body after a timeout. What property do we expect?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Idempotence: One or more identical PUTs must leave the resource in the same state, even though the log or metrics may record multiple requests.

</details>

**Exam focus:**

- Agent is not an object: the key difference is autonomy, an agent can refuse to fulfill a request.
- BDI is deciphered as beliefs (knowledge about the world), desires (goals), intentions (chosen plans) - often asked to decipher.
- "REST is a protocol?" — a classic trap. No, it's an architectural style; the protocol is SOAP.
- GET, PUT, DELETE are idempotent, POST is not.
- Stateless in REST means that the server does not keep the state of the SESSION; the data in the database is, of course, saved.
- Contract Net: announcements — bids — contract award — execution and report. Exactly in this order.

---

<a id="q27"></a>

## 27. Distributed computing infrastructure and cloud systems (IaaS/PaaS/SaaS). Cloud deployment

> **Learning outcome.** After this chapter, you will be able to differentiate between IaaS, PaaS, and SaaS and plan cloud deployments with elasticity, observability, and fault tolerance. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Imagine that you need electricity. You can buy your own generator, take care of it, fill it up — or you can just plug it into the outlet and pay for what you use. The cloud is the same idea, only with computers: instead of buying servers, you "rent" someone else's over the Internet and pay only for what you actually use.

Distributed computing uses multiple networked computers to perform coordinated work. A cluster is a group of tightly coupled computers managed as one system; its nodes need not be identical or located in one room. A grid coordinates more loosely coupled resources, often across organizations; BOINC is a classic example that uses volunteers' computers for scientific workloads. A cloud exposes pooled computing resources as on-demand network services; examples include AWS, Google Cloud, and Azure.

The widely used NIST definition identifies five essential characteristics of cloud computing:

1. **On-demand self-service** — consumers provision resources without requiring human interaction with the provider.
2. **Broad network access** — capabilities are available over a network through standard mechanisms.
3. **Resource pooling** — the provider pools physical and virtual resources for multiple consumers, while tenant workloads remain logically isolated.
4. **Rapid elasticity** — resources can scale out and in quickly as demand changes.
5. **Measured service** — usage is metered, monitored, and reported, enabling pay-as-you-go charging.

Now the main thing is three service models. The best analogy is with pizza. You cook everything yourself at home — it's your own server. **IaaS** (infrastructure as a service) — you were given a kitchen with an oven, but you knead and bake the dough yourself: the provider supplies virtual machines, disks, and networks, while you manage the operating system and everything above it. Examples include AWS EC2 and DigitalOcean. **PaaS** (platform as a service) — ready-made pizza was brought to you, and you only set the table: the provider supplies the runtime and managed services, while you deploy your application code. Examples include Heroku and Google App Engine. **SaaS** (software as a service) — you simply visit a pizzeria: you use a complete application, such as Gmail or Microsoft 365, without managing its platform or infrastructure.

A rule to remember: the "higher" model (IaaS → PaaS → SaaS), the more layers the provider takes on, and you are left with less and less trouble — but also less control. There are also newer options: serverless/FaaS (AWS Lambda) — you write only individual functions that are triggered in response to events, and you don't think about servers at all.

The technical foundation of IaaS is virtualization: a special program, a hypervisor, "cuts" one physical server into several virtual machines, each with its own operating system.

Apart from service models, there are deployment models — "who is this cloud for?": public (offered to external customers, as with AWS), private (dedicated to one organization), hybrid (a combination of public and private clouds), and community cloud (shared by organizations with common requirements).

How are applications deployed in the cloud? A virtual machine includes a guest operating system and its own kernel. A container packages an application and its user-space dependencies but shares the host kernel, so it is usually smaller and starts faster. Docker containers are therefore not virtual machines. Kubernetes orchestrates large numbers of containers: you declare the desired state in configuration files ("run three replicas of my service"), and Kubernetes works to maintain it, replacing failed replicas and scaling workloads when configured to do so.

Practices grew around this: "infrastructure as code" (servers are described by text files and created automatically - Terraform), CI/CD (automatic pipeline "build → test → roll out"). The new version is rolled out carefully: rolling update (replacing instances one at a time), blue-green (two full environments, traffic switches instantly - it's easy to roll back) or canary ("canary": at first only 5% of users see the new version, and if everything is fine, the share is increased).

The CAP theorem says that when a network partition occurs, a distributed system cannot guarantee both consistency and availability for every affected request. Because communication can fail, the practical choice during a partition is whether to reject or delay some requests to preserve consistency, or to serve them while accepting possible inconsistency.

Here is a practical example using the "nines" of reliability. A promise of 99.9% availability allows 8760 · 0.001 = 8.76 hours of downtime per year. Each additional nine reduces the allowed downtime tenfold: 99.99% permits about 53 minutes per year.


### Reinforcement

**Additional worked example.** A virtual machine with a self-managed OS is IaaS; a platform that runs uploaded code without OS management is PaaS; hosted email in a browser is SaaS. At each level, the provider assumes more responsibility.

**Transfer example.** CPU-based autoscaling can add replicas to many applications, but it is simplest and safest when replicas do not depend on critical local state. External session storage and object storage for files make replicas interchangeable.

**Active recall.** Close the explanation above and answer without peeking: Who controls the OS in IaaS, PaaS and SaaS? How is elasticity different from a simple large server?

**Mini practice — check yourself.** The API has uneven traffic and can be run multiple times. Name at least three components of a reliable deployment.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** A load balancer, health checks, multiple replicas in separate zones, autoscaling, centralized logs and metrics, and externalized state storage are suitable components. Any three with an explanation of their role will suffice.

</details>

**Exam focus:**

- Do not confuse service models (IaaS/PaaS/SaaS — "what they provide") with deployment models (public/private/hybrid — "for whom").
- Heroku is PaaS, not IaaS (you don't manage the operating system). EC2 is IaaS, Gmail is SaaS.
- Being able to name all five characteristics of the cloud according to NIST is a classic question.
- CAP applies during a network partition: the system must sacrifice either consistency or availability for affected requests.
- The container shares the OS kernel with the host, the virtual machine has its own OS.

---

<a id="q28"></a>

## 28. Mathematical modeling: principles, parameter estimation, adequacy, validation, and verification

> **Learning outcome.** After this chapter, you will be able to build a model from goal and assumptions to parameters and distinguish between calibration, verification, validation, and sensitivity analysis. First, try to explain the topic in your own words, and then test yourself with the block at the end.

A mathematical model is a "simplified copy" of a real object written by formulas. Like a city map: it is not the city itself, it does not have every tree, but it is convenient to plan a route. The whole art of modeling is to discard the unimportant, keep the essential, and then honestly check whether the model is lying.

**Principles of construction.** First, purposefulness: a model is built for a specific purpose. The same river requires different models for a hydropower engineer and an ecologist. Second, simplicity (Occam's razor): the model should be only as complex as necessary, because extra parameters may merely fit noise in the data. Third, hierarchy: start with a coarse model and refine it gradually. Fourth, balance laws underpin many models: accumulation = inflow − outflow + internal production. Dimensional analysis is another basic check: quantities with incompatible units, such as metres and seconds, cannot be added.

The modeling process itself is a cycle: formulated the problem in words → came up with a concept → wrote down the math → solved (analytically or numerically) → compared with reality → clarified → and circled again.

**Kinematic analogies.** Processes that differ physically can be described by equations of the same form. The equation dx/dt = −kx describes exponential decay: examples include radioactive decay, capacitor discharge, population decline, and Newtonian cooling when x denotes the temperature difference from the surroundings. Mechanical spring-mass oscillations and electrical RLC oscillations provide another analogy. In the standard force-voltage analogy, mass corresponds to inductance, damping to resistance, and spring stiffness to the inverse of capacitance. Engineers can therefore study one kind of system using an analogue of another.

**Compartmental analysis.** A system is represented as a set of well-mixed "reservoirs" or compartments, with material or individuals flowing between them. A balance is written for each compartment: how much enters, leaves, or is produced within it. This approach is used in pharmacokinetics, where blood and tissues can be modeled as compartments, and in epidemiology. The SIR model divides a population into susceptible, infectious, and recovered compartments. A key quantity is the basic reproduction number:

R₀ = β/γ

In the normalized deterministic SIR model, R₀ = β/γ is the expected number of secondary cases caused by one infectious individual in a wholly susceptible population. R₀ > 1 implies initial epidemic growth, whereas R₀ < 1 implies decline. With β = 0.6 and γ = 0.2, R₀ = 3. Under homogeneous mixing and a perfectly effective vaccine, the critical vaccination fraction is 1 − 1/R₀ = 2/3.

**Identification of parameters.** The model is written, but there are unknown coefficients in it - they must be extracted from the data. There is a distinction between structural identification (selection of the type of model: linear or exponential) and parametric (selection of numbers in the already selected model). The main working tool is the method of least squares: we choose such parameters that the sum of squares of deviations of the model from the data is the smallest. A simple example: points (1,2), (2,3), (3,5) and the model "y equals a x". The best a is calculated as the sum of products of x by y divided by the sum of the squares of x: (2+6+15)/(1+4+9) = 23/14 ≈ 1.64. There are other approaches: the maximum likelihood method (we take the parameters for which the observed data are the most likely), Bayesian estimation (we add prior knowledge about the parameters), the Kalman filter (updates the estimate with each new measurement - this is how GPS navigation works).

An important concept is identifiability: whether parameters can be uniquely inferred from the available observations. If an output is only weakly sensitive to a parameter, that parameter may be estimated with large uncertainty. Exact non-identifiability means that no amount of the same kind of data can determine it uniquely.

**Soft modeling** (the term of the mathematician Volodymyr Arnold) is an approach for cases when the exact form of the formulas is unknown. Instead of specific formulas, they take a whole class of functions with qualitative properties ("this dependence increases", "this one decreases") and look for conclusions that survive for the entire class. The logic is simple: if the conclusion of a "hard" model collapses from the slightest change in the formula, it is unreliable and should not be trusted.

**Model Check.** Here lives the most insidious pair of exam terms:

- **Verification** — "are we solving the equation correctly?" It's about the code and the numerical method: does the program implement the math without errors.
- **Validation** — "are we solving the correct equations?" This is about reality: comparing model behavior with observations, ideally using data that were not used for calibration.

Mnemonic: verification is about code, validation is about reality. Adequacy is assessed statistically by inspecting residuals ("data minus model," ideally without systematic structure), calculating measures such as R² and average prediction errors, and checking assumptions. Cross-validation trains the model on one subset of the data and evaluates it on another to detect overfitting, where the model fits the training data but performs poorly on new observations.


### Reinforcement

**Additional worked example.** For coffee cooling, the `dT/dt = -k(T - T_room)` model ignores evaporation and temperature heterogeneity. The parameter k is estimated from measurements, and then the prediction is tested on new data.

**Transfer example.** A queue model may implement its equations correctly (verification) yet predict a real store poorly because its assumptions about customer arrivals are wrong (failed validation).

**Active recall.** Close the explanation above and answer without peeking: How is structure identification different from parameter estimation? Which question does verification ask, and which question does validation ask?

**Mini practice — check yourself.** The model reproduces the training measurements well but is wrong in the new week. What checks would be appropriate next?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Validate the model on independent data, analyze parameter sensitivity and stability, and revisit structural assumptions instead of merely refitting coefficients.

</details>

**Exam focus:**

- Verification vs. Validation is a major pitfall. "Solve it right" vs. "solve the right one."
- In the normalized SIR model, R₀ = β/γ and the threshold is one. Do not confuse R₀ with the R (recovered) compartment.
- A model is not "true" - it is useful or not within the limits of its purpose.
- Calibrating and validating on the same data is a methodological error (overfitting).

---

<a id="q29"></a>

## 29. Simulation models: discrete events, Monte Carlo, Petri nets, system dynamics, and multi-agent simulation

> **Learning outcome.** After this chapter, you will be able to choose discrete-event, Monte Carlo, Petri nets, system dynamics, or agent modeling according to the nature of the system. First, try to explain the topic in your own words, and then test yourself with the block at the end.


Simulation modeling means reproducing a system's behavior on a computer, much as a flight simulator reproduces a real flight. Simulation is especially useful for complex, stochastic, or interacting systems, even when some analytical formulas exist. It is used to study bank queues, epidemics, and production lines. This chapter contrasts five important approaches: discrete-event simulation, Monte Carlo methods, Petri nets, system dynamics, and agent-based modeling.

**Discrete-event simulation (DES).** The key idea is that the system state changes only at event times ("customer arrives," "service completes"). The model maintains a calendar of future events. Its main loop removes the next event from the calendar, advances the simulation clock to that event's timestamp, processes the event, schedules any resulting events, and repeats.

Consider one checkout where customers arrive at minutes 0, 4, and 5, and each service takes 3 minutes. The first customer begins immediately and finishes at minute 3. The second arrives at minute 4, begins immediately, and finishes at minute 7. The third arrives at minute 5, waits until minute 7, and finishes at minute 10, so that customer's waiting time is 2 minutes. The simulation clock jumps 0 → 3 → 4 → 5 → 7 → 10 instead of checking every intervening instant. A fixed-time-step simulation checks the state at regular intervals; it may be simpler, but its efficiency and accuracy depend on the chosen step size.

**The Monte Carlo method** estimates a quantity by drawing many random samples and averaging the resulting observations or estimates. A classic example estimates π by generating random points in a unit square and counting how many fall inside the inscribed quarter-circle. Its area is π/4, so π ≈ 4 · (the proportion of hits). If 782 out of 1000 points hit, π ≈ 3.128. The standard sampling error decreases as

O(1/√N)

where N is the number of trials. In other words, reducing the standard error by a factor of ten usually requires one hundred times as many samples. The canonical O(1/√N) rate does not explicitly worsen with dimension, which can make Monte Carlo attractive for high-dimensional integrals where grid methods become prohibitively expensive. However, the error constant and variance can still depend strongly on the dimension and the integrand. The method requires pseudorandom number generators and techniques for sampling from the required distributions.

**Petri nets** are a graphical language for concurrent processes. A Petri net is a bipartite graph with places (circles) and transitions (bars or rectangles); arcs connect places to transitions or transitions to places. Tokens occupy places. A transition is enabled when every input place contains the tokens required by its incoming arc weights. When the transition fires, it consumes those input tokens and produces tokens in its output places.

Example: place two tokens in "raw material" and one token in "free machine." A "process part" transition consumes one token from each input place and produces one token in "finished part" and one back in "free machine." It can fire twice before the raw material is exhausted. If two enabled transitions compete for one token, the choice is nondeterministic. Important Petri-net properties include reachability; boundedness, which asks whether token counts remain bounded; deadlock freedom; and liveness, which asks whether every transition can eventually fire again from every reachable marking.

**System dynamics** (associated with Jay Forrester) takes a top-down view without modeling individuals separately. Its central concepts are stocks, such as population, capital, or inventory; flows, such as births, spending, or shipments; and feedback loops. Reinforcing feedback amplifies change (more population → more births → still more population), while balancing feedback counteracts change. Such models are commonly expressed as systems of ordinary differential equations and integrated numerically.

**Agent-based modeling (ABM)** takes a bottom-up view: many autonomous agents follow local rules, and complex system-level behavior can emerge from their interactions. In Schelling's segregation model, even a mild preference for similar neighbors can produce substantial neighborhood segregation without any agent explicitly seeking city-wide separation. Another classic is Reynolds's boids model, in which three local steering rules produce realistic flocking.

A complete computer-simulation cycle is: problem formulation → conceptual model → implementation → verification (was the model implemented correctly?) → validation (does it represent the real system adequately for its purpose?) → experiments and analysis. One stochastic run is not enough: use independent replications and report uncertainty, for example with confidence intervals. For steady-state simulations, an initial warm-up period may be discarded to reduce initialization bias.


### Reinforcement

**Additional worked example.** In the bank model, the events "customer arrived" and "service completed" change the state of the queue. Jumping from event to event is more efficient than checking every millisecond without change.

**Transfer example.** To estimate the area of ​​a circle, we generate points in the square [-1,1]². The fraction of points inside x² + y² ≤ 1 approaches π/4; 78,500 out of 100,000 give π ≈ 4 · 0.785 = 3.14.

**Active recall.** Close the explanation above and answer without peeking: When is a discrete-event model better than a fixed time step? What should be reported with the Monte Carlo result?

**Mini practice — check yourself.** How will the standard Monte Carlo random error decrease if the number of trials is increased by a factor of 4?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** About 2 times, because the standard error decays as 1/√N.

</details>

**Exam focus:**

- Model time is not equal to real time: in the event algorithm, the clock jumps between events.
- Monte Carlo: reducing standard error tenfold usually requires one hundred times as many samples; the O(1/√N) rate does not explicitly contain dimension, but variance can still depend on it.
- A Petri net is bipartite: arcs connect places and transitions, and firing consumes input tokens and produces output tokens.
- Three broad simulation paradigms often contrasted are DES — queues and events, system dynamics — stocks and flows, and ABM — individual agents and emergence.
- One stochastic run is not a result; replications are required.

---

<a id="q30"></a>

## 30. Systems analysis, complex systems, decomposition, aggregation, and perturbation methods

> **Learning outcome.** After this chapter, you will be able to decompose a complex system, identify boundaries and relationships, and explain when the separation of time scales warrants simplification. First, try to explain the topic in your own words, and then test yourself with the block at the end.

Systems analysis examines a complex object, such as a city, factory, or organism, as a whole made of interacting parts. The whole can exhibit behavior that no component has by itself: no single neuron thinks, yet a brain can. The appearance of such system-level properties is called **emergence**.

**Basic concepts:** system, element (a part treated as indivisible at the chosen level of analysis), subsystem, relationship, structure, environment (everything external that interacts with the system), input and output, state, goal, and feedback (when information about an output influences subsequent inputs or behavior).

**Principles:** integrity (the system is not equal to the sum of the parts); hierarchy (every system is a subsystem of something bigger: cell → organ → organism); structurality (behavior is determined not only by elements, but also by connections between them); purposefulness (the analysis is based on the goal); multiplicity of descriptions (one system is described by several models at once — what it does, what it consists of, how information flows).

**Methodology — stages:** formulate the problem and build a tree of goals (break the main goal into sub-goals); define the system boundary; identify subsystems and connections; build models; generate and evaluate alternatives using methods such as anonymous rounds of Delphi surveys or the Analytic Hierarchy Process with Saaty pairwise comparisons; make a decision; and monitor its implementation.

**Theory of complex systems.** Common features of a complex system include many elements and connections, hierarchy, nonlinearity, randomness, feedback, adaptability, and emergence. Complexity can be structural (many interacting components), dynamic (intricate behavior over time, including chaos and bifurcations), or computational. Related areas include Wiener's cybernetics, Bertalanffy's general systems theory, synergetics and self-organization, and complex-network theory. Small-world networks have short paths between most pairs of nodes, while scale-free networks contain hubs with exceptionally many connections.

**Description of the structure.** A structure is stable connections between elements, and the natural language for it is a graph: elements become vertices, connections become edges. Typical structures: linear (chain), tree-like (boss → subordinates), star (center and rays), ring, matrix (double subordination), fully connected network. Example: a company where the director is directly connected to four departments is a star. The path between any two departments goes through the director, that is, the longest of the shortest paths (diameter) is equal to two. The director is the most central point, and this is the vulnerability of the star: remove him and the system crumbles. Add direct connections between departments — stability will increase, but coordination will become more difficult. This is a typical system compromise.

**Decomposition and aggregation** are complementary techniques in an analysis-synthesis cycle. Decomposition divides a complex system into simpler blocks from the top down. A useful decomposition should be complete enough not to omit essential behavior and should create blocks with strong internal cohesion and relatively weak coupling to one another. Aggregation moves in the opposite direction, combining detailed elements into larger subsystems or summary indicators, such as GDP instead of millions of individual transactions. For example, the goal "increase store profit" can be decomposed into "increase traffic," "increase conversion," and "reduce costs." The branches must later be reconciled because one action, such as advertising, can increase both traffic and costs.

**Method of a small parameter (regular perturbation).** Suppose a problem differs only slightly from a simpler one, with the difference controlled by a small parameter ε, such as 0.01. The solution can often be sought as a series:

x = x₀ + ε·x₁ + ε²·x₂ + …

First solve the unperturbed problem at ε = 0 to obtain x₀, then calculate successive corrections x₁, x₂, and so on. For example, consider x² + εx − 1 = 0 and its positive root. At ε = 0, the root is 1. The first-order approximation is x ≈ 1 − ε/2. For ε = 0.1 this gives 0.95, while the exact positive root is about 0.95125, an error of about 0.00125.

**Method of singular perturbations.** In an important class of singularly perturbed problems, a small parameter multiplies the highest-order derivative, for example ε·dy/dt = g(x, y). Setting ε to zero then changes a differential equation into an algebraic equation, reducing the system's order so that all original conditions can no longer be imposed on the reduced problem. The dynamics separate into fast and slow parts, and a thin initial or spatial boundary layer may appear where the fast variable changes rapidly.

A thermometer in a slowly heated room gives useful intuition. The thermometer is a fast subsystem: within seconds its reading catches up with the temperature, while the room warms over hours. The initial boundary layer is the short interval after inserting the thermometer, when its needle moves rapidly. Afterward, the reading can be approximated by the room temperature. Roughly, Tikhonov's theorem states that, under suitable regularity conditions and stability of the fast subsystem, the fast variable approaches its quasi-steady value rapidly relative to the slow time scale; outside the boundary layer, one can analyze the reduced slow system.

For systems analysis, singular perturbation provides a mathematical basis for aggregation: fast processes can be approximated by their quasi-steady values, reducing the problem's dimension by separating its time scales.


### Reinforcement

**Additional worked example.** An online store can be decomposed into catalog, order, payment and delivery, but optimizing each separately does not guarantee the optimality of the whole system: fast payment will not help if the warehouse is a bottleneck.

**Transfer example.** In a drone stabilization model, electrical processes can be much faster than mechanical motion. The singular perturbation method analyzes the fast and slow subsystems separately and then reconciles them.

**Active recall.** Close the explanation above and answer without peeking: How does decomposition differ from aggregation? Why is the property of the whole system not always the sum of the properties of the parts?

**Mini practice — check yourself.** After local improvement of the two subsystems, the overall time became worse. Name the system explanation.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Interactions or a bottleneck changed: the local optimizations ignored queues, synchronization, or a shared resource. Evaluate the system-wide objective and the coupling between subsystems.

</details>

**Exam focus:**

- In a regular perturbation, the solution varies smoothly as ε → 0 and can often be expanded in powers of ε. In a singular perturbation, setting ε = 0 changes the order or type of the problem, so boundary layers or multiple time scales can appear.
- Tikhonov's theorem requires stability of the fast subsystem — without it, there will be no "sticking".
- Emergence — to be able to give an example: brain and neuron, traffic jam and individual cars.
- Decomposition without weak inter-block connections is useless: if everything is connected to everything, partitioning does not simplify anything.
- Decomposition — from top to bottom (breakdown), aggregation — from bottom to top (consolidation); these are inverse operations of one cycle.

---

<a id="q31"></a>

## 31. System optimization: linear and nonlinear programming, optimality, constraints, and search methods

> **Learning outcome.** After this chapter, you will be able to formulate an objective and constraints, verify basic optimality conditions, and compare Lagrangian, KKT, penalties, and random search. First, try to explain the topic in your own words, and then test yourself with the block at the end.


### The essence of the question
Optimization is the search for the best option: the cheapest production plan, the shortest route, or the highest profit. Imagine looking for the lowest point in a mountainous area in thick fog: you can see only the local slope and take steps downhill. Different optimization methods are different strategies for this search, while constraints are fences that cannot be crossed. The word "programming" here means planning, not writing code; the term predates the era of mass computers.

### Key concepts
There are three things in any optimization problem. First, the **objective function** is what we want to minimize (cost) or maximize (profit). Second, the **decision variables** are the quantities we can choose. Third, the **feasible set** contains all choices that satisfy the constraints, such as a budget, capacity, or non-negativity requirement.

A **local minimum** is no worse than all sufficiently nearby points; it may not be the best point overall. A **global minimum** is no worse than every feasible point. For a convex objective over a convex feasible set, every local minimum is global, which is why convex optimization has especially strong guarantees.

### Linear programming and the simplex method
**Linear programming (LP)** is the case where the objective and constraints are linear: there are no squares or products of decision variables. The feasible set is a convex polyhedron. If a linear objective has a finite optimum over a nonempty polytope, at least one optimum occurs at a **vertex** (extreme point). Enumerating every vertex is usually impractical, however, so LP algorithms exploit the geometry more intelligently.

The **simplex method**, developed by George Dantzig in the 1940s, moves between vertices (basic feasible solutions) of the feasible polyhedron. A pivot normally moves to an adjacent vertex with a better objective value; a degenerate pivot can leave the objective unchanged, and a suitable anti-cycling rule prevents cycling. The method stops when the reduced costs satisfy the optimality condition; their required sign depends on the tableau convention. If an improving entering variable has no eligible leaving variable, the LP is unbounded in that direction.

A small example: maximize the profit of 3 hryvnias per unit of the first product plus 2 per unit of the second, subject to a total of at most 4 units and at most 3 units of the first product. The vertices of the feasible region and their profits are:

| Vertex (first, second) | Profit |
|---|---|
| (0, 0) | 0 |
| (3, 0) | 9 |
| (3, 1) | **11** |
| (0, 4) | 8 |

The optimum is 11 at the vertex (3, 1). The simplex method might follow (0,0) → (3,0) → (3,1), improving the objective at each pivot. Klee–Minty examples show exponential worst-case behavior for some pivot rules, although simplex is often fast in practice. Polynomial-time LP algorithms include ellipsoid and interior-point methods.

### Nonlinear problems: how to recognize the optimum
For a differentiable, unconstrained problem, an interior local minimum must have zero **gradient** (the vector of first partial derivatives). This is only a necessary condition: maxima and saddle points can also have zero gradient. A positive-definite Hessian at a stationary point is a sufficient condition for a strict local minimum; a negative-definite Hessian indicates a local maximum, while an indefinite Hessian indicates a saddle point.

This zero-gradient condition is for an unconstrained interior point. A constrained optimum can lie on a boundary, where the gradient need not be zero; feasibility and the constraint geometry must then be included in the optimality conditions.

The main working method is **gradient descent**:

```
x_new = x_old − step · gradient
```

that is, "take a step down the slope." Example: minimize f(x)=x², start at x=4, and use step size 0.25. Since f'(4)=8, the next point is 4 − 0.25·8 = 2; subsequent points are 1, 0.5, and 0.25, approaching the minimum at zero. Newton's method incorporates curvature and can converge quadratically near a well-behaved solution, but it requires second-derivative information or an approximation. Derivative-free methods include Nelder–Mead, pattern search, and the one-dimensional golden-section search.

### Optimization with constraints
The **method of Lagrange multipliers** handles differentiable equality constraints. We form a Lagrangian by adding each constraint multiplied by an unknown multiplier, then solve the stationarity and feasibility equations. Under suitable regularity conditions, a multiplier can be interpreted as the sensitivity, or "shadow price", of the optimum to a small change in the corresponding constraint. For example, minimizing x² + y² subject to x + y = 2 gives x = y = 1.

The **Karush–Kuhn–Tucker (KKT) conditions** extend this idea to inequality constraints. For a minimization problem written as gᵢ(x) ≤ 0, they comprise stationarity, primal feasibility, dual feasibility (λᵢ ≥ 0), and **complementary slackness** (λᵢgᵢ(x) = 0). Thus an inactive inequality has multiplier zero; a tight constraint may have a nonzero multiplier, but need not. Under a suitable constraint qualification the conditions are necessary for a local optimum. They are also sufficient when the objective and inequality constraints are convex (with affine equality constraints).

A **penalty method** replaces a constrained problem with a sequence of unconstrained problems whose objective adds an increasing cost for constraint violations. A **barrier method** instead adds a term that tends to infinity at the boundary and keeps iterates strictly inside the feasible region. Interior-point methods develop this barrier idea with carefully controlled central-path steps.

### Pseudogradient and random search
Sometimes the exact gradient is unavailable because measurements are noisy or the objective is produced by a simulation. A **stochastic approximation** or pseudogradient method uses a noisy direction whose expected value is aligned with the true gradient. A standard step-size condition is that step sizes decay but not too quickly: their sum diverges while the sum of their squares converges (Robbins–Monro conditions). **Stochastic gradient descent (SGD)** applies this idea to gradients estimated from data samples and is widely used to train neural networks.

**Random search** includes sampling candidate points and retaining the best, trying random directions and accepting improvements, or adapting the step length according to the success history. Such methods do not require derivatives and can tolerate noise or multiple local minima, but they may use many objective evaluations. More elaborate randomized global-search methods appear in question 32.


### Reinforcement

**Additional worked example.** We maximize f(x, y) = x + y subject to x + 2y ≤ 8 and x,y ≥ 0. A linear objective reaches its maximum at a vertex of the feasible polygon; checking (0,0), (8,0), and (0,4) shows that (8,0) is best.

**Transfer example.** To minimize x² + y² for x + y = 2, the Lagrangian L = x² + y² + λ(x+y−2). The conditions give x = y = 1: the closest point to the origin on the line.

**Active recall.** Close the explanation above and answer without peeking: What are the components of an optimization problem? When is zero gradient only a candidate and not a guarantee of a minimum?

**Mini practice — check yourself.** Find the stationary point f(x) = (x − 3)² and classify it by the second derivative.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** f'(x)=2(x−3)=0 gives x=3; f''(x)=2>0, so it is a strict minimum, f(3)=0.

</details>

**Exam focus:**

- "Programming" means planning; Dantzig's simplex method is unrelated to the Nelder–Mead simplex—they are different algorithms.
- When a finite LP optimum is attained on a polytope, some vertex is optimal; if two adjacent vertices are optimal, every point on the edge between them is also optimal.
- A zero gradient is only a necessary condition: it can be both a saddle and a maximum.
- In KKT, dual feasibility and complementary slackness are often tested: an inactive inequality has a zero multiplier.
- The simplex method is exponential in the worst case, although linear programming itself is solved in polynomial time by other methods.

---

<a id="q32"></a>

## 32. Discrete and Boolean optimization, Gomory cuts, branch-and-bound, and metaheuristics

> **Learning outcome.** After this chapter, you will be able to recognize discrete constraints, explain pruning in branch-and-bound, and describe the role of metaheuristics without optimality guarantees. First, try to explain the topic in your own words, and then test yourself with the block at the end.


### The essence of the question
In discrete optimization, decision variables take values from a discrete set: there are no 2.7 buses or 0.4 factories—only integers or yes/no choices. The search space has gaps, so ordinary gradient methods cannot move smoothly through it. Integer-programming optimization is generally **NP-hard**; its associated decision versions are often **NP-complete**. Unless P = NP, no polynomial-time algorithm solves every instance.

A natural first step is to allow fractional values and solve the **LP relaxation**. Simply rounding its solution is unsafe: the rounded point may violate a constraint or be far from the true integer optimum.

### Boolean programming
Here, each variable is a yes/no decision: select an item or not, build a facility or not. The classic example is the **0/1 knapsack problem**: choose items with weights and values to maximize total value without exceeding capacity. With capacity 5 and items A=(weight 2, value 3), B=(3,4), and C=(4,5), taking the individually most valuable item C gives 5, but A and B together give 7. Dynamic programming solves this instance class in O(nW), where W is the numeric capacity. This is **pseudo-polynomial**, because W may be exponentially large relative to the number of bits used to encode it.

Other binary or combinatorial problems include set cover and the traveling-salesperson problem. The assignment problem is a related but polynomial-time case solved, for example, by the Hungarian algorithm. Common tools include implicit enumeration, branch-and-bound, cutting planes, and reductions to Boolean satisfiability (SAT).

### Gomory cuts (cutting-plane method)
The cutting-plane idea is to solve the LP relaxation and then add valid inequalities that remove fractional solutions without removing any feasible integer solution. For a pure integer program, a **Gomory fractional cut** can be derived from a simplex-tableau row using fractional parts; mixed-integer programs use corresponding mixed-integer cuts. The cut excludes the current fractional optimum, after which the LP is reoptimized. Pure cutting-plane procedures have finite-convergence results under standard rationality and algorithmic assumptions, but modern solvers usually combine cuts with branch-and-bound in **branch-and-cut**.

### Branch-and-bound method
This is an exact implicit-enumeration method. It builds a search tree but prunes an entire subtree whenever a bound proves that the subtree cannot improve the best known feasible solution.

Three components are essential. **Branching:** if an LP relaxation gives x = 2.6, split the subproblem into x ≤ 2 and x ≥ 3; together these branches retain every integer solution. **Bounding:** for a maximization problem, the relaxation value is an upper bound on what that branch can achieve (for minimization it is a lower bound). **Incumbent:** the best feasible integer solution found so far. Fathom a node if its constraints are infeasible, its relaxation is integral (and supplies a feasible integer solution), or its bound cannot beat the incumbent.

Example for maximization: the root relaxation gives an upper bound of 10 and a fractional variable. The x≤2 branch produces an integer solution of value 8, which becomes the incumbent. The x≥3 branch has upper bound 7.5, so it is pruned because it cannot beat 8. Once every remaining node is fathomed, 8 is proved optimal. Worst-case running time is exponential, but branch-and-cut is highly effective in modern solvers such as CPLEX and Gurobi.

### Discrete optimization on graphs
Many graph-optimization problems are polynomial-time solvable: shortest paths (Dijkstra for nonnegative edge weights, Bellman–Ford when negative edges may occur, and Floyd–Warshall for all pairs), minimum spanning trees (Kruskal or Prim), maximum flow, and maximum matching. Other decision problems—including Hamiltonian cycle and the usual decision versions of traveling salesperson, graph coloring, clique, and vertex cover—are NP-complete; their optimization versions are NP-hard. Exact exponential algorithms, branch-and-bound, approximation algorithms, and metaheuristics are used for hard cases. For the symmetric metric TSP, for example, Christofides' algorithm returns a tour of length at most 1.5 times the optimum.

### Metaheuristics
A **metaheuristic** is a high-level search strategy, often randomized, that seeks good solutions without generally proving global optimality. Its central trade-off is between **exploration** of new regions and **exploitation** of promising ones.

**Simulated annealing** borrows its analogy from slowly cooling a material. It randomly perturbs the current solution: an improving move is accepted, while a worsening move with objective increase Δ>0 is accepted with temperature-dependent probability

```
P(accept a worse move) = exp(−Δ / T)
```

As long as the temperature is high, the algorithm is more willing to accept worsening moves and can escape local minima; as it cools, it becomes increasingly greedy. Numerically, with Δ = 2 and T = 10, the acceptance probability is about 0.82, whereas at T = 0.5 it is about 0.018.

**Genetic algorithms** imitate evolution. We keep a population of encoded solutions ("chromosomes", often bit strings), evaluate each with a fitness function, and over generations apply selection (better solutions are more likely to become parents), crossover (parents exchange segments), and mutation (random changes that maintain diversity). With elitism, the best solutions are copied unchanged into the next generation.

**Ant-colony optimization** models artificial ants that construct probabilistic routes. Edges are favored when they have more pheromone (search experience) or a stronger heuristic value, such as shorter distance. Better routes receive more pheromone, while evaporation prevents unlimited accumulation and encourages continued exploration. The method is often applied to routing problems such as TSP.

Other examples include **tabu search** (which records recently visited moves or states), **particle swarm optimization**, and evolutionary strategies. Because these methods are stochastic, their quality should be evaluated statistically over multiple independent runs.


### Reinforcement

**Additional worked example.** In the 0/1 knapsack problem, xᵢ ∈ {0,1}: an item is either selected or not. The linear relaxation allows fractional values and gives an upper bound; if even that bound is worse than the best known solution, the branch can be pruned.

**Transfer example.** Simulated annealing sometimes takes a worse step, especially at high temperature, to get out of a local minimum. At the end, the temperature drops and the behavior becomes almost greedy.

**Active recall.** Close the explanation above and answer without peeking: What is a bound in branch-and-bound? Why is the best result of a genetic algorithm not a proof of the global optimum?

**Mini practice — check yourself.** For binary x,y, list all feasible points satisfying x+y ≤ 1 and maximize 3x+2y.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** The feasible points are (0,0), (1,0), and (0,1), with objective values 0, 3, and 2. Therefore (1,0) is optimal with value 3.

</details>

**Exam focus:**

- Rounding an LP-relaxation solution can produce an infeasible or far-from-optimal integer point.
- Branch-and-bound is exact once the entire search is fathomed; a metaheuristic normally provides no optimality certificate.
- A valid Gomory cut preserves every feasible integer point while excluding a fractional LP solution.
- Simulated annealing accepts some worsening moves, helping it escape local minima.
- Knapsack optimization is NP-hard, yet O(nW) dynamic programming is possible: the algorithm is pseudo-polynomial because W is a numeric value, not the length of its binary encoding.

---

<a id="q33"></a>

## 33. Multi-objective optimization, Pareto-optimal solutions, minimax, and criterion aggregation

> **Learning outcome.** After this chapter, you will be able to identify Pareto dominance and explain how weights, trade-offs, and minimax criteria transform multiple objectives into a decision. First, try to explain the topic in your own words, and then test yourself with the block at the end.


### The essence of the question
In life, there is rarely one criterion: you want a laptop that is cheap, powerful, and light — and these desires pull in different directions. Multi-objective optimization studies reasonable compromises when it is impossible to improve every objective at once. Formally, several objective functions are optimized over one feasible set; an option that is best for every objective simultaneously is often unattainable.

### Dominance and Pareto set
We say that option A **dominates** option B if A is no worse than B in every objective and strictly better in at least one. A dominated option can be discarded when searching for Pareto-optimal solutions. A **Pareto-optimal** option is one that is not dominated by any feasible option: improving one objective requires worsening another (or leaving the Pareto set). The set of such solutions is the **Pareto set**, and its image in objective space is the **Pareto front**.

Example: five laptops, criteria "quality" and "economy" (both - the more, the better):

| Option | Quality | Economy | Status |
|---|---|---|---|
| A| 9 | 2 | Pareto-optimal |
| B | 7 | 5 | Pareto-optimal |
| C| 4 | 8 | Pareto-optimal |
| D | 6 | 4 | dominated by B (7 vs. 6 and 5 vs. 4) |
| E| 3 | 3 | dominated by B and C |

The Pareto front is A, B, and C. Mathematics alone does not choose between them: A has the best quality, while C has the best economy. The final choice requires a **decision-maker (DM)**. The **ideal point** is the (possibly unattainable) vector of the best value of every objective; here it is (quality 9, economy 8). The **nadir point** is the vector of worst objective values among the Pareto-optimal solutions.

### The method of the main criterion and the method of concessions
The simplest move is **the method of the main criterion**: we choose the most important indicator and maximize it, and turn the rest into restrictions ("the quality is maximum, but the economy is not lower than 4").

**The method of successive concessions** is a related procedure. We order the objectives by importance, optimize the first, and then relax its optimum by an allowed **concession**. We optimize the second subject to that bound, then continue in the same way. With zero concessions this becomes a lexicographic choice; with positive concessions it makes an explicit trade-off. In the table, the maximum quality is 9; a concession of 2 imposes quality ≥ 7, after which B is more economical than A.

### Minimax methods
Here, the philosophy is "take care of the weakest link": among the options, we maximize the worst normalized objective. A related **ideal-point method** chooses an option closest to the ideal point; with the weighted Chebyshev metric, distance is measured by the largest weighted deviation:

```
min over x [ max over criteria  wᵢ · |idealᵢ − fᵢ(x)| ]
```

That is, for each option we find the objective with the largest weighted shortfall from the ideal and minimize that shortfall. The weights and normalization must be specified before comparing distances. Unlike a plain weighted sum, an appropriately augmented or weighted Chebyshev scalarization can recover Pareto points on non-convex parts of the front.

### Scalarization of objectives
**Scalarization** replaces the vector of objective values with a single score, after which the problem becomes single-objective. The most common form is a **weighted sum**:

```
F(x) = λ₁·f₁(x) + λ₂·f₂(x) + ... , weights are nonnegative and sum to one
```

Normalization is essential before combining objectives with different units or scales; "grams are not added to hryvnias." Changing the weights can produce different Pareto points, but a plain weighted sum generally misses unsupported points on non-convex parts of the front. In the **ε-constraint method**, one objective is optimized and the others become constraints with chosen thresholds; varying those thresholds can recover supported and unsupported Pareto points. Under suitable monotonicity and feasibility assumptions, a scalarized optimum is Pareto-optimal. Evolutionary methods such as NSGA-II and SPEA2 approximate a whole front while maintaining population diversity.

### System optimization
In a system-optimization or hierarchical formulation, we optimize not only a solution within a fixed framework but also aspects of the framework itself. If the objectives are unattainable in the current feasible region, upper-level decisions can change the system structure or resource limits; the lower level then solves the resulting multi-objective problem. This is useful for planning large energy, transport, or economic systems.


### Reinforcement

**Additional worked example.** Car A: price 20, consumption 8; B: 22, 6; C: 25, 9, where both criteria are minimized. C is dominated by A, because A is both cheaper and more economical. A and B do not dominate each other and form a Pareto front.

**Transfer example.** The weighted sum of 0.7·price + 0.3·time makes sense only after normalization: hryvnias and hours cannot be honestly added on raw scales.

**Active recall.** Close the explanation above and answer without peeking: What does Pareto dominance mean? Why is the Pareto-optimal point not necessarily the only "best" one?

**Mini practice — check yourself.** To maximize two criteria, given A=(5,5), B=(6,3), C=(4,4). What points are not dominated?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** A and B are non-dominated. A dominates C because 5 ≥ 4 in both criteria and is strictly better in both; B does not dominate A, and A does not dominate B.

</details>

**Exam focus:**

- There are often many Pareto-optimal solutions: finding the Pareto set does not mean finding a unique answer; the final selection belongs to the decision-maker.
- Dominance requires "nowhere is worse and at least somewhere is strictly better"; if A is better in one criterion and B in another - they do not dominate each other, both can be in the Pareto set.
- A plain weighted sum may miss unsupported points on non-convex parts of the front; Chebyshev scalarization and the ε-constraint method can reach more of the front.
- Before a weighted sum, the objectives must be normalized when their measurement units or scales differ.
- The Pareto set lives in decision space; the Pareto front is its image in objective space. Do not confuse them.

---

<a id="q34"></a>

## 34. Decision theory, utility, preferences, AHP, uncertainty, and risk

> **Learning outcome.** After this chapter, you will be able to structure alternatives, criteria, utility, and risk, and perform simple expected value selection, or AHP. First, try to explain the topic in your own words, and then test yourself with the block at the end.


### The essence of the question
Decision theory studies how to choose among alternatives when outcomes and information vary. It distinguishes **certainty** (consequences are known), **risk** (probabilities of relevant states are known or estimated), and **uncertainty** (those probabilities are not known). **Conflict**, where strategic opponents affect the outcome, is the subject of game theory in question 35.

### Relationship of preference and utility
The **preference relation** is a rule for comparing pairs of options: "A is no worse than B." A rational weak order is complete (any two options can be compared) and transitive (if A is no worse than B and B is no worse than C, then A is no worse than C). Under suitable assumptions, such preferences can be represented by a **utility function** that preserves the ordering. For deterministic choices, any strictly increasing transformation of utility represents the same preferences; the numerical differences acquire additional meaning only in models such as expected utility.

**Expected utility theory** (von Neumann–Morgenstern) extends this to lotteries—random outcomes with specified probabilities. If preferences satisfy axioms such as completeness, transitivity, continuity, and independence, a lottery is evaluated by the weighted average of the utilities of its outcomes:

```
U(lottery) = sum over outcomes: P(outcome) × utility(outcome)
```

The shape of the utility function encodes risk attitude. If utility of money is √m, the lottery "1,000 UAH or nothing, with equal probabilities" has expected utility about 15.8, whereas a guaranteed 500 UAH has utility about 22.4. A risk-averse person therefore chooses the guarantee even though both options have expected monetary value 500 UAH. Concave utility represents risk aversion. Real people sometimes violate the axioms (the Allais and Ellsberg paradoxes); prospect theory, associated with Kahneman and Tversky, is a descriptive alternative.

### Alternative selection procedures
Dominated options can first be removed, leaving the Pareto set. Other procedures include lexicographic selection (use the next criterion only to break a tie), Simon's **satisficing** (choose the first option meeting all aspiration thresholds), and the ELECTRE and PROMETHEE outranking methods, which use concordance and discordance information in pairwise comparisons.

### Analytic Hierarchy Process (Saaty)
AHP (Saaty's Analytic Hierarchy Process) chooses among alternatives through pairwise comparisons. Build a hierarchy of goal → criteria → alternatives; fill pairwise-comparison matrices using Saaty's 1–9 scale; enforce reciprocal entries (if A is three times as important as B, then B is 1/3 as important as A); estimate priorities, commonly from the principal eigenvector (row geometric means are an approximation); check the consistency ratio; and compute each alternative's global score as the weighted sum of its local priorities. A consistency ratio around 0.10 or less is a common rule of thumb, not a mathematical law.

Example: the decision-maker says that price is three times as important as quality, giving criterion weights 0.75 and 0.25. Alternative X has local priorities 0.6 (price) and 0.3 (quality); Y has 0.4 and 0.7. Their global scores are X: 0.75·0.6 + 0.25·0.3 = 0.525, and Y: 0.75·0.4 + 0.25·0.7 = 0.475. We choose X.

### Choice under uncertainty
Given a payoff table, the payoff of each strategy is listed for each state of the environment, but the state probabilities are unknown. For example:

| | State 1 | State 2 | State 3 | the worst | the best |
|---|---|---|---|---|---|
| a1 | 2 | 6 | 10 | 2 | 10 |
| a2 | 5 | 4 | 6 | 4 | 6 |

- **Wald's maximin criterion** (extreme pessimism, "prepare for the worst"): for each strategy, take its worst payoff and choose the strategy with the largest such payoff. We choose a2 because 4 > 2.
- **Savage's minimax-regret criterion:** form a regret table by subtracting each payoff from the best payoff in its state, then minimize the maximum regret. Here, a1's maximum regret is 3 and a2's is 4, so we choose a1.
- **Hurwicz criterion:** with optimism coefficient α, score a strategy as α·(best payoff) + (1−α)·(worst payoff). For α = 0.5, a1 scores 6 and a2 scores 5, so we choose a1. α = 0 gives Wald; α = 1 gives the maximax criterion.
- **Laplace criterion:** when no probabilities are known, treat the states as equally likely and maximize the arithmetic mean. Here, a1 gives 6 and a2 gives 5, so we choose a1.

Different criteria can give different answers; the selected criterion expresses the decision-maker's attitude toward uncertainty.

### Selection under conditions of risk and statistical methods
When state probabilities are known, the **Bayes (expected-payoff) criterion** chooses the strategy with the largest expected payoff. With probabilities 0.5, 0.3, and 0.2, a2 gives 4.9 versus 4.8 for a1, so we choose a2. Risk measures such as variance may also matter, especially when expected payoffs are close. **Decision trees** use square decision nodes and circular chance nodes; evaluate them from the leaves back to the root by backward induction.

In **Bayesian decision theory**, observed data update a prior distribution to a posterior via Bayes' rule, and the action minimizing posterior expected loss is selected. The **expected value of perfect information (EVPI)** is the value of knowing the state before choosing: the expected payoff when one can choose after observing the state minus the best payoff when choosing without that information. In hypothesis testing, a Type I error is a false positive and a Type II error is a false negative. Wald's sequential probability-ratio test allows evidence to be evaluated incrementally.


### Reinforcement

**Additional worked example.** A lottery pays 100 with probability 0.6 and 0 with probability 0.4, so its expected monetary value is 60. A guaranteed 55 has a lower expected value, but a risk-averse person may still choose it because utility need not be linear in money.

**Transfer example.** In AHP, if quality is three times as important as price, set `a_quality,price = 3` and `a_price,quality = 1/3`. The consistency check flags materially contradictory judgments.

**Active recall.** Close the explanation above and answer without peeking: How does expected value differ from expected utility? Why does AHP check the consistency ratio?

**Mini practice — check yourself.** Alternative A gives 80 for sure; B is 150 with probability 0.6 and 0 otherwise. Which one will a risk-neutral person choose?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** EV(A)=80; EV(B)=150·0.6=90. A risk-neutral person will choose B; another utility function may change the choice.

</details>

**Exam focus:**

- Risk means known probabilities; uncertainty means unknown probabilities. Bayes applies to expected payoff under risk, while Wald, Savage, Hurwicz, and Laplace are common uncertainty criteria.
- Savage minimizes maximum regret, using a regret table rather than applying minimax directly to payoffs.
- In Hurwicz's formula, α weights the best payoff and 1−α weights the worst; α = 0 is Wald and α = 1 is maximax.
- Expected utility is not the same as expected monetary payoff: nonlinear utility can make a risk-averse person reject a higher-EMV gamble.
- In AHP, reciprocal comparisons and a consistency-ratio check are important; 0.10 is a conventional guideline, not an absolute requirement.

---

<a id="q35"></a>

## 35. Decision-making under conflict: game theory, collective choice, and Bayesian networks

> **Learning outcome.** After this chapter, you will be able to find best answers and equilibria in games and update probabilities in a simple Bayesian network. First, try to explain the topic in your own words, and then test yourself with the block at the end.

The essence of the question is simple: what should you do when your result depends on other decision-makers who also pursue their own objectives? **Game theory** models such strategic interaction. In chess or rock-paper-scissors, each player's move and the rules together determine the outcome. The other topic is a **Bayesian network**, a probabilistic graph that can answer questions such as "the grass is wet—did it rain?"

**What is a game?** A game has players, each with strategies (complete plans of action), and a payoff for every combination of strategies. A **zero-sum** game has payoffs that sum to zero (for example, matching pennies); a **non-zero-sum** game permits mutual gains or losses. Games may be cooperative or non-cooperative, and static (simultaneous choices) or dynamic (sequential choices represented by a game tree).

**Matrix game and saddle point.** In a finite two-player zero-sum matrix game, rows are the first player's actions, columns are the second player's actions, and each cell gives the first player's payoff (the second receives its negative). The cautious first player takes the best of the row minima: this **maximin** is the lower value of the game. The second player takes the minimum of the column maxima: this **minimax** is the upper value. The lower value never exceeds the upper value.

If these two values match, the game has a **saddle point**: a cell that is the minimum in its row and the maximum in its column. Neither player can improve by deviating unilaterally. Example:

| | B1 | B2 | B3 |
|---|---|---|---|
| A1 | 4 | 5 | 6 |
| A2 | 3 | 1 | 2 |

The minimum in row A1 is 4 and in row A2 is 1, so the first player chooses A1 and guarantees 4. The column maxima are 4, 5, and 6; the second player chooses B1, minimizing the maximum payoff conceded. The values match: (A1, B1) is the saddle point and the game value is 4.

**Mixed strategies.** If the lower and upper values differ (as in rock-paper-scissors), no pure action secures the game value. A **mixed strategy** randomizes over pure actions. Von Neumann's minimax theorem states that every finite zero-sum matrix game has optimal mixed strategies, with maximin equal to minimax. The probabilities can be found by indifference equations or by reducing the game to a linear program; strictly dominated strategies can be removed first.

**Nash equilibrium.** For general-sum games and games with multiple players, a Nash equilibrium is a profile of strategies such that no player can improve their payoff by changing only their own strategy while the others' strategies remain fixed. Nash's theorem guarantees at least one equilibrium in mixed strategies for every finite game. A saddle point is a special case of Nash equilibrium in a two-player zero-sum game.

Classic — **prisoner's dilemma**. Two suspects are interrogated separately, each can remain silent or betray:

| | Player 2 stays silent | Player 2 betrays |
|---|---|---|
| Player 1 stays silent | 1 year, 1 year | 10 years, 0 years |
| Player 1 betrays | 0 years, 10 years | 5 years, 5 years |

Regardless of the other's action, each prisoner receives a lower sentence by betraying. Thus mutual betrayal is the Nash equilibrium (5 years each), although mutual silence would give both only 1 year. A Nash equilibrium is stable, not necessarily socially optimal. In finite perfect-information sequential games, backward induction analyzes the game tree from the terminal positions.

**Collective choice.** Group decisions have their own pitfalls. **Majority rule** compares alternatives pairwise; the **Borda count** assigns points by rank; and a **Condorcet winner** defeats every other alternative in a pairwise majority vote. The **Condorcet paradox** shows that majority preferences can cycle even when each voter is consistent. For example, voters may rank A > B > C, B > C > A, and C > A > B: collectively A beats B, B beats C, and C beats A. Arrow's impossibility theorem states that, with at least three alternatives and an unrestricted preference domain, no voting rule can simultaneously satisfy standard fairness conditions (including Pareto efficiency and independence of irrelevant alternatives) and be non-dictatorial. The Delphi method uses anonymous expert rounds; in cooperative games, the **Shapley value** divides a coalition's surplus according to average marginal contribution across all player orderings.

**Bayesian networks.** A Bayesian network is a directed acyclic graph (DAG): vertices are random variables, and each vertex has a conditional-probability table given its parents. The joint distribution factors as

```
P(x₁, …, xₙ) = ∏ᵢ P(xᵢ | parents(xᵢ)).
```

This factorization can make storage and inference much cheaper than representing the full joint table. An arrow represents a conditional dependency, not necessarily a physical cause.

Example: suppose P(Rain) = 0.20, P(Wet | Rain) = 0.90, and P(Wet | ¬Rain) = 0.10. Then Bayes' rule gives:

P(Rain | Wet) = P(Wet | Rain) · P(Rain) / P(Wet) = 0.18 / 0.26 ≈ 0.69

Thus observing wet grass raises the rain probability from 20% to about 69%; this is probabilistic inference. Exact inference in an arbitrary Bayesian network is generally #P-hard, although it is tractable for structures such as trees and polytrees using methods such as variable elimination, junction-tree propagation, or belief propagation. Approximate methods include Monte Carlo and Gibbs sampling. Adding decision and utility nodes produces an **influence diagram**, which supports action selection by expected utility.


### Reinforcement

**Additional worked example.** In a prisoner's dilemma, betrayal is the best response regardless of the other's action, so mutual betrayal is a Nash equilibrium, even though mutual cooperation would give both parties a better outcome. Individual stability is not equal to the social optimum.

**Transfer example.** If a disease has a prior of 1%, a test sensitivity of 90%, and a false positive of 5%, then a positive result does not mean a 90% probability of the disease: P(D|+) = 0.009/(0.009+0.0495) ≈ 15.4%.

**Active recall.** Close the explanation above and answer without peeking: What must be done in a Nash equilibrium? Why is P(test+|disease) not equal to P(disease|test+)?

**Mini practice — check yourself.** The game has a saddle point if maximin = minimax. What does this mean for the need for mixed strategies?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Pure strategies already form an equilibrium at the saddle point; mixing is not required to secure the game price.

</details>

**Exam focus:**

- The saddle point is a special case of Nash equilibrium; the Nash equilibrium always exists (in mixed strategies), the saddle point in pure strategies does not always exist.
- The lower value of the game is never higher than the upper value; equality is the criterion for a pure-strategy saddle point.
- Nash equilibrium does not mean "best for everyone" - the prisoner's dilemma clearly shows this.
- A Bayesian network is a DAG; an arrow denotes a conditional dependency, not necessarily physical causation.
- Exact inference is tractable in some sparse graph structures but #P-hard in general, motivating approximate methods.

---

<a id="q36"></a>

## 36. Knowledge-based systems, knowledge representation and inference, semantic networks, frames, rules, and ontologies

> **Learning outcome.** After this chapter, you will be able to compare logical, semantic, frame, production, and ontological representations of knowledge and perform simple inference. First, try to explain the topic in your own words, and then test yourself with the block at the end.

The essence of the question: how to record human knowledge so that a computer can reason with it. A knowledge-based system is an "expert" program, in which knowledge lies in a separate explicit "reference book" (for example, rules: "if temperature and rash - suspect measles"), and a separate "engine" is able to draw conclusions from this reference book. The question covers the four classic ways of recording knowledge—logical formulas, concept networks, "questionnaire"-frames, and "if-then" rules—plus ontologies, shared concept vocabularies for the web.

**Knowledge-based system architecture.** Three main parts: a knowledge base (knowledge of the subject area), an inference engine (the mechanism that applies that knowledge), and working memory (facts of the current task). An explanation subsystem ("why did you decide that way?") and an interface are also common. The key idea is to separate knowledge from the reasoning mechanism: the base can be changed without rewriting the engine. A historical classic is the MYCIN medical system, which diagnosed infections using rules with certainty factors.

**Logical models.** Here, knowledge is represented by formulas of formal logic. The simplest case is propositional logic: propositions are true or false and are connected by "and", "or", "not", and "if." Checking whether a propositional formula can be made true is the SAT problem, which is NP-complete. First-order predicate logic adds properties, relations ("Socrates is a man"), and the quantifiers "for all" and "there exists." First-order validity and entailment are semidecidable: if a statement follows, a complete proof procedure can eventually prove it, but if it does not follow, the procedure may run forever.

Inference derives new facts from existing ones. A basic rule is modus ponens: "A is true, and A implies B; therefore B" ("it is raining" and "if it rains, the street is wet" imply that the street is wet). A general proof technique is **resolution refutation**: convert the formulas to clause form, add the negation of the goal, and derive a contradiction. For example, from "every person is mortal" and "Socrates is a person," instantiate the universal rule with `x = Socrates` and derive "Socrates is mortal," contradicting the negated goal. This step is universal instantiation; **unification** is the more general process of finding substitutions that make terms or literals match. Resolution is sound and refutation-complete. The two common rule-application strategies are **forward chaining**, from facts toward consequences, and **backward chaining**, from a goal toward facts that would prove it; Prolog uses goal-directed SLD resolution.

**Semantic networks.** This is a graph of concepts: vertices are objects and classes, and arcs are relations: "a dog is an animal" (class-subclass), "Rex is a dog" (an instance of a class), and "a tail is part of a dog." The main mechanism is inheritance: if "an animal breathes," then a dog breathes and Rex breathes, although those consequences were not written explicitly. Default exceptions are possible ("birds normally fly, but a penguin is a bird that does not fly"), which motivates non-monotonic reasoning: new knowledge can invalidate an earlier default conclusion. The representation is intuitive and visual, but early semantic networks lacked strict formal semantics; description logics later provided formal semantics for important classes of such representations.

**Frame model.** A frame (Minsky's idea) is a "questionnaire" for a typical situation or object. A frame has slots (fields) with default values, constraints, and attached "daemon" procedures that are triggered when values are requested or changed. For example, a `Room` frame may have slots for `walls` (default 4), `windows` (0 to 10), and `area` (computed as length times width when requested). A `Bedroom` frame can inherit those slots from `Room` and add a `bed` slot. Frames closely resemble object-oriented classes and objects and influenced later knowledge-representation and object systems.

**Production model.** Knowledge is a set of "IF condition, THEN action or conclusion" rules. The engine runs a three-step cycle: find rules whose conditions match the current facts; if several match, choose one by a conflict-resolution strategy such as priority or specificity; then fire it and update the facts. For example, use the rules "IF fever AND cough, THEN suspect flu" and "IF flu, THEN recommend bed rest" with the facts `{fever, cough}`. The first cycle adds `flu`, and the second adds `bed rest`. To avoid checking every rule against every fact after each change, the Rete algorithm caches partial matches. The rules are modular and understandable to experts, but in a large knowledge base their interactions can become difficult to predict.

**Ontologies, RDF, OWL.** An ontology is a formal description of domain concepts: classes, their hierarchy, properties, constraints, and instances. The goal is for people and programs to interpret terms consistently, for example on the Semantic Web or during data integration. OWL is grounded in description logics: a knowledge base can be divided into a TBox (terminological axioms about classes) and an ABox (assertions about individuals), and a reasoner can, for example, infer a class hierarchy.

Three levels of languages, which are important not to confuse:

- **RDF** — a data model whose basic statements are subject–predicate–object triples. Subjects are IRIs or blank nodes, predicates are IRIs, and objects are IRIs, blank nodes, or literals.
- **RDFS** — a simple dictionary on top of RDF: classes, subclasses, subproperties, domains and ranges of properties; gives base inheritance.
- **OWL** — a more expressive ontology language that supports class equivalence and disjointness, cardinality restrictions ("a person has exactly two parents"), and transitive and inverse properties. OWL 2 profiles trade expressiveness for tractable reasoning; OWL 2 EL, for example, is designed for large ontologies with many classes and properties.

Queries to RDF graphs are written in SPARQL (often described as "SQL for RDF graphs"). Protégé is a popular ontology editor, and FOAF and schema.org are well-known vocabularies.


### Reinforcement

**Additional worked example.** The fact `Bird(Tweety)` and rule `Bird(x) → HasWings(x)` allow us to derive `HasWings(Tweety)`. However, a rule saying “birds fly” needs exceptions for penguins—the classic motivation for non-monotonic reasoning.

**Transfer example.** The RDF triples `Kyiv type City` and `City subclassOf Settlement`, together with RDFS semantics, let a reasoner infer that Kyiv is a Settlement without storing that fact explicitly.

**Active recall.** Close the explanation above and answer without peeking: How does a production rule differ from a frame? What are the roles of RDF, RDFS and OWL?

**Mini practice — check yourself.** There are rules A→B, B∧C→D and facts A,C. What new facts will be obtained by direct inference?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** First B with A, then D with B and C. Closure of facts: {A, C, B, D}.

</details>

**Exam focus:**

- "Class-subclass" vs. "class instance": "dog is an animal" is one thing, "Rex is a dog" is another.
- Forward chaining proceeds from facts to consequences; backward chaining proceeds from a goal to facts that would establish it. Prolog uses goal-directed SLD resolution.
- Resolution refutation adds the negation of the goal and derives a contradiction.
- First-order validity and entailment are semidecidable, not decidable — a favorite trap question.
- RDF, RDFS and OWL are three layers (data, dictionary, logic), not synonyms.

---

<a id="q37"></a>

## 37. Artificial neural networks, activation, backpropagation, recurrent networks, Hopfield networks, and Hamming networks

> **Learning outcome.** After this chapter, you will be able to trace forward and backpropagation, select activation, and explain the differences between feed-forward, recurrent, Hopfield, and Hamming networks. First, try to explain the topic in your own words, and then test yourself with the block at the end.


The crux of the matter: a neural network is a bunch of very simple "bricks", each of which can do one thing - multiply its inputs by weights, add up and pass the result through a non-linear function. But connected in layers, these bricks are able to approximate almost any dependence. "Training" is the selection of weights so that the responses of the network coincide with the desired ones, and the main working tool is error backpropagation: the error at the output is "propagated" back through the network, and each weight learns exactly how much it is to blame.

**Neuron and architectures.** One neuron calculates the weighted sum of the inputs plus a bias and applies an activation function:

y = f(Σ wᵢxᵢ + b)

Here, each weight controls the influence of an input, the bias shifts the activation threshold, and `f` adds nonlinearity. Without nonlinear activations, any number of stacked layers collapses to a single affine transformation. Numerical example: inputs (1, 0.5), weights (0.8, −0.4), and bias 0.1 give a pre-activation of 0.7; its sigmoid is approximately 0.67.

Rosenblatt's single-layer perceptron was an important early trainable neural model and is essentially a linear classifier: it draws a straight line (in higher dimensions, a hyperplane) between classes. Its famous weakness is the **XOR problem**: labels assigned to opposite corners of a square cannot be separated by one straight line. Adding a hidden layer makes XOR representable by a multilayer perceptron (MLP), in which signals flow from the input through hidden layers to the output. Cybenko's theorem states that a one-hidden-layer network with a suitable sigmoidal activation can approximate any continuous function on a compact domain arbitrarily well. This is an existence theorem: it does not say how many neurons are needed or how to train them.

**Activation functions** — a short cheat sheet in words:

- threshold ("step") — historically important but non-differentiable, so it is unsuitable for ordinary gradient training;
- sigmoid — smooth, squeezes everything in the range from 0 to 1; the trouble is the attenuation of the gradient: its derivative is small, and in a deep network the product of such derivatives goes to zero, the first layers stop learning;
- hyperbolic tangent (`tanh`) is zero-centered, unlike sigmoid, but it also saturates and can suffer from vanishing gradients;
- ReLU (`max(0, x)`) is a standard activation in deep networks: it is fast and does not saturate on positive inputs; a neuron whose pre-activation remains negative has zero gradient and may stop updating, becoming a "dead ReLU";
- softmax — at the output: converts numbers into probabilities of classes that add up to one.

**How networks learn.** In supervised learning, examples pair inputs with target outputs, and training minimizes a loss such as squared error for regression or cross-entropy for classification. In unsupervised learning there are no target labels; methods include clustering, dimensionality reduction, representation learning, and Hebbian learning. Gradient-based training moves weights in the direction opposite the loss gradient, with step size controlled by the learning rate. Neural networks are commonly optimized with mini-batch stochastic gradient methods such as SGD or Adam.

**Backpropagation.** This is not an optimizer by itself, but an efficient way to calculate gradients for an optimizer. The four steps are: (1) perform a forward pass and compute all activations; (2) compute the loss at the output; (3) use the chain rule in a reverse pass to obtain the derivative of the loss with respect to each parameter; and (4) update the parameters in the direction chosen by the optimizer. For a linear neuron `y = wx` with loss `L = ½(y − t)²`, let `w = 0.5`, `x = 1`, and `t = 1`. Then `y = 0.5`, `dL/dw = (y − t)x = −0.5`, and a gradient-descent step of 0.1 gives `w = 0.55`, moving the next prediction toward 1. Reverse-mode differentiation computes all parameter gradients at a cost proportional to the operations in the forward computation. Practical difficulties include vanishing or exploding gradients and choosing an appropriate learning rate.

**Recurrent networks (RNNs).** The new hidden state depends on both the current input and the previous hidden state, so the network can model sequences such as text or sound. Training unfolds the recurrence through time and applies backpropagation through time (BPTT). Ordinary RNNs often struggle with long-range dependencies because gradients can vanish or explode. LSTMs and GRUs use gates to control the flow of information; a GRU is a simpler gated alternative to an LSTM.

**Hopfield network** is a fully connected recurrent network that works as associative memory: given a noisy or incomplete pattern, it can recover a stored pattern. In the classical model, weights are symmetric, self-connections are zero, and neuron states are `+1` or `−1`. Patterns can be stored with a Hebbian rule. During recall, neurons are updated asynchronously, one at a time, according to the sign of their weighted input. Under these conditions an energy function cannot increase, so the network reaches a fixed point:

E = −½ Σ wᵢⱼ sᵢ sⱼ

The network descends into an attractor of the energy landscape, but not every attractor must be a stored pattern: spurious attractors can occur. For random, unbiased patterns stored by the standard Hebbian rule, the asymptotic reliable-recall capacity is approximately `0.138N` patterns for `N` neurons. A tiny example: store the pattern `(+1, −1, +1)` and present the corrupted input `(+1, +1, +1)`. If the learned weights give the second neuron a negative local field, its asynchronous update changes it to `−1` and restores the pattern.

**Hamming network** is a classifier based on Hamming distance, the number of positions at which two binary vectors differ; for example, the distance between `1011` and `1001` is 1. The first layer measures the similarity of the input to each stored prototype. A competitive MAXNET layer then suppresses weaker responses until the closest prototype wins. A Hamming network classifies an input by direct comparison with stored prototypes, whereas a Hopfield network iteratively reconstructs an attractor. Its storage cost grows with the number of prototypes.


### Reinforcement

**Additional worked example.** The neuron has x=(1,2), w=(0.5,−1), b=0.5. Weighted sum z=1·0.5+2·(−1)+0.5=−1; ReLU returns 0 and sigmoid returns approximately 0.269. The same linear part gives different nonlinear behavior.

**Transfer example.** During backpropagation, the output error is passed backward using the chain rule. If a ReLU neuron consistently has z < 0, its derivative is 0 and its weights may stop updating; this is called a “dead ReLU.”

**Active recall.** Close the explanation above and answer without peeking: why does a network need nonlinear activation? Which two passes are repeated during each training step?

**Mini practice — check yourself.** For the sigmoid output y=0.8 and target t=1, calculate the sign of the error y−t and say in which direction the logit should be changed to reduce the loss.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** y−t=−0.2; the logit must be increased so that the sigmoid output approaches 1. The exact step depends on the learning rate and the derivatives of the previous layers.

</details>

**Exam focus:**

- Why a single-layer perceptron cannot represent XOR: the classes are not linearly separable; a hidden layer makes the pattern representable.
- Cybenko's theorem is about the existence of an approximation, not about a learning algorithm.
- Backprop is a fast gradient calculation, not a separate method from gradient descent.
- With symmetric weights, zero self-connections, and asynchronous updates, a classical Hopfield network has non-increasing energy and converges to a fixed point.
- The `0.138N` Hopfield capacity estimate assumes random, unbiased patterns and the standard Hebbian rule.
- To be able to name the disadvantage of each activation: sigmoid - attenuation of the gradient, ReLU - dead neurons.

---

<a id="q38"></a>

## 38. Self-organizing and competitive-learning networks, SOM, stochastic adaptation, and CNNs

> **Learning outcome.** After this chapter, you will be able to explain competitive learning and SOM and calculate the output size of a simple CNN convolution. First, try to explain the topic in your own words, and then test yourself with the block at the end.

The crux of the matter: self-organization is when a network finds structure in the data on its own without any prompts, like a person sorting a bunch of photos into stacks of similar ones. In competitive learning, neurons "compete" to see who can better recognize the input, and the winner adapts to it even more. The Kohonen network turns multidimensional data into a visual two-dimensional "similarity map." And a convolutional network (CNN) is a neural network for images: a small "magnifier filter" slides over the picture and looks for the same pattern (edge, corner) anywhere.

**Competitive learning.** There are no target labels, only a stream of examples. Each neuron stores a weight vector that acts as a prototype. For an input `x`, the neuron whose prototype is closest to `x`, often by Euclidean distance, wins; this is the winner-take-all principle. The winner moves its prototype toward the input according to `w ← w + η(x − w)`, where `η` is the learning rate. For example, if `w = (1, 1)`, `x = (0.9, 0.8)`, and `η = 0.5`, the updated prototype is `(0.95, 0.9)`. After many examples, prototypes approximate cluster centers, much like online k-means. A potential problem is a "dead" neuron that never wins and therefore never learns; remedies include conscience mechanisms that penalize frequent winners or winner-take-most variants that also update nearby neurons.

**Kohonen network (SOM).** Its neurons occupy nodes of a usually two-dimensional grid. For each input, find the best matching unit (BMU), the neuron with the closest prototype. Move both the BMU and its **lattice neighbors** toward the input; the update becomes smaller as grid distance from the BMU increases, often according to a Gaussian neighborhood function. Both the learning rate and neighborhood radius decrease over time: a broad neighborhood first establishes global order, and a smaller neighborhood later performs local refinement. The result is a topologically ordered map in which similar input vectors tend to activate nearby grid cells.

A critically important nuance: neighborhood is measured by distance **on the map grid**, not in data space. And it is the updating of the neighbors (and not just the winner) that distinguishes SOM from simple competitive learning and generates regularity: neighbors "pull" after the winner, so their prototypes become similar, and smooth regions appear on the map. Application: visualization of multidimensional data, clustering, compression of a set of vectors to a small set of prototypes (vector quantization); the supervised version is called LVQ.

**Step-size conditions.** SOM learning is related to stochastic approximation. A common diminishing learning-rate schedule uses the Robbins–Monro conditions

Σ η(t) = ∞ and Σ η²(t) < ∞

The divergent first sum prevents the steps from shrinking too quickly; the finite second sum makes the influence of noise diminish. A typical schedule is `η(t) = c/t`: the harmonic series diverges, whereas the sum of inverse squares converges. A constant step violates the second condition, and a `1/t²` step violates the first. These step-size conditions are useful but do not by themselves prove SOM convergence. Rigorous convergence results require additional assumptions or restricted variants; there is no general convergence theorem for the standard multidimensional SOM.

**Convolutional neural networks (CNNs).** This architecture is designed for grid-structured data, especially images; LeCun's LeNet-5 is a classic example. Its three key ideas are:

1. **Local receptive fields:** a neuron sees a small window, such as 3×3 pixels, rather than the whole image.
2. **Shared weights:** the same small weight matrix, or kernel, is applied at every image position, so it detects the same feature anywhere.
3. **Pooling:** each window is summarized by its maximum or average, reducing spatial size and adding robustness to small shifts.

Here is a convolution step by step. Take a 3×3 image and a 2×2 filter. Place the filter in the upper-left corner, multiply corresponding values, and add the products to obtain one output value. Shift the filter by one pixel and repeat. The four valid positions produce a 2×2 feature map showing how strongly the filter pattern responds at each location. With input size `N`, padding `P`, kernel size `K`, stride `S`, and dilation 1, the output size along one axis is `⌊(N + 2P − K)/S⌋ + 1`; without padding, `P = 0`. Fully connected layers and softmax can then perform classification.

Why do CNNs often outperform ordinary MLPs on images? Local connectivity and weight sharing use far fewer parameters: one 3×3 kernel has 9 weights per input–output channel pair, reused at every spatial position, rather than a separate weight for every pixel–neuron pair. This parameter efficiency can reduce overfitting. Successive layers can build a hierarchy of edges, textures, object parts, and whole objects. Training still uses backpropagation. Well-known architectures include AlexNet, VGG, and ResNet, whose residual connections help gradients pass through very deep networks.


### Reinforcement

**Additional worked example.** For each input vector, a SOM finds the nearest winning neuron and moves both its weights and its map neighbors toward the input. The neighborhood radius gradually decreases, so global ordering gives way to local refinement.

**Transfer example.** For a 32×32 image, a 3×3 kernel, stride 1, and padding 0, the output size is (32−3)/1+1 = 30, so the feature map is 30×30. With padding 1, the spatial size would remain 32×32.

**Active recall.** Close the explanation above and answer without peeking: what is updated during competitive learning? Which parameters determine the spatial size of a convolution's output?

**Mini practice — check yourself.** Input 28×28, kernel 5, stride 2, padding 1. Find the output size along one axis.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** ⌊(28 + 2·1 − 5)/2⌋ + 1 = ⌊25/2⌋ + 1 = 13.

</details>

**Exam focus:**

- SOM is not just winner-take-all: lattice neighbors are also updated, and this is what gives the topological order.
- Neighborhood in SOM is on the map grid, not in the input data space.
- Robbins–Monro conditions: the infinite sum prevents steps from shrinking too quickly, and the finite sum of squared steps attenuates noise; constant steps and `1/t²` do not satisfy both conditions.
- These learning-rate conditions alone do not prove SOM convergence; general multidimensional SOM convergence requires additional assumptions or restricted variants.
- Shared weights are the main reason for the small number of CNN parameters; pooling gives robustness to small shifts, not "lossless compression".

---

<a id="q39"></a>

## 39. Machine learning, empirical risk, overfitting, and the bias–variance trade-off

> **Learning outcome.** After this chapter, you will be able to distinguish underfitting from overfitting, explain the bias–variance decomposition, and apply data splitting and complexity control. First, try to explain the topic in your own words, and then test yourself with the block at the end.

**The essence of the question.** In machine learning, a system learns a pattern from examples instead of following only hand-written rules. Imagine a student preparing from last year's exam questions: understanding the underlying ideas helps with new questions, whereas memorizing the old answers may fail as soon as the wording changes. This "memorized but did not generalize" behavior is called **overfitting**. The central question is why learning from a finite sample can work on new data and how to avoid fitting noise.

Mitchell's classic definition sounds something like this: a program learns if, with the accumulation of experience (data), it performs its task better and better according to the selected quality measure. For example: the task is to recognize spam, the experience is ten thousand letters marked as "spam / not spam", the measure is the percentage of correctly sorted letters.

**Three learning paradigms** (I explain in words):

| Paradigm | What is given | Life example |
|---|---|---|
| Supervised learning | examples + target labels or values | spam filter, apartment price forecast |
| Unsupervised (no teacher) | only examples, no answers | divide customers into similar groups |
| Reinforcement learning | the agent acts and receives rewards | teach the program to play chess |

**Statistical learning theory (Vapnik and Chervonenkis)** asks when performance on a finite sample predicts performance on new data from the same distribution. The **population risk** is the expected loss under the unknown data-generating distribution. We want to minimize it, but cannot calculate it exactly because the distribution is unknown. The **empirical risk**, by contrast, is the average loss on the observed sample and is easy to calculate.

**Empirical Risk Minimization (ERM)** is the honest name of a simple idea: since the true error is not visible, let's minimize the one that is visible - the average error on the available examples:

```
R_emp(h) = (1/n) · Σ L(yᵢ, h(xᵢ))
```

That is, you simply take all n examples, see how much the model was wrong on each one, and average it. The main question: when does a small error on the sample guarantee a small error on the new data?

One relevant capacity measure is **VC dimension**: the largest `d` for which some set of `d` points can be **shattered**, meaning that the model class can realize every binary labeling of that fixed set. Affine linear classifiers in the plane have VC dimension 3. Three non-collinear points can be shattered, but no set of four points can be shattered; for four corners of a square, the XOR labeling is not linearly separable.

The main result of the theory is **Vapnik's estimate**, which can be read as follows:

> true error ≤ sampling error + "penalty for model complexity",

and the penalty grows with model-class capacity and decreases with the number of examples. The conclusions are intuitive: a more flexible model usually needs more data. Infinite VC dimension rules out distribution-free uniform generalization guarantees from finite samples under the standard PAC assumptions unless further restrictions are imposed. Structural risk minimization therefore compares model classes of increasing capacity and balances empirical error against a complexity penalty. Another famous result is the "No Free Lunch" theorem: averaged uniformly over all possible problems, no learning algorithm is universally superior; useful learning requires assumptions about the problem.

**Overfitting** means that a model has fitted noise or accidental patterns in the training sample. A common symptom is low training error but substantially higher validation error. The opposite problem is underfitting: the model is too limited to fit even the training data well.

A classic example uses 10 noisy points near a straight line. A straight line with two parameters may capture the trend and predict new points well. A ninth-degree polynomial can interpolate 10 points with distinct input coordinates and achieve zero training error, yet it may oscillate between them and generalize poorly.

Common remedies include collecting more representative data, using regularization, choosing a simpler model, early stopping, selecting hyperparameters by cross-validation, applying valid data augmentation, and averaging models in an ensemble.

The bias–variance tradeoff explains where the error comes from. For quadratic losses, it is divided into three parts:

```
Error = Bias² + Variance + noise
```

Bias is systematic error: the model's average prediction misses the target, often because the model class is too simple. Variance measures how much the fitted prediction changes when the training sample changes and is often high for very flexible models. Irreducible noise is variation that no predictor based on the available features can eliminate.

The standard analogy is target shooting: bias is a misaligned sight, while variance is a shaky hand that scatters shots. A simple model often has high bias and low variance; a flexible model often has lower bias and higher variance. In practice, capacity is controlled by hyperparameters such as polynomial degree, regularization strength, or tree depth and selected using validation data or cross-validation. Classical test-error curves are often U-shaped as complexity grows. Some modern models instead exhibit **double descent**: test error falls, rises near the interpolation threshold, and then falls again.


### Reinforcement

**Additional worked example.** A polynomial of the first degree does not describe wavy data: high training and validation errors mean high bias. A polynomial of the 20th degree passes all training points, but makes mistakes on new ones: low training and high validation error mean high variance.

**Transfer example.** If you repeatedly use test-set results to select a model, the test set effectively becomes validation data and the resulting performance estimate is optimistic. Keep the test set untouched during model selection and evaluate it once after the model and hyperparameters are fixed.

**Active recall.** Close the explanation above and answer without peeking: How to distinguish bias and variance from training/validation curves? Why more data often helps variance, but not necessarily bias?

**Mini practice — check yourself.** Training accuracy 99%, validation 72%. Name two relevant actions.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Reduce complexity/add regularization and collect or augment data. Also check for leakage and the correctness of the distribution; simply training longer is likely to make overfitting worse.

</details>

**Exam focus:**

- We minimize empirical risk on a sample, but want low population risk; capacity-dependent generalization bounds connect the two.
- VC-dimensionality is not the number of parameters; for straight lines on the plane, it is equal to 3.
- Overfitting is not "the model has learned poorly"; on the contrary, it is far too good at training data.
- The question "increase the complexity of the model - what will happen?": bias decreases, variance increases.

---

<a id="q40"></a>

## 40. Supervised learning, cross-validation, regularization, SVMs, and kernel methods

> **Learning outcome.** After this chapter, you will be able to distinguish between classification and regression, correctly build cross-validation and explain L1/L2, margin and kernel trick. First, try to explain the topic in your own words, and then test yourself with the block at the end.


**The essence of the question.** In supervised learning, each training example includes an input and a target. Predicting a numeric target, such as an apartment price or tomorrow's temperature, is **regression**; predicting a category, such as "cat or dog" or "spam or not," is **classification**. Three practical questions are how to estimate performance honestly (validation and testing), how to control overfitting (regularization), and how to construct an effective decision boundary (for example, with an SVM and kernels).

For regression, a basic tool is linear regression: the prediction is a weighted sum of features plus an intercept, and ordinary least squares chooses coefficients that minimize the sum of squared residuals. Classification quality can be measured by accuracy, precision (the fraction of predicted positives that are truly positive), recall (the fraction of actual positives that were found), and the F1 score, the harmonic mean of precision and recall.

**Logistic regression** — despite the name, it is a classifier, a classic exam pitfall. It computes a weighted sum of features and passes it through a sigmoid, a smooth S-shaped function that maps any finite real number to a value strictly between 0 and 1:

```
P(class 1) = 1 / (1 + e^(−(w·x + b)))
```

The output models the conditional probability of class 1. For example, let the weight be 2, bias −1, and input 2. The logit is `2·2 − 1 = 3`, so the sigmoid output is approximately 0.95. A threshold of 0.5 corresponds to a zero logit, so the decision boundary is a line, or a hyperplane in higher dimensions. Logistic regression is trained by maximizing likelihood, equivalently minimizing log loss. The loss is convex, so it has no spurious local minima; uniqueness requires additional rank or regularization conditions, and linearly separable unregularized data may have no finite minimizer. Multinomial logistic regression commonly uses softmax.

**Validation and testing** address honest evaluation: measuring a model on the same data used to train it is like allowing a student to take an exam using the exact questions they memorized. In a train/validation/test split, fit parameters on the training set, select models and hyperparameters on the validation set, and make the final assessment once on the untouched test set.

**K-fold cross-validation** divides the available training data into `K` folds, commonly 5 or 10. Fit the model `K` times: each run uses `K − 1` folds for training and the remaining fold for validation. Average the `K` validation scores. With 100 examples and `K = 5`, each fold has 20 examples. If the validation errors are 0.10, 0.14, 0.08, 0.12, and 0.16, their mean is 0.12, and each example served as validation data exactly once. Leave-one-out cross-validation uses `K = n`; it often has low bias for the `n − 1` training-set procedure but can have high variance and computational cost. For imbalanced classification, stratified folds preserve class proportions as closely as possible.

**Regularization** adds a complexity penalty to the data-fitting loss: minimize prediction error while discouraging overly large coefficients. Its strength is controlled by the hyperparameter `λ`. Two common options are:

- **L2 (ridge)** penalizes the sum of squared coefficients. It typically shrinks coefficients without inducing sparsity. For linear least squares with `λ > 0`, ridge gives a unique coefficient solution even when features are collinear.
- **L1 (lasso)** penalizes the sum of absolute coefficient values. It can drive some coefficients exactly to zero and thereby perform feature selection.

Combining L1 and L2 penalties gives the elastic net. **Algorithmic stability** means that replacing one training example changes the learned predictor only slightly. Under suitable assumptions, regularization can improve stability, and stability can yield generalization bounds; neither statement is unconditional for every regularized method.

**SVM (support vector machine, Vapnik).** If many hyperplanes separate the classes, an SVM chooses one with maximum margin: the greatest distance to the nearest training points on either side. Imagine laying the widest possible road between two villages. The boundary is determined by **support vectors**, the points on or inside the margin. In one dimension, if the negative class is at 1 and 2 and the positive class at 6 and 8, the boundary lies midway between the nearest opposing points 2 and 6, at 4; those two points are support vectors. Moving 1 or 8 does not change the boundary as long as they remain non-support vectors. A soft-margin SVM permits margin violations and penalizes them using `C`. A large `C` penalizes violations strongly and may overfit; a small `C` permits more violations and generally favors a wider margin.

**Kernel methods.** If classes are not linearly separable in the original space, a nonlinear feature map may make a linear separator possible in another space. The **kernel trick** avoids constructing that feature map explicitly: an SVM depends on examples through inner products, and a kernel evaluates the corresponding inner product in an implicit feature space. A valid kernel normally produces a positive-semidefinite Gram matrix; it is not an arbitrary similarity function. Common choices include linear, polynomial, and Gaussian radial basis function (RBF) kernels. Classical kernel SVM training and prediction can become expensive as the number of examples and support vectors grows, so linear or approximate methods are often preferred for very large datasets.


### Reinforcement

**Additional worked example.** In 5-fold cross-validation, split the data into five folds and run five fits. Each fit trains on four folds and validates on the remaining fold. Every example appears in a validation fold exactly once; the mean and standard deviation of the scores summarize performance and variation across folds.

**Transfer example.** L1 can drive weak coefficients to zero and therefore perform feature selection; L2 smoothly shrinks all weights. Both penalties control complexity, but their solution geometries differ.

**Active recall.** Close the explanation above and answer without peeking: Why should preprocessing fit inside each fold? What does SVM maximize?

**Mini practice — check yourself.** Scaler was trained on all data prior to cross-validation. What is the problem and how to fix it?

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** This is leakage: validation data affected the preprocessing statistics. Place the scaler and model in one pipeline and fit that pipeline only on the training portion of each fold.

</details>

**Exam focus:**

- Logistic regression is classification, not regression (a favorite pitfall question).
- In K-fold cross-validation, each example appears in a validation fold exactly once, and the model is fitted `K` times.
- L1 can drive coefficients to zero and select features; L2 generally shrinks coefficients without inducing sparsity.
- In SVM, a larger gap corresponds to a smaller norm of weights; larger C - tighter to errors, narrower gap.
- The kernel trick is valuable because the high-dimensional mapping is never built explicitly.

---

<a id="q41"></a>

## 41. Unsupervised learning, clustering, PCA, reinforcement learning, and Q-learning

> **Learning outcome.** After this chapter, you will be able to perform one-step k-means and PCA intuitively and explain state, action, reward, and Q-update in reinforcement learning. First, try to explain the topic in your own words, and then test yourself with the block at the end.


**The essence of the question.** There is no teacher here. Clustering is "sorting a bunch of things into similar piles by yourself," like sorting a pile of socks without instructions. PCA is "compress the data, leaving the essentials", like the shadow of a three-dimensional object on the wall: the dimensions are fewer, and the shape is recognizable. Reinforcement learning is training: the agent tries actions, receives a "treat" (reward) for good ones, and gradually learns to behave optimally.

### Clustering

**k-means clustering.** The goal is to divide the points into k groups so that each point is as close as possible to the center of its group. The algorithm is simple:

1. Initialize k centers (preferably with k-means++, which tends to choose well-separated starting centers).
2. Assign each point to the nearest center.
3. Move each center to the arithmetic mean of its points.
4. Repeat steps 2-3 until nothing changes.

Example on a straight line, k=2, points {1, 2, 3, 10, 11, 12}, starting centers 2 and 3. At first, {1, 2} is closer to center 2, {3, 10, 11, 12} to center 3. We move the centers: the first becomes 1.5, the second becomes 9. Now point 3 runs to the first center (it is closer to 1.5 than to 9). The centers become 2 and 11, and everything stabilizes: the clusters {1, 2, 3} and {10, 11, 12} are exactly what the eye can see.

What should be remembered: the algorithm converges to a local optimum, so the result depends on the initialization and the algorithm is usually run several times. The number of clusters k must be specified in advance; it can be selected with the elbow method or the silhouette coefficient. One iteration costs O(nkd), where n is the number of points, k the number of clusters, and d the number of features; this is linear in n when k and d are fixed. The method favors roughly spherical clusters of similar size and is sensitive to outliers.

**Hierarchical clustering** builds not one partition, but a whole tree of mergers — a **dendrogram**. In the agglomerative approach ("bottom-up"), each point starts as a separate cluster; at each step, the two nearest clusters are merged until only one remains. The dendrogram can be built without choosing k, although a cut level or cluster count must still be selected to obtain a flat partition. Distance between clusters can be defined by the closest pair of points (single linkage, which can produce long chains), the farthest pair (complete linkage), the average pairwise distance, or Ward's method (which minimizes the increase in within-cluster variance). The disadvantage is computational cost: a naive implementation can take cubic time in the number of points.

### Principal Component Analysis (PCA)

Suppose the data has a hundred features, but we want to reduce it to two or three while retaining its essential structure. PCA rotates the coordinate system so that the new axes point in the directions of greatest variance, then keeps the first few axes. The recipe is to center the data (subtract each feature's mean), calculate the covariance matrix, and find its eigenvectors and eigenvalues. The eigenvectors define the new axes, while the eigenvalues give the variance along those axes. We sort the components by decreasing eigenvalue and keep enough to preserve, say, 95% of the total variance.

For example, people's height and weight are strongly correlated, forming a cloud of points stretched along a diagonal. The first principal component follows the cloud (roughly, "body size") and might explain 90% of the variance; the second runs across it and explains only 10%. We can therefore keep one coordinate instead of two while discarding only 10% of the variance. Applications include compression, visualization, and noise reduction. Nonlinear dimensionality-reduction methods include kernel PCA, t-SNE, UMAP, and autoencoders.

### Reinforcement learning and Q-learning

An agent lives in an environment: it sees a state, performs an action, receives a reward, and enters a new state. The goal is to produce a **policy** (action selection rule) that maximizes the total reward, with future rewards slightly discounted by the discount factor γ (close to one γ means a "far-sighted" agent).

The central object is the function Q(s, a): "how much total reward will I collect on average, if in state s I do action a, and then play optimally." **Q-learning** updates this estimate after each step:

```
Q(s,a) ← Q(s,a) + α · [ r + γ · max Q(s',·) − Q(s,a) ]
```

In words: new estimate = old estimate + learning rate α multiplied by the temporal-difference error: the received reward plus the discounted best estimate from the next state, minus the old estimate.

Example: two steps to the goal, γ=0.9, α=0.5, all Q are initially zero. The agent in the penultimate state steps to the goal and gets 10: his Q becomes 0 + 0.5·(10 − 0) = 5. The next time, standing at the start, he takes a step to the penultimate state (there is no reward, but from there it "smells" like a five): the start Q becomes 0.5·0.9·5 = 2.25. So the reward gradually "flows" from the target back to the start, and over time the agent knows the value of each step.

A central challenge in reinforcement learning is the **exploration–exploitation trade-off**: should the agent try new actions or use the best action currently known? The simplest strategy is ε-greedy: with probability ε the agent takes a random action; otherwise, it takes the best known action. The value of ε is often reduced gradually. Other exploration strategies include softmax action selection, whose temperature controls randomness, and upper-confidence-bound methods, which favor actions tried less often. Q-learning is off-policy because its update uses the maximum over next-state actions even if the behavior policy selects another action. SARSA is on-policy because it updates from the action actually selected. For large state spaces, a neural network can approximate Q(s,a), as in a Deep Q-Network (DQN).


### Reinforcement

**Additional worked example.** Points 1, 2, 9, 10 with centers 1 and 10 after assignment form clusters {1,2} and {9,10}; new centers 1.5 and 9.5. The next assignment will not change—the algorithm has converged.

**Transfer example.** If two features lie almost on the same diagonal, the first principal component is directed along the largest spread and can replace the two coordinates with one with little loss of information.

**Active recall.** Close the explanation above and answer without peeking: What optimizes the k-means assignment step and update step? How does reward differ from return in RL?

**Mini practice — check yourself.** For Q=4, reward=3, max Q(next)=5, α=0.5, γ=0.9, perform Q-learning update.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Target = 3 + 0.9·5 = 7.5; new Q = 4 + 0.5·(7.5−4) = 5.75.

</details>

**Exam focus:**

- k-means always converges, but only to a local minimum - so multiple runs or k-means++.
- k-means requires specifying k in advance; hierarchical - no (cut the dendrogram anywhere).
- Data must be centered before PCA; the principal components are the eigenvectors of the covariance matrix.
- Q-learning is off-policy because its update uses the maximum over next-state actions; SARSA is on-policy because it uses the action actually selected.
- RL is not supervised: there are no correct answers, there is only a reward, often delayed in time.

---

<a id="q42"></a>

## 42. Fuzzy systems, membership functions, fuzzy inference, defuzzification, and fuzzy neural networks

> **Learning outcome.** After this chapter, you will be able to calculate basic fuzzy set operations and trace fuzzification, rule evaluation, aggregation and defuzzification. First, try to explain the topic in your own words, and then test yourself with the block at the end.


**The essence of the question.** Real-world concepts often have gradual boundaries: is a person tall at 180 cm but not at 179 cm? **Fuzzy logic** (Lotfi Zadeh, 1965) allows partial membership in a set: for example, a person 179 cm tall may belong to the fuzzy set "tall" with degree 0.7. Fuzzy systems process human-readable rules such as "IF the temperature is high, THEN run the fan at high speed" and produce a crisp output such as fan speed.

A **fuzzy set** generalizes a classical set: instead of only "belongs" or "does not belong," each element has a **degree of membership** from 0 to 1, specified by the **membership function** μ. In a classical set μ is only 0 or 1; in a fuzzy set it can take intermediate values. For the fuzzy set "tall," for example, μ(160 cm)=0, μ(175 cm)=0.5, and μ(190 cm)=1. The **support** contains all elements with membership greater than 0; the **core** contains those with membership 1; and an **α-cut** contains all elements whose membership is at least α.

The forms of membership functions are most often simple: triangular (linearly increases to a peak and linearly decreases), trapezoidal (with a "plateau" of full membership), Gaussian (bell), sigmoid. An example with a triangular "comfort temperature" with zeros at 20 and 30 and a peak at 25: temperature 22 corresponds to degree 0.4 (two-fifths of the way from 20 to the peak), temperature 25 - exactly 1, and 31 - already 0.

A **linguistic variable** is a variable whose values are words rather than numbers. Its components include a name ("temperature"), a set of linguistic terms ("low," "medium," "high"), a universe of discourse (for example, 0–50 °C), syntactic rules that generate modified terms ("very high," "not low"), and semantic rules that map each term to a fuzzy set, that is, to a membership function μ.

**Operations on fuzzy sets** generalize classical set operations. With the standard Zadeh operators, negation is one minus the membership degree, intersection (AND) takes the minimum, and union (OR) takes the maximum. Other choices include the product t-norm and probabilistic-sum t-conorm. If two membership degrees are 0.7 and 0.4, their intersection is 0.4, their union is 0.7, and the negation of the first is 0.3. With the standard operators, the classical laws of excluded middle and non-contradiction do not generally hold: if μ_A(x)=0.7, then μ_{A∩¬A}(x)=min(0.7,0.3)=0.3 rather than 0.

**Fuzzy logic** allows truth degrees between 0 and 1. AND and OR can be calculated with minimum and maximum or with other t-norms and t-conorms. A central inference pattern is generalized modus ponens: from the rule "if x is A, then y is B" and the observation that x matches A to some degree, we infer that y matches B to a corresponding degree. Fuzzy logic therefore represents gradual concepts that classical two-valued logic cannot express directly.

**Fuzzy inference system** is a pipeline of six steps: fuzzification (input numbers are converted into degrees of membership of terms) → rule base → aggregation of the conditions of each rule into one degree of activation α → activation (degree α "cuts" or scales the conclusion of the rule) → accumulation (we combine the conclusions of all rules) → defuzzification (we make one number from the fuzzy result).

**Four classic inference algorithms** differ in the consequent (the "THEN" part) and in how they obtain a crisp output:

| Algorithm | Consequent ("THEN" part) | How to obtain the output |
|---|---|---|
| Mamdani | fuzzy set | clip each consequent fuzzy set at activation level α, aggregate the results, then defuzzify, commonly with the centroid |
| Larsen | fuzzy set | scale each consequent membership function by α, aggregate, then defuzzify |
| Tsukamoto | fuzzy set with a monotonic membership function | solve μ_B(z)=α for each rule, then take the activation-weighted average of the resulting crisp values |
| Sugeno (TSK) | a constant or a function of the inputs | take the activation-weighted average of rule outputs; no fuzzy output set needs defuzzification |

An example from zero-order Sugeno. Rules: "IF the temperature is low, THEN heating 80" and "IF high, THEN heating 10". The 22-degree input is fuzzified: "low" by 0.3, "high" by 0.6. The output is a weighted average: (0.3·80 + 0.6·10) / (0.3+0.6) ≈ 33.3 is the specific heating power. In Mamdani, instead, two sets of conclusions would be cut at the levels 0.3 and 0.6, aggregated into one shape, and its center of gravity would be sought.

**Defuzzification** transforms the final fuzzy set into a single crisp number. The most popular method is the **centroid** (center of gravity): imagine the area under the μ graph cut from cardboard and find its balance point. Other methods include the bisector, which divides the area in half, and the mean of maxima, which averages the points where μ is maximal. For membership degrees 0.2, 0.8, 0.8, and 0.2 at points 1, 2, 3, and 4, both the centroid and the mean of maxima give 2.5.

**Fuzzy neural networks** combine a fuzzy inference structure, whose rules remain interpretable, with learning that tunes membership-function parameters. A classic example is **ANFIS**, a five-layer network that implements a Sugeno model: fuzzification → rule strengths → normalization → weighted rule outputs → summation. Its hybrid learning algorithm estimates consequent parameters by least squares and premise membership-function parameters by backpropagation.

A **Neo-Fuzzy Neuron** is a neuron in which each input passes through its own nonlinear synapse: a set of triangular membership functions forming a partition of unity, each with its own weight. For fixed membership functions, the output is linear in the weights, so fitting the weights with a least-squares loss is a convex problem; a unique minimum additionally requires suitable rank conditions or regularization. **Cascading neo-fuzzy networks** (Bodyanskyi et al.) build a growing architecture from these neurons. Each new neuron receives the original inputs plus the outputs of previous neurons. The neurons are trained sequentially, and already trained weights may be frozen while the new neuron is fitted, including with online least-squares algorithms.


### Reinforcement

**Additional worked example.** If μ_warm(24°)=0.7 and μ_humid(24°)=0.4, then for standard operators AND = min(0.7,0.4)=0.4, OR = max(...)=0.7, NOT warm = 1−0.7=0.3.

**Transfer example.** In fan control, the rule “IF temperature is high AND humidity is high, THEN speed is high” can fire to a partial degree. Mamdani inference produces a fuzzy output set, which is then converted to a crisp number, for example by the centroid method.

**Active recall.** Close the explanation above and answer without peeking: What are the stages of the fuzzy-inference pipeline described above? How does membership degree differ from probability?

**Mini practice — check yourself.** For μ_A(x)=0.2 and μ_B(x)=0.9, find the membership of the intersection, union, and complement of A by the min/max operators.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** μ_A∩B=0.2, μ_A∪B=0.9, μ_¬A=0.8.

</details>

**Exam focus:**

- A membership degree of 0.7 is not a probability: it describes how well an element satisfies a vague concept, not the likelihood of an event. Membership degrees across unrelated fuzzy sets do not have to sum to 1.
- The intersection of a set with its negation is not empty — the law of non-contradiction is violated (show with numbers).
- Mamdani produces a fuzzy set (needs defuzzification), Sugeno gives an immediate number (weighted average).
- Larsen differs from Mamdani only in activation: multiplication instead of cutting.
- Tsukamoto requires monotonic functions in the conclusions — otherwise the equation "μ equals α" has more than one solution.
- A neo-fuzzy neuron's output is linear in its weights, not necessarily in its inputs; with fixed membership functions, least-squares fitting is convex in those weights.

---

<a id="q43"></a>

## 43. Fuzzy clustering: Fuzzy C-Means and Gustafson–Kessel

> **Learning outcome.** After this chapter, you will be able to perform the Fuzzy C-Means step and explain how Gustafson–Kessel adapts the cluster shape. First, try to explain the topic in your own words, and then test yourself with the block at the end.

**The essence of the question.** Conventional clustering works like a strict watchman: each data point gets exactly one "drawer" - either you are in cluster #1 or #2, the third is not given. But life is rarely so black and white. A song can be "a little rock, a little pop" and a bank customer "mostly reliable, but with a touch of risk." Fuzzy clustering is exactly that: instead of a hard yes/no, each point gets degrees of belonging to all clusters at once—say, 70% to one, 25% to the second, and 5% to the third. The only condition: these percentages for one point must add up to exactly 1, i.e. 100%. Plus common sense: no cluster should be empty and none should "eat up" all points completely.

This is learning without a teacher: no one tells you in advance which groups are correct - the algorithm itself looks for structure in the data.

**Fuzzy C-Means (FCM).** The most widely used fuzzy clustering algorithm was introduced by Dunn (1973) and developed further by Bezdek (1981). It chooses cluster centers and memberships to minimize the objective function

```
J_m = Σ Σ u_ik^m · ||x_k − v_i||²
```

It is easy to read: for each "point - cluster" pair, we take the square of the distance from the point to the center of the cluster and multiply it by the membership to the power of m. That is, the algorithm punishes situations when a point is strongly "tied" to a distant center. The smaller the value of J, the better the partitioning.

The parameter m>1 is the **fuzzifier**, usually set to m=2. As m approaches 1, the partition becomes almost crisp and approaches ordinary K-means. As m becomes very large, memberships approach equal values for all clusters, which removes useful structure. Thus, K-means is a limiting case of FCM as m approaches 1.

**How it works in steps:**

1. Choose the number of clusters, the value of m, and a convergence tolerance; initialize the centers or membership matrix.
2. Calculate each center as a weighted average of the points using weights u_ik^m: points with greater membership pull the center more strongly toward them.
3. Update memberships: the closer a point is to one center relative to the others, the greater its membership in that cluster. If a point exactly coincides with a center, assign membership 1 to that cluster and 0 to the others.
4. Stop when the membership matrix changes by less than the tolerance; otherwise, return to step 2.

**A small example.** Consider four points on a line: 1, 2, 6, and 7. We seek two clusters with starting centers 2 and 6. After calculating the memberships, we obtain approximately:

| point | to the left cluster | to the right |
|---|---|---|
| 1 | 0.96 | 0.04 |
| 2 | 1.00 | 0.00 |
| 6 | 0.00 | 1.00 |
| 7 | 0.04 | 0.96 |

Each row gives a total of 1. The point is visible: point 1 is "almost completely" in the left cluster, but has a tiny share and in the right one - there is no hard division. Then the centers are recalculated (they converge at approximately 1.5 and 6.5 — within their groups), and so on in a circle until the changes become negligible.

**Important properties of FCM.** The algorithm converges to a local minimum, not necessarily the global optimum, so its result depends on initialization and it is commonly run several times. The number of clusters must be specified in advance, and quality can be evaluated with fuzzy-cluster validity indices such as the Xie–Beni index, where smaller values indicate compact, well-separated clusters. Standard FCM uses Euclidean distance and therefore favors roughly spherical clusters. It is also sensitive to outliers: a distant point must still distribute a total membership of 1 among the clusters and can shift their centers.

**The Gustafson–Kessel algorithm (1979).** This method addresses standard FCM's preference for spherical clusters. If a real cluster is elongated, Euclidean distance may describe it poorly. Gustafson–Kessel (GK) gives each cluster its own adaptive distance metric. A fuzzy covariance matrix describes the directions and magnitudes of the cluster's spread, and a Mahalanobis-type distance makes separation along a high-variance direction count less than separation across it. The algorithm can therefore model ellipsoidal clusters with different orientations.

There is one technical nuance: if the algorithm could change each metric arbitrarily, it could shrink all distances and make the optimization degenerate. GK therefore constrains the determinant of each cluster's metric matrix, commonly to 1. The cluster's shape and orientation may change, but its metric cannot collapse without limit.

**Comparison in brief:** FCM uses Euclidean distance and therefore favors spherical clusters; GK adapts a metric for each cluster and can model ellipsoidal clusters of arbitrary orientation. GK is more expensive because each iteration estimates covariance/metric matrices and performs matrix operations. Extensions include Gath–Geva clustering, which models clusters with different volumes and densities, and possibilistic clustering, which removes the constraint that each point's memberships sum to 1. An outlier can then have low membership in every cluster, making the method more robust to anomalies.


### Reinforcement

**Additional worked example.** A point with memberships (0.8, 0.2) does not belong exclusively to one cluster. For m=2, its weights in the center updates are 0.8²=0.64 and 0.2²=0.04, so the point contributes much more strongly to the first cluster center.

**Transfer example.** FCM with Euclidean distance favors spherical clusters. Gustafson–Kessel adapts a separate metric matrix for each cluster, so it can represent an elongated cloud as an ellipsoid.

**Active recall.** Close the explanation above and answer without peeking: What does the condition of the sum of memberships for one point mean? How does the m parameter change the fuzziness?

**Mini practice — check yourself.** The point has memberships 0.6 and 0.4. Check the sum and find the weights to update the centers for m=2.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** The sum is 1, so the membership constraint is satisfied. The weights are 0.6²=0.36 and 0.4²=0.16; when calculating a center, each weighted sum is normalized by the total weight of all points in that cluster.

</details>

**Exam focus:**

- "How does FCM differ from K-means?" — memberships are not restricted to 0 or 1; K-means is the limiting case as m approaches 1.
- The membership degrees for each point sum to 1, and no cluster may be empty.
- Parameter m controls the fuzziness; a common choice is m=2.
- Pitfall: FCM only finds the local minimum, not the global minimum.
- Why fix the volume of a GK cluster? To prevent the metric optimization from degenerating.
- Do not confuse them: standard FCM favors spherical clusters; GK can model ellipsoidal clusters.

---

<a id="q44"></a>

## 44. Intelligent distributed information systems, information retrieval, the Semantic Web, and agents

> **Learning outcome.** After this chapter, you will be able to calculate precision/recall, explain inverted index and PageRank, and relate RDF/OWL and agent-based platforms to search. First, try to explain the topic in your own words, and then test yourself with the block at the end.


**The point of the question.** This is about how "smart" programs do not work alone, but in a network: how a search engine finds the right page among billions, how to teach computers to understand the content of pages, not just the coincidence of words, and how small autonomous agent programs negotiate with each other, like employees in an office.

**Intelligent distributed system** is a set of autonomous nodes (services, agents, knowledge repositories), scattered over different machines, which together solve problems using AI methods. Key properties: each node acts independently; nodes "understand" each other through common protocols and dictionaries; the system can be expanded without rework; and the fall of one node does not kill the whole.

**Information Retrieval (IR)** is the process of finding relevant documents or information items in a large collection in response to a user's query. Main models include:

- Boolean: query — logical expression ("cat AND dog NOT hamster"); a document either fits or it doesn't, without ranking.
- Vector: both the document and the request are vectors of word weights, and similarity is the "angle" between them (cosine measure). The weight of a word is calculated according to the TF-IDF principle: a word is important if it occurs frequently in this document, but rarely in the collection as a whole. The word "and", which is in every document, gets zero weight - and rightly so, it says nothing. A practical improvement of this idea is the BM25 formula, which is used by real search engines.

**Search quality** is commonly measured with two complementary metrics. **Precision** is the proportion of retrieved documents that are relevant; **recall** is the proportion of all relevant documents that were retrieved. If the system retrieves 10 documents, 6 of which are relevant, while the collection contains 12 relevant documents in total, precision=6/10=0.6 and recall=6/12=0.5. They are combined by the harmonic mean:

```
F1 = 2PR / (P + R)
```

Here F1≈0.55. The harmonic mean penalizes imbalance: high precision cannot compensate for very low recall, or vice versa.

**The heart of the search engine is the inverted index**: the dictionary "word → list of documents where it is". It's like a subject index at the end of a book: we don't flip through the entire book, but immediately open the right pages.

**Search engine** consists of four parts: a crawler (a robot that goes around the Web by links, respecting robots.txt), an indexer (breaks texts into words, reduces them to the basics and builds an inverted index), a ranker and a query processor (corrects errors, adds synonyms). The most famous ranking idea is PageRank:

```
PR(p) = (1−d)/N + d · Σ_{q→p} PR(q)/outdeg(q)
```

In other words, a page is important if important pages link to it, and each page distributes its authority equally among its outgoing links. This is the **random-surfer model**: the surfer usually follows a link and occasionally, with probability 1−d (commonly 0.15), jumps to a random page. The update is iterated until the PageRank values converge.

**Ontology** is a formal, machine-readable description of a domain: classes of concepts ("Cat is an Animal"), relations, attributes, constraints, and specific instances (a particular cat named Murchyk). Ontologies can help search systems expand queries with synonyms, disambiguate words such as "bank" (a financial institution or the side of a river), build faceted filters, and organize knowledge graphs.

**The Semantic Web** is Tim Berners-Lee's vision of publishing data with machine-readable meaning, so that a computer can recognize "Kyiv" as a city rather than merely a string. Its technology stack includes **Uniform Resource Identifiers (URIs)**; RDF, a data model based on subject–predicate–object **triples** that form a graph; RDFS, a basic vocabulary for classes and properties; OWL, a more expressive ontology language that supports automated reasoning; and SPARQL, a query language for RDF graphs. Semantic Web service approaches such as OWL-S and WSMO describe service capabilities in machine-readable form so that software can discover, compose, and invoke services.

**Intelligent agent** is a program that perceives the environment and acts on it, having four signature properties: autonomy (works without constant commands), reactivity (responds to changes in time), proactivity (itself initiates actions for the sake of the goal) and sociality (communicates with other agents). Architectures are reactive (simple stimulus-reaction rules), deliberative BDI (Beliefs–Desires–Intentions: beliefs — what the agent knows about the world, desires — what it wants, intentions — what it decided to do) and hybrid.

A **multi-agent system** contains multiple agents that coordinate using mechanisms such as the FIPA ACL message language and performatives including INFORM, REQUEST, and PROPOSE. Coordination protocols include Contract Net, in which a manager announces a task, agents submit proposals, and the manager awards the task, as well as auctions and negotiation. FIPA-style platforms provide services such as the Agent Management System (AMS) for agent lifecycle and platform management, the Directory Facilitator (DF) for service discovery, and a Message Transport Service (MTS) for communication. Implementations include JADE (Java), Jason (BDI), and SPADE (Python).


### Reinforcement

**Additional worked example.** The system found 8 documents, 6 relevant; a total of 10 relevant ones. Precision=6/8=0.75, recall=6/10=0.6, F1=2·0.75·0.6/(1.35)≈0.667.

**Transfer example.** The inverted index entry `graph: [d1,d3]` immediately returns candidate documents without scanning the entire collection. An ontology can expand a query for "automobile" with the synonym "car" and the broader class Vehicle, but expansion must be controlled to avoid reducing precision.

**Active recall.** Close the explanation above and answer without looking: Which denominators have precision and recall? What does an inverted index store?

**Mini practice — check yourself.** The search returned 20 results, 5 are relevant, and there are 25 relevant in the collection. Calculate precision and recall.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** Precision=5/20=0.25; recall=5/25=0.20. Both metrics are low, but they answer different questions.

</details>

**Exam focus:**

- Precision vs recall: precision divides relevant retrieved documents by all retrieved documents; recall divides relevant retrieved documents by all relevant documents in the collection.
- Why the logarithm in TF-IDF is to suppress ubiquitous words: a word that is everywhere gets a weight of zero.
- PageRank: It is not the number of links that matters, but the importance of those who link.
- Do not confuse RDF (triples), RDFS (basic class/property vocabulary), and OWL (expressive ontologies with inference); SPARQL is a query language, not an ontology language.
- Remember all four properties of an intelligent agent: autonomy, reactivity, proactivity, and social ability.
- Expand BDI and know the responsibilities of the FIPA platform components AMS, DF, and MTS.

---

<a id="q45"></a>

## 45. Applications: Data Mining, BI, image processing, computer vision, NLP, and decision support

> **Learning outcome.** After this chapter, you will be able to compare a business problem with Data Mining, BI, image processing, computer vision, NLP or DSS and choose the appropriate metric. First, try to explain the topic in your own words, and then test yourself with the block at the end.

**Gist of the question.** This overview asks where the methods are used in practice. Data mining discovers useful patterns in large datasets; business intelligence turns organizational data into reports and decisions; image processing improves or transforms an image; computer vision interprets what an image contains; NLP works with human language; and decision support systems (DSS) help people compare choices.

**Data mining** discovers non-trivial, previously unknown, and practically useful patterns in data. A standard project lifecycle is CRISP-DM, whose six iterative phases are business understanding → data understanding → data preparation → modeling → evaluation → deployment. Main tasks include:

- Classification — predict a category from labeled training examples, using methods such as decision trees, naive Bayes, k-nearest neighbors, or random forests.
- Regression — to predict a number, not a category (price, demand).
- Clustering — grouping without labels (K-means, DBSCAN, fuzzy FCM from question 43).
- Association rules — "customers who buy X often also buy Y." The Apriori algorithm uses the anti-monotonicity property: if an itemset is infrequent, every larger itemset containing it is also infrequent, so those candidates can be pruned.
- Detection of anomalies and forecasting of time series.

**Association-rule example.** There are 100 transactions: bread appears in 40, butter in 25, and both appear together in 20. For the rule "bread → butter," support=20/100=0.2, so the pair occurs in one fifth of all transactions; confidence=20/40=0.5, so half of the transactions containing bread also contain butter; and lift=0.5/0.25=2, so butter is twice as likely when bread is present as it is overall. Typical data-mining applications include credit scoring, customer-churn prediction, market-basket analysis, and fraud detection.

**Business intelligence (BI).** An ETL process extracts data from multiple sources, cleans and transforms it, and loads it into a data warehouse. In a classic **star schema**, a fact table sits at the center and dimension tables surround it. OLAP then provides multidimensional cubes, such as sales by product × region × month. Common operations are slice — select one value ("March only"); roll-up — aggregate upward (cities to countries); drill-down — show finer detail (years to months); and pivot — rotate the axes. Dashboards present KPIs in tools such as Power BI or Tableau, while predictive analytics asks "what will happen?" and prescriptive analytics asks "what should we do?"

**Image processing** transforms an image into another image, often to improve it. Main tools include histogram equalization for contrast, convolutional filters in which a small kernel slides over the image, Gaussian filtering for blur, median filtering for salt-and-pepper noise, and frequency-domain methods based on the Fourier transform. For edge detection, the Canny algorithm smooths the image, computes the intensity gradient, applies non-maximum suppression to produce thin edges, and uses two thresholds: strong edges are retained, while weak edges are retained only when connected to strong ones. Segmentation divides an image into regions; Otsu's method automatically selects a threshold between foreground and background. In mathematical morphology, erosion shrinks foreground objects, dilation expands them, and combinations of the two can remove small noise or fill holes.

**Computer vision** transforms images into an understanding of a scene. Earlier systems used hand-crafted features such as SIFT or HOG followed by a classifier; today convolutional neural networks (CNNs) dominate many tasks and learn useful visual features directly. Convolutional layers detect local patterns, pooling reduces spatial resolution, and later layers produce predictions. Milestones include AlexNet (the 2012 breakthrough), ResNet (residual or skip connections that enabled very deep networks), and Vision Transformers. Tasks include classification ("this is a cat"), object detection ("the cat is here, inside this box," using models such as YOLO or Faster R-CNN), pixel-wise segmentation, face recognition, and OCR. Applications include autonomous vehicles, medical-image analysis, and industrial quality control.

**NLP (natural language processing).** A classic pipeline tokenizes text, stems or lemmatizes words, tags parts of speech, and recognizes named entities such as people, places, and dates. Mathematical models represent text with numeric vectors: earlier approaches used count vectors and TF-IDF, word2vec learned similar vectors for words used in similar contexts, and transformers introduced an attention mechanism:

```
Attention(Q, K, V) = softmax(QK^T / √d_k) V
```

In words, each token attends to other tokens and weights how relevant they are to its representation. This leads to models such as BERT, an encoder model trained in part to predict masked tokens, and GPT, a decoder model trained to predict the next token. NLP tasks include sentiment analysis, machine translation, summarization, and chatbots. In retrieval-augmented generation (RAG), the system first retrieves relevant documents and then generates an answer grounded in them.

**Decision support systems (DSS)** help a person in tasks without a ready-made formula, where there are several conflicting criteria (price vs. quality vs. deadlines):

- The **Analytic Hierarchy Process** (AHP, Saaty) builds a hierarchy "goal → criteria → alternatives." An expert compares elements pairwise on a 1–9 scale, and weights are derived from those comparisons. The consistency ratio (CR) must be checked; CR>0.1 commonly indicates that the judgments should be reviewed.
- TOPSIS: choose the alternative closest to the "ideal" and the farthest from the "anti-ideal".
- Decisions under conditions of uncertainty - when it is not known what state of the world will occur: Wald's criterion (pessimist: we choose the best of the worst outcomes), Savage's criterion (minimize the maximum "regret" - missed gain), Hurwitz (pessimist/optimist compromise), Laplace (all states are equally likely).
- Expert systems — "IF–THEN" rules plus an inference engine and an explanation subsystem; MYCIN is a classic medical example.

**An example of the Wald criterion.** Two projects, the profit depends on the state of the market:

| | Bad market | Good market | Minimum |
|---|---|---|---|
| Project A | 10 | 100 | 10 |
| Project B | 40 | 60 | 40 |

For each project, Wald's criterion takes the worst possible payoff and then chooses the larger of those minima. Project A has minimum 10 and Project B has minimum 40, so the criterion selects B and guarantees at least 40 under the listed scenarios.


### Reinforcement

**Additional worked example.** A filter that removes noise from a scan performs image processing; a model that locates a license plate performs computer vision; OCR then converts the plate image into text. One product can combine several fields sequentially.

**Transfer example.** For the rule "tea → lemon" in 100 transactions, tea appears in 20, lemon in 25, and both appear in 10. Support=0.10, confidence=10/20=0.50, and lift=0.50/0.25=2: lemon is twice as likely in transactions containing tea as it is overall.

**Active recall.** Close the explanation above and answer without peeking: How does classification differ from clustering? How does descriptive BI differ from predictive analytics?

**Mini practice — check yourself.** The company wants: (1) a dashboard of past sales; (2) demand forecast; (3) automatically group customers without labels. Name the approaches.

<details markdown="1">
<summary><strong>Show answer</strong></summary>

**Answer.** (1) BI/OLAP; (2) supervised regression or time-series forecasting; (3) unsupervised clustering. Each task requires a separate quality metric.

</details>

**Exam focus:**

- Classification vs regression: category vs number; both are supervised-learning tasks, whereas clustering is unsupervised.
- Image processing ("image → improved image") is not the same as computer vision ("image → understanding"); this distinction is often tested.
- The confidence of the rule "A → B" is not equal to the confidence of "B → A": the denominators are different.
- CRISP-DM — name all six stages in order.
- OLAP: roll-up aggregates to a higher level; drill-down reveals finer detail.
- AHP: consistency threshold CR < 0.1.
- Wald chooses the best worst-case payoff; Savage minimizes maximum regret.
- BERT — encoder (understanding), GPT — decoder (generation).
