# Static Telemetry Comment Examples

## Python Example - Business Logic

### Input Code
```python
def calculate_total(items: List[Item], tax_rate: float) -> float:
    """Calculate total with tax."""
    if not items:
        raise ValueError("Items cannot be empty")
    subtotal = sum(item.price for item in items)
    return subtotal + (subtotal * tax_rate)
```

### With Static Telemetry Comment
```python
# TELEMETRY:START
# {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"calculate_total","function_type":"business_logic","file_path":"src/billing/calculator.py","line_number":15,"language":"python","parameters":[{"name":"items","type":"List[Item]","required":true},{"name":"tax_rate","type":"float","required":true}],"return_type":"float","complexity":{"cyclomatic":2,"lines_of_code":4},"data_flow":{"inputs":[{"name":"items","source":"user_request","validation":"non_empty_list"},{"name":"tax_rate","source":"config","validation":"range_0_to_1"}],"outputs":[{"name":"total","type":"float","destination":"return"}]},"error_handling":{"raises":[{"type":"ValueError","condition":"items is empty","line":17}],"validates":[{"param":"items","check":"non_empty","on_failure":"raise ValueError"}]},"classifiers":{"purpose":"Calculate total cost including tax for billing","category":"business_logic","subcategory":"financial_calculation","tags":["billing","tax","calculation","critical"],"criticality":"high","data_sensitivity":"pii"},"calls":{"outgoing":[{"function":"sum","module":"builtins","line":19}]},"usage":{"caller_count":5,"test_coverage":95,"is_public_api":true}}
# TELEMETRY:END
def calculate_total(items: List[Item], tax_rate: float) -> float:
    """Calculate total with tax."""
    if not items:
        raise ValueError("Items cannot be empty")
    subtotal = sum(item.price for item in items)
    return subtotal + (subtotal * tax_rate)
```

## JavaScript Example - API Handler

### Input Code
```javascript
async function processOrder(orderId, options = {}) {
    const order = await fetchOrder(orderId);
    if (!order) throw new NotFoundError('Order not found');
    await validateOrder(order);
    return await saveOrder(order, options);
}
```

### With Static Telemetry Comment
```javascript
// TELEMETRY:START
// {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"processOrder","function_type":"api_handler","file_path":"src/api/orders.js","line_number":42,"language":"javascript","parameters":[{"name":"orderId","type":"string","required":true},{"name":"options","type":"Object","required":false,"default":"{}"}],"return_type":"Promise<Order>","is_async":true,"complexity":{"cyclomatic":2,"lines_of_code":5},"data_flow":{"inputs":[{"name":"orderId","source":"user_request","validation":"uuid"},{"name":"options","source":"user_request"}],"outputs":[{"name":"order","type":"Order","destination":"database"}],"side_effects":[{"type":"database_write","description":"Saves order to database","target":"orders_table"}]},"error_handling":{"raises":[{"type":"NotFoundError","condition":"order not found","line":44}],"validates":[{"param":"order","check":"exists","on_failure":"throw NotFoundError"}]},"classifiers":{"purpose":"Process and validate order before saving to database","category":"api_handler","subcategory":"order_processing","tags":["orders","api","async","database"],"criticality":"high","performance_profile":"io_bound"},"calls":{"outgoing":[{"function":"fetchOrder","module":"./db","line":43},{"function":"validateOrder","module":"./validators","line":45},{"function":"saveOrder","module":"./db","line":46}]},"usage":{"caller_count":3,"test_coverage":88,"is_public_api":true}}
// TELEMETRY:END
async function processOrder(orderId, options = {}) {
    const order = await fetchOrder(orderId);
    if (!order) throw new NotFoundError('Order not found');
    await validateOrder(order);
    return await saveOrder(order, options);
}
```

## Go Example - Database Access

### Input Code
```go
func GetUserByID(id string) (*User, error) {
    if id == "" {
        return nil, errors.New("id cannot be empty")
    }

    var user User
    err := db.QueryRow("SELECT * FROM users WHERE id = ?", id).Scan(&user)
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return &user, err
}
```

