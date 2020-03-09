# Minimizing menu selection costs with Integer Programming (and PuLP)
## *(And how re-framing problems can lead to simple solutions)*

Tata Sky is an Indian TV network provider that - like all such providers - offers a byzantine array of standalone and bundled channels for its users to choose from. Given my sparse usage, I've long been wanting to minimize my rather hefty expenditure on their services, but actually going through their catalog and calculating whether I'd be better off with a pack, or a standalone channel, or some convoluted combination of the two, was a rather daunting task, and something I'd put off until I realized that it was one I could solve with Integer Programming.

What fascinated me, after I'd come up with an approach, was how trivial the problem became once it had been framed in a slightly different manner. That reframing, rather than just the technicalities of the solution, is what motivated me to write this piece.

### The Problem

Tata Sky offers over 300 paid channels (and 275 Free to Air (FTA) ones), along with about 170 packs, consisting of a median around 54 channels, with a few large packs numbering in the hundreds. The objective is to pick the combination of standalone channels and packs that provide one with all of the saas-bahu serials one needs while minimizing expenditure. To find the optimal combination for this task requires one to search through the entire space of packs and standalone channels, which being a rather arduous task to do manually is where Integer Programming comes in.

### Linear & Integer Programming

A general Linear Programming problem can be represented as thus:

![](/images/integerprogramming_1.png)

We're essentially trying to minimize a linear function given a set of linear constraints and real-valued decision variables. Numerous problems in economics, business, vehicle routing, scheduling can be modeled and solved as a linear program (LP).

What we're going to use however is a variation on LP in which the decision variables are further constrained to be integers, and in our case, just binary. A common problem is this vein is the knapsack problem.

![](/images/integerprogramming_2.png "The Basic Knapsack problem @ codescope.com")

Simply put, the objective here to maximize the value of items in the bag, while satisfying its carrying constraint. What we're solving for, however, is the inverse of this problem: we're trying to minimize cost while retaining the ability to view the selected channels.

Computationally, this is not really an easy solution. While most linear programs are solvable in polynomial time, there is no such guarantee for Integer programs, which in the worst-case have exponential complexity. If we select 50 channels we want to be able to view, and we know that there are 100 packs that have at least one of these channels, then we have 2¹⁵⁰ combinations to search through!

![](/images/integerprogramming_3.png "While a Linear Program can take on any value in the grey area, an IP solution can only be one of the blue dots.")

To accomplish this, I'm going to be using PuLP, which is a Python modeling library for linear optimization problems.

However, first the data.

### Data, Data, Data…

As is often said, the hard part of ML - though this is not really ML - is the data, and a majority of the time spent on creating this solution was spent on the ETL. And since Tata Sky did not make this easy for me by providing a readily available open-source data set, the very first step was figuring out a way to build it.

The most straight-forward solution here would have been to scrape the data from their site, but since I wasn't sure about the legalities behind that, I decided to err on the side of caution. Thankfully, I found a couple of pdfs floating around the web with channels and packs in relatively well-formatted tables. Since, the pdfs were text-based, ie. I could click and drag to select text from the document, I could use Python's Camelot library to extract data from them.

Extracting individual channel info from a pdf comprising almost entirely of a single table of channels was relatively straight-forward.

![](/images/integerprogramming_4.png "Yes, from the 4th to the 578th executed cell - this is a kludge of a notebook!")

The packs, however, were significantly harder, had a lot of errors and inconsistencies, and required a lot more in terms of validation.

![](/images/integerprogramming_5.png "Kludgy, yes, but still kinda fun to figure out.")

The end result of this long-drawn data step was a table consisting of all 588 unique channels, their standalone prices, and a list of all of the packs they belonged to.

![](/images/integerprogramming_6.png "The Popular column, for now, is just a simple count of all the packs a channel belongs to, which is admittedly not a good estimator for a channel's popularity.")

And that lead, finally, to the modeling.

### The Solution 

My initial framing of the problem had a rather fundamental gap: I could not figure out a way to link packs and channels within the general IP framework. In the knapsack problem, and its inverse, each item is essentially a singular thing and can be excluded or included in the optimal solution with a {0,1} decision variable. Here, however, we have two tiers of items:

- Singular items, ie. the standalone channels.
- Packs, which comprise of a large number of standalone channels.

The inclusion of a pack - say English News - implied the inclusion of a large number of channel as well. Initially, I'd envisioned something like a lookup table for each pack to try and encode these relationships into the problem, but that wouldn't fit within the LP framework, and definitely could not be solved by a standard solver.

However, eventually, I realized that all it needed was a slight tweaking of the constraints. Instead of trying to directly create some sort of connection between the channels and packs, I could do so indirectly using the constraints.

Let me illustrate this with an example.

Suppose, we select 5 channels.

![](/images/integerprogramming_7.png)

The minimization function generated consists of the 5 channels themselves, and the 33 unique packs that contain at least 1 of these channels. The channels and packs are encoded as boolean decision variables - the c's and x's - and multiplied with their respective prices.

![](/images/integerprogramming_8.png "5 channels and 33 packs.")

Alongside that, we have 5 constraints, 1 each for each of the 5 selected channels. Each constraint comprises of a channel variable, and all of the packs that contain that channel. For example, the constraint for HBO HD contains 16 decision variables - 1 for the channel, and 15 for all of the distinct packs that contain HBO HD.

![](/images/integerprogramming_9.png "Constraint generated for HBO HD by PuLP")

What this encodes is the idea that since we want HBO HD, we need to pay for the channel itself, or at least 1 of the packs that contain it. It's an inequality constraint because as long as overall spend is minimized, we don't care how many times we subscribe to any particular channel, as long as we do so at least once.
This neatly solves the problem we had in regards to linking packs and channels. By specifying the packs in the constraint function for a channel, we don't need to explicitly link between them, since the selection of a pack explicitly satisfies the constraint for the channels it contains.

And that's it.

The optimal solution in this, almost degenerate case, is to choose each individual channel.

![](/images/integerprogramming_10.png "Constraint generated for HBO HD by PuLP")

However, we can get far more interesting suggestions, given a larger list of channel selections. With some 100-odd randomly selected channels, we get this, which is definitely far cheaper than choosing the bigger packs - which can cost up to a thousand - or just buying the channels individually, which costs above 500.

![](/images/integerprogramming_11.png)

### Conclusion and Code 

The Integer Programming code can be found in my [GitHub](https://github.com/AbhishekKaps/Integer-Programing-with-PuLP), along with the data for the channels and packs.

The PuLP documentation is an excellent resource for an introduction to this domain and can be found [here](https://github.com/AbhishekKaps/Integer-Programing-with-PuLP).

My next step with this is going to be to figure out a way to keep the pack & channel info up to date. In addition, there are some complexities I left out of this initial solution- such as cost for multi-tv support, amongst others - and I plan to add them in as well. And finally, deploy this via my (very first) web app.