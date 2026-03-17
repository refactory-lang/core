# Tier 3: Async Pattern Resolution

You are resolving async/await patterns for Rust code produced by an automated translation pipeline. The target runtime is Tokio.

## Context

The following Rust code has async patterns that need Tokio-specific handling:

### Compiler error:
{compiler_error}

### Rust code (with TODO markers for async patterns):
{rust_code}

### Original source (for semantic context):
{original_source}

## Rules

1. Use `tokio::select!` for concurrent operations that need cancellation.
2. Use `Pin<Box<dyn Future>>` for dynamic dispatch of async operations.
3. Use `tokio::spawn` for fire-and-forget concurrent tasks.
4. Use `tokio::sync::Mutex` (not `std::sync::Mutex`) for shared state in async code.
5. Add `Send + 'static` bounds to spawned futures.
6. Use `tokio::time::timeout` for deadline-based cancellation.
7. Prefer structured concurrency (`tokio::join!`) over manual task management.
8. Do NOT change the logic or rename any variables.
9. Return ONLY the corrected Rust code, no explanation.

## Escalation

If the async pattern requires a fundamentally different architecture (e.g., actor model, channel-based communication), emit a `// TODO(t3:async): architecture change required` comment and leave the code unchanged.
