+++
title = "UML Diagram"
author = ["zhi"]
date = 2026-06-29T00:00:00-04:00
tags = ["uml"]
categories = ["tutorial"]
type = "posts"
draft = false
weight = 1002
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [UML Diagrams in Org-Mode](#uml-diagrams-in-org-mode)
- [Class Diagram](#class-diagram)
    - [Inheritance](#inheritance)
    - [Association](#association)
    - [Aggregation](#aggregation)
    - [Composition](#composition)

</div>
<!--endtoc-->

Unified Modeling Language (UML) diagram is a standardized visual
diagram used to design and document the structure and behavior
of complex software systems.
Here is [YouTube video](https://www.youtube.com/watch?v=WnMQ8HlmeXc) that talks about the diagram usage.
There are many different types of diagram, and here I
keep a record of them for my personal usage.


## UML Diagrams in Org-Mode {#uml-diagrams-in-org-mode}

One of the ways to create a UML diagram in Org-Mode
is to use [PlantUML](https://plantuml.com/), a simple language used for writing
UML diagrams.

To use PlantUML, first install the language.
On Fedora, do

```bash
sudo dnf install plantuml
```

Now set up the emacs config file to set the path to the
plantuml.jar file. You can find where that is via

```bash
rpm -ql plantuml | grep -i jar
```

Now in the emacs config file, put

```elisp
;; Set the path. Put whatever the previous command gives
(setq org-plantuml-jar-path "/usr/share/java/plantuml.jar")

;; Loads the language plantuml language
(org-babel-do-load-languages
 'org-babel-load-languages
 '((plantuml . t)))
```

Now we can run plantuml code in the org src code block.

```text
#+begin_src plantuml :file class_diagram.png
class User {
- String username
- String password
+ login()
+ logout()
}
#+end_src
```

Now we can evalaute the src block via `C-c C-c`.

---


## Class Diagram {#class-diagram}

A UML diagram gives you a visual relation between different
classes. See this [YouTube video](https://www.youtube.com/watch?v=6XrL5jXmTwM) for more detail.

A basic class object in the class diagram looks like:

{{< figure src="class_diagram.png" >}}

Each class have different attributes, 2nd row,
and different methods, 3rd row, which ends with `()`.
Now both attribute or method can have different levels
of _visibility_.

1.  `-` private
2.  `#` protected
3.  `+` public
4.  `~`  package/default

Now all we need to do is to connect different classes
together to create our diagram. There are different types
of relation between different classes.
Let's describe the common types:

| Relationship | Arrow Type        | Description                                   |
|--------------|-------------------|-----------------------------------------------|
| Inheritance  | Hollow Triangle   | One class is the subclass of another          |
| Association  | Straight Line     | Weakly related with each other                |
| Aggregation  | Hollow Diamond    | The part can exist independently of the whole |
| Composition  | Fulfilled Diamond | The part CANNOT exist withou the whole        |

---


### Inheritance {#inheritance}

If a class is a subclass of another class -- inheritance.
Then we should connect the two via a _hollow triangular arrow_
pointing from the child class to the parent class.
Here is an example

{{< figure src="inheritance.png" >}}


### Association {#association}

If a class is only weakly associated with another class,
we can simply draw a _line_ between them. This can be used
to indicate that class A uses class B is some way.
This is typically when class B  is not directly stored
in class A, but class A uses class B as some argument
in one of the methods.
To expand upon the previous example, we can say _Customer_ buys
_Product_.

{{< figure src="association.png" >}}


### Aggregation {#aggregation}

Aggregation is when a class (A) can exist independently of the whole
(B). This is denoted by a hollow diamond shape
where the diamond is connected from class A to class B.
This likely when class B is stored as part of the attribute,
and it is created as an argument during initialization.
So that class B was already created.
We can again expand upon the previous example, where we
introduce a new class called _ShoppingCart_.
Now _ShoppingCart_ can contain 0 to many _Product_,
but _Product_ itself can live perfectly fine without _ShoppingCart_.
This is called an aggregation.

{{< figure src="aggregation.png" >}}


### Composition {#composition}

Lastly, we have composition. This relation is similar to aggregation.
However, in this case the part (A) CANNOT exist without the whole (B).
This is denoted by a solid diamond shape.
This is when class A directly calls class B and stores it as
an attribute or somehow uses it inside class A.
From the previous example, consider an _Order_ object.
This depends on the _Customer_. But without _Customer_ there won't be any _Order_.
This is then a composition.

{{< figure src="aggregation.png" >}}

---
