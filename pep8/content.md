
- pep8
- autopep8
- pep8radius
- yapf

---

#PEP8-ifying code

"Before I say what I'm going to say"

---

## What is PEP8?

> This document gives coding conventions for the Python code comprising the
standard library in the main Python distribution. 

---

> One of Guido's key insights is that code is read much more often than it is
> written. The guidelines provided here are intended to improve the readability
> of code and make it consistent across the wide spectrum of Python code.

---

## A Foolish Consistency is the Hobgoblin of Little Minds

> A style guide is about consistency. Consistency with this style guide is
> important. Consistency within a project is more important. Consistency within
> one module or function is most important.

---

TODO: History??

- A standard for writing python code
- Consistent among all codebases

...or at least that's the idea.

---

## What I'm going to talk about

- How to test code for PEP8
- How to fix code for PEP8 (and talk about two different implementations/approaches)
- Why you probably shouldn't do this

---

## Why we fight

TODO: Bad code examples (hidden bugs)

---

## Where's are the problems

pep8 by @jcrocholl (now at PyCQA)

Given python code, return all the bad things.

---

### Usage

```sh
$ pep8 myscript.py
$ pep8 myproj
```

TODO: Examples, can do these live!

side note: landscape/codeclimate extend this idea to catch "code smells",
lint-type errors.

---

### Describe pep8 algorithms

TODO

---

### Running pep8 on every commit

Some (open-source) projects run pep8 as a test on every commit. i.e. if the
code doesn't pass pep8 the build fails.

Not unheard of heard of in production: "Supercharge your python developers"
https://www.jeffknupp.com/blog/2013/11/15/supercharge-your-python-developers/

https://www.dominicrodger.com/build-breaking.html

Note: pep8 exits with non-zero status if there are infractions.

---

## Fix the problem

autopep8 by @hhatto.
(special mention to Stephen Myint for tireless work on both this and pep8, and other similar packages e.g. docformatter.)

---

### Idea

loop through hits and fix them:

---

### Usage

```sh
$ autopep8 myscript.py --diff | cdiff
$ autopep8 myscript.py --in-place

$ autopep8 myproj --recursive --in-place
```

---

### Example:

```py
code = open("myscript.py").readlines()
```

Now compare to the list of pep8 failures (note: lines are 1-index).

Write "tricks" to fix up each failure (TODO example).

---

## A different approach

yapf by @google (Bill Wendling and Eli Bendersky)

---

### The problem

Some sections of code have multiple ways to fix, which is best?

```py
f(argument1="something quite long", argument2="something even longer", argument3="over 80 chars)
```

Could become (both pass pep8):

```py
f(argument1="something quite long", argument2="something even longer",
  argument3="over 80 chars")

f(argument1="something quite long",
  argument2="something even longer",
  argument3="over 80 chars")
```

which is best?

*Note: autopep8 will fix it "just enough".*

---

### The solution

Avoid arguments/discussion, just accept yapf (will give you a canonical answer).

```sh
$ yapf myscript.py --diff | cdiff
$ yapf myproj --recursive --in-place
```

Note: also accepts local style if you want to deviate from PEP8,
e.g. `indent_width = 2` (you heathen).

*Similar tools exist for go (gofmt) and C++ (clang-format by Daniel Jasper).*

---

### How it works

Take the AST, give each formatting option a "badness" score.
Minimize the total badness.

This becomes a graph theoretic problem (use Dijikstra's; TODO check).

---

#### Example

```py
f(argument1="something quite long",[NEWLINE|SPACE]argument2="something even longer",[NEWLINE|SPACE]argument3="over 80 chars)

# (SPACE, SPACE) 
f(argument1="something quite long", argument2="something even longer", argument3="over 80 chars)

# (SPACE, NEWLINE)
f(argument1="something quite long", argument2="something even longer",
  argument3="over 80 chars")

# (NEWLINE, NEWLINE) 
f(argument1="something quite long",
  argument2="something even longer",
  argument3="over 80 chars")
```

TODO what is their badness??

The latter wins!

TODO add figure for this graph.

---

## Doing this, probably a bad idea...

---

### Why

- Diffs are massive
- May break code (if there is a bug in the reformatter)
- All unmerged work needs to resolve merge conflicts (oh, they'll be conflicts!)
- Difficult to explain/value spending time on this...

> Because the code in question predates the introduction of the guideline and
> there is no other reason to be modifying that code.

PEP8.

---

## But we want to make code better...

pep8radius (by me!, idea by @y-p for pandas development)

PEP8 fix just the code which you've been working on (since the last commit /
merge-base of a branch).

---

## Usage

```sh
$ pep8radius myscript.py --diff | cdiff
$ pep8radius myproj --in-place

$ pep8radius myproj --inplace --yapf
```

Gets the lines which have changed, passes them to autopep8 or yapf.

---

TODO simplez meerkat image.

---

Note: there are also plugins now which do this within your text editor,
either on blocks of code or running pep8radius via a command.


---

## And another thing

There's also docformatter, which fixes docstrings for PEP257.

(You can even pass --docformatter and options to pep8radius.)
