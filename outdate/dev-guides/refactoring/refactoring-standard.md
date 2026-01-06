# Refactoring Standard Instructions

**Goal**: Language-agnostic refactoring principles that make code human-readable and maintainable.

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler

## Quantitative Principles: Measurable Criteria

### Principle 1: Cyclomatic Complexity ≤ 10
**Keep control flow simple and testable**
- Cyclomatic complexity measures the number of independent paths through code
- Calculated as: decision points + 1
- Recommended range: 1-10 (low risk)
- Values over 10 significantly increase defect rates and make code difficult to test

Example:
```
NG:
function processOrder(order):
  if order.status == "pending":
    if order.items.length > 0:
      for item in order.items:
        if item.stock > 0:
          if item.price > 0:
            process item normally
          else if item.onSale:
            process item with sale
  else if order.status == "cancelled":
    handle cancellation
  else if order.status == "completed":
    handle completion

OK:
function processOrder(order):
  if not isProcessable(order):
    return
  processItems(order.items)

function isProcessable(order):
  return order.status == "pending" and order.items.length > 0

function processItems(items):
  validItems = items.filter(item => hasStock(item))
  for each item in validItems:
    processItem(item)
```

---

### Principle 2: Cognitive Complexity ≤ 15
**Minimize mental effort required to understand code**
- Cognitive complexity measures how difficult code is to understand
- Unlike cyclomatic complexity, it penalizes nesting depth more heavily
- Recommended maximum: 15 per function
- Deep nesting exponentially increases cognitive load

Example:
```
NG:
function validateUser(user):
  if user exists:                          // +1
    if user.email exists:                  // +2 (nested)
      if user.email contains "@":          // +3 (nested)
        if user.age >= 18:                 // +4 (nested)
          return true
  return false
// Cognitive Complexity: 10

OK:
function validateUser(user):
  if not user:                             // +1
    return false
  if not user.email:                       // +1
    return false
  if not user.email contains "@":          // +1
    return false
  if user.age < 18:                        // +1
    return false
  return true
// Cognitive Complexity: 4
```

---

### Principle 3: 80/24 Rule
**Code that fits in your brain: 80 characters wide, 24 lines long**
- Maximum 80 characters per line (industry standard since IBM punch cards)
- Maximum 24 lines per function/method
- Code fitting in an 80x24 box reduces cognitive load dramatically
- Enables side-by-side code review, split screens, and better readability

Example:
```
NG:
function processAndGenerateCustomerOrderReportWithValidationAndFormatting(data):
  // Validate customer data
  if not data:
    throw error "No data"
  for each item in data:
    if not item.isValid:
      throw error "Invalid"

  // Transform and calculate totals
  transformedData = empty list
  for each item in data:
    transformed = transform item
    transformedData.add(transformed)

  // Generate and format report with headers, footers, and styling
  report = create new report
  report.addHeader()
  for each item in transformedData:
    report.addRow(item)
  report.addFooter()

  // Write to file
  file = open report file
  file.write(report.content)
  file.close()
  return report
// Violates both rules:
// - Function name > 80 chars
// - Function body > 24 lines (30 lines total)

OK:
function generateReport(data):
  validated = validateData(data)
  transformed = transformData(validated)
  formatted = formatReport(transformed)
  return writeReport(formatted)
// Function: 5 lines, all under 80 chars

function validateData(data):
  if not data:
    throw error "No data provided"
  for each item in data:
    if not item.isValid:
      throw error "Invalid item"
  return data
// Function: 7 lines, all under 80 chars

function formatReport(data):
  report = create new report
  report.addHeader()
  for each item in data:
    report.addRow(item)
  report.addFooter()
  return report
// Function: 7 lines, all under 80 chars
```

---

### Principle 4: Nesting Depth ≤ 3-4 Levels
**Avoid deeply nested control structures**
- Nesting depth measures how many levels of control structures are nested
- Recommended maximum: 3-4 levels
- Deep nesting makes it hard to track context and available variables
- Use guard clauses and early returns to reduce nesting

Example:
```
NG:
function processData(data):
  if data exists:                                // Level 1
    if data.isValid:                             // Level 2
      for each item in data.items:               // Level 3
        if item.active:                          // Level 4
          if item.value > 0:                     // Level 5
            process item
          else:
            skip item

OK:
function processData(data):
  if not data or not data.isValid:
    return

  activeItems = data.items.filter(item => item.active and item.value > 0)
  for each item in activeItems:                  // Level 1
    processItem(item)

function processItem(item):                      // Level 1
  // process logic here
```

