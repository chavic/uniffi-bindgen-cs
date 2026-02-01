# External Types Support Implementation Plan

## Background
NordSecurity/uniffi-bindgen-cs issue #40 - External types are unsupported.
This blocks payjoin-ffi from generating C# bindings.

## Current State (in chavic/external-types-support branch)

### Files to Modify

1. **bindgen/src/lib.rs**
   - Lines 105-120: Uncomment `update_component_configs` external types logic
   - Currently commented out with "TODO: external types are not supported"

2. **bindgen/src/gen_cs/external.rs**
   - Currently stub implementation
   - Need full `CodeType` implementation for external types

3. **bindgen/src/gen_cs/mod.rs**
   - `AsCodeType` implementation missing `Type::External` match arm
   - Need to add around line 337

4. **Templates** (in bindgen/templates/)
   - Types.cs - may need external type handling
   - wrapper.cs - namespace imports for external packages

## Implementation Steps

### Step 1: Enable External Package Configuration
**File:** `bindgen/src/lib.rs`

Uncomment and fix the `update_component_configs` method to:
- Build `external_packages` HashMap
- Map crate names to namespace names
- Support cross-crate type references

### Step 2: Implement ExternalCodeType
**File:** `bindgen/src/gen_cs/external.rs`

Current:
```rust
impl CodeType for ExternalCodeType {
    fn type_label(&self, _ci: &ComponentInterface) -> String {
        self.name.clone()  // Just returns name, no namespace
    }
    // ...
}
```

Needed:
- Lookup external type's namespace from config
- Generate fully qualified name: `namespace.TypeName`
- Handle FFI converter for external types
- Handle imports/using statements

### Step 3: Add Type Mapping
**File:** `bindgen/src/gen_cs/mod.rs`

Add to `AsCodeType::as_codetype()`:
```rust
Type::External { name, crate_name, .. } => {
    Box::new(external::ExternalCodeType::new(name, crate_name))
}
```

### Step 4: Template Updates
**Files:** Templates in bindgen/templates/

- Add `using ExternalNamespace;` for each external package
- Generate FFI converters that reference external crate's converter

## Testing Plan

1. Create test fixture with external types
2. Generate C# bindings
3. Compile C# project
4. Run integration tests

## References

- Original issue: https://github.com/NordSecurity/uniffi-bindgen-cs/issues/40
- Similar implementation in uniffi-kotlin
- UniFFI external types spec: https://mozilla.github.io/uniffi-rs/udl/ext_types.html

## Acceptance Criteria

- [ ] Can generate bindings for UDL with `[External="crate"]` types
- [ ] Generated C# code compiles
- [ ] Cross-crate FFI calls work correctly
- [ ] All existing tests still pass

---
**Branch:** chavic/external-types-support
**Created:** 2026-02-01
**Owner:** chavic
