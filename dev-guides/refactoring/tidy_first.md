# Tidy First - Clean Code Before Merge

## Philosophy

Tidy First is about **small, focused improvements** that take **minutes to an hour**, done **solo**, at the structural and behavioral level. The goal is to achieve **low coupling and high cohesion**.
Consider four factors: Does tidying reduce costs, increase revenue, reduce elements that need changing, or concentrate changes in a smaller scope?

**Tidy First is NOT big refactoring** (multi-person, multi-week, architectural changes). It's elemental cleanup that **makes the next change easier**.

---

## 1. Guarding

**Guard conditions early** before diving into details. This makes **preconditions explicit** and code easier to analyze.
**Nested branches complicate understanding**, while **guard clauses at the top clarify logic flow**.

```
NG:
if (condition)
  if (nested_condition)
    return result
  main_process()
if (another_condition)
  return other_result
more_process()

OK:
if (not condition) return
if (not nested_condition) return
if (not another_condition) return

main_process()
```

---

## 2. Delete Dead Code

**Remove code that never executes**. It serves no purpose and clutters the codebase.
**Verify the code is truly unused before deletion**. Check for reflection in static languages or dynamic variable expansion in dynamic languages.

```
NG:
function process(data)
  unused_value = calculate()  // never referenced
  return transform(data)

OK:
function process(data)
  return transform(data)
```

---

## 3. Normalize Symmetry

**Use consistent approaches for identical operations**. **Inconsistency implies different intent when none exists**.
Follow existing conventions and linter rules to maintain consistency.

```
NG:
function_a()
  return foo != nil ? foo : foo = initialize()

function_b()
  foo = foo != nil ? foo : initialize()
  return foo

OK:
function_a()
  if foo is nil
    foo = initialize()
  return foo

function_b()
  if foo is nil
    foo = initialize()
  return foo
```

---

## 4. Reading Order

**Code is read more than written**. Arrange important information in the order readers need it.
**Place high-level logic before implementation details** so readers understand intent before mechanics.

```
NG:
private function helper(x)
  return x * 2

public function main()
  return helper(value)

OK:
public function main()
  return helper(value)

private function helper(x)
  return x * 2
```

---

## 5. Cohesion Order

**Place code that changes together physically close together**. When separation is difficult, increase cohesion first.
Ensure **cost of tidying plus cost of change is less than cost of scattered code plus cost of change**.

```
NG:
function process_order(order)
  validate(order)
  return calculate(order)

function unrelated_function()
  // unrelated code

function validate(order)
  // validation logic

function calculate(order)
  // calculation logic

OK:
function process_order(order)
  validate(order)
  return calculate(order)

function validate(order)
  // validation logic

function calculate(order)
  // calculation logic

function unrelated_function()
  // unrelated code
```

---

## 6. Move Declaration and Initialization Together

**Keep variable declaration close to initialization and usage**. **Scattered declarations obscure intent**.

```
NG:
function process()
  int a
  some_code_not_using_a()
  a = value

  int b
  more_code_using_a_not_b()
  b = calculate(a)
  code_using_b()

OK:
function process()
  some_code_not_using_a()

  int a = value
  int b = calculate(a)
  code_using_b()
```

---

## 7. Explaining Variables

**Extract complex expressions into variables with clear names**. This **reveals intent at first glance**.

```
NG:
return new Point(
  (x1 + x2) / 2 + offset * cos(angle),
  (y1 + y2) / 2 + offset * sin(angle)
)

OK:
center_x = (x1 + x2) / 2 + offset * cos(angle)
center_y = (y1 + y2) / 2 + offset * sin(angle)

return new Point(center_x, center_y)
```

---

## 8. Explaining Constants

**Create symbolic constants with descriptive names**. **Avoid over-simplification that loses meaning**.

```
NG:
if user.status == 1

NG:
STATUS = 1
if user.status == STATUS

OK:
ACTIVE_STATUS = 1
if user.status == ACTIVE_STATUS
```

