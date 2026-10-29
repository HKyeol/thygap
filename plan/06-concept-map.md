# 6. The concept map

## 6.1 Purpose

Every diagnosis walks a map of mathematics. When a student misses a step, the app does not stop at "you do not know linear equations": it walks down to what that step stands on (combining like terms, the distributive property, operations with integers, fractions) until it finds the exact place where the student's knowledge breaks. A gap can sit far below the problem, such as a fraction slip under a high-school algebra step, so the whole of school and early university mathematics forms **one connected map**.

The same map drives the map check. A composed problem mixes a main concept with support concepts and aims at a named knowledge element, often a well-known slip. That is only possible when the map knows, for every concept, its parts, its classic mistakes and which other concepts it is really used with.

The map is plain data, written once, checked by code and loaded unchanged. It is **world mathematics**, not one country's syllabus: the bands name school stages, and the topics are the ones taught almost everywhere.

## 6.2 What a node is

The map is a list of **ConceptNode** entries. A node is one of two kinds:

| kind | meaning | examples of the idea |
|---|---|---|
| concept | one idea a short problem can test on its own. Not one per textbook chapter, not one per exercise type | fractions with different denominators; the Pythagorean theorem; the matrix product |
| skill | an ability used across many topics that is not a topic of its own. A skill has no elements and is never probed directly | translating words into an expression |

Skills are few. A skill's id starts with `skill-`.

### Fields of a ConceptNode

| field | meaning |
|---|---|
| id | lower-case English words joined by single hyphens that name the idea (`linear-equations`); never a grade, a course or a book number. Unique across the whole map |
| name | the usual Korean textbook name and a short English name (at most 40 characters). Both unique across the whole map. The Korean name contains Hangul; the English name contains none |
| kind | concept or skill |
| band | the school stage where the idea is usually first taught (6.3) |
| strand | the area of mathematics it belongs to (6.3); always the strand of its file |
| description | one or two English sentences (at most 300 characters) saying what a student can do, with one small example with numbers, in plain-text math (`3/4`, `x^2`, `sqrt(2)`), no LaTeX |
| prerequisites | the concepts a student must know directly before this one (6.6) |
| combines with | concepts used together with this one in problems although neither builds on the other (6.5); concepts only |
| elements | the parts of the idea, each probe-able alone (6.4); concepts only |
| figure | how much a problem on this concept needs a figure: figure needed (hard to state without one), figure helpful (such as a fraction bar or a number line) or figure not needed. Only concepts where a figure is needed or helpful get figure problems |

## 6.3 Bands and strands

**Bands** order the map by school stage, from the lowest up:

| bands | stage |
|---|---|
| E1 – E6 | elementary school, years 1 to 6 |
| M1 – M3 | middle school, years 1 to 3 |
| H1 – H3 | high school, years 1 to 3 |
| U1 – U4 | university, years 1 to 4 |

The app aims at students up to high school year 1 (**target band H1**). The target decides how much detail the model sees in the catalog (6.9) and which concepts a map check may ask before the student has met them (5.1, 11.10). The map itself reaches from E1 to U4.

**Strands**: number, algebra, function, geometry, calculus, prob-stat, linear-algebra, discrete. Every strand has nodes.

## 6.4 Elements

A concept is not one yes/no fact. Each concept has **2 to 6 Elements**: the parts of the idea that a short probe can ask about alone. An Element has an id (unique within its concept, same id format as nodes), a type, a short English name, a short Korean name and a one-sentence English description in plain-text math. The elements of one concept never repeat each other.

| type | the part of the idea it names |
|---|---|
| definition | what it is |
| condition | when it applies, and when it does not |
| procedure | how it is done |
| representation | how it is written or drawn (notation, graphs, diagrams) |
| interpretation | what a result means |
| pitfall | one well-known wrong belief students hold about the idea |

### Why pitfalls matter

A student can pass a problem by knowing only the big idea. The app's promise is to find the exact element that is missing, and the most telling elements are the classic slips. Therefore:

