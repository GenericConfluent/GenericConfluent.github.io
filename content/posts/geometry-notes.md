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

**Def:** Given two lines $l_1, l_2$ if they intersect then they are **non-parallel** written $l_1 \not \parallel l_2$, if they intersect then they are **parallel** written $l_1 \parallel l_2$. (If they only intersect at one point then they are **distinct** lines).

**Def:** A **line segment** is written $\overline{AB}$ or $AB$ if we don't care about direction, a **ray** may be created by extending that segment indefinetly in either direction written $\overrightarrow{AB}$ or $\overleftarrow{AB}$, and a **line** may be written $\overleftrightarrow{AB}$.

Course Axioms:
1. There is a unique line through two distinct points.
2. A full rotation measures $360 \degree$, the angle along a line is $180 \degree$, and a right angle is $90 \degree$.
3. A point and distance define a unique circle.
4. (Triangle Inequality) Any side of a triangle is less than the sum of the remaining two. (Alternatively: the shortest distance between two points is a line)
5. (The fifth) We use both the parallel postulate and playfairs axiom. Parallel Postulate: For two lines $l, m$ and a transversal $t$, with 
6. (SAS) $AB \equiv DE, \angle B \equiv \angle D, BC \equiv EF \iff \triangle ABC \equiv \triangle DEF$

## Directed Distance

1. $\overline{AB} = -\overline{BC}$
2. $\overline{AB} + \overline{BC} = \overline{AC}$
3. $\overline{AB} = \overline{AC} \iff A = C$

## Sameness
There are multiple ways things can be the same in geometry and so we have multiple relevant equivalence relations to indicate the sense in which two objects are the same.

**Def:** When objects differ only by positioning in space (same up to translation, rotation, and reflection) they are **congruent** written $a \equiv b$.

**Def:** We can loosen the idea of congruence even more by allowing scaling and then we get **similarity**. Two **similar** are written $a \sim b$.

**Thm (ASA):** $\angle B \equiv \angle E, BC \equiv EF, \angle C \equiv \angle F \iff \triangle ABC \equiv \triangle DEF$

**Thm (AAS)** $\angle A \equiv \angle D, \angle B \equiv \angle E, BC \equiv EF \iff \triangle ABC \equiv \triangle DEF$

**Thm (SSS)** $AB \equiv DE, BC \equiv EF, AC \equiv DF \iff \triangle ABC \equiv \triangle DEF$

## Misc Ideas
**Def:** Given a line segment $AB$ there exists a point $M \in AB$ s.t. $AM \equiv MB$. $M$ is called the **midpoint**.

## Lines

## Triangles
**Def:** A triangle is **isosceles** if any two sides are equal.

**Thm (Iso Tri):** For iso $\triangle ABC$, $AB \equiv AC \iff \angle B \equiv \angle C$

**Def:** A triangle is **equilateral** if all sides are of equal length.

**Thm (Eq Tri):** For eq $\triangle ABC$, $AB \equiv BC \equiv AC \iff \angle A \equiv \angle B \equiv \angle C = 60 \degree$

**Def:** A triangle with any angle $90 \degree$ is called a *right angle triangle*.

**Def:** For a right angled triangle the side opposite to the right angle is called the **hypoteneus**.

**Thm:** The hypoteneus is the longest side of a right angle triangle.

**Lem:** $\triangle ABC \equiv \triangle DEF$ iff ($AB \equiv DE, BC \equiv EF, CA \equiv FD$ and $\angle A \equiv \angle D, \angle B \equiv \angle E, \angle C \equiv \angle F$)

The corresponding angles and side lengths need to be the same for the triangles to be the same.

**Thm (Angle Side Inequality):** For $\triangle ABC$, $\angle B > \angle C \iff AC > AB$

**Thm (Open Jaw Inequality):** For $\triangle ABC$ and $\triangle DEF$ where $AB \equiv DE$ and $BC \equiv EF$ then $\angle B > \angle E \iff AC > DF$


**Def:** Given $\angle ABC$ and point $S$, we call $BS$ the **angle bisector** for $\angle ABC$ iff $\angle ABS \equiv \angle CBS$.

**Thm (Characterization of Angle Bisector):**

**Def:** Let $V$ be some vertex for $\triangle ABC$ and $M$ the midpoint of the opposite side. Then the line $VM$ is called a **median** of $\triangle ABC$.

**Thm (Iso Implies):** For iso $\triangle ABC$ with $AB \equiv AC$ then the median, angle bisector, and perpendicular bisector all from $A$ are the same line.

**Thm (Implies Iso)** For $\triangle ABC$ if any *two* of the median, angle bisector, or perpendicular bisector are the same, then $\triangle ABC$ is iso.

## Quadrilaterals

## Thale's

**Thm (Thale's):** Let $A,B,C \in C(O, r)$, then $m(\overgroup{AC}) = 2 \angle ABC$

**Coro:**  Let $AB$ and $CD$ be two chords intersecting at $X$. Then 

## Ceva and Menelaus