---

### Principle 5: Parameter Count ≤ 3
**Limit function parameters to maintain simplicity**
- Functions with many parameters are hard to understand and test
- Recommended: 0-3 parameters (more requires special justification)
- Test case combinations explode: 3 params = 8 cases, 6 params = 64 cases
- Use parameter objects or configuration objects for complex cases

Example:
```
NG:
function createUser(name, email, age, address, phone, role, department, manager):
  user = new User()
  user.name = name
  user.email = email
  user.age = age
  user.address = address
  user.phone = phone
  user.role = role
  user.department = department
  user.manager = manager
  return user

OK:
function createUser(userConfig):
  return new User(
    name: userConfig.name,
    email: userConfig.email,
    age: userConfig.age,
    address: userConfig.address,
    phone: userConfig.phone,
    role: userConfig.role,
    department: userConfig.department,
    manager: userConfig.manager
  )

// Even better: Use builder pattern
userBuilder = new UserBuilder()
user = userBuilder
  .withName("John")
  .withEmail("john@example.com")
  .withAge(30)
  .build()
```

---

## Qualitative Principles: Conceptual Criteria

### Principle 6: Single Responsibility Principle
**Each function or class should have one reason to change**
- A function should do one thing and do it well
- If a function has multiple responsibilities, split it into separate functions
- Makes code easier to understand, test, and maintain
- Reduces coupling and increases cohesion

Example:
```
NG:
class UserService:
  function saveUser(user):
    // Validate user
    if not user.email contains "@":
      throw error "Invalid email"

    // Save to database
    database.insert("users", user)

    // Send notification
    emailService.send(user.email, "Welcome!")

    // Log activity
    logger.log("User created: " + user.name)

OK:
class UserValidator:
  function validate(user):
    if not user.email contains "@":
      throw error "Invalid email"

class UserRepository:
  function save(user):
    return database.insert("users", user)

class NotificationService:
  function sendWelcomeEmail(user):
    return emailService.send(user.email, "Welcome!")

class UserService:
  constructor(validator, repository, notificationService):
    this.validator = validator
    this.repository = repository
    this.notificationService = notificationService

  function createUser(user):
    this.validator.validate(user)
    this.repository.save(user)
    this.notificationService.sendWelcomeEmail(user)
```

---

### Principle 7: Intention-Revealing Names
**Names should clearly communicate purpose and usage**
- Variable, function, and class names should answer: Why? What? How?
- Use meaningful names that reduce the need for comments
- Follow conventions: classes are nouns, functions are verbs, booleans start with is/has/can
- Avoid abbreviations and single-letter names (except loop indices)

Example:
```
NG:
function calc(d):
  t = d * 24
  return t

x = getUserData()
mgr = new Manager()

if u.a and u.p:
  process()

OK:
function convertDaysToHours(days):
  hoursPerDay = 24
  return days * hoursPerDay

authenticatedUser = getUserData()
connectionManager = new ConnectionManager()

if user.isAuthenticated and user.hasPermission:
  processRequest()

// Boolean naming examples
if user.isActive and order.hasItems and payment.canProcess:
  completeTransaction()
```

---

### Principle 8: DRY (Don't Repeat Yourself)
**Each piece of knowledge should exist in one place**
- Avoid duplicating code or logic
- Extract repeated code into reusable functions or constants
- Changes should only need to be made in one location
- Copy-paste is a warning sign of potential DRY violation

Example:
```
NG:
data = loadData()

totalApples = 0
for each item in data:
  if item.kind == "apple":
    totalApples = totalApples + item.cost
print("Apple total: " + toString(totalApples))

totalOranges = 0
for each item in data:
  if item.kind == "orange":
    totalOranges = totalOranges + item.cost
print("Orange total: " + toString(totalOranges))

totalBananas = 0
for each item in data:
  if item.kind == "banana":
    totalBananas = totalBananas + item.cost
print("Banana total: " + toString(totalBananas))

OK:
data = loadData()

totals = empty map
for each item in data:
  kind = item.kind
  if kind not in totals:
    totals[kind] = 0
  totals[kind] = totals[kind] + item.cost

for each kind in keys(totals):
  print(kind + " total: " + toString(totals[kind]))

// Alternative with helper function
function calculateTotalByKind(data, kind):
  total = 0
  for each item in data:
    if item.kind == kind:
      total = total + item.cost
  return total

for each kind in ["apple", "orange", "banana"]:
  total = calculateTotalByKind(data, kind)
  print(kind + " total: " + toString(total))
```