- **Every well-known slip of a concept is its own pitfall element.** Each concept has at least one; an idea with two classic slips has two.
- A pitfall's description states the wrong belief in one sentence, so that a probe or a composed problem can change one detail and see whether the student really holds the idea. Examples of the intended level:
  - fractions: a larger denominator means a larger fraction;
  - matrices: a product equal to the zero matrix means one factor is the zero matrix; and AB equals BA;
  - determinants: the determinant of a sum is the sum of the determinants;
  - series: terms that tend to 0 make the series converge;
  - implications: "if p then q" is false whenever p is false;
  - p-values: a p-value is the probability that the null hypothesis is true.
- Pitfall elements are what the map check aims at first, and a wrong answer that equals a known slip's answer is recorded against that pitfall.

A step of a solution, a probe and a teach card can point at a concept as a whole or at one element of it, written `concept-id#element-id`.

## 6.5 "Combines with" and the learned co-use counts

Many problems join two concepts that do not build on each other: a matrix product with a linear system, a quadratic function with the discriminant, a rectangle's area with a linear equation, a fraction of an amount with division, an inverse matrix with a system of equations, a confidence interval with a hypothesis test. Prerequisites alone cannot describe such pairs, and a composed problem that only stacks a concept on its own prerequisite is too easy to see through.

The **combines with** list records these pairs in the data:

- 0 to 4 entries per concept; most concepts list one or two.
- Each entry is a concept (never a skill), never the node itself, never one of its prerequisites (direct or not) and never a concept that builds on it (direct or not).
- A pair is written once, on one side; the map reads it in both directions.
- A pair across two files is written in the later file (6.7).

**Learned co-use counts** complement the written pairs. Each time an analysis of a real problem is verified, its main concept and its support concepts are added to a shared count (main → support → times seen together), shared by all users. A draft, failed or cancelled analysis adds nothing; skills and unknown ids are skipped. The composer may use as a support any concept that is a prerequisite of the main, a written "combines with" partner, or a concept counted together with the main at least once; the most often counted ones are preferred.

## 6.6 Prerequisite rules

The repair walk goes down prerequisite links, so the links must be sound:

1. **Direct only.** A node lists 0 to 4 prerequisites (more than 4 is a warning), and none may follow from another listed one. If A needs B and B needs C, A lists B, not C.
2. **Never a later band.** A prerequisite's band is never later than the node's band.
3. **No self link, no cycle.**
4. **Every node reaches a root.** A root is a node with no prerequisite. Each file has only a few roots; the roots of the whole map are the first ideas of counting.
5. **Grow out of what came before.** A middle-school idea that grows out of an elementary idea lists it (integers build on whole-number operations and fractions; middle-school angles on elementary angles). University files start from school concepts: linear algebra from school matrices, systems of linear equations and vectors; discrete mathematics from sets and conditions, prime factors, sequences and induction, permutations and combinations; advanced calculus from limits, the derivative, the definite integral and vectors; advanced probability from conditional probability, random variables, the binomial and normal distributions and sampling.
6. **Never written twice.** An idea an earlier file already has is a prerequisite in a later file, never a second node.

## 6.7 The twelve data files

The map is written as twelve files, one strand part each, in this fixed order. Links only go **backward across files**: a node's prerequisites and "combines with" entries are in its own file or an earlier one, never a later one. Ids and names are unique across all files.

