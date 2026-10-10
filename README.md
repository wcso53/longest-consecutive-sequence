# longest consecutive sequence: how the hash-set "find the start" trick gets you O(n), plus the edge cases interviewers actually follow up on

Sorting the array solves this problem in about six lines. It's also the answer that gets you a follow-up question, because LeetCode 128 explicitly says your algorithm must run in O(n) time, and no comparison sort does. That single constraint is what makes this problem worth studying — it's less about consecutive numbers and more about the habit of checking whether you've done the same work twice.

## What the problem is actually asking

Given an unsorted array of integers, return the length of the longest consecutive elements sequence. Order in the array is irrelevant. Only which values exist matters.


Input:  nums = [100,4,200,1,3,2]
Output: 4           # [1,2,3,4]


A second example makes the "order doesn't matter" part harder to ignore:


Input:  nums = [0,3,7,2,5,8,4,6,0,1]
Output: 9           # [0..8], with the duplicate 0 counted once


The stated limits are 0 ≤ nums.length ≤ 10⁵ and −10⁹ ≤ nums[i] ≤ 10⁹. Negative values, duplicates, and an empty array are all fair game, and each of them shows up in someone's failed submission.

One naming trap: this is not the longest *contiguous subarray*. `[1,3,5,4,7]` contains `1,3,4,5` as a consecutive sequence of length 4, even though those values aren't adjacent in the array. Reading "consecutive" as "adjacent indices" turns a medium problem into a different, easier one.

## Why the sorting solution is a trap

Sorting groups consecutive values together, then one linear scan counts runs. Skipping duplicates and resetting the counter on a gap gives the right answer, in O(n log n), with O(1) or O(n) extra space depending on the sort.

It's a legitimately good answer to say out loud, because it shows you can get to correct code fast. The problem is that the O(n) requirement in the statement exists specifically to rule it out. On a whiteboard, "sort first" often reads as "I didn't notice the constraint." Mention it, name its complexity, then move on.

## The trick: only count from the start of a run

Put every number in a hash set for O(1) membership checks. Then, before extending anything, ask one question: is `num - 1` in the set? If it is, `num` is somewhere in the middle of a run and its run will be counted when you reach the actual first element. If it isn't, `num` is the smallest value of its run, and you can walk forward until the streak breaks.

python
class Solution:
    def longestConsecutive(self, nums: list[int]) -> int:
        num_set = set(nums)          # dedupes for free
        longest = 0

        for num in num_set:
            if num - 1 not in num_set:   # num is the start of a run
                length = 1
                while num + length in num_set:
                    length += 1
                longest = max(longest, length)

        return longest


Returning 0 for an empty array falls out of this for free. So does the single-element case.

### Why the nested loop is still linear

This is the part that gets asked about, because a `while` inside a `for` looks quadratic. It isn't, and the reason is worth saying out loud in an interview.

The inner loop only runs from a number with no left neighbor. For any given run of length k, exactly one element — the first one — passes that test, and the inner loop advances through the remaining k−1 elements of that same run exactly once. Elements in the middle of a run fail the guard, so they never trigger an inner loop of their own. Every value is therefore examined a constant number of times: once when the loop reaches it as a candidate start, and at most once while some run is being extended. Total work is O(n) time and O(n) space for the set.

The concrete failure mode when you drop that guard: an already-sorted input like `[1,2,3,...,n]`. Every element passes, every element walks the rest of the array, and you're back to O(n²) — with a correct final answer, which is why it can slip past a quick test.

## Where people actually lose points

A handful of small things, roughly in order of how often they cost an offer:

- **No start check.** Covered above. It's the single most common issue.
- **Iterating the original array instead of the set.** Not incorrect, but with heavy duplicates you re-examine the same values repeatedly. Iterating the set also gets you deduplication without a separate step.
- **Forgetting duplicates exist.** `[1,0,1,2]` is 3, not 4. The set handles this, but the sorting version needs an explicit `nums[i] == nums[i-1]` skip.
- **Empty input.** `0 ≤ nums.length` means an empty array is valid input and the expected output is 0. `max()` over an empty sequence throws in Python, so a `longest = 0` starting value matters.
- **Negative numbers.** The set-based approach doesn't care; a "counting array from 0 to max" approach does, and it also blows up on the 10⁹ upper bound.
- **Overflow worries.** With values capped at 10⁹, `num + 1` can't overflow a 32-bit signed int, so no guard is needed here. Worth knowing that if the constraint were 2³¹−1 instead, `current + 1` would wrap and the loop would misbehave in Java or C++.
- **"How would you also return the sequence?"** You can't get it from this loop alone. It's usually answered by tracking the start value and the best length you've seen — or by using a map of run boundaries.

## Two other O(n) angles, and one you can skip