### With Static Telemetry Comment
```go
// TELEMETRY:START
// {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"GetUserByID","function_type":"database_access","file_path":"internal/db/users.go","line_number":25,"language":"go","parameters":[{"name":"id","type":"string","required":true}],"return_type":"(*User, error)","complexity":{"cyclomatic":3,"lines_of_code":9},"data_flow":{"inputs":[{"name":"id","source":"user_request","validation":"non_empty","sanitization":"sql_escape"}],"outputs":[{"name":"user","type":"*User","destination":"return"}],"side_effects":[{"type":"database_read","description":"Queries users table","target":"users"}]},"error_handling":{"raises":[{"type":"error","condition":"id is empty","line":27}],"catches":[{"type":"sql.ErrNoRows","action":"return nil","line":32}],"validates":[{"param":"id","check":"non_empty","on_failure":"return error"}]},"classifiers":{"purpose":"Fetch user by ID from database","category":"database_access","subcategory":"user_repository","tags":["database","users","read","repository"],"criticality":"high","data_sensitivity":"pii","performance_profile":"io_bound"},"calls":{"outgoing":[{"function":"QueryRow","module":"database/sql","line":31}]},"security":{"input_sanitization":true,"sql_injection_risk":"none"},"usage":{"caller_count":12,"test_coverage":100,"is_public_api":true}}
// TELEMETRY:END
func GetUserByID(id string) (*User, error) {
    if id == "" {
        return nil, errors.New("id cannot be empty")
    }

    var user User
    err := db.QueryRow("SELECT * FROM users WHERE id = ?", id).Scan(&user)
    if err == sql.ErrNoRows {
        return nil, nil
    }
    return &user, err
}
```

## Python Example - Data Processing

### Input Code
```python
def transform_user_data(raw_data: Dict) -> User:
    """Transform raw API response to User object."""
    user_id = raw_data.get('id')
    if not user_id:
        logger.warning("Missing user ID in raw data")
        return None

    return User(
        id=user_id,
        name=raw_data.get('name', 'Unknown'),
        email=raw_data.get('email'),
        created_at=parse_timestamp(raw_data.get('created'))
    )
```

### With Static Telemetry Comment
```python
# TELEMETRY:START
# {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"transform_user_data","function_type":"transformation","file_path":"src/services/user_transformer.py","line_number":10,"language":"python","parameters":[{"name":"raw_data","type":"Dict","required":true}],"return_type":"User","complexity":{"cyclomatic":2,"lines_of_code":10},"data_flow":{"inputs":[{"name":"raw_data","source":"external_api"}],"outputs":[{"name":"user","type":"User","destination":"return"}],"side_effects":[{"type":"log","description":"Logs warning if user ID missing"}]},"error_handling":{"validates":[{"param":"raw_data.id","check":"exists","on_failure":"log warning and return None"}]},"classifiers":{"purpose":"Transform raw API response data into structured User object","category":"transformation","subcategory":"data_mapping","tags":["transformation","user","api","mapping"],"criticality":"medium","data_sensitivity":"pii","performance_profile":"fast"},"calls":{"outgoing":[{"function":"get","module":"dict","line":12},{"function":"warning","module":"logger","line":14},{"function":"parse_timestamp","module":"utils.time","line":20}]},"usage":{"caller_count":8,"test_coverage":92,"is_public_api":false}}
# TELEMETRY:END
def transform_user_data(raw_data: Dict) -> User:
    """Transform raw API response to User object."""
    user_id = raw_data.get('id')
    if not user_id:
        logger.warning("Missing user ID in raw data")
        return None

    return User(
        id=user_id,
        name=raw_data.get('name', 'Unknown'),
        email=raw_data.get('email'),
        created_at=parse_timestamp(raw_data.get('created'))
    )
```

## TypeScript Example - Validation

### Input Code
```typescript
function validateEmail(email: string): boolean {
    if (!email || email.length === 0) {
        throw new ValidationError('Email cannot be empty');
    }

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const isValid = emailRegex.test(email);

    if (!isValid) {
        logger.warn(`Invalid email format: ${email}`);
    }

    return isValid;
}
```

