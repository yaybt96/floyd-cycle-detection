# fast and slow pointers: Floyd's tortoise and hare explained, with the template for cycle detection, middle node, and Happy Number

Most people who fail a linked list question don't fail because they can't write the loop. They fail because nobody told them the problem was a linked list problem.

Find the Duplicate Number is the classic example. The statement talks about an array of integers, not a list of nodes, and yet the intended solution is exactly the same two-pointer walk you'd use on LeetCode 141. Same algorithm, different costume. That's the real gap: not knowing Floyd's cycle detection, but knowing when it applies.

So this is a walkthrough of the pattern itself — the template, the problems it covers, the mistakes that cost people the interview — plus a note on where structured pattern training actually helps.

## What the pattern actually does

Two pointers start at the head. Slow moves one node per iteration, fast moves two. That's the whole setup.

The interesting part is why they're guaranteed to meet inside a cycle. Once both pointers are inside the loop, fast gains exactly one position on slow per iteration. The gap shrinks by one each step, and a gap that shrinks by one always reaches zero. No probability, no edge case. If a cycle exists, the meeting is inevitable.

If there's no cycle, fast hits the end of the list first. That's your negative case, and it terminates in at most n/2 iterations.

For finding the middle, the same mechanism does the work. Fast travels twice the distance of slow, so when fast reaches the last node, slow has covered exactly half the list. One pass, no length counter, no second walk.

## When to reach for it — and when it's the wrong tool

The pattern fits when the problem describes a sequence that might revisit a state, or when you need a structural property of a singly linked structure without extra memory.

Signals that usually mean fast and slow:

- The words cycle, loop, circular, or "repeated state" appear in the statement
- You need the middle node in a single pass
- You need the start of a cycle, not just whether one exists
- A visited-set solution obviously works but costs O(n) space
- The input is a linked list with no length field and no random access

Cases where it's the wrong answer:

- Trees and DAGs. Unless you can flatten the structure into a sequence, pointer speed differences tell you nothing useful about branching.
- Arrays where you just need the midpoint. `arr[len(arr) // 2]` is O(1). Introducing pointers there is a step backwards.
- Problems where you need to record every visited node, not just detect a repeat. A set is the honest answer there.

There's also a cousin pattern people blur together with this one: fixed-gap pointers, where fast starts k nodes ahead and both then move one step at a time. That's how you find the k-th node from the end. AlgoMonster's decision flowchart literally branches on "Fast/slow or fixed-gap pointers?", which is a decent sign that the distinction comes up in practice.

## The template, and the one guard people get wrong


slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        return True

return False


The condition `fast and fast.next` trips people up constantly, and the confusion is almost always about the *second* access. Look at what the loop body does: `fast.next.next`. That's two dereferences. So before executing it, both `fast` and `fast.next` have to exist. Checking `fast.next.next` in the while condition is unnecessary — if it's null, the assignment just sets `fast` to null, which is fine and gets caught by the loop guard on the next pass.

So: check what you're about to dereference, not the value you're about to assign.

## The problems this pattern covers

### Linked List Cycle (141)

Straight template application. Initialize both pointers at head, run the loop, return true when they collide.

The naive solution — push every visited node into a hash set — also runs in O(n) time but costs O(n) space. In an interview, naming both and explaining the tradeoff is worth doing. Reviewers consistently flag that the useful signal isn't which one you write, it's whether you can explain why O(1) space matters.

### Linked List Cycle II (142) — finding the entry node

Detection is phase one. Finding *where* the cycle starts is two phases, and the second one surprises people.

After slow and fast meet, reset one pointer to head, leave the other at the meeting point, and advance both one step at a time. They meet again at the cycle entry.

The reason is a distance identity, not a trick: the distance from head to the cycle entry equals the distance from the meeting point to the cycle entry, modulo the cycle length. So a pointer walking from head and a pointer walking around the loop from the meeting point arrive at the same node at the same time.

The mistake here is resetting *both* pointers to head. Then you've thrown away the meeting point information and you're just running the detection again.

### Find the Duplicate Number (287)

This is the array-in-disguise problem. Values are in the range 1 to n, in an array of length n+1, so at least one value repeats by pigeonhole. Treat each index as a node and `nums[i]` as its next pointer, and the duplicate value becomes the entry point of a cycle.

Same two-phase algorithm as 142. O(n) time, O(1) space, array untouched. If you can spot that mapping under pressure, you've demonstrated the pattern rather than memorized a problem.

### Middle of the Linked List (876)

The simplest use of the speed difference. Run the loop until fast or fast.next runs out, then return slow.

One detail worth knowing before you submit: for even-length lists, slow lands on the *second* of the two middle nodes. That's what LeetCode 876 expects, and it's also the convention that makes Palindrome Linked List and Maximum Twin Sum work correctly downstream.

### Palindrome Linked List (234)

Three steps stacked together: find the middle with fast and slow, reverse the second half in place, then compare the two halves node by node. Space is O(1) if you reverse in place rather than copying values into an array.

The array approach is easier to get right under time pressure, and interviewers know it. If you have the time, doing it with pointer manipulation is the stronger showing — and you can restore the second half afterward if mutation is a concern.

### Happy Number (202)

Repeatedly replace a number with the sum of the squares of its digits. Either you reach 1, or you fall into a loop that never reaches 1. Because the sequence of values is deterministic, the loop is a cycle — and fast and slow pointers detect it without a visited set.