A **boundary map** stores, for each value, the length of the consecutive run it belongs to. When you insert `n`, if `n-1` and `n+1` are both present, you merge two runs: the new length is `m[n-1] + m[n+1] + 1`. Update the two endpoints of the merged run with that length, and the answer is the largest value you ever store. It's the same asymptotics as the set version with more bookkeeping, and it can hand you the run boundaries directly.

Union-Find is genuinely applicable here — the problem is tagged with it — by mapping each value to an index and unioning indexes whose values differ by one, then taking the largest component size. It works, but the constant factor and the implementation length make it hard to justify under interview time pressure unless you were asked for it.

One variation worth recognizing: instead of the `num - 1` guard, you can delete values from the set as you consume them. AlgoMonster's writeup takes that route — once a run starting at `x` is counted, those numbers are removed so no later iteration reconsiders them. Same O(n), and for some people it's easier to argue the invariance: a value is either in the set and not yet counted, or it's gone and already counted. If you've already read that walkthrough, the guard version above and the removal version are the same idea with different bookkeeping. You can 👉 [read the full Longest Consecutive Sequence explanation and code walkthrough on AlgoMonster](https://algo.monster/p/longest-consecutive-sequence?fpr=li69) if you want to see the diagram-heavy version side by side with the code.

The Union-Find route is the one I'd skip unless you're specifically drilling that structure. If you are, this problem is a reasonable union-find exercise; it's just not the reason the problem is in the question bank.

## Practicing the pattern, not the problem

Solving LeetCode 128 once teaches you this problem. What you want is the reflex: see an unsorted array, a "streak" or "consecutive" keyword, and a stated O(n) bound, and think hash set plus a check for where the run begins. That's pattern recognition, and it's the thing that transfers to the next unfamiliar problem.

AlgoMonster's format is text plus interactive diagrams rather than video, organized around reusable templates, with the set-based approach to this problem living inside its hash table material. A few things about it are relevant if you're deciding whether to pay for practice material at all:

- The explanations are written as pattern-first, so a problem page tells you which family it belongs to and why, which is the part LeetCode's discussion section leaves you to infer.
- Content is text and diagrams. There are no video walkthroughs, which is either a feature or a dealbreaker depending on how you learn.
- The editor is in-browser, so you can run against the problem's test cases without leaving the page.
- Speedrun mode gives you timed review passes over patterns you've already covered — useful in the last two weeks before an interview, when you want recall rather than new material.
- An AI assistant is billed as unlimited on paid plans, for explanations and hints when you're stuck rather than for handing you a solution.

Worth knowing: the platform is DSA-focused. System design coverage is introductory, and there's no live interview simulation, so if you're targeting L5+ roles you'll be pairing it with something else.

## What it costs right now

AlgoMonster runs three tiers. Both paid plans are currently listed at what third-party promo trackers describe as roughly half of list price, with $198/year and $378 shown as the undiscounted figures.

| Plan | What you get | Price | Billing | Link |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems | $0 | — | [Start with the free tier](https://algo.monster/subscribe?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; company question updates; 325 lessons and problems, 678 illustrations, 3,000+ editorials; member discount on live bootcamps | $99/year (about $8.25/mo) | Billed annually, 3-day free trial, cancel or upgrade anytime | [Check the yearly plan](https://algo.monster/subscribe?fpr=li69) |
| Pro – Lifetime | Everything in the yearly plan plus lifetime content updates and an internal referral to a top tech company for qualified candidates | $189 | One-time payment | [Get lifetime access](https://algo.monster/subscribe?fpr=li69) |

A couple of notes on the numbers, since pricing pages move around. There's no monthly option — you're choosing between annual and lifetime. The plan of record on the subscribe page has listed $99/year and $189 one-time for a while, but promotion windows have changed the lifetime price in both directions before, including a $99 lifetime period that users asked the company to bring back on its own forum. If the exact price matters to your decision, check the page rather than trusting any figure you read in a blog post, including this one.

Fee math worth doing: at roughly $8.25/month billed annually, the yearly plan costs less than two months of LeetCode Premium and gets you structured explanations instead of a problem bank. If you interview once and stop, the yearly plan is the right call. The lifetime plan only pays off if you expect to come back — job changes every couple of years, or pattern review before promotion loops. At $189 it breaks even against two renewals, which is a long enough horizon that plenty of people should just take the year.

Two limits to go in with clear eyes. The problem bank is in the low hundreds, not thousands, so volume practice still means LeetCode. And there's no video, so if you learn by watching someone solve it, this is the wrong shape of product.

## The 30-second version

Hash set, then only start counting at values whose predecessor is absent. O(n) time, O(n) space, and the reason it stays linear is that each value gets touched a constant number of times. Sorting is correct but misses the stated bound, Union-Find works but isn't worth the code, and the two things most likely to cost you the round are a missing start check and forgetting that an empty array must return 0.

If you want the pattern drilled rather than just this one solution, 👉 [compare the AlgoMonster plans and current pricing](https://algo.monster/subscribe?fpr=li69) — the free tier is enough to see whether the text-and-diagram format fits how you study before you spend anything.
