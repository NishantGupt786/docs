# Migration Guide: Using System Tool Exports

This guide helps existing Laddr users migrate to the new system tool export feature.

## What Changed?

### New in This Release

✅ **Base system tool classes are now exportable**

You can now import and use:
- `TaskDelegationTool`
- `ParallelDelegationTool`
- `ArtifactStorageTool`

✅ **Enables composable overrides**

You can now override system tools while reusing base functionality.

### Backward Compatibility

✅ **100% backward compatible** - No breaking changes

- All existing code works without modification
- Default behavior unchanged
- Existing overrides work the same way
- No migration required unless you want new features

## Before vs. After

### Before: Limited Override Options

Previously, when overriding system tools, you had two options:

#### Option 1: Complete Reimplementation

```python
@override_system_tool("system_delegate_task")
async def custom_delegate(agent_name, task_description, task, ...):
    # Had to reimplement ALL delegation logic
    # - Message bus publishing
    # - Response stream creation
    # - Timeout handling
    # - Error handling
    # - Artifact storage
    # ... 100+ lines of code
```

**Problems:**
- ❌ Code duplication
- ❌ Hard to maintain
- ❌ Easy to introduce bugs
- ❌ Missed core improvements/fixes

#### Option 2: No Override (Use Default)

```python
# Just use the default system_delegate_task
# No customization possible
```

**Problems:**
- ❌ Can't add logging
- ❌ Can't add metrics
- ❌ Can't add rate limiting
- ❌ Can't add custom behavior

### After: Composable Overrides

Now you can override AND reuse base functionality:

```python
from laddr import override_system_tool, TaskDelegationTool

@override_system_tool("system_delegate_task")
async def custom_delegate(
    agent_name: str,
    task_description: str,
    task: str,
    task_data: dict = None,
    timeout_seconds: int = 300,
    _message_bus=None,
    _artifact_storage=None,
    _agent=None
):
    """Add logging while reusing core delegation."""
    
    # Your custom logic (5 lines)
    logger.info(f"Delegating to {agent_name}")
    
    # Reuse base tool (100+ lines of tested code)
    delegation_tool = TaskDelegationTool(_message_bus, _artifact_storage, _agent)
    result = await delegation_tool.delegate_task(
        agent_name=agent_name,
        task_description=task_description,
        task=task,
        task_data=task_data,
        timeout_seconds=timeout_seconds
    )
    
    # More custom logic
    logger.info("Delegation complete")
    return result
```

**Benefits:**
- ✅ Minimal code
- ✅ Reuse tested core logic
- ✅ Easy to maintain
- ✅ Automatic bug fixes/improvements
- ✅ Add any custom behavior you need

## Migration Scenarios

### Scenario 1: You're NOT Using Overrides

**You don't need to do anything!**

The new feature is opt-in. Your existing code continues to work exactly as before.

### Scenario 2: You Have Custom Overrides

If you have existing custom overrides, you can optionally refactor them to use base tools.

#### Example Migration

**Before (all custom code):**

```python
# old_custom_delegation.py
import asyncio
from laddr import override_system_tool

@override_system_tool("system_delegate_task")
async def custom_delegate(agent_name, task_description, task, task_data=None, timeout_seconds=300, _message_bus=None, _artifact_storage=None, _agent=None):
    """Custom delegation with logging."""
    
    import logging
    logger = logging.getLogger(__name__)
    logger.info(f"Delegating to {agent_name}")
    
    # 50+ lines of custom message bus logic
    response_stream = f"laddr:response:{_agent.job_id}:{agent_name}"
    message = {
        "agent_name": agent_name,
        "task": task,
        "task_description": task_description,
        "task_data": task_data or {},
        "response_stream": response_stream,
        "job_id": _agent.job_id
    }
    
    # Publish to message bus
    await _message_bus.publish(f"laddr:agent:{agent_name}", message)
    
    # Wait for response with timeout
    start_time = asyncio.get_event_loop().time()
    while True:
        if asyncio.get_event_loop().time() - start_time > timeout_seconds:
            raise TimeoutError(f"Delegation timed out after {timeout_seconds}s")
        
        # Poll for response
        response = await _message_bus.read_stream(response_stream, count=1)
        if response:
            # Parse and return result
            result = response[0]["data"]
            logger.info("Delegation complete")
            return result
        
        await asyncio.sleep(0.1)
```

**After (using base tool):**

