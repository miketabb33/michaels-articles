This article is derived from my notes on Dragan Stepanović's presentation on "[Async Code Reviews Are Killing Your Company’s Throughput](https://www.youtube.com/watch?v=fYFruezJEDs)" from NDC Copenhagen 2022. The photos in this article were sourced from the presentation.

The problem with async code review flows are they put throughput and quality at odds. Take a moment to look at this diagram of a typical PR-based async code review flow.

![pr-based-async-code-review](https://raw.githubusercontent.com/miketabb33/michaels-articles/master/img/async-code-reviews-killing-throughput/pr-based-async-code-review.png)

- As you can see, pull requests (PR's) lead to context switching and feedback lag.
- It’s not uncommon for most of a single ticket's lead time to be spent in waiting for review.

The following analyzes the engagement, wait time, and size of async code reviews.

## Engagement
**Systemic effects of delayed and ‘choked’ feedback:** The inherent delay’s of PR’s cause slow feedback, which kills engagement. To imagine this clearly, think of the effectiveness of a phone call with 10-second delays.

**High latency, low throughput feedback:** Written communication is more expensive than verbal communication. Written communication takes longer to write, and it is inherently delayed. Verbal timely feedback is higher quality.

**Bigger PRs lead to less engagement per line of code.** *PR size measured in lines of code (LoC) and engagement measured total comment count per PR.*

**A lack of engagement and feedback leads to lower quality code.** More feedback does not equal higher quality code, but more ‘quality feedback’ does equal higher quality code.

**Anecdote:** 10 lines of code - 10 issues. 500 lines of code - lgtm.

## Wait Time and Size
Software’s #1 priority is to deliver value to its customers. Any wait time delays value delivery. *Not all value is equal, but each change has some impact.*

**Inventory expands until it matches the delays of the system.** For example: If it takes 20 minutes to run tests, it makes sense to batch many changes together before running the tests. On the flip side, there’s a cost to a large batch of changes. They are risky, have longer feedback cycles, and lack fast course correction.


**We need to seek quality feedback. Anecdote:** The hotter/colder game; when you get closer to the item, someone says hotter and vice versa.  Consider 2 teams, one providing hotter/colder feedback to their teammate every 1 min and the other every 1 second. Can you guess which team will find the item first?

PR size is a lose/lose balancing act, sacrificing either quality or throughput.

Few large PRs negatively impacts quality:
- (-) Takes longer to write and review
- (-) Low feedback and quality
- (-) Riskier due to many changes
- (-) Low engagement
- (-) Higher lead time
- (-) Lower deployment frequency
- (+) Fewer team interruptions

Many small PRs equate to higher feedback/quality but more team interruptions. Team interruptions create longer wait times and negatively impacts throughput.
- (+) Shorter time to write and review
- (+) High feedback and quality
- (+) Less risky due to fewer changes
- (+) Higher engagement
- (+) Shorter lead time to change
- (+) Higher deployment frequency
- (-) More team interruptions

PR's put throughput and quality at odds.

![throughput-vs-quality](https://raw.githubusercontent.com/miketabb33/michaels-articles/master/img/async-code-reviews-killing-throughput/throughput-vs-quality.png)

**Optimal batch size:** The point at which gets the best of throughput and quality. This concept comes with accepting that throughput and quality/stability are at odds with each other.

![batch-size](https://raw.githubusercontent.com/miketabb33/michaels-articles/master/img/async-code-reviews-killing-throughput/batch-size-and-cost.png)

## The Fool’s Choice: Throughput vs Quality
Throughput and quality are not mutually exclusive, they can both be achieved.

**Small PR's and changes** are desired because there are many benefits:
- Quicker to write and review
- Less time allocated for review
- Higher engagement
- Less risky
- Shorter lead time to change
- Higher deployment frequency

**Continuous code review**:
- In order to not exponentially lose throughput while reducing the average size of the PR, people need to get exponentially closer and closer in time.
- While small PR's exhibit many pro's, the big negative to small PRs is team interruptions. However, if your pairing or mobbing on the same work, you can’t be interrupted.

**Co-creation patterns** (Pairing and Mobbing):
- **Size is low**, because we are reviewing a line at a time, as we co-create.
- **Wait time is low**, because the review wait time it effectively 0.
- **Engagement is high**, because communication is verbal and immediate. We are able to do fast course correction.

## Summary

Throughput and quality can both be achieved if teams shift away from async code reviews. Instead, value small changes, continuous code reviews, and co-creation patterns. Embracing practices like mobbing/pairing and [trunk based delivery](https://dora.dev/devops-capabilities/technical/trunk-based-development/) will increase a company's capability to deliver value to its customers.