| # | file | strand | bands | concepts (roughly) | coverage |
|---|---|---|---|---|---|
| 1 | number-elementary | number | E1–E6 | 30–40 | counting, whole numbers and place value, addition and subtraction; multiplication and division, number patterns, the missing number in a box; fractions as parts of a whole, decimals, comparing and adding them; factors and multiples, fractions with different denominators, multiplying fractions and decimals; dividing fractions and decimals, ratio and rate |
| 2 | geometry-elementary | geometry | E1–E6 | 25–35 | flat and solid shapes, comparing and measuring length, telling time, units of measure; angles, kinds of triangles and quadrilaterals, perimeter, area of rectangles and other polygons; symmetry and congruent figures, positions on a grid, volume of boxes, circumference and area of a circle; reading tables, pictographs, bar, line and circle graphs |
| 3 | number | number | M1–H2 | 25–35 | integers and rational numbers, powers, prime factors, greatest common divisor and least common multiple, order of operations, percent; repeating decimals, rounding; square roots, real numbers, calculations with radicals; sets, propositions and conditions (converse, contrapositive, necessary and sufficient); rational exponents and logarithms as numbers |
| 4 | algebra | algebra | M1–H2 | 45–60 | expressions, linear equations and word problems; monomials and polynomials, linear inequalities, systems of linear equations; multiplying and factoring polynomials, quadratic equations; polynomial division, the remainder theorem, complex numbers, the discriminant, roots and coefficients, higher-degree equations, systems of inequalities, matrices with sum, scalar multiple and product; arithmetic and geometric sequences, sums and sigma notation, induction |
| 5 | function | function | M1–H3 | 30–40 | the coordinate plane, direct and inverse proportion; linear functions and their graphs, a linear equation as a line; quadratic functions and their graphs; functions, composite and inverse functions, rational and irrational functions, quadratic functions with equations and inequalities; exponential, logarithmic and trigonometric functions and their graphs |
| 6 | geometry | geometry | M1–H3 | 45–60 | basic figures, angles, parallel lines, constructions, congruent triangles, polygons, circles and sectors, solids, surface area and volume; properties of triangles and quadrilaterals, similarity, the Pythagorean theorem; trigonometric ratios, circle theorems; coordinate geometry (distance, dividing a segment, equations of lines and circles, moving figures); laws of sines and cosines; conic sections, vectors, space geometry |
| 7 | calculus | calculus | H2–H3 | 35–50 | limits and continuity, the derivative, derivatives of polynomials, tangent lines, increase and decrease, maxima and minima, antiderivatives, definite integrals, areas; limits of sequences, series, derivatives of exponential, logarithmic and trigonometric functions, product, quotient and chain rules, substitution and integration by parts, areas and volumes |
| 8 | prob-stat | prob-stat | M1–H2 | 25–35 | frequency tables, histograms, relative frequency; counting cases, probability of an event; mean, median, mode, variance, scatter plots; permutations and combinations; the binomial theorem, conditional probability, independence, random variables, binomial and normal distributions, sampling and estimation |
| 9 | linear-algebra | linear-algebra | U1–U2 | 30–40 | systems as matrices, row reduction and echelon form, rank, determinants and their properties, the inverse matrix, vectors in n dimensions; vector spaces and subspaces, span, linear independence, basis and dimension, linear maps and their matrices, kernel and image, change of basis, eigenvalues and eigenvectors, diagonalisation, inner products, orthogonality and projection, Gram-Schmidt, symmetric matrices |
| 10 | discrete | discrete | U1–U4 | 35–45 | truth tables and logical equivalence, quantifiers and their negation, direct proof, contradiction and contrapositive, strong induction, power sets and Cartesian products, relations and equivalence relations, injective, surjective and bijective functions, inclusion-exclusion, pigeonhole, recurrences; divisibility and the Euclidean algorithm, congruences, primes and unique factorisation, graphs, paths and cycles, Euler and Hamilton paths, trees and spanning trees, generating functions; groups, subgroups, cyclic and permutation groups, homomorphisms, rings and fields; quotient groups and isomorphism theorems, polynomial rings, field extensions |
| 11 | calculus-advanced | calculus | U1–U4 | 35–45 | convergence tests, absolute and conditional convergence, power series and radius of convergence, Taylor series, improper integrals, partial derivatives, gradient and directional derivatives, multivariable chain rule, extrema in two variables, multiple integrals, change of variables and the Jacobian, first-order ODEs; vector fields, line integrals, Green's theorem, surface integrals, divergence and Stokes' theorems, second-order linear ODEs; epsilon-delta limits, completeness of the reals, uniform convergence; analytic functions and Cauchy-Riemann, contour integrals and Cauchy's theorem, Laurent series and residues |
| 12 | prob-stat-advanced | prob-stat | U1–U3 | 30–40 | geometric and Poisson distributions, densities (uniform, exponential), cumulative distribution functions, rules for expectation and variance, joint and marginal distributions, covariance and correlation, independence of random variables; law of large numbers, central limit theorem, estimators and bias, maximum likelihood, confidence intervals, t and chi-square distributions, hypothesis tests, p-values, errors of the first and second kind; simple linear regression and least squares, Bayes' rule in inference, prior and posterior |