---

### Principle 9: KISS (Keep It Simple, Stupid)
**Choose the simplest solution that works**
- Avoid unnecessary complexity and over-engineering
- Prefer clear code over clever code
- Don't add functionality until it's needed (YAGNI principle)
- Explicit is better than implicit

Example:
```
NG:
// Overly clever and complex
result = arr.reduce(function(accumulator, value):
  return merge(accumulator, createMap(value.id, value))
, emptyMap())

// Over-engineered abstraction
class AbstractFactoryProviderManagerSingletonAdapter:
  function createAbstractFactory():
    factory = new AbstractFactory()
    return new FactoryAdapter(factory)

OK:
// Simple and clear
result = empty map
for each item in arr:
  result[item.id] = item

// Simple and sufficient
class UserFactory:
  function create(userData):
    return new User(userData)

// Simple conditional
if user.age >= 18:
  allowAccess()
else:
  denyAccess()

// Instead of clever one-liner
accessAllowed = user.age >= 18
handleAccess(user, accessAllowed)
```

---

### Principle 10: Self-Documenting Code
**Code should explain itself without excessive comments**
- Write code that is clear enough to understand without comments
- Use descriptive names and clear structure instead of comments
- Comments should explain "why", not "what" or "how"
- Good comments: legal info, intent explanation, warnings, public API docs

Example:
```
NG:
// Get user by ID
function getUserById(id):
  // Query the database
  return database.find("users", id)

// Loop through array and check each element
for i from 0 to arr.length:
  // Check if status is active and age is over 18
  if arr[i].status == "active" and arr[i].age > 18:
    // Process the user
    process(arr[i])

OK:
// Self-documenting with clear naming
function findUserById(userId):
  return userRepository.findById(userId)

function processAdultActiveUsers(users):
  eligibleUsers = users.filter(isAdultActiveUser)
  for each user in eligibleUsers:
    processUser(user)

function isAdultActiveUser(user):
  return user.status == "active" and user.age > 18

// Acceptable comment: explains intent/reason
// Performance optimization: Cache frequently accessed data
// Benchmarks show this improves response time by 40%
cache = new LRUCache(maxSize: 1000)

// Acceptable comment: warning
// WARNING: This operation is expensive and may take several minutes
// for large datasets. Consider running during off-peak hours.
function rebuildSearchIndex(documents):
  // implementation
```

## Quality Analysis Table

| Principle | Recommended | Warning | Critical | Impact |
|-----------|-------------|---------|----------|--------|
| Cyclomatic Complexity | 1-10 | 11-20 | 21+ | Defect rate increases |
| Cognitive Complexity | 0-15 | 15-25 | 25+ | Understanding difficulty |
| 80/24 Rule | ≤80 chars, ≤24 lines | 80+ chars, 24-40 lines | >40 lines | Maintenance burden |
| Nesting Depth | 1-3 levels | 4-5 levels | 5+ levels | Cognitive load |
| Parameter Count | 0-3 params | 4 params | 5+ params | Testing complexity |

## Appendix 1: Application Guidelines

When refactoring code with Claude Code:

1. Start with quantitative metrics - they are measurable and objective
2. Apply qualitative principles to improve overall design
3. Use these techniques:
   - Extract Method: Break large functions into smaller ones
   - Guard Clauses: Use early returns to reduce nesting
   - Parameter Objects: Group related parameters
   - Extract Constants: Replace magic numbers with named constants
   - Extract Variables: Make complex expressions readable

4. Remember: These are guidelines, not rigid rules
5. Aim for gradual improvement (Boy Scout Rule: leave code cleaner than you found it)
6. Code is read far more often than it is written - optimize for readability

## Appendix 2: References
- Martin Fowler. (2018). Refactoring: Improving the Design of Existing Code (2nd Edition)
- Robert C. Martin. (2008). Clean Code: A Handbook of Agile Software Craftsmanship
- Buse & Weimer. (2010). Learning a metric for code readability
- McCabe. (1976). A complexity measure
- SonarSource. Cognitive Complexity White Paper
- Mark Seemann. (2019). The 80/24 rule. https://blog.ploeh.dk/2019/11/04/the-80-24-rule/
