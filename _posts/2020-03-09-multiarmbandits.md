# Multi-Arm Bandits: a potential alternative to A/B tests

![](/images/multiarmbandit_1.png "Cartoon Octopus")

Imagine this scenario: you run a website with rare but valuable conversions, and you want to experiment with variations of a landing page in order to maximize your conversion rate. The go-to methodology for such an optimization is A/B testing.

In its most straightforward implementation, a single A/B test consists of essentially one phase of exploration, where you divvy up traffic between the control and test versions of the landing page, and a follow-up period of exploitation, where you direct all traffic to the best performing variation from phase one.
The downside of this methodology is that during the testing phase, some proportion of your visitors will necessarily be going to the worse performing variation. The more valuable your conversions are, the more you tend to lose out during this phase.

Some degree of loss during the exploration phase cannot be avoided: if we do not test at least a few potential variations, it would not be possible to know how they rank against each other. Nevertheless, while we cannot take this potential loss down to zero, can we reduce it while retaining the ability to infer statistically significant results?

This is where Multi-arm Bandit policies enter the picture.

### What are MABs?

MABs are a class of problems where you must make choices over a space of possibilities (landing page design, medical drug testing, advertising campaigns) where each choice has a stochastic reward distribution and where the choice of an inferior option results in some sort of loss (lost conversions, or worse health outcomes, etc). While there are many strategies to solve a MAB-type problem, each attempting to minimize the loss, in general, they differ from A/B testing in the sense that they do not wait for a statistically significant result before routing more traffic to the best performing variation at any point in time. In other words, the exploration (learning) & exploitation (earning) phase occur mostly simultaneously with MAB policies, while they are clearly distinct in A/B testing.

![](/images/multiarmbandit_2.png "AB and MAB")
*An illustration of phases of an A/B Test (left) and a Multi-Arm Bandit policy (right). Photos from automizy.com*

The central tension in these problems comes from the exploration-exploitation dilemma, and each policy has its own specific tradeoffs in regards to it.

In this post, the policies I'll focus on are:

- Epsilon-Greedy
- Thompson Sampling

So, let's begin.

---

### Epsilon-Greedy

Returning to the landing page hypothetical, let a single round encompass the total number of views and conversions over some period - 6 hrs, 1 day, 1 week, etc. With A/B testing we'd wait for our experiment to reach some pre-defined stopping criterion - usually either statistical significance, minimum sample size for each variation, or a certain number of days/rounds - prior to choosing the best performing variation.

With a ɛ-greedy policy, after some initial round or two of pure exploration, we would always choose the best variation - the variation with the highest expected reward - at the start of any round, except for ɛ% of the time, where we choose a random variation from the rest.

![](/images/multiarmbandit_3.png "epsilon-greedy")
*A single epsilon-greedy round. Photo from yhat*

If we consider it from the point of view of the loss incurred from sending traffic to a lower performing variation, then ɛ-greedy improves on the A/B test by ensuring that (1 - ɛ)% of the time, only the best performing variation is being seeded with visitors. The downside to that is the issue of local optima, or the fact that since the rewards are stochastic a 'hot streak' by a lower-quality variation at the start of the experiment might lead to it being incorrectly routed a majority of the traffic.

To some extent, the algorithm counters that by ensuring that exploration occurs ɛ% of the time, so even if the lower-quality variation performs well initially, over time random exploration will ensure that other, potentially better-performing Arms also get traffic.

Consequently, just like with AB testing, some degree of loss is built-in to the strategy, since we will explore ɛ % of the time, regardless of how much better any particular variation has fared so far.

The graph below is the result of a simulation of a ɛ-greedy strategy with 1 control and 3 potential variations, of which Arm 1 is the best of the lot. The simulation was run 200 times, and as we can see, by round 200 the algorithm is routing traffic to Arm 1 75% of the time. It hits a ceiling of around 85% (ɛ equals 15%) by around the 300th round.

![](/images/multiarmbandit_4.png "bandit in R")

While potentially better than A/B testing, this pre-determined ceiling is something of a drawback with Greedy strategies, one that Thompson Sampling (amongst others) tries to improve upon.

### Thompson Sampling

With Thompson Sampling, instead of choosing an Arm based upon the highest expected reward at the start of any round, the policy instead samples from the reward distribution of each Arm and directs traffic to the one with the highest sample value at the start of the round. The theory here is quite interesting, so let's unpack this.

Firstly, we now model the entire distribution of the conversion rate, instead of just the expected value. Since an individual visitor is either converted or non-converted by the landing page, we can think of conversions as a Bernoulli process. When we sum it up across visitors, we get a Binomial distribution. What we want to know is the distribution of p, the probability of success for the Bernoulli process. To do so, we will use Bayesian updating, with p being updated at the end of every round.

At the start of the experiment we have no information about what the conversion rate for a variation looks like. So, we initialize the algorithm with a completely uninformed prior - plot 1. This encapsulates the idea that the conversion rate for a variation can be anywhere between 0 & 1 with equal probability. As the experiment progresses, and we gain more information about what the actual conversion for a variation look like, we update this initial uninformed guess - hence the distribution centers and becomes narrower around the actual value.

![](/images/multiarmbandit_5.png "Conversion rate distribution")
*Empirical distribution of the conversation rate - a parameter for the Bernoulli.*

*(Statistical Aside: we model p as a Beta Distribution, which is the conjugate prior for the Binomial. That essentially means that if the prior for a Binomial random variable has a Beta distribution, the posterior will too! For example: plot 1 is Beta(1,1) and is prior for Round 1. If in round 1 we see 5 successful conversions and 95 failed ones, the posterior becomes Beta(6,96), which is also the prior for round 2!)*

With Greedy policies, we would always pick the variation with the highest expected conversion rate (the vertical line). With Thompson, we instead first take a sample from the distribution and then pick the Arm with the best sample.

![](/images/multiarmbandit_6.png "sampling thompson")
*Arm 2 is picked despite having a lower mean conversion rate.*

Despite Arm 1 having a higher expected value, we will use Arm 2, as the sample value in this round is higher (In the next round, another random sample is taken for the posterior distribution). This ensures that Arms that we haven't explored much can be picked with a much higher probability than with Greedy. Essentially, keeping everything else equal, the probability that a variation will get picked is inversely proportional to the uncertainty around it. This dynamism ensures that we don't have a deterministic ceiling on the best variation as we do with greedy policies. Running the same simulation as before - 200 experiments, 1 control and 3 variations - we see that by round 200 the algorithm has converged to picking the best Arm over 90% of the time, and approaches 100% by round 300.

### Conclusion 

Whether you should use Bandits or A/B testing is highly dependent on context. Vanilla Bandit policies have certain additional mathematical assumptions that need to be fulfilled in order to be directly applicable; AB testing is rather robust to violations of these assumptions. A quick simulation of the two policies (AB testing & Thompson Sampling), highlights some of the factors that can help choose between one of the other.

*(Note: Regret is simply the expected loss in conversions due to the exploration phase. The lower the better)*

![](/images/multiarmbandit_7.png "regret by procedure")
*In all these cases, Bandits take longer but have a lower regret than A/B tests*

In general, Bandits are useful for very short experiments (think News Headlines, as the duration in which they are fresh & relevant in short), continuous optimization of site components, Ad targeting (Google uses Bandits extensively with Ad Sense), etc. Statistically, Bandits would likely perform better than A/B tests with higher significance requirements and larger differences in null and alternative hypothesis.

Nevertheless, despite the additional complexity they bring, Bandit policies are quite a useful tool and can provide results superior to A/B testing in many, diverse scenarios.