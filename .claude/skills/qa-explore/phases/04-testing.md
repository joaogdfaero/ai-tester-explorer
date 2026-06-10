# Phase 4: Testing (~80% of time)

**Goal:** Test the in scope features and areas of the application.

This is NOT feature-by-feature testing. This is "walk through the app as a real user would." using your QA expertise to find issues. Don't make any "weird" behavior that a normal user would never do (ex. clicking the play button multiple times in quick succession). 

## Test main scenarios
Execute Scenarios: One by one, work through each area you identified earlier, exploring every function in that section. Think of what a real end-user might do. Come up with use cases or scenarios and execute them. Then, think of unusual variations and execute those. Use the results of these tests to spark new ideas.

## Iteration
Iterate: Once you have exhausted a specific area or function, move on to your next point of interest. As you repeat this process, remember what you have learned and use that information to influence your subsequent tests. 

## How to test (guide)
Use the instructions below as inspiration during your tests, but don't hesitate to follow other valid paths or your own ideais to find issues, use your creativity and prior knowledge.

### 1. Execute the Happy Path First

Complete the full journey with valid data to see if it works as expected, but NEVER:
- Make any purchases, payments 
- Make changes to the account (changing password, profile data, payment methods and etc). 

### 2. Verify Data at Every Step

Validate that data updates correctly after every action:
- counters
- badges
- totals
- balances
- lists
- statuses
- history
- confirmations

Refresh the page or navigate away and back to verify the data persists correctly.


### 3. Check Cross-Page Consistency

Compare the same information across different pages and components:
- titles
- prices
- images
- counts
- statuses
- labels
- recommendations

Make sure the same data remains consistent everywhere.

### 4. Verify the End State

Confirm the action was actually completed successfully:
- verify confirmation messages
- history updates
- saved state
- updated UI
- resulting behavior

Always look for proof that the operation completed correctly.

### 5. Test Navigation and User Flow

Validate:
- buttons
- links
- menus
- banners
- tabs
- breadcrumbs
- carousels
- redirects
- browser back/forward navigation

Ensure every interaction opens the correct destination and keeps the expected state.

### 6. Explore Edge Cases and Boundaries

Test:
- empty states
- long text
- last items
- invalid input
- very large values
- no search results
- slow loading
- expired sessions

Boundary conditions frequently expose bugs. Use your creativity to find issues.

### 7. Arithmetic Verification (e-commerce and fintech)

**Never trust a number that "looks right" — calculate it.**

When the app involves prices, quantities, counters, or totals, verify the math explicitly before moving on. This is where subtle billing bugs hide.

#### Cart / basket checks

After every add, remove, or quantity change, verify these equations:

```
badge_count         = sum(item.quantity) for all items in cart
                      NOT count of distinct products

item.line_total     = item.unit_price × item.quantity
                      TEST WITH qty ≥ 3 — never rely on qty=2 alone
                      (2 × half_price can numerically equal 1 × full_price,
                      masking two bugs at once)

cart.subtotal       = sum(item.line_total) for all items

cart.total          = cart.subtotal + shipping − discount
```

Run these checks at: cart page, every checkout step, and the order confirmation. Any inconsistency between pages is a billing bug regardless of which number looks "right."

#### PLP add-to-cart isolation

To verify the listing "+" button adds exactly 1 unit:
1. Start from an empty cart (or clear it first)
2. Click "+ Carrinho" exactly once for one product
3. Navigate to cart: verify qty=1, badge=1
4. If badge=2 or qty=2 → the button is adding duplicates (file it)

Do NOT test this with a non-empty cart — existing items make it impossible to detect the bug.

#### Price trail audit

Follow every price from its first appearance to the final confirmation:

```
PLP price  →  PDP price  →  cart unit price  →  cart line total
→  cart subtotal  →  checkout subtotal  →  checkout total
→  confirmation subtotal  →  confirmation total
```

Document each value. Any step where a number changes without a user-triggered reason (apply coupon, choose shipping) is a bug.

#### Coupon / discount persistence