---

## 9. Explicit Parameters

Split routines to clarify what inputs are actually needed. **Make required parameters explicit rather than implicit**.

```
NG:
function process(config)
  use config.a
  use config.b  // error: undefined
  use config.x  // error: undefined

OK:
function process(config)
  process_impl(config.a, config.b, config.x)

function process_impl(a, b, x)
  // explicit parameters make requirements clear
```

---

## 10. Chunk Statements

**Insert blank lines between logical chunks**. This helps readers understand "first do A, then do B."
Avoid over-design. **Keep it simple** and remember to include actual changes.

```
NG:
function process(data)
  result = validate(data)
  if not result.valid
    return error
  transformed = transform(data)
  saved = save(transformed)
  notify(saved)
  return success

OK:
function process(data)
  // Validate input
  result = validate(data)
  if not result.valid
    return error

  // Transform and save
  transformed = transform(data)
  saved = save(transformed)

  // Notify completion
  notify(saved)
  return success
```

---

## 11. Extract Helper

**Extract code with clear purpose and limited interaction** into helper functions. Also known as method extraction.

```
NG:
routine_a()
  unchanged_part_1()
  shared_interaction()
  unchanged_part_2()

routine_b()
  shared_interaction()
  unchanged_part_3()

OK:
helper()
  shared_interaction()

routine_a()
  unchanged_part_1()
  helper()
  unchanged_part_2()

routine_b()
  helper()
  unchanged_part_3()
```

---

## 12. Chunk Consolidation

**Over-fragmented code obscures understanding**. Small parts with complex interactions hide the big picture.
**Before refactoring, consolidate related code in one place** through inlining. This reveals the full flow and structure.
**After understanding the whole, re-extract at appropriate granularity** with meaningful abstractions.

Signs you need consolidation: **long repeated parameter lists**, **repeated code especially conditionals**, **unclear helper names**, **shared mutable data structures**.

```
NG:
function calculate_total(items)
  subtotal = calculate_subtotal(items)
  tax = calculate_tax(subtotal)
  discount = calculate_discount(subtotal)
  shipping = calculate_shipping(subtotal, discount)
  return apply_final_calculation(subtotal, tax, discount, shipping)

function calculate_subtotal(items)
  sum = 0
  for item in items
    sum += get_item_price(item)
  return sum

function get_item_price(item)
  return item.price * item.quantity

OK-Step1: Consolidate to understand the whole
function calculate_total(items)
  subtotal = 0
  for item in items
    subtotal += item.price * item.quantity

  tax = subtotal * 0.08

  discount = 0
  if subtotal > 10000
    discount = subtotal * 0.1

  shipping = (subtotal - discount > 5000) ? 0 : 500

  return subtotal + tax - discount + shipping

OK-Step2: Re-extract with meaningful units
function calculate_total(items)
  subtotal = sum(item.price * item.quantity for item in items)
  tax = subtotal * TAX_RATE
  discount = get_volume_discount(subtotal)
  shipping = get_free_shipping_or_fee(subtotal - discount)

  return subtotal + tax - discount + shipping
```

---

## 13. Explaining Comments

**Comment "why" not "what"**. Answer questions future readers will have.
**Document context, constraints, and tradeoffs** that code cannot express. **Avoid restating variable names or listing execution order**.

```
NG:
// Set max to 100 if greater than 100
if max > 100 then
  max = 100

OK:
// IMPORTANT: UI component crashes with 100+ elements (bug #789).
// This is a temporary workaround until the component is fixed.
if max > 100 then
  max = 100
```

---

## Applying Tidy First

1. **Before making behavioral changes**, identify which tidy patterns would make the change easier
2. **Apply tidies as separate, minimal commits** (one tidy per commit when possible)
3. **Each tidy should take minutes, not hours**
4. If a tidy becomes complex, **stop and commit what you have**
5. **After tidying, make your behavioral change in a separate commit**

Remember: **Tidy First is about making the next change easier, not perfecting the codebase**.
