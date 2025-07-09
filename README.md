# axuspace

User space memory access utilities for kernel-level operations.

`axuspace` provides safe abstractions for reading and writing user space memory from kernel code. It offers type-safe pointer wrappers and automatic memory validation.

## Core Types

- `UserPtr<T>` - Mutable user space pointer wrapper
- `UserConstPtr<T>` - Immutable user space pointer wrapper  
- `UserSpace<A>` - High-level interface for user space operations
- `UserReadable<T>` - Trait for unified read operations

## Example

```rust
use axuspace::{UserPtr, UserConstPtr, UserSpace};

// Create user space pointers
let user_ptr: UserPtr<i32> = UserPtr::from(0x1000);
let str_ptr: UserConstPtr<c_char> = UserConstPtr::from(0x2000);

// Create UserSpace instance
let uspace = UserSpace::new(my_uspace_access);

// Read single value
let value: &i32 = uspace.read(user_ptr)?;

// Read string
let string: &str = uspace.read_str(str_ptr)?;

// Read slice
let slice: &[u8] = uspace.read_slice(ptr, 10)?;

// Write value
uspace.write(user_ptr, 42i32)?;

// Write slice
let data = [1, 2, 3, 4, 5];
uspace.write_slice(ptr, &data)?;

// Read string array (like argv)
let argv_ptr: UserConstPtr<UserConstPtr<c_char>> = /* ... */;
let strings: Vec<String> = uspace.read_str_array(argv_ptr)?;

// Handle nullable pointers
use axuspace::nullable;
let result: Option<&str> = nullable!(uspace.read_str(maybe_null_ptr))?;
```

## Pointer Operations

```rust
// Basic pointer operations
let addr = user_ptr.address();           // Get virtual address
let is_null = user_ptr.is_null();        // Check for null
let offset_ptr = user_ptr.offset(4);     // Add offset
let typed_ptr = user_ptr.cast::<u64>();  // Type conversion

// Copy to kernel buffer
let mut buffer = [0u8; 256];
uspace.read_slice_to(ptr, &mut buffer)?;
```

## Custom UserSpaceAccess

```rust
use axuspace::UserSpaceAccess;

struct MyUserSpaceAccess;

impl UserSpaceAccess for MyUserSpaceAccess {
    fn check_region_access(&self, range: VirtAddrRange, flags: MappingFlags) -> LinuxResult<()> {
        // Implement access permission checking
        Ok(())
    }
    
    fn populate_region(&self, range: VirtAddrRange, flags: MappingFlags) -> LinuxResult<()> {
        // Implement page population
        Ok(())
    }
}
```