```python
# new_custom_delegation.py
import logging
from laddr import override_system_tool, TaskDelegationTool

logger = logging.getLogger(__name__)

@override_system_tool("system_delegate_task")
async def custom_delegate(
    agent_name: str,
    task_description: str,
    task: str,
    task_data: dict = None,
    timeout_seconds: int = 300,
    _message_bus=None,
    _artifact_storage=None,
    _agent=None
):
    """Custom delegation with logging."""
    
    logger.info(f"Delegating to {agent_name}")
    
    # All the complex logic is handled by base tool
    delegation_tool = TaskDelegationTool(_message_bus, _artifact_storage, _agent)
    result = await delegation_tool.delegate_task(
        agent_name=agent_name,
        task_description=task_description,
        task=task,
        task_data=task_data,
        timeout_seconds=timeout_seconds
    )
    
    logger.info("Delegation complete")
    return result
```

**Improvements:**
- 📉 50+ lines → 15 lines
- ✅ Same functionality
- ✅ Easier to maintain
- ✅ Automatic bug fixes
- ✅ Better tested (reuses core code)

### Scenario 3: You Want to Add Monitoring

You can now easily add monitoring to existing agents without reimplementing delegation.

```python
# monitoring.py
from laddr import override_system_tool, TaskDelegationTool
from prometheus_client import Counter, Histogram
import time

# Metrics
delegation_counter = Counter('delegation_total', 'Total delegations', ['agent_name', 'status'])
delegation_duration = Histogram('delegation_duration_seconds', 'Delegation duration', ['agent_name'])

@override_system_tool("system_delegate_task")
async def monitored_delegation(
    agent_name: str,
    task_description: str,
    task: str,
    task_data: dict = None,
    timeout_seconds: int = 300,
    _message_bus=None,
    _artifact_storage=None,
    _agent=None
):
    """Delegation with Prometheus metrics."""
    
    start_time = time.time()
    
    try:
        delegation_tool = TaskDelegationTool(_message_bus, _artifact_storage, _agent)
        result = await delegation_tool.delegate_task(
            agent_name=agent_name,
            task_description=task_description,
            task=task,
            task_data=task_data,
            timeout_seconds=timeout_seconds
        )
        
        # Record success
        delegation_counter.labels(agent_name=agent_name, status='success').inc()
        delegation_duration.labels(agent_name=agent_name).observe(time.time() - start_time)
        
        return result
        
    except Exception as e:
        # Record failure
        delegation_counter.labels(agent_name=agent_name, status='failure').inc()
        delegation_duration.labels(agent_name=agent_name).observe(time.time() - start_time)
        raise
```

Just import this module in your agent files and monitoring is automatically added!

## Step-by-Step Migration

### Step 1: Identify Your Overrides

Find all uses of `@override_system_tool` in your codebase:

```bash
grep -r "@override_system_tool" .
```

### Step 2: Analyze Each Override

For each override, ask:
1. Does it reimplement core delegation/storage logic?
2. Could it benefit from using a base tool?
3. Is the custom logic focused on one specific enhancement?

### Step 3: Refactor Using Base Tools

If yes to questions above, refactor:

1. Import the base tool class
2. Instantiate with injected dependencies
3. Call base tool methods
4. Keep your custom logic

**Template:**

```python
from laddr import override_system_tool, TaskDelegationTool  # 1. Import

@override_system_tool("system_delegate_task")
async def my_override(
    agent_name: str,
    task_description: str,
    task: str,
    task_data: dict = None,
    timeout_seconds: int = 300,
    _message_bus=None,
    _artifact_storage=None,
    _agent=None
):
    # Your custom BEFORE logic
    
    # 2. Instantiate base tool
    delegation_tool = TaskDelegationTool(_message_bus, _artifact_storage, _agent)
    
    # 3. Call base method
    result = await delegation_tool.delegate_task(
        agent_name=agent_name,
        task_description=task_description,
        task=task,
        task_data=task_data,
        timeout_seconds=timeout_seconds
    )
    
    # Your custom AFTER logic
    
    return result
```

### Step 4: Test

1. Run your existing tests
2. Verify behavior unchanged
3. Check logs/metrics still work

### Step 5: Clean Up

Remove any duplicated code that's now handled by base tools.

## Common Migration Patterns

### Pattern 1: Adding Pre-Processing

**Before:**
```python
@override_system_tool("system_delegate_task")
async def validate_and_delegate(...):
    # Validation
    if not agent_name:
        raise ValueError("agent_name required")
    
    # Copy-pasted delegation logic (50+ lines)
    ...
```

**After:**
```python
from laddr import override_system_tool, TaskDelegationTool

@override_system_tool("system_delegate_task")
async def validate_and_delegate(..., _message_bus=None, _artifact_storage=None, _agent=None):
    # Validation
    if not agent_name:
        raise ValueError("agent_name required")
    
    # Reuse base tool
    return await TaskDelegationTool(_message_bus, _artifact_storage, _agent).delegate_task(...)
```

### Pattern 2: Adding Post-Processing

**Before:**
```python
@override_system_tool("system_delegate_task")
async def transform_result(...):
    # Copy-pasted delegation logic (50+ lines)
    result = ...
    
    # Transform result
    result["transformed"] = True
    return result
```

**After:**
```python
from laddr import override_system_tool, TaskDelegationTool

@override_system_tool("system_delegate_task")
async def transform_result(..., _message_bus=None, _artifact_storage=None, _agent=None):
    # Delegate using base tool
    result = await TaskDelegationTool(_message_bus, _artifact_storage, _agent).delegate_task(...)
    
    # Transform result
    result["transformed"] = True
    return result
```

### Pattern 3: Adding Side Effects

**Before:**
```python
@override_system_tool("system_delegate_task")
async def logged_delegation(...):
    logger.info("Starting")
    
    # Copy-pasted delegation logic (50+ lines)
    result = ...
    
    logger.info("Done")
    return result
```

**After:**
```python
from laddr import override_system_tool, TaskDelegationTool

@override_system_tool("system_delegate_task")
async def logged_delegation(..., _message_bus=None, _artifact_storage=None, _agent=None):
    logger.info("Starting")
    
    result = await TaskDelegationTool(_message_bus, _artifact_storage, _agent).delegate_task(...)
    
    logger.info("Done")
    return result
```

## Testing After Migration

### Verify Override Registration

```python
from laddr.core.system_tools import get_tool_override

override = get_tool_override("system_delegate_task")
assert override is not None, "Override not registered!"
assert override.__name__ == "my_override", "Wrong override!"
```

### Verify Behavior

```python
# Test that delegation still works
result = await coordinator.delegate_task(
    agent_name="researcher",
    task_description="Test",
    task="test"
)
assert result is not None
```

### Verify Custom Logic

```python
# Test that your custom logic executes
# (e.g., check logs, metrics, side effects)
assert "Starting delegation" in captured_logs
assert metrics["delegation_count"] == 1
```

## Rollback Plan

If you need to rollback:

1. **Remove the import:**
   ```python
   # from laddr import TaskDelegationTool  # Remove this
   ```

2. **Restore original override code:**
   ```python
   @override_system_tool("system_delegate_task")
   async def my_override(...):
       # Your original implementation
   ```

3. **Test and redeploy**

Your system will work exactly as before the migration.

## FAQ

### Q: Do I have to migrate?

**A: No.** This is an opt-in feature. Existing code works without changes.

### Q: Will my old overrides still work?

**A: Yes.** 100% backward compatible.

### Q: What if I don't use overrides at all?

**A: No action needed.** The new feature is for users who want to customize system tools.

### Q: Can I mix old and new patterns?

**A: Yes.** You can have some overrides using base tools and others not.

### Q: Is there a performance difference?

**A: No.** Base tools use the same code that was already running internally.

### Q: Can I partially use base tools?

**A: Yes.** You can use base tools for some parts and custom logic for others.

Example:
```python
@override_system_tool("system_delegate_task")
async def hybrid(..., _message_bus=None, _artifact_storage=None, _agent=None):
    # Custom message bus logic
    await _message_bus.publish(...)
    
    # But use base tool for response handling
    delegation_tool = TaskDelegationTool(_message_bus, _artifact_storage, _agent)
    # Use specific methods as needed
```

## Support

If you encounter issues during migration:

1. Check the [Complete Guide](custom-system-tools.md)
2. Review [Examples](../../examples/custom_parallel_delegation.py)
3. Run [Verification Tests](../../examples/verify_system_tool_exports.py)
4. Open an issue on GitHub

## Summary

✅ **Backward compatible** - No breaking changes  
✅ **Opt-in** - Migration not required  
✅ **Easy migration** - Simple refactoring pattern  
✅ **Better code** - Less duplication, easier maintenance  
✅ **Same behavior** - Functionality unchanged  

**Recommendation:** Migrate gradually as you touch existing override code. No rush!