Same pattern, no linked list anywhere. This is the problem that convinces most people the pattern is about *structure*, not about nodes.

## Complexity, and what you're trading away

Both pointers traverse at most n nodes each, so time is O(n). Only two variables are allocated, so space is O(1), independent of list length.

The tradeoff is that Floyd's algorithm tells you a cycle exists and where it starts, but it doesn't enumerate the nodes. If you need the full set of visited states, a hash set is the right data structure and the O(n) space is a genuine cost you should state out loud rather than hide.

## Common mistakes, in order of how often they show up

1. Checking `slow == fast` before the first move. Both pointers start at head, so the comparison fires immediately and returns a false positive on a list with no cycle. Move first, compare after.
2. Dropping the `fast.next` half of the guard. `while fast:` alone crashes on an even-length list.
3. Resetting both pointers to head in phase two of cycle detection.
4. Returning the meeting point when the question asked for the cycle entry. They're different nodes.
5. Reaching for pointers on a sorted array where index arithmetic already gives you the midpoint.
6. Confusing fast/slow with the opposite-end two-pointer pattern. Two pointers usually start at opposite ends and compare values; fast/slow starts at the same place and exploits speed difference to reveal structure.

## Where structured pattern training fits in

You can learn all of the above from free LeetCode discussions — the writeups are genuinely good, and the editorials for 141 and 142 explain the phase-two proof as well as any paid course. The gap isn't information availability. It's recognition speed: seeing `nums` and thinking "implicit linked list" within the first minute of reading a problem.

That's the specific thing [👉 AlgoMonster's pattern curriculum](https://algo.monster/subscribe?fpr=li69) is built around, and it's the reason reviewers keep describing it as a course rather than a problem bank.

The format is text and interactive diagrams, no video. Lessons are organized around named patterns with code templates, and the linked list section is where slow-fast pointers get introduced — the platform's own Blind 75 practice list flags that section as "[introduces the use of slow-fast pointers, reversing a linked list, and merging two lists](https://algo.monster?fpr=li69)". The decision flowchart is the part that maps most directly onto the recognition problem: it walks through questions like whether the input is sorted, whether you need a contiguous subarray, and whether this particular shape wants fast/slow or fixed-gap pointers.

What reviewers say it does well: the curriculum is designed to be finished in roughly 4–8 weeks at 1–2 hours a day, the diagrams make abstract structure concrete, and Speedrun mode adds a clock that self-paced problem sets don't have. What they consistently flag as limits: no video walkthroughs, a smaller problem bank than LeetCode's 3,000+, introductory-level system design coverage, and shallow behavioral content. Several reviews also recommend pairing it with LeetCode for volume rather than treating it as a complete prep stack.

## AlgoMonster plans and current pricing

The subscribe page currently shows two paid plans, annual and lifetime, plus a free tier. There's no monthly billing option.

| Plan | What you get | Price | Billing | Link |
| --- | --- | --- | --- | --- |
| Free | Limited access to sample patterns, lessons, and problems — enough to judge the teaching style | $0 | — | [ Start on the free tier and browse the lessons](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems; unlimited AI Assistant; unlimited Speedrun; real questions from top tech companies, updated monthly; 500+ lessons across 6 courses; 1,000+ illustrations; 3,000+ editorials; member discount on live bootcamps; bonus Foundation and Build an LLM courses | $8.25/mo, billed as $99/year | Annual, cancel or upgrade anytime | [ Get Pro Yearly](https://algo.monster/subscribe?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan, plus lifetime updates and an internal referral to a top tech company for qualified candidates | $189 | One-time, permanent access | [ Get Pro Lifetime](https://algo.monster/subscribe?fpr=li69) |

Two things worth knowing before you click.

First, third-party coupon pages trace the same offer back to list prices of $198/year and $378 lifetime with a promotional discount applied — which is consistent with the $99 and $189 you see at checkout. Codes from partner sites are reported to stack on top of that, but they get rotated and expired without notice, so check what the checkout page actually charges rather than trusting a code from a blog post.

Second, you'll find reviews quoting $459 for lifetime access or $119–$299 for annual. Those numbers come from different promotional periods and appear to be stale. The figures on the subscribe page are what you'll be charged today.

Which plan makes sense depends on your timeline. If you have interviews scheduled inside the next two months, the annual plan covers the window at a lower upfront cost. If you expect to go through the loop again when you change jobs — which for most engineers is every two to three years — lifetime pays for itself on the second cycle, and there's no renewal to think about when a recruiter emails you out of nowhere.

**Bonus courses:** Both paid plans include the Foundation Course (CS fundamentals from the ground up) and a Build an LLM course covering tokenization, attention, and training without requiring an ML background. Some specialized courses developed with external partners, such as Quant Interview Prep, are sold separately.

## What to take away

Fast and slow pointers is one of the smaller patterns to learn and one of the more rewarding to recognize — the code barely changes across six or seven problems, and the recognition cue is usually a single word in the statement. Learn the template cold, understand why the gap-closing argument guarantees a meeting, and spend your practice time on the mapping step: array to implicit list, number sequence to cycle. That's the part interviews actually test.

If you want the pattern drills plus the decision flowchart in one place, [👉 the current plans are here](https://algo.monster/subscribe?fpr=li69), and the free tier is enough to see whether text-and-diagram teaching suits how you learn.