Apply a coupon or discount in the cart. Then follow it through:
1. Note the discount amount at the cart
2. Navigate to checkout step 1 — is the discount still in the summary?
3. Step 2, step 3 — same check
4. On the confirmation page — is the discount reflected in the total?

A coupon that disappears mid-funnel is a trust and revenue bug.

### 8. Boundary Value Testing

For every threshold in the application — free shipping cutoff, minimum order, stock limits, form field lengths — test the boundary itself, not just values that are obviously inside or outside.

**The three-point rule:**

| Position | Test value | Expected result |
|----------|-----------|----------------|
| Below boundary | `threshold − 1 unit` | Condition does NOT apply |
| Exactly at boundary | `threshold` | Depends on `>` vs `>=` — use UI copy as oracle |
| Above boundary | `threshold + 1 unit` | Condition DOES apply |

**How to identify the intended operator:**
- UI says "acima de R$299" / "above $50" / "mehr als" → `>` is intended → at exactly R$299 condition should NOT apply
- UI says "a partir de R$299" / "from $50" / "ab" → `>=` is intended → at exactly R$299 condition SHOULD apply
- If the UI copy and the code behavior disagree, that is a bug

**Common boundary targets in e-commerce:**
- Free shipping threshold (test exact amount, −R$0,01, +R$0,01)
- Minimum order amount
- Stock limit (add qty = stock, then +1 more)
- Quantity minimum (try qty=0 and qty=−1 via the "−" button)
- Coupon usage limits (if shown)

**How to construct a cart at exactly R$299,00:**
Use the product price visible on the PLP/PDP (not the cart price, which may be wrong). Add items or adjust quantities until the cart subtotal matches the target. If no single product combination reaches the exact value, note this as untested and move on — do not spend more than 2 minutes on it.

### 9. Input Variation Testing

Do not test a feature only with the input you happen to think of first. Vary systematically:

#### Search
- Test the same query in: all lowercase, all uppercase, mixed case, with/without accents
- All variants should return the same result count
- A search that returns different results for "tênis" vs "Tênis" is case-sensitive — file it

#### Forms with auto-fill (CEP, postcode, ZIP)
- Test with a value that should produce a DIFFERENT city/state than any obvious default
- If you test with a São Paulo CEP and get "São Paulo/SP", you cannot detect a hardcoded default
- Use a CEP from a distant state: Rio de Janeiro (20000-000), Porto Alegre (90000-000), Manaus (69000-000)
- If all CEPs return the same city, the lookup is hardcoded — file it

#### Quantity controls — PDP vs cart are independent components
Test each separately:
- **PDP quantity selector**: click "−" when qty=1. Must NOT allow 0 or negative. Stays at 1 or disables button.
- **Cart quantity control**: click "−" when qty=1. Either removes item (acceptable) or blocks at 1 (acceptable). Silent removal with wrong toast is a bug.

### 10. Transient UI — Capture Before It Disappears

Toast notifications, inline alerts, and success/error flashes are **transient** — they appear for 2-3 seconds and then vanish. A snapshot taken after the page settles will not contain them.

**Rule: after every cart add, remove, or update action, take a screenshot immediately** before issuing the next command.

```bash
playwright-cli click e45          # the cart action
playwright-cli screenshot --filename=screenshots/toast-after-remove.png   # immediately
playwright-cli snapshot           # only then read the settled state
```

Check the toast text matches the action performed:
- Add to cart → message should say "added" / "adicionado"
- Remove from cart → message should say "removed" / "removido"
- A remove that triggers an "added" message is a distinct, reportable bug

If no toast appeared at all when one was expected (e.g., silent add failure when size not selected), that too is a finding.

### 11. Investigate Any Inconsistent Behavior

Treat unexpected behavior as suspicious:
- intermittent failures
- flickering UI
- inconsistent data
- delayed updates
- broken transitions

### 12. Errors

Investigate only the **NEW errors** flagged in Phase 3 (those not matched by the `known_errors` baseline). Verify that each new error is not causing a visible issue for the customer on the front end.

Do NOT re-investigate baseline errors — they were already classified as expected/analytics-only. If a baseline error starts producing a customer-visible symptom on a specific page, that is a new finding; file it as a bug and note the discrepancy.
