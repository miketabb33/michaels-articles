Dragan Stepanović - NDC Copenhagen 2022 - link to talk

![pr-based-async-code-review](https://raw.githubusercontent.com/miketabb33/michaels-articles/master/img/async-code-reviews-killing-throughput/pr-based-async-code-review.png)

Above is a diagram of a typical PR-based async code review.
- PR's promote context switching and feedback lag.
- It’s not uncommon for most of a single tickets lead time to be spent in waiting for review.

The next parts cover aspects of async code reviews and explores how teams can achieve both throughput and stability.

## Engagement
- **Systemic effects of delayed and ‘choked’ feedback:** The inherent delay’s of PR’s cause slow feedback, which kills engagement. For example, A phone call with delays ends up killing the engagement.
- **High latency, low throughput feedback:** Written communication is more expensive than verbal communication. Written communication takes longer to write, and it is inherently delayed. Verbal timely feedback is higher quality
- Bigger PRs (more lines of code - LoC) lead to less engagement per line of code (engagement being measured in comments).
- A lack of engagement and feedback leads to lower quality code. (More feedback does not equal higher quality code, but more ‘quality feedback’ does equal higher quality code.)
- **Anecdote:** 10 lines of code - 10 issues. 500 lines of code - lgtm.

Wait Time and Size
Software’s number 1 priority is to deliver value to its customers. Any wait time delays value delivery. Not all value is equal, but each change has some impact.
Inventory expands until it matches the delays of the system. Example: if it takes 20 mins to run tests, it makes sense to batch many changes together before running the tests. On the flip side, there’s a cost to a large batch of changes. They are risky, have longer feedback cycles, and longer gaps of course correction.
We need to seek quality feedback.
Anecdote: The hotter/colder game; when you get closer to the item, someone says hotter and vice versa.  Consider 2 teams, one providing hotter/colder feedback to their teammate every 1 min and the other every 1 second. Can you guess which team will find the item first? This anecdote applies to providing value for customers.
PR size is a lose/lose balancing act, sacrificing either quality or throughput.
Few large PRs equate:
- Takes longer to write and review
- Low feedback and quality
- Riskier due to many changes
- Low engagement
- Higher lead time
- Lower deployment frequency
+ Fewer team interruptions
Many small PRs equate to higher feedback/quality but more team interruptions. Team interruptions create longer wait times and negatively impacts throughput.
+ Shorter time to write and review
+ High feedback and quality
+ Less risky due to fewer changes
+ Higher engagement
+ Shorter lead time to change
+ Higher deployment frequency
- More team interruptions

![throughput-vs-quality](https://raw.githubusercontent.com/miketabb33/michaels-articles/master/img/async-code-reviews-killing-throughput/throughput-vs-quality.png)

Optimal batch size: The point at which gets the best of throughput and quality. This concept comes with accepting that throughput and quality/stability are at odds with each other.

![batch-size](https://raw.githubusercontent.com/miketabb33/michaels-articles/master/img/async-code-reviews-killing-throughput/batch-size-and-cost.png)

The Fool’s Choice: Throughput vs Quality
Throughput and quality can both be achieved.
Small PR (or changes) are desired because there are many benefits:
Quicker to write and review
Less time allocated for review
Higher engagement
Less risky
Shorter lead time to change
Higher deployment frequency
Continuous Code Review: In order to not exponentially lose throughput while reducing the average size of the PR, people need to get exponentially closer and closer in time.
The big negative to small PRs is team interruptions. You can’t be interrupted if you are not doing anything else.
Co-Creation Patterns:
Pairing and Mobbing
Size is low:
We are reviewing a line at a time, as we co-create.
Wait time is Low:
This is because the review wait time it effectively 0.
Engagement is High:
Communication is verbal and immediate. We are able to do fast course correction.