The concept counts are rough target ranges, not quotas. Every band in a file's range is covered by concepts of that file.

Each later file will be written with the earlier files at hand, so every link to an earlier file names an id that really exists there.

**The small test map.** A separate small map of under twenty concepts, around a middle-school linear-equation problem, will be written during the event for the tests and the offline mock mode. It must be a subset of the full map and follow the same rules, so the worked test problem and the mock mode also run on the real map.

## 6.8 Module boundaries

| module | responsibility | depends on |
|---|---|---|
| concept graph | Answers every question the other modules ask about a map (prerequisites near and far, a stable order from the roots up, "combines with" partners, which concepts of a step a repair walk may probe), checks a map against 6.10 and builds the model's catalog (6.9). Works on any map: the full map, the small test map, or a map built inside a test. | the shared types and constants |
| concept map data | Joins the twelve files, in file order, into the one map the app uses everywhere, stamped with its version. | the twelve data files, the shared types and constants |

The graph tolerates malformed input without crashing; the validator reports every problem.

The loader never adds, drops, merges or changes a node.

## 6.9 The catalog for the model

The model that splits a problem into steps must name concepts and elements by id. It gets a catalog: one line per node, in topological order. A full line holds the id, the English name, the band and the description; a short line holds the id, the English name and the band. A concept's line ends with its elements (id and type), so the model can name a step's element as `concept-id#element-id`.

- A small map gets only full lines.
- On the full map, the concepts the problem text mentions, and their prerequisites, are given in full, as is every node at or below the target band; the rest by name.

## 6.10 Checks the loader runs

The check will run on every data file during the build and on the joined map. Each problem is one issue naming its rule and its node; an error stops the build, a warning is shown.

| check | refuses |
|---|---|
| unique id, id format | a repeated node id; an id that is not lower-case words joined by single hyphens |
| known band, strand, kind | a band or strand the map does not declare; a kind other than concept or skill |
| description | a description that is empty, too long, or has formatting markup |
| names | a Korean name without Hangul, an English name with Hangul or over 40 characters; a name used twice anywhere in the map |
| skills | a skill with elements or "combines with" entries |
| element count and pitfall | a concept with fewer than 2 or more than 6 elements, or with no pitfall |
| element fields | a repeated or badly formed element id, an unknown element type, an empty description, a Korean element name without Hangul |
| prerequisites | a node listing itself; a prerequisite that is not a node; a prerequisite from a later band; a prerequisite from a later file; a prerequisite implied by another listed one |
| combines with | an entry that is not a concept of the map, is the node itself, or is related to it by prerequisites in either direction; more than 4 entries; an entry from a later file |
| acyclic, roots | a node in a cycle or depending on one; a node that does not reach a root |
| warnings | more than 4 prerequisites |
| coverage | an empty file; a band in a file's range with no concept in that file; a strand with no nodes |

On the joined map, the loader checks that the map is acyclic, every concept reaches a root, and no prerequisite link is redundant.

## 6.11 Acceptance criteria

1. The joined map must have every node of the twelve files exactly once, in file order, reach from an E1 concept to U4 concepts, have nodes in every strand, and the validator must find no error in it.
2. A map in which a node lists a prerequisite from a later band, lists itself, or sits in a cycle must be refused, and the issue must name the rule and the node.
3. Every concept must have 2 to 6 elements, at least one of them a pitfall whose description states a wrong belief; a concept missing a pitfall must be refused.
4. A "combines with" pair written on one side must be reported as a partner of both concepts, and an entry that is a prerequisite or descendant of the concept must be refused.
5. Walking down from any node by prerequisites must end at a root, and the repair walk must never receive a skill or an unknown id as a probe target.
6. For a problem whose words name a concept, the catalog must give that concept and all its prerequisites a full line even on the full map, and every concept's line must list its elements.
