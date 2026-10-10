# palindrome linked list: Solve LeetCode 234 in O(n) Time and O(1) Space Without Copying the List

The problem statement for LeetCode 234 fits in two lines: given the `head` of a singly linked list, return `true` if it reads the same forwards and backwards, and `false` otherwise.


Input:  1 -> 2 -> 2 -> 1
Output: true

Input:  1 -> 2
Output: false


Then comes the line that makes people close the tab:

> Follow up: Could you do it in O(n) time and O(1) space?

With an array or a string, a palindrome check is close to trivial. You set one pointer at index 0, another at `len - 1`, walk them toward each other, and compare. The data structure hands you backwards access for free.

A singly linked list gives you nothing. Each node knows only where the next one is. There is no `prev` pointer, no indexing, no length property. So the whole problem reduces to one question: **how do you read a singly linked list backwards without allocating extra memory?**

There are four honest answers, and only one of them satisfies the follow-up.

## The four approaches, compared

| Approach | Core idea | Time | Extra space | What it costs you |
| --- | --- | --- | --- | --- |
| Copy to array | Dump values into a list, use two pointers from both ends | O(n) | O(n) | Fails the follow-up, but it's the fastest way to a correct submission |
| Stack | Push half the nodes, pop and compare | O(n) | O(n/2) | Half the memory of the array version, still not O(1) |
| Recursion | Recurse to the tail, compare on the way back up | O(n) | O(n) call stack | Hits Python's recursion limit at 10⁵ nodes |
| Reverse the second half | Fast/slow pointers find the middle, reverse in place, compare | O(n) | O(1) | Mutates the list (restorable), more code |

A note on the constraints, because they matter more than they look: the node count runs up to 100,000, and values are single digits from 0 to 9.

That recursion limit issue is not theoretical. Python's default recursion limit is 1000 frames, so a recursive solution blows up on large test cases with `RecursionError`, not with a wrong answer. If you're writing the recursion version to show off elegance, at least know why the judge rejects it.

## Why the array and stack versions are still worth writing first

If you're in an interview and the clock is running, writing the array version takes about 40 seconds and proves you understand the problem. Save the optimal solution for after you've confirmed the mechanics.

In Python it's four meaningful lines:

python
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        vals = []
        cur = head
        while cur:
            vals.append(cur.val)
            cur = cur.next
        return vals == vals[::-1]


The stack version is the same idea with a different container, and it exists mostly to demonstrate that you know LIFO ordering reverses a sequence. Not much more to it.

The real interview signal is whether you can get to O(1) space, because that requires recognizing a pattern: **fast and slow pointers plus in-place reversal**. Both of those are reusable techniques, and this problem just bolts them together.

## The O(1) space solution, step by step

Take `1 -> 2 -> 3 -> 2 -> 1`.

**Step 1 — find the middle with two pointers.** Start both at the head. Move `slow` one node per iteration and `fast` two. When `fast` runs off the end, `slow` is sitting at the middle (odd length) or at the first node of the second half (even length).

For `1 -> 2 -> 3 -> 2 -> 1`, `slow` lands on `3`.

**Step 2 — reverse everything from `slow` onward.** Standard in-place reversal with three pointers (`prev`, `cur`, `next`). The tail of the original list becomes the head of the reversed half.

After this, the list reads `1 -> 2 -> 3` on the left and `1 -> 2 -> 3` on the right, where the right half is a separate chain pointing backwards relative to the original.

**Step 3 — compare node by node.** Walk one pointer from the original head and one from the head of the reversed half. The moment two values differ, return `false`. If you make it through the whole reversed half, every pair matched, so return `true`.

The reversed half is always the shorter or equal half, which is why the loop condition checks only the right pointer. The middle node in an odd-length list doesn't need a partner — it's a palindrome by definition.

python
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        slow, fast = head, head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # Reverse from slow to the end of the list
        prev = None
        while slow:
            nxt = slow.next
            slow.next = prev
            prev = slow
            slow = nxt

        # Compare the first half against the reversed second half
        left, right = head, prev
        while right:
            if left.val != right.val:
                return False
            left = left.next
            right = right.next

        return True


Two implementation details that separate working code from buggy code:

- **Where `fast` starts.** Initializing `fast = head` and looping while `fast and fast.next` is the version that needs no odd/even special-casing. Some editorials initialize `fast = head.next` and add a branch for odd lengths — both are correct, but the first is shorter and easier to defend in an interview.
- **Not splitting the list.** You don't need to set `slow.next = None` before reversing. It's a common habit from other problems, but here it's unnecessary work as long as your comparison loop only runs over the reversed half.

## Edge cases that actually break submissions

Most wrong answers on this problem come from one of five places.

1. **Single node or empty list.** `[1]` is a palindrome. If your middle-finding loop assumes at least two nodes, you'll crash or return the wrong value.
2. **Two nodes.** `[2,2]` is `true`; `[2,1]` is `false`. Two-element lists are where off-by-one errors in the `slow`/`fast` initialization show up.
3. **Odd vs. even length.** `[1,2,3,2,1]` and `[1,2,2,1]` take different paths through the middle-finding step. Test both.
4. **Duplicate values everywhere.** `[1,1,1,1,1]` is `true`. `[1,1,1,1,2]` is `false` and will fail if you compare the wrong halves.
5. **Mutating the input.** The O(1) solution rewires real pointers. LeetCode accepts this because the function only returns a boolean, but if your caller keeps using the list afterward, it's now corrupted. Restoring it means reversing the second half one more time and reconnecting — extra code that interviewers sometimes ask for as a follow-up.

