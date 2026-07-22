# Defensive Programming

This guide reduces the impact of invalid state, unexpected input, and partial failures.

## Failure Strategy

| Situation | Preferred Response | Rationale |
| --- | --- | --- |
| Invalid input | Reject early and report context. | Prevents invalid state from spreading. |
| Missing optional dependency | Use a safe fallback. | Preserves expected operation when possible. |
| Missing required dependency | Stop the operation safely. | Avoids undefined behavior. |
| Recoverable external failure | Retry only with clear bounds. | Limits repeated work and failure loops. |

## Guard Clauses

Validate assumptions at the boundary of an operation and return before work begins.

```cpp
bool FInventoryService::AddItem(const FItemId& ItemId)
{
    if (!ItemId.IsValid() || !Catalog.Contains(ItemId))
    {
        return false;
    }

    Inventory.Add(ItemId);
    return true;
}
```

## Diagnostics

- Include enough context to reproduce and investigate a failure.
- Avoid exposing sensitive data in logs or error messages.
- Distinguish expected validation failures from unexpected faults.

## Checklist

- [ ] Inputs are validated before state changes occur.
- [ ] Optional dependencies have safe fallback behavior.
- [ ] Required dependencies fail safely when unavailable.
- [ ] Cleanup runs on every exit path.
- [ ] Failures provide actionable diagnostic context.

## Related Guidance

- [C++ Coding Standard](CppCodingStandard.md)
- [Lifecycle and Ownership](LifecycleAndOwnership.md)
