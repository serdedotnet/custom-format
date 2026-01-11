# Serde Format Test Plan

This document captures test cases that are format-agnostic and should be implemented across all Serde serialization formats (JSON, XML, TOML, etc.).

## Exception Handling

### Deserialization exceptions should not be hidden by Dispose exceptions

**Scenario:** When deserialization fails partway through parsing, the `Dispose` method may also fail (e.g., because the reader isn't at the expected position). The original deserialization exception should be propagated to the caller, not hidden by a secondary exception from `Dispose`.

**Test approach:**
- Provide malformed input that causes deserialization to fail mid-parse
- Ensure the input also leaves the deserializer in a state where `Dispose` would throw
- Assert that the original parsing exception is thrown, not a Dispose-related exception

**Example (XML):**
```xml
<BoolStruct>
  <BoolField>notaboolean</BoolField>
  <ExtraContent>causes Dispose to fail</ExtraContent>
</BoolStruct>
```

---

## Round-Trip Consistency

### Serialize then deserialize should produce equal objects

**Scenario:** Any serializable object should survive a round-trip through serialization and deserialization unchanged.

**Test approach:**
- Create an instance with known values
- Serialize to string
- Deserialize back to object
- Assert equality with original

---

## Null and Optional Handling

### Null values should serialize/deserialize correctly

**Scenario:** Nullable fields and reference types set to null should be handled consistently.

**Test approach:**
- Test nullable value types (`int?`, `bool?`)
- Test nullable reference types (`string?`, custom classes)
- Verify round-trip preserves null vs default distinction where applicable

### Missing fields during deserialization

**Scenario:** When deserializing, fields missing from the input should receive appropriate default values or throw meaningful errors.

**Test approach:**
- Deserialize input missing required fields
- Deserialize input missing optional fields
- Verify behavior matches expectations (default value vs exception)

---

## Edge Cases

### Empty collections

**Scenario:** Empty arrays, lists, and dictionaries should serialize and deserialize correctly.

**Test approach:**
- Serialize object with empty `[]` or `{}`
- Deserialize and verify empty collection is restored

### Nested types

**Scenario:** Deeply nested object graphs should serialize/deserialize correctly.

**Test approach:**
- Test 2-3 levels of nesting
- Verify all nested properties survive round-trip

### Special characters in strings

**Scenario:** Strings containing format-specific special characters should be properly escaped.

**Test approach:**
- Include characters that require escaping in the format (e.g., `<`, `>`, `&` for XML; `"`, `\` for JSON)
- Verify round-trip preserves exact string content

---

## Numeric Boundaries

### Integer boundary values

**Scenario:** Min/max values for all integer types should serialize correctly.

**Test approach:**
- Test `byte.MaxValue`, `int.MinValue`, `long.MaxValue`, etc.
- Verify no overflow or precision loss

### Floating point special values

**Scenario:** Special floating point values should be handled appropriately.

**Test approach:**
- Test `double.NaN`, `double.PositiveInfinity`, `double.NegativeInfinity`
- Document expected behavior (some formats may not support these)

---

## Enum Handling

### Enum serialization format

**Scenario:** Enums should serialize as their string name (not numeric value) by default.

**Test approach:**
- Serialize enum value
- Verify output contains name, not integer
- Verify deserialization from name works

### Unknown enum values during deserialization

**Scenario:** Deserializing an unrecognized enum string should produce a clear error.

**Test approach:**
- Attempt to deserialize invalid enum string
- Verify meaningful exception is thrown

---

## Adding New Test Cases

When adding a test case to this plan:
1. Describe the scenario in format-agnostic terms
2. Provide a concrete test approach
3. Note any format-specific considerations
4. Implement the test in all format libraries