### With Static Telemetry Comment
```typescript
// TELEMETRY:START
// {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"validateEmail","function_type":"validation","file_path":"src/validators/email.ts","line_number":5,"language":"typescript","parameters":[{"name":"email","type":"string","required":true}],"return_type":"boolean","complexity":{"cyclomatic":3,"lines_of_code":12},"data_flow":{"inputs":[{"name":"email","source":"user_request","validation":"non_empty"}],"outputs":[{"name":"isValid","type":"boolean","destination":"return"}],"side_effects":[{"type":"log","description":"Logs warning for invalid email format"}]},"error_handling":{"raises":[{"type":"ValidationError","condition":"email is empty","line":7}],"validates":[{"param":"email","check":"non_empty","on_failure":"throw ValidationError"},{"param":"email","check":"email_format","on_failure":"log warning"}]},"classifiers":{"purpose":"Validate email address format using regex pattern","category":"validation","subcategory":"input_validation","tags":["validation","email","regex","input"],"criticality":"high","data_sensitivity":"pii","performance_profile":"fast"},"calls":{"outgoing":[{"function":"test","module":"RegExp","line":11},{"function":"warn","module":"logger","line":14}]},"security":{"input_sanitization":true,"xss_risk":"none"},"usage":{"caller_count":25,"test_coverage":100,"is_public_api":true}}
// TELEMETRY:END
function validateEmail(email: string): boolean {
    if (!email || email.length === 0) {
        throw new ValidationError('Email cannot be empty');
    }

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const isValid = emailRegex.test(email);

    if (!isValid) {
        logger.warn(`Invalid email format: ${email}`);
    }

    return isValid;
}
```

## Python Example - Complex Function with Multiple Concerns

### Input Code
```python
@cache(ttl=3600)
@validate_params
def calculate_risk_score(
    user: User,
    transaction: Transaction,
    historical_data: Optional[List[Transaction]] = None
) -> RiskScore:
    """
    Calculate risk score for transaction.

    Uses ML model + rule-based checks.
    """
    # Validate inputs
    if not user.is_verified:
        raise UnverifiedUserError("User must be verified")

    # Feature engineering
    features = extract_features(user, transaction)
    if historical_data:
        features.update(extract_historical_features(historical_data))

    # ML prediction
    try:
        ml_score = risk_model.predict(features)
    except ModelError as e:
        logger.error(f"ML model failed: {e}")
        ml_score = 0.5  # Default to medium risk

    # Rule-based overrides
    if transaction.amount > user.daily_limit:
        ml_score = max(ml_score, 0.8)  # Boost to high risk

    # Save to audit log
    save_risk_calculation(user.id, transaction.id, ml_score)

    return RiskScore(
        score=ml_score,
        category=categorize_risk(ml_score),
        factors=features
    )
```