On that last point, if you're asked to restore the list, the clean way is to keep a reference to the node before the reversed section (or use `slow`'s position) so you can point `next` back at it after a second reversal pass.

## Complexity, stated honestly

| Version | Time | Space | Passes the follow-up? |
| --- | --- | --- | --- |
| Array copy | O(n) | O(n) | No |
| Stack | O(n) | O(n) | No |
| Recursion | O(n) | O(n) | No |
| Reverse second half | O(n) | O(1) | Yes |

The time complexity is O(n) in every case; the constant factor differs. The reverse-half version traverses the list roughly 2.5 times (middle-finding, reversal, comparison), which is still linear and nowhere near a performance concern at n = 100,000.

If a solution claims O(1) space while using recursion, that claim is wrong. The call stack grows with the input, so recursive palindrome checks are O(n) space no matter how elegant the code looks.

## The pattern behind it, and where else it shows up

Palindrome Linked List is a composite problem. It's really two separate techniques glued together:

- **Fast/slow pointers** for finding the middle in one pass without knowing the length in advance
- **In-place linked list reversal** for getting backwards traversal without a second data structure

Both are used constantly. If you've solved Reverse Linked List (206) and Middle of the Linked List (876), you already have every ingredient — this problem just asks you to combine them under a memory constraint.

The same pattern family covers Linked List Cycle, Reorder List, and Palindrome Pairs. On the string side, Valid Palindrome and Longest Palindromic Substring test the same core intuition with different mechanics. If you're working through a structured list of linked list problems, this one is usually the checkpoint that tells you whether you actually understand pointer manipulation or have just been pattern-matching on syntax.

That "which technique does this problem need" step is where a lot of candidates lose time — not coding ability, but recognition speed. That's the specific gap a pattern-based curriculum is built to close, and it's why a platform like AlgoMonster organizes its content by technique rather than by problem number: each pattern comes with a template you reuse, and the practice problems exist to make the template stick.

AlgoMonster's own editorial for this problem, [👉 read the full 234 walkthrough and code in Java, C++ and Python](https://algo.monster/liteproblems/234?fpr=li69), follows the same structure used above — find the middle, reverse the second half, compare — and includes a step-by-step trace of `[1 -> 2 -> 3 -> 2 -> 1]` node by node if you want to see the pointer movements visually before writing your own.

## AlgoMonster plans and what a subscription actually unlocks

If this problem is the kind of thing you're studying systematically rather than one-off, here's what the platform costs. It keeps it simple: a free tier plus two paid options, no monthly billing, no team plan.

| Plan | What you get | Price | Billing | Purchase link |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems — enough to judge whether the text-and-diagram teaching style works for you | $0 | — | [ Start on the free tier before paying anything](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI Assistant usage; unlimited Speedrun; company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations and 3,000+ editorials; member discount on live bootcamps; 12 months of access | $99/year as currently listed (`$8.25/mo` billed annually); list price $198 | Billed annually, 12 months of full access | [ Check the current yearly price at checkout](https://algo.monster/subscribe?fpr=li69) |
| Pro – Lifetime | Everything in the yearly plan, plus lifetime updates — no renewal, no price increases | $189 one-time as currently listed; list price $378 | One-time payment, access kept forever | [ See the lifetime option and today's applied price](https://algo.monster/subscribe?fpr=li69) |

Two things worth knowing before you compare these numbers with what you find elsewhere.

**Prices move with promotions.** Search for "AlgoMonster price" and you'll find third-party blogs quoting everything from $89/year to $459 lifetime, some of them clearly stale. The subscribe page applies the current promotion automatically, so the price at checkout is the only one that matters. Discount codes from partner creators can stack additional percentage off in some cases, but I'd rather you see the real number on the page than trust a figure from a blog post that hasn't been touched in a year.

**The free tier is genuinely a preview, not a trial.** You get a subset of the patterns and problems — useful for deciding whether the diagram-heavy, text-first format fits how you learn. It is not enough to prep for interviews on its own. If you've been grinding problems with no structure and can't tell a two-pointer problem from a sliding-window problem on sight, that's the gap the paid content targets.

## Quick answers to the questions people actually search

**Do I have to reverse the list, or is there another O(1) approach?**
Reversing the second half is the standard answer and the one interviewers expect. Some solutions reverse the first half while walking with slow/fast pointers in a single pass, which works and saves one traversal, but the code is harder to read and the complexity gain is negligible on interview-sized inputs.

**Is comparing the whole list against its reverse valid?**
Checking whether values read the same forwards and backwards is the right idea, but if you reverse the entire list with extra memory you've solved a different problem. In-place reversal of just the second half is what gets you to O(1).

**What if the list has an even number of nodes?**
Then both halves are the same length and the comparison loop runs to the end of the reversed half. No special case needed.

**Why does my recursive solution fail on big inputs?**
Stack depth. Python caps recursion at 1000 frames by default, and the constraint allows 100,000 nodes. Even in languages without that limit, the recursion is O(n) space, so it doesn't satisfy the follow-up anyway.

**Does the solution have to restore the original list?**
Not for LeetCode. The judge only checks the return value. It becomes a real requirement if the function is called inside a larger program that still needs the list intact — worth mentioning out loud in an interview, since it shows you think about side effects.

**Is this a good problem to learn in-place reversal from?**
It's better as a check *after* you've learned it. Reverse Linked List (206) is where the three-pointer reversal actually clicks; Palindrome Linked List is where you prove you can apply it under a constraint. Doing them out of order is a common reason people get stuck here.
