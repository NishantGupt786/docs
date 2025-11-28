# System Tool Exports - Documentation Index

Complete documentation for Laddr's system tool export feature, enabling users to create custom delegation and storage implementations while reusing core functionality.

## 📚 Documentation Structure

### 1. [Quick Start Guide](custom-system-tools-quickstart.md)
**5-minute introduction** - Get started immediately with copy-paste examples

**Best for:**
- First-time users
- Quick prototyping
- Learning the basics

**Contents:**
- Basic example (copy-paste ready)
- Common use cases (logging, metrics, rate limiting, retries)
- Available tools reference
- Common mistakes

**Time:** ~5 minutes

---

### 2. [Complete Guide](custom-system-tools.md)
**Comprehensive reference** - Everything you need to know

**Best for:**
- Production implementations
- Advanced patterns
- Deep understanding
- Reference documentation

**Contents:**
- Import patterns (3 ways)
- System tool architecture explained
- 5 complete implementation patterns:
  1. Pre/post processing with logging
  2. Rate limiting
  3. Retry logic with exponential backoff
  4. Circuit breaker pattern
  5. Custom artifact storage with compression
- Full API reference for all 3 base tools
- Best practices
- Testing strategies
- Troubleshooting guide

**Time:** ~30 minutes

---

### 3. [Migration Guide](migration-system-tools.md)
**Upgrade existing code** - Refactor old overrides to use base tools

**Best for:**
- Existing Laddr users
- Upgrading custom overrides
- Understanding changes

**Contents:**
- What changed (backward compatible)
- Before vs. after comparisons
- 3 migration scenarios
- Step-by-step migration process
- Common migration patterns
- Testing after migration
- Rollback plan
- FAQ

**Time:** ~20 minutes

---

### 4. [Implementation Summary](SYSTEM_TOOLS_EXPORT_SUMMARY.md)
**Technical overview** - What was implemented and how

**Best for:**
- Understanding the implementation
- Contributors
- Technical review

**Contents:**
- Files modified (8 files)
- Export API design
- Verification test results
- Key features
- Next steps

**Time:** ~10 minutes

---

## 🚀 Quick Navigation

### I want to...

**Get started quickly**  
→ [Quick Start Guide](custom-system-tools-quickstart.md)

**Build a production feature**  
→ [Complete Guide](custom-system-tools.md)

**Upgrade existing overrides**  
→ [Migration Guide](migration-system-tools.md)

**See working examples**  
→ [Examples Directory](../../examples/)
- [custom_parallel_delegation.py](../../examples/custom_parallel_delegation.py) - Two complete examples
- [verify_system_tool_exports.py](../../examples/verify_system_tool_exports.py) - Test suite

**Understand the implementation**  
→ [Implementation Summary](SYSTEM_TOOLS_EXPORT_SUMMARY.md)

---

## 📖 Learning Path

### Beginner Track
1. Read [Quick Start](custom-system-tools-quickstart.md)
2. Try the basic example
3. Experiment with common use cases

### Intermediate Track
1. Read [Complete Guide](custom-system-tools.md) - Sections 1-6
2. Review [Examples](../../examples/custom_parallel_delegation.py)
3. Implement your first custom override
4. Test using patterns from guide

### Advanced Track
1. Read [Complete Guide](custom-system-tools.md) - Full document
2. Study all 5 implementation patterns
3. Review [Migration Guide](migration-system-tools.md)
4. Implement advanced patterns (circuit breaker, retries)
5. Contribute new patterns

---

## 🎯 Common Use Cases

### Use Case: Add Logging
- **Guide:** [Quick Start](custom-system-tools-quickstart.md) - "Add Logging" section
- **Example:** [Complete Guide](custom-system-tools.md) - Pattern 1

### Use Case: Add Metrics/Monitoring
- **Guide:** [Quick Start](custom-system-tools-quickstart.md) - "Add Metrics" section
- **Example:** [Complete Guide](custom-system-tools.md) - Pattern 1
- **Migration:** [Migration Guide](migration-system-tools.md) - Scenario 3

### Use Case: Rate Limiting
- **Guide:** [Quick Start](custom-system-tools-quickstart.md) - "Add Rate Limiting" section
- **Example:** [Complete Guide](custom-system-tools.md) - Pattern 2

### Use Case: Retry Logic
- **Guide:** [Quick Start](custom-system-tools-quickstart.md) - "Add Retries" section
- **Example:** [Complete Guide](custom-system-tools.md) - Pattern 3

### Use Case: Circuit Breaker
- **Example:** [Complete Guide](custom-system-tools.md) - Pattern 4

### Use Case: Custom Storage
- **Example:** [Complete Guide](custom-system-tools.md) - Pattern 5

### Use Case: Migrate Existing Override
- **Guide:** [Migration Guide](migration-system-tools.md) - Full document
- **Example:** [Migration Guide](migration-system-tools.md) - Example Migration section

---

## 🔍 API Reference

### Base Tool Classes

