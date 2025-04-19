# P0000R0: Optional Fallthrough Control in `switch` Statements

## Abstract

This proposal introduces an optional way to disable implicit fallthrough behavior in C++ `switch` statements. By allowing a directive such as `fall_through(false);`, developers can prevent unintended fallthrough and reduce the need for repetitive `break` statements. This feature enhances code clarity, safety, and aligns with modern language design trends.

## Motivation

While `switch` statements in C++ offer concise branching, the default fallthrough behavior has long been a source of subtle bugs, especially for beginners or in large, evolving codebases. Forgetting to write a `break;` can lead to unexpected behavior, and although C++17 introduced `[[fallthrough]]`, it is merely a way to document intentional fallthrough — it does not prevent unintentional ones.

In contrast, `if`-`else` chains are verbose but safe — each condition is clearly separated. Developers often prefer them despite being more repetitive.

This proposal aims to bridge that gap: preserve the structure of `switch`, but make it safer and more intuitive.

## Proposal

We introduce an optional directive: `fall_through(bool enabled);`

When `fall_through(false);` is used directly before a `switch` block, the compiler will treat each `case` as if it ends with an implicit `break;`, unless explicitly overridden with `[[fallthrough]];`.

When omitted or `fall_through(true);` is used, the traditional C++ behavior is preserved.

### Behavior Summary

| Setting              | Fallthrough by Default | `break` Needed? | `[[fallthrough]]` Allowed |
|----------------------|------------------------|------------------|----------------------------|
| `fall_through(false)` | ❌ Disabled             | ❌ No             | ✅ Yes                     |
| `fall_through(true)`  | ✅ Enabled (default)    | ✅ Yes            | ✅ Yes                     |

## Examples

### Traditional Behavior (Risk of accidental fallthrough)

```cpp
switch(a) {
    case 1: std::cout << "One\n"; // fallthrough?
    case 2: std::cout << "Two\n";
}
With fall_through(false);
cpp
Copy
Edit
fall_through(false);
switch(a) {
    case 1: std::cout << "One\n"; // auto-break here
    case 2: std::cout << "Two\n"; // auto-break here
}
Equivalent to:

cpp
Copy
Edit
switch(a) {
    case 1: std::cout << "One\n"; break;
    case 2: std::cout << "Two\n"; break;
}
Explicit fallthrough remains possible
cpp
Copy
Edit
fall_through(false);
switch(a) {
    case 1: std::cout << "One\n"; [[fallthrough]];
    case 2: std::cout << "Two\n";
}
Alternatives Considered
Rely solely on [[fallthrough]]: doesn't prevent missing break;.

Require compiler warnings for fallthrough: already exists, but not enforced.

Language redesign: not practical for legacy compatibility.

Technical Specification
fall_through(bool) is a compiler directive that affects the immediately following switch block.

When false, the compiler injects implicit break; after each case, unless:

A [[fallthrough]] attribute is present, or

A break, return, or goto is manually inserted.

This is syntactic sugar; no ABI or runtime cost.

Impact on Existing Code
None. This is opt-in. All existing code behaves exactly the same unless fall_through(false); is explicitly used.

Implementation Considerations
This can be implemented via compiler support or macro preprocessing.

Suggested as a compile-time feature, possibly via attribute or pragma in early compilers before standardization.

Conclusion
switch should remain powerful — but also safer and more beginner-friendly.
With fall_through(false);, developers gain control over a well-known footgun in C++ without breaking compatibility or giving up expressiveness.
It’s a small addition with meaningful impact on code correctness.