### With Static Telemetry Comment
```python
# TELEMETRY:START
# {"event_type":"static_function_metadata","timestamp":"2025-11-04T12:34:56.789Z","function_name":"calculate_risk_score","function_type":"business_logic","file_path":"src/risk/calculator.py","line_number":15,"language":"python","decorators":["@cache","@validate_params"],"parameters":[{"name":"user","type":"User","required":true},{"name":"transaction","type":"Transaction","required":true},{"name":"historical_data","type":"Optional[List[Transaction]]","required":false,"default":"None"}],"return_type":"RiskScore","complexity":{"cyclomatic":5,"cognitive":12,"lines_of_code":35,"max_nesting":3},"data_flow":{"inputs":[{"name":"user","source":"database","validation":"is_verified"},{"name":"transaction","source":"user_request","validation":"valid_transaction"},{"name":"historical_data","source":"database"}],"outputs":[{"name":"risk_score","type":"RiskScore","destination":"return"}],"side_effects":[{"type":"log","description":"Logs ML model errors"},{"type":"database_write","description":"Saves risk calculation to audit log","target":"risk_audit"}]},"error_handling":{"raises":[{"type":"UnverifiedUserError","condition":"user not verified","line":20}],"catches":[{"type":"ModelError","action":"fallback","line":27}],"validates":[{"param":"user.is_verified","check":"is_true","on_failure":"raise UnverifiedUserError"}]},"classifiers":{"purpose":"Calculate transaction risk score using ML model and rule-based checks","category":"business_logic","subcategory":"risk_assessment","tags":["risk","ml","transaction","fraud","security","critical"],"criticality":"critical","data_sensitivity":"pii","performance_profile":"cpu_bound"},"calls":{"outgoing":[{"function":"extract_features","module":"risk.features","line":24},{"function":"extract_historical_features","module":"risk.features","line":26,"conditional":true},{"function":"predict","module":"risk_model","line":30},{"function":"error","module":"logger","line":32},{"function":"max","module":"builtins","line":38},{"function":"save_risk_calculation","module":"risk.audit","line":41},{"function":"categorize_risk","module":"risk.categories","line":45}]},"security":{"requires_auth":true,"requires_authorization":["risk_calculation"],"input_sanitization":true},"usage":{"caller_count":15,"test_coverage":85,"is_public_api":true}}
# TELEMETRY:END
@cache(ttl=3600)
@validate_params
def calculate_risk_score(
    user: User,
    transaction: Transaction,
    historical_data: Optional[List[Transaction]] = None
) -> RiskScore:
    """
    Calculate risk score for transaction.

    Uses ML model + rule-based checks.
    """
    # Validate inputs
    if not user.is_verified:
        raise UnverifiedUserError("User must be verified")

    # Feature engineering
    features = extract_features(user, transaction)
    if historical_data:
        features.update(extract_historical_features(historical_data))

    # ML prediction
    try:
        ml_score = risk_model.predict(features)
    except ModelError as e:
        logger.error(f"ML model failed: {e}")
        ml_score = 0.5  # Default to medium risk

    # Rule-based overrides
    if transaction.amount > user.daily_limit:
        ml_score = max(ml_score, 0.8)  # Boost to high risk

    # Save to audit log
    save_risk_calculation(user.id, transaction.id, ml_score)

    return RiskScore(
        score=ml_score,
        category=categorize_risk(ml_score),
        factors=features
    )
```

## Human-Readable Summary

For each function, the comment contains:

1. **Basic Info**: Name, type, location, language
2. **Signature**: Parameters (with types), return type, decorators
3. **Complexity**: Cyclomatic, cognitive, LOC, nesting
4. **Data Flow**: Where data comes from, where it goes, side effects
5. **Error Handling**: What can be raised, what's caught, what's validated
6. **Purpose**: One-sentence description + categorization + tags
7. **Call Graph**: What it calls, what calls it (estimated)
8. **Security**: Auth requirements, sanitization, risk levels
9. **Usage**: How many callers, test coverage, public/private

## Parsing Example

To extract functions for visualization:

```python
import json
import re

def extract_static_telemetry(source_code: str):
    """Extract all static telemetry comments from source code."""
    pattern = r'# TELEMETRY:START\s*\n# (.*?)\n# TELEMETRY:END'
    matches = re.findall(pattern, source_code, re.MULTILINE | re.DOTALL)

    functions = []
    for match in matches:
        data = json.loads(match)
        functions.append(data)

    return functions

# Build call graph
def build_call_graph(functions):
    """Build directed graph from function metadata."""
    graph = {}
    for func in functions:
        name = func['function_name']
        calls = [c['function'] for c in func.get('calls', {}).get('outgoing', [])]
        graph[name] = calls
    return graph
```

## Comparison: Static vs Full Telemetry

| Feature | Static Comment | Full Telemetry |
|---------|----------------|----------------|
| **Overhead** | Zero | Adds telemetry calls |
| **Data Source** | Code analysis | Runtime execution |
| **Accuracy** | High (declared) | Perfect (observed) |
| **Call Graph** | Estimated | Actual |
| **Dead Code** | Can detect | Cannot detect (relies on execution) |
| **Performance** | N/A | Duration, counts |
| **Coverage** | 100% of code | Only executed paths |
| **Setup** | Instant | Requires instrumentation + execution |

## Benefits of Hybrid Approach

Use both static comments + full telemetry for maximum insight:

1. **Static Comments**: Architecture, design intent, complete coverage
2. **Full Telemetry**: Runtime behavior, performance, actual usage

The static comments provide the "blueprint", while runtime telemetry shows the "reality".