#### TaskDelegationTool
- **Purpose:** Single-task delegation to other agents
- **Constructor:** `TaskDelegationTool(message_bus, artifact_storage, agent)`
- **Methods:** `delegate_task(agent_name, task_description, task, task_data, timeout_seconds)`
- **Override:** `system_delegate_task`
- **Reference:** [Complete Guide - TaskDelegationTool](custom-system-tools.md#taskdelegationtool)

#### ParallelDelegationTool
- **Purpose:** Parallel multi-task delegation (fan-out)
- **Constructor:** `ParallelDelegationTool(message_bus, artifact_storage, agent)`
- **Methods:** `delegate_parallel(agent_name, tasks, timeout_seconds)`
- **Override:** `system_delegate_parallel`
- **Reference:** [Complete Guide - ParallelDelegationTool](custom-system-tools.md#paralleldelegationtool)

#### ArtifactStorageTool
- **Purpose:** Storage and retrieval of large data artifacts
- **Constructor:** `ArtifactStorageTool(storage_backend, default_bucket)`
- **Methods:** `store_artifact(...)`, `retrieve_artifact(...)`
- **Override:** `system_store_artifact`, `system_retrieve_artifact`
- **Reference:** [Complete Guide - ArtifactStorageTool](custom-system-tools.md#artifactstoragetool)

### Helper Functions

- `@override_system_tool(name)` - Register custom override
- `get_tool_override(name)` - Get registered override
- `clear_tool_overrides()` - Clear all overrides
- `list_tool_overrides()` - List all registered overrides

**Reference:** [Complete Guide - Testing Your Overrides](custom-system-tools.md#testing-your-overrides)

---

## 💡 Code Examples

### Minimal Example (5 lines)
```python
from laddr import override_system_tool, TaskDelegationTool

@override_system_tool("system_delegate_task")
async def my_override(..., _message_bus=None, _artifact_storage=None, _agent=None):
    return await TaskDelegationTool(_message_bus, _artifact_storage, _agent).delegate_task(...)
```

### With Logging (10 lines)
```python
from laddr import override_system_tool, TaskDelegationTool
import logging

logger = logging.getLogger(__name__)

@override_system_tool("system_delegate_task")
async def logged_delegation(..., _message_bus=None, _artifact_storage=None, _agent=None):
    logger.info(f"Delegating to {agent_name}")
    result = await TaskDelegationTool(_message_bus, _artifact_storage, _agent).delegate_task(...)
    logger.info("Complete")
    return result
```

### With Metrics (20 lines)
See: [Quick Start - Add Metrics](custom-system-tools-quickstart.md#2-add-metrics)

### With Rate Limiting (15 lines)
See: [Quick Start - Add Rate Limiting](custom-system-tools-quickstart.md#3-add-rate-limiting)

### With Retries (15 lines)
See: [Quick Start - Add Retries](custom-system-tools-quickstart.md#4-add-retries)

### Complete Production Example (50+ lines)
See: [Complete Guide - Pattern 1](custom-system-tools.md#pattern-1-enhance-with-prepost-processing)

---

## ✅ Verification

### Run Tests
```bash
cd tester
docker compose cp ../examples/verify_system_tool_exports.py researcher_worker:/tmp/verify_test.py
docker compose exec researcher_worker python /tmp/verify_test.py
```

Expected output: **5/5 tests passed** ✅

### Test Results
- ✅ Import Test - Verifies all import patterns work
- ✅ Instantiation Test - Verifies base tools can be created
- ✅ Method Signature Test - Verifies API compatibility
- ✅ Override Test - Verifies decorator works
- ✅ Documentation Test - Verifies all classes documented

**Reference:** [Implementation Summary - Verification](SYSTEM_TOOLS_EXPORT_SUMMARY.md#verification)

---

## 🛠️ Troubleshooting

### Import Errors
See: [Complete Guide - Troubleshooting - Override Not Being Called](custom-system-tools.md#override-not-being-called)

### Runtime Injection Not Working
See: [Complete Guide - Troubleshooting - Runtime Injection Not Working](custom-system-tools.md#runtime-injection-not-working)

### Base Tool Not Working
See: [Complete Guide - Troubleshooting - Base Tool Not Working](custom-system-tools.md#base-tool-not-working)

### Common Mistakes
See: [Quick Start - Common Mistakes](custom-system-tools-quickstart.md#common-mistakes)

---

## 📦 What's Included

### Documentation (4 files)
- ✅ Quick Start Guide (~200 lines)
- ✅ Complete Guide (~750 lines)
- ✅ Migration Guide (~400 lines)
- ✅ Implementation Summary (~300 lines)

### Examples (2 files)
- ✅ Custom Parallel Delegation (164 lines)
- ✅ Verification Tests (313 lines)

### Code Changes (3 core files)
- ✅ `laddr/core/system_tools.py` - Added exports
- ✅ `laddr/core/__init__.py` - Added imports
- ✅ `laddr/__init__.py` - Top-level exports

### Configuration (2 files)
- ✅ `README.md` - Added feature section
- ✅ `mkdocs.yml` - Added navigation

**Total:** 11 files created/modified

---

## 🎓 Additional Resources

### Source Code
- [system_tools.py](../../lib/laddr/src/laddr/core/system_tools.py) - Implementation
- [agent_runtime.py](../../lib/laddr/src/laddr/core/agent_runtime.py) - Usage in runtime

### API Reference
- [laddr.core.system_tools](../reference/evenage.core.system_tools.md) - API docs

### Main Documentation
- [Getting Started](../getting-started.md) - Laddr basics
- [Configuration](../configuration.md) - System configuration
- [Recipes](recipes.md) - Other patterns and recipes

---

## 📝 Summary

This documentation covers everything you need to:

✅ **Learn** - Quick start to complete guide  
✅ **Implement** - Working examples and patterns  
✅ **Migrate** - Upgrade existing code  
✅ **Test** - Verification and testing strategies  
✅ **Troubleshoot** - Common issues and solutions  
✅ **Reference** - Complete API documentation  

**Start here:** [Quick Start Guide](custom-system-tools-quickstart.md) ⚡

---

*Last updated: November 8, 2025*
