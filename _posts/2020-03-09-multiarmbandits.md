# Multi-Arm Bandits: a potential alternative to A/B tests

![](/images/multiarmbandit_1 "Cartoon Octopus")

Imagine this scenario: you run a website with rare but valuable conversions, and you want to experiment with variations of a landing page in order to maximize your conversion rate. The go-to methodology for such an optimization is A/B testing.

In its most straightforward implementation, a single A/B test consists of essentially one phase of exploration, where you divvy up traffic between the control and test versions of the landing page, and a follow-up period of exploitation, where you direct all traffic to the best performing variation from phase one.
The downside of this methodology is that during the testing phase, some proportion of your visitors will necessarily be going to the worse performing variation. The more valuable your conversions are, the more you tend to lose out during this phase.

Some degree of loss during the exploration phase cannot be avoided: if we do not test at least a few potential variations, it would not be possible to know how they rank against each other. Nevertheless, while we cannot take this potential loss down to zero, can we reduce it while retaining the ability to infer statistically significant results?

This is where Multi-arm Bandit policies enter the picture.

### What are MABs?

MABs are a class of problems where you must make choices over a space of possibilities (landing page design, medical drug testing, advertising campaigns) where each choice has a stochastic reward distribution and where the choice of an inferior option results in some sort of loss (lost conversions, or worse health outcomes, etc). While there are many strategies to solve a MAB-type problem, each attempting to minimize the loss, in general, they differ from A/B testing in the sense that they do not wait for a statistically significant result before routing more traffic to the best performing variation at any point in time. In other words, the exploration (learning) & exploitation (earning) phase occur mostly simultaneously with MAB policies, while they are clearly distinct in A/B testing.

![](/images/multiarmbandit_2 "AB and MAB")
*An illustration of phases of an A/B Test (left) and a Multi-Arm Bandit policy (right). Photos from automizy.com*



Here's the table of contents:

1. TOC
{:toc}

## Basic setup

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-filename.md`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `filename` is whatever file name you choose, to remind yourself what this post is about. `.md` is the file extension for markdown files.

The first line of the file should start with a single hash character, then a space, then your title. This is how you create a "*level 1 heading*" in markdown. Then you can create level 2, 3, etc headings as you wish but repeating the hash character, such as you see in the line `## File names` above.

## Basic formatting

You can use *italics*, **bold**, `code font text`, and create [links](https://www.markdownguide.org/cheat-sheet/). Here's a footnote [^1]. Here's a horizontal rule:

---

## Lists

Here's a list:

- item 1
- item 2

And a numbered list:

1. item 1
1. item 2

## Boxes and stuff

> This is a quotation

{% include alert.html text="You can include alert boxes" %}

...and...

{% include info.html text="You can include info boxes" %}

## Images

![](/images/logo.png "fast.ai's logo")

## Code

General preformatted text:

    # Do a thing
    do_thing()

Python code and output:

```python
# Prints '2'
print(1+1)
```

    2

## Tables

| Column 1 | Column 2 |
|-|-|
| A thing | Another thing |

## Footnotes

[^1]: This is the footnote.

