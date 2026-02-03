# Regex Patterns

## Common Patterns for Data Validation and Transformation

Reference patterns for use in custom validation rules and transformations.

---

### Email Addresses

```regex
# Basic email validation
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

# Strict RFC 5322 compliant
^(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])$
```

---

### Phone Numbers

```regex
# US phone (various formats)
^(\+1)?[-.\s]?\(?[0-9]{3}\)?[-.\s]?[0-9]{3}[-.\s]?[0-9]{4}$

# International (E.164 format)
^\+[1-9]\d{1,14}$

# Generic (digits with separators)
^[\d\s\-\.\(\)]+$
```

---

### Dates and Times

```regex
# ISO 8601 date (YYYY-MM-DD)
^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$

# US date (MM/DD/YYYY)
^(0[1-9]|1[0-2])/(0[1-9]|[12]\d|3[01])/\d{4}$

# European date (DD/MM/YYYY)
^(0[1-9]|[12]\d|3[01])/(0[1-9]|1[0-2])/\d{4}$

# ISO 8601 datetime with timezone
^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d+)?(Z|[+-]\d{2}:\d{2})?$

# Time (HH:MM:SS)
^([01]\d|2[0-3]):([0-5]\d):([0-5]\d)$
```

---

### Identifiers

```regex
# UUID v4
^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$

# UUID (any version)
^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$

# Alphanumeric ID
^[a-zA-Z0-9]+$

# Alphanumeric with underscore/dash
^[a-zA-Z0-9_-]+$
```

---

### Financial

```regex
# Credit card (basic)
^\d{13,19}$

# Credit card (formatted)
^\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}$

# Currency (US format)
^\$?[\d,]+(\.\d{2})?$

# Currency (European format)
^[\d\s]+,\d{2}\s?€?$

# IBAN
^[A-Z]{2}\d{2}[A-Z0-9]{4}\d{7}([A-Z0-9]?){0,16}$
```

---

### US-Specific

```regex
# Social Security Number
^\d{3}-?\d{2}-?\d{4}$

# ZIP code (5 digit)
^\d{5}$

# ZIP code (5+4)
^\d{5}(-\d{4})?$

# State abbreviation
^[A-Z]{2}$

# EIN (Employer ID)
^\d{2}-?\d{7}$
```

---

### Network

```regex
# IPv4 address
^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$

# IPv6 address (simplified)
^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$

# MAC address
^([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})$

# URL
^https?://[^\s/$.?#].[^\s]*$

# Domain name
^(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,}$
```

---

### Data Quality

```regex
# Contains only whitespace
^\s*$

# Contains any digits
\d

# Contains only letters
^[a-zA-Z]+$

# Contains special characters
[^a-zA-Z0-9\s]

# Multiple consecutive spaces
\s{2,}

# Leading/trailing whitespace
^\s+|\s+$
```

---

### Using Patterns in Sensei

#### Custom Validation Rule

```yaml
validation_rules:
  - name: valid_email
    column: email
    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    action: flag # or 'reject'
    message: 'Invalid email format'
```

#### Transformation Filter

```yaml
transformations:
  - column: phone
    filter:
      pattern: "^\\+1" # Only US numbers
    transform: "REGEXP_REPLACE(phone, '[^0-9]', '')"
```

---

### Pattern Testing

Test patterns before using:

```python
import re

pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
test_values = [
    'user@example.com',
    'invalid.email',
    'user+tag@domain.co.uk'
]

for value in test_values:
    match = re.match(pattern, value)
    print(f"{value}: {'PASS' if match else 'FAIL'}")
```

---

### Notes

- Escape backslashes in YAML (`\\d` instead of `\d`)
- Most patterns are case-insensitive by default in Sensei
- Test patterns with your actual data before production use
- Consider performance for very complex patterns on large datasets

→ [Custom Validation Rules](../../product/platform/kora/validation-types.md)
→ [Transformation Rules](../../product/platform/maba/transformation-engine/README.md)
