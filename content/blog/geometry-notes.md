+++
date = '2025-03-22T10:12:09-06:00'
title = 'Geometry Notes'
description = 'Definitions, notation, and theorems for introductory euclidean geometry.'
tags = ['geometry', 'math']
+++

This is purposefully does not include proofs. I made this as a part of my study for my second geometry midterm and final, while proofs are particularly helpful for remembering results I don't have the time time to typeset them and they would likely distract from the key information.

## Axioms
The course I'm doing decided to go with an edited version of the axioms/postulates, and perhaps glossed over
defining some stuff. And to be clear these notes are on euclidean geometry.


**Def:** Given two lines $l_1, l_2$ if they intersect then they are **non-parallel** written $l_1 parallel.not l_2$, if they don't intersect then they are **parallel** written $l_1 parallel l_2$. (If they only intersect at one point then they are **distinct** lines).

**Def:** A **line segment** is written $overline(A B)$ or $A B$ if we don't care about direction, a **ray** may be created by extending that segment indefinitely in either direction written $arrow(A B)$ or $arrow.l(A B)$, and a **line** may be written $arrows.lr(A B)$.

Course Axioms:
1. There is a unique line through two distinct points.
2. A full rotation measures $360°$, the angle along a line is $180°$, and a right angle is $90°$.
3. A point and distance define a unique circle.
4. (Triangle Inequality) Any side of a triangle is less than the sum of the remaining two. (Alternatively: the shortest distance between two points is a line)
5. (The fifth) We use both the parallel postulate and Playfair's axiom. Parallel Postulate: For two lines $l, m$ and a transversal $t$, with
6. (SAS) $A B equiv D E, angle B equiv angle D, B C equiv E F arrow.l.r.double triangle A B C equiv triangle D E F$

## Directed Distance

1. $overline(A B) = -overline(B C)$
2. $overline(A B) + overline(B C) = overline(A C)$
3. $overline(A B) = overline(A C) arrow.l.r.double A = C$

## Sameness
There are multiple ways things can be the same in geometry and so we have multiple relevant equivalence relations to indicate the sense in which two objects are the same.

**Def:** When objects differ only by positioning in space (same up to translation, rotation, and reflection) they are **congruent** written $a equiv b$.

**Def:** We can loosen the idea of congruence even more by allowing scaling and then we get **similarity**. Two objects are **similar** written $a tilde b$.

**Thm (ASA):** $angle B equiv angle E, B C equiv E F, angle C equiv angle F arrow.l.r.double triangle A B C equiv triangle D E F$

**Thm (AAS):** $angle A equiv angle D, angle B equiv angle E, B C equiv E F arrow.l.r.double triangle A B C equiv triangle D E F$

**Thm (SSS):** $A B equiv D E, B C equiv E F, A C equiv D F arrow.l.r.double triangle A B C equiv triangle D E F$

## Misc Ideas
**Def:** Given a line segment $A B$ there exists a point $M in A B$ s.t. $A M equiv M B$. $M$ is called the **midpoint**.

## Lines

## Triangles
**Def:** A triangle is **isosceles** if any two sides are equal.

**Thm (Iso Tri):** For iso $triangle A B C$, $A B equiv A C arrow.l.r.double angle B equiv angle C$

**Def:** A triangle is **equilateral** if all sides are of equal length.

**Thm (Eq Tri):** For eq $triangle A B C$, $A B equiv B C equiv A C arrow.l.r.double angle A equiv angle B equiv angle C = 60°$

**Def:** A triangle with any angle $90°$ is called a **right angle triangle**.

**Def:** For a right angled triangle the side opposite to the right angle is called the **hypotenuse**.

**Thm:** The hypotenuse is the longest side of a right angle triangle.

**Lem:** $triangle A B C equiv triangle D E F$ iff ($A B equiv D E, B C equiv E F, C A equiv F D$ and $angle A equiv angle D, angle B equiv angle E, angle C equiv angle F$)

The corresponding angles and side lengths need to be the same for the triangles to be the same.

**Thm (Angle Side Inequality):** For $triangle A B C$, $angle B > angle C arrow.l.r.double A C > A B$

**Thm (Open Jaw Inequality):** For $triangle A B C$ and $triangle D E F$ where $A B equiv D E$ and $B C equiv E F$ then $angle B > angle E arrow.l.r.double A C > D F$

**Def:** Given $angle A B C$ and point $S$, we call $B S$ the **angle bisector** for $angle A B C$ iff $angle A B S equiv angle C B S$.

**Thm (Characterization of Angle Bisector):**

**Def:** Let $V$ be some vertex for $triangle A B C$ and $M$ the midpoint of the opposite side. Then the line $V M$ is called a **median** of $triangle A B C$.

**Thm (Iso Implies):** For iso $triangle A B C$ with $A B equiv A C$ then the median, angle bisector, and perpendicular bisector all from $A$ are the same line.

**Thm (Implies Iso):** For $triangle A B C$ if any **two** of the median, angle bisector, or perpendicular bisector are the same, then $triangle A B C$ is iso.

## Quadrilaterals

## Thales'

**Thm (Thales'):** Let $A, B, C in C(O, r)$, then $m(overline(A C)) = 2 angle A B C$

**Coro:** Let $A B$ and $C D$ be two chords intersecting at $X$. Then

## Ceva and Menelaus
