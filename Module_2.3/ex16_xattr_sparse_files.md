# [Module 2.3] - Exercise 16: Extended Attributes & Sparse Files

## Metadonnees

```yaml
module: "2.3 - File Systems"
exercise: "ex16"
title: "Extended Attributes & Sparse Files"
difficulty: intermediaire
estimated_time: "4 heures"
prerequisite_exercises: ["ex12", "ex13"]
concepts_requis: ["filesystem", "metadata", "file_holes"]
score_qualite: 98
```

---

## Concepts Couverts

### 2.3.28: Extended Attributes (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.28.a | xattr concept | Key-value metadata |
| 2.3.28.b | `xattr` crate | Rust bindings |
| 2.3.28.c | `xattr::get()` | Read attribute |
| 2.3.28.d | `xattr::set()` | Set attribute |
| 2.3.28.e | `xattr::remove()` | Delete attribute |
| 2.3.28.f | `xattr::list()` | List all xattrs |
| 2.3.28.g | Namespaces | user, system, security |
| 2.3.28.h | Use cases | Tags, capabilities |
| 2.3.28.i | `nix::sys::xattr` | Low-level API |
| 2.3.28.j | SELinux labels | Security context |

### 2.3.29: Sparse Files & Holes (9 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.29.a | Sparse file concept | Holes in file |
| 2.3.29.b | Creating holes | Seek past end + write |
| 2.3.29.c | `SEEK_HOLE` | Find holes |
| 2.3.29.d | `SEEK_DATA` | Find data |
| 2.3.29.e | `nix::fcntl::lseek()` | With whence |
| 2.3.29.f | `fallocate()` | Allocate/punch holes |
| 2.3.29.g | `FALLOC_FL_PUNCH_HOLE` | Create hole |
| 2.3.29.h | `nix::fcntl::fallocate()` | Rust binding |
| 2.3.29.i | Copy efficiency | Preserve holes |

---

## Partie 1: Extended Attributes (2.3.28)

### Exercice 1.1: Understanding Extended Attributes (2.3.28.a, b)

Extended attributes (xattrs) allow associating arbitrary key-value metadata with files beyond standard permissions and timestamps.

```rust
//! Extended Attributes Manager
//! Demonstrates xattr operations in Rust

use std::path::Path;
use std::io;

// 2.3.28.b: Using xattr crate
// Add to Cargo.toml: xattr = "1.0"

/// XAttr namespace types (2.3.28.g)
#[derive(Debug, Clone, Copy)]
pub enum XAttrNamespace {
    /// User namespace (user.*)
    User,
    /// System namespace (system.*)
    System,
    /// Security namespace (security.*)
    Security,
    /// Trusted namespace (trusted.*)
    Trusted,
}

impl XAttrNamespace {
    fn prefix(&self) -> &'static str {
        match self {
            XAttrNamespace::User => "user.",
            XAttrNamespace::System => "system.",
            XAttrNamespace::Security => "security.",
            XAttrNamespace::Trusted => "trusted.",
        }
    }
}

/// Extended attribute wrapper
pub struct ExtendedAttribute {
    pub name: String,
    pub value: Vec<u8>,
    pub namespace: Option<XAttrNamespace>,
}

impl ExtendedAttribute {
    pub fn new(name: impl Into<String>, value: impl Into<Vec<u8>>) -> Self {
        Self {
            name: name.into(),
            value: value.into(),
            namespace: None,
        }
    }

    /// Create attribute with namespace (2.3.28.g)
    pub fn with_namespace(
        namespace: XAttrNamespace,
        key: impl Into<String>,
        value: impl Into<Vec<u8>>,
    ) -> Self {
        let full_name = format!("{}{}", namespace.prefix(), key.into());
        Self {
            name: full_name,
            value: value.into(),
            namespace: Some(namespace),
        }
    }
}

fn main() -> io::Result<()> {
    println!("Extended Attributes Concept (2.3.28.a)");
    println!("=====================================\n");

    println!("XAttrs are key-value pairs attached to files:");
    println!("  - Stored in filesystem metadata");
    println!("  - Survive copy/move operations (if supported)");
    println!("  - Have namespaces: user, system, security, trusted\n");

    println!("Common use cases (2.3.28.h):");
    println!("  - File tags and labels");
    println!("  - Security contexts (SELinux)");
    println!("  - Capabilities");
    println!("  - ACL storage");
    println!("  - Checksums/hashes");

    Ok(())
}
```

### Exercice 1.2: Setting and Getting Attributes (2.3.28.c, d)

```rust
use std::path::Path;
use std::io::{self, Write};

/// XAttr manager using xattr crate (2.3.28.b)
pub struct XAttrManager;

impl XAttrManager {
    /// Set an extended attribute (2.3.28.d)
    pub fn set<P: AsRef<Path>>(
        path: P,
        name: &str,
        value: &[u8],
    ) -> io::Result<()> {
        xattr::set(path.as_ref(), name, value)
    }

    /// Get an extended attribute (2.3.28.c)
    pub fn get<P: AsRef<Path>>(
        path: P,
        name: &str,
    ) -> io::Result<Option<Vec<u8>>> {
        match xattr::get(path.as_ref(), name) {
            Ok(value) => Ok(value),
            Err(e) if e.kind() == io::ErrorKind::NotFound => Ok(None),
            Err(e) => Err(e),
        }
    }

    /// Set string attribute
    pub fn set_string<P: AsRef<Path>>(
        path: P,
        name: &str,
        value: &str,
    ) -> io::Result<()> {
        Self::set(path, name, value.as_bytes())
    }

    /// Get string attribute
    pub fn get_string<P: AsRef<Path>>(
        path: P,
        name: &str,
    ) -> io::Result<Option<String>> {
        match Self::get(path, name)? {
            Some(bytes) => {
                String::from_utf8(bytes)
                    .map(Some)
                    .map_err(|e| io::Error::new(io::ErrorKind::InvalidData, e))
            }
            None => Ok(None),
        }
    }
}

fn demonstrate_xattr() -> io::Result<()> {
    use std::fs::File;

    // Create test file
    let test_path = "/tmp/xattr_test.txt";
    File::create(test_path)?;

    // 2.3.28.d: Set attribute
    println!("Setting attribute 'user.description'...");
    XAttrManager::set_string(test_path, "user.description", "Test file")?;

    // 2.3.28.c: Get attribute
    if let Some(desc) = XAttrManager::get_string(test_path, "user.description")? {
        println!("Got attribute: {}", desc);
    }

    // Set binary data
    let checksum: [u8; 4] = [0xDE, 0xAD, 0xBE, 0xEF];
    XAttrManager::set(test_path, "user.checksum", &checksum)?;

    if let Some(data) = XAttrManager::get(test_path, "user.checksum")? {
        println!("Got checksum: {:02X?}", data);
    }

    // Cleanup
    std::fs::remove_file(test_path)?;

    Ok(())
}
```

### Exercice 1.3: Listing and Removing Attributes (2.3.28.e, f)

```rust
use std::path::Path;
use std::io;

impl XAttrManager {
    /// List all extended attributes (2.3.28.f)
    pub fn list<P: AsRef<Path>>(path: P) -> io::Result<Vec<String>> {
        let names: Vec<String> = xattr::list(path.as_ref())?
            .filter_map(|name| name.into_string().ok())
            .collect();
        Ok(names)
    }

    /// Remove an extended attribute (2.3.28.e)
    pub fn remove<P: AsRef<Path>>(path: P, name: &str) -> io::Result<()> {
        xattr::remove(path.as_ref(), name)
    }

    /// Remove all user attributes
    pub fn remove_all_user<P: AsRef<Path>>(path: P) -> io::Result<usize> {
        let path = path.as_ref();
        let names = Self::list(path)?;
        let mut count = 0;

        for name in names {
            if name.starts_with("user.") {
                Self::remove(path, &name)?;
                count += 1;
            }
        }

        Ok(count)
    }
}

/// File tagger using xattrs (2.3.28.h - Use case: Tags)
pub struct FileTagger {
    tag_attr: String,
}

impl FileTagger {
    pub fn new() -> Self {
        Self {
            tag_attr: "user.tags".to_string(),
        }
    }

    /// Add tag to file
    pub fn add_tag<P: AsRef<Path>>(&self, path: P, tag: &str) -> io::Result<()> {
        let mut tags = self.get_tags(&path)?;
        if !tags.contains(&tag.to_string()) {
            tags.push(tag.to_string());
            self.save_tags(path, &tags)
        } else {
            Ok(())
        }
    }

    /// Remove tag from file
    pub fn remove_tag<P: AsRef<Path>>(&self, path: P, tag: &str) -> io::Result<()> {
        let mut tags = self.get_tags(&path)?;
        tags.retain(|t| t != tag);
        self.save_tags(path, &tags)
    }

    /// Get all tags
    pub fn get_tags<P: AsRef<Path>>(&self, path: P) -> io::Result<Vec<String>> {
        match XAttrManager::get_string(&path, &self.tag_attr)? {
            Some(data) => {
                Ok(data.split(',')
                    .map(|s| s.trim().to_string())
                    .filter(|s| !s.is_empty())
                    .collect())
            }
            None => Ok(Vec::new()),
        }
    }

    fn save_tags<P: AsRef<Path>>(&self, path: P, tags: &[String]) -> io::Result<()> {
        if tags.is_empty() {
            XAttrManager::remove(path, &self.tag_attr)
        } else {
            XAttrManager::set_string(path, &self.tag_attr, &tags.join(","))
        }
    }
}

fn demonstrate_listing() -> io::Result<()> {
    use std::fs::File;

    let test_path = "/tmp/xattr_list_test.txt";
    File::create(test_path)?;

    // Set multiple attributes
    XAttrManager::set_string(test_path, "user.author", "Alice")?;
    XAttrManager::set_string(test_path, "user.project", "Demo")?;
    XAttrManager::set_string(test_path, "user.version", "1.0")?;

    // 2.3.28.f: List all attributes
    println!("Listing attributes:");
    for name in XAttrManager::list(test_path)? {
        if let Some(value) = XAttrManager::get_string(test_path, &name)? {
            println!("  {} = {}", name, value);
        }
    }

    // 2.3.28.e: Remove attribute
    println!("\nRemoving 'user.version'...");
    XAttrManager::remove(test_path, "user.version")?;

    println!("After removal:");
    for name in XAttrManager::list(test_path)? {
        println!("  {}", name);
    }

    std::fs::remove_file(test_path)?;
    Ok(())
}
```

### Exercice 1.4: Low-Level API with nix (2.3.28.i)

```rust
use std::os::unix::io::AsRawFd;
use std::path::Path;
use std::ffi::CString;

/// Low-level xattr operations using nix (2.3.28.i)
pub mod low_level {
    use nix::sys::xattr;
    use std::path::Path;
    use std::io;

    /// Get xattr using nix
    pub fn getxattr<P: AsRef<Path>>(path: P, name: &str) -> io::Result<Vec<u8>> {
        xattr::getxattr(path.as_ref(), name)
            .map_err(|e| io::Error::new(io::ErrorKind::Other, e))
    }

    /// Set xattr using nix
    pub fn setxattr<P: AsRef<Path>>(
        path: P,
        name: &str,
        value: &[u8],
    ) -> io::Result<()> {
        xattr::setxattr(path.as_ref(), name, value)
            .map_err(|e| io::Error::new(io::ErrorKind::Other, e))
    }

    /// List xattrs using nix
    pub fn listxattr<P: AsRef<Path>>(path: P) -> io::Result<Vec<String>> {
        let list = xattr::listxattr(path.as_ref())
            .map_err(|e| io::Error::new(io::ErrorKind::Other, e))?;

        Ok(list.into_iter()
            .filter_map(|name| name.into_string().ok())
            .collect())
    }

    /// Remove xattr using nix
    pub fn removexattr<P: AsRef<Path>>(path: P, name: &str) -> io::Result<()> {
        xattr::removexattr(path.as_ref(), name)
            .map_err(|e| io::Error::new(io::ErrorKind::Other, e))
    }

    /// lgetxattr - don't follow symlinks
    pub fn lgetxattr<P: AsRef<Path>>(path: P, name: &str) -> io::Result<Vec<u8>> {
        xattr::lgetxattr(path.as_ref(), name)
            .map_err(|e| io::Error::new(io::ErrorKind::Other, e))
    }
}

/// SELinux context helper (2.3.28.j)
pub struct SELinuxContext;

impl SELinuxContext {
    const SELINUX_ATTR: &'static str = "security.selinux";

    /// Get SELinux context of a file (2.3.28.j)
    pub fn get_context<P: AsRef<Path>>(path: P) -> io::Result<Option<String>> {
        match XAttrManager::get(path, Self::SELINUX_ATTR)? {
            Some(bytes) => {
                // SELinux context is null-terminated
                let context = bytes.iter()
                    .take_while(|&&b| b != 0)
                    .cloned()
                    .collect::<Vec<_>>();

                String::from_utf8(context)
                    .map(Some)
                    .map_err(|e| io::Error::new(io::ErrorKind::InvalidData, e))
            }
            None => Ok(None),
        }
    }

    /// Parse SELinux context into components
    pub fn parse_context(context: &str) -> Option<SELinuxComponents> {
        let parts: Vec<&str> = context.split(':').collect();
        if parts.len() >= 4 {
            Some(SELinuxComponents {
                user: parts[0].to_string(),
                role: parts[1].to_string(),
                type_: parts[2].to_string(),
                level: parts[3..].join(":"),
            })
        } else {
            None
        }
    }
}

#[derive(Debug)]
pub struct SELinuxComponents {
    pub user: String,
    pub role: String,
    pub type_: String,
    pub level: String,
}
```

---

## Partie 2: Sparse Files & Holes (2.3.29)

### Exercice 2.1: Understanding Sparse Files (2.3.29.a, b)

```rust
//! Sparse Files - Files with holes
//!
//! Sparse files contain "holes" - regions that don't allocate
//! physical disk blocks. Reading holes returns zeros.

use std::fs::{File, OpenOptions};
use std::io::{self, Read, Seek, SeekFrom, Write};
use std::os::unix::io::AsRawFd;

/// Sparse file information
#[derive(Debug)]
pub struct SparseFileInfo {
    pub path: String,
    pub logical_size: u64,
    pub physical_size: u64,
    pub hole_count: usize,
    pub data_regions: Vec<(u64, u64)>,  // (offset, length)
    pub hole_regions: Vec<(u64, u64)>,
}

impl SparseFileInfo {
    pub fn sparseness_ratio(&self) -> f64 {
        if self.logical_size == 0 {
            0.0
        } else {
            1.0 - (self.physical_size as f64 / self.logical_size as f64)
        }
    }
}

/// Create a sparse file by seeking past end (2.3.29.b)
pub fn create_sparse_file(path: &str, size: u64) -> io::Result<()> {
    let mut file = File::create(path)?;

    // Seek to position near end (2.3.29.b)
    file.seek(SeekFrom::Start(size - 1))?;

    // Write single byte to set file size
    file.write_all(&[0])?;

    println!("Created sparse file: {} bytes logical", size);

    // Check actual disk usage
    let metadata = file.metadata()?;
    println!("Blocks allocated: {}", metadata.len());

    Ok(())
}

/// Write data at specific offset, creating sparse regions
pub fn write_sparse_data(path: &str, data: &[u8], offset: u64) -> io::Result<()> {
    let mut file = OpenOptions::new()
        .write(true)
        .open(path)?;

    file.seek(SeekFrom::Start(offset))?;
    file.write_all(data)?;

    Ok(())
}

fn demonstrate_sparse() -> io::Result<()> {
    println!("Sparse Files Concept (2.3.29.a)");
    println!("===============================\n");

    let path = "/tmp/sparse_demo.dat";

    // Create 1GB sparse file (uses almost no disk space)
    let one_gb = 1024 * 1024 * 1024;
    create_sparse_file(path, one_gb)?;

    // Write some data at different offsets
    write_sparse_data(path, b"START", 0)?;
    write_sparse_data(path, b"MIDDLE", one_gb / 2)?;
    write_sparse_data(path, b"END", one_gb - 10)?;

    // Reading from hole returns zeros
    let mut file = File::open(path)?;
    let mut buf = [0u8; 10];
    file.seek(SeekFrom::Start(1000))?;
    file.read_exact(&mut buf)?;
    println!("Data from hole: {:?} (all zeros)", buf);

    std::fs::remove_file(path)?;
    Ok(())
}
```

### Exercice 2.2: Finding Holes with SEEK_HOLE/SEEK_DATA (2.3.29.c, d, e)

```rust
use std::fs::File;
use std::io::{self, Seek, SeekFrom};
use std::os::unix::io::AsRawFd;
use nix::unistd;

/// Seek whence values for hole detection
#[cfg(target_os = "linux")]
mod seek_constants {
    pub const SEEK_DATA: i32 = 3;  // 2.3.29.d
    pub const SEEK_HOLE: i32 = 4;  // 2.3.29.c
}

/// Find data and hole regions in a file (2.3.29.c, d)
pub struct HoleFinder {
    fd: i32,
    file_size: u64,
}

impl HoleFinder {
    pub fn new(file: &File) -> io::Result<Self> {
        let metadata = file.metadata()?;
        Ok(Self {
            fd: file.as_raw_fd(),
            file_size: metadata.len(),
        })
    }

    /// Seek to next data region (2.3.29.d)
    pub fn seek_data(&self, offset: u64) -> io::Result<Option<u64>> {
        self.seek_whence(offset, seek_constants::SEEK_DATA)
    }

    /// Seek to next hole (2.3.29.c)
    pub fn seek_hole(&self, offset: u64) -> io::Result<Option<u64>> {
        self.seek_whence(offset, seek_constants::SEEK_HOLE)
    }

    /// Low-level lseek with custom whence (2.3.29.e)
    fn seek_whence(&self, offset: u64, whence: i32) -> io::Result<Option<u64>> {
        // Using nix::unistd::lseek (2.3.29.e)
        let result = unsafe {
            libc::lseek(self.fd, offset as i64, whence)
        };

        if result < 0 {
            let err = io::Error::last_os_error();
            if err.raw_os_error() == Some(libc::ENXIO) {
                // No more data/holes
                Ok(None)
            } else {
                Err(err)
            }
        } else {
            Ok(Some(result as u64))
        }
    }

    /// Map all data and hole regions
    pub fn map_regions(&self) -> io::Result<SparseFileInfo> {
        let mut data_regions = Vec::new();
        let mut hole_regions = Vec::new();
        let mut pos = 0u64;

        while pos < self.file_size {
            // Find next data region
            match self.seek_data(pos)? {
                Some(data_start) => {
                    // Find where data ends (next hole)
                    match self.seek_hole(data_start)? {
                        Some(hole_start) => {
                            data_regions.push((data_start, hole_start - data_start));

                            // Check for hole between current pos and data_start
                            if data_start > pos {
                                hole_regions.push((pos, data_start - pos));
                            }

                            pos = hole_start;
                        }
                        None => {
                            // Data extends to end of file
                            data_regions.push((data_start, self.file_size - data_start));
                            if data_start > pos {
                                hole_regions.push((pos, data_start - pos));
                            }
                            break;
                        }
                    }
                }
                None => {
                    // Rest of file is a hole
                    if pos < self.file_size {
                        hole_regions.push((pos, self.file_size - pos));
                    }
                    break;
                }
            }
        }

        // Calculate physical size (sum of data regions)
        let physical_size: u64 = data_regions.iter()
            .map(|(_, len)| len)
            .sum();

        Ok(SparseFileInfo {
            path: String::new(),
            logical_size: self.file_size,
            physical_size,
            hole_count: hole_regions.len(),
            data_regions,
            hole_regions,
        })
    }
}

fn demonstrate_hole_finding() -> io::Result<()> {
    println!("\nHole Detection (2.3.29.c, d)");
    println!("============================\n");

    let path = "/tmp/sparse_holes.dat";

    // Create sparse file with known pattern
    create_sparse_file(path, 1024 * 1024)?;  // 1MB
    write_sparse_data(path, b"DATA1", 0)?;
    write_sparse_data(path, b"DATA2", 100_000)?;
    write_sparse_data(path, b"DATA3", 500_000)?;

    let file = File::open(path)?;
    let finder = HoleFinder::new(&file)?;

    let info = finder.map_regions()?;

    println!("Sparse file analysis:");
    println!("  Logical size: {} bytes", info.logical_size);
    println!("  Physical size: {} bytes", info.physical_size);
    println!("  Sparseness: {:.1}%", info.sparseness_ratio() * 100.0);
    println!("\nData regions:");
    for (offset, len) in &info.data_regions {
        println!("  offset: {}, length: {}", offset, len);
    }
    println!("\nHole regions: {}", info.hole_count);
    for (offset, len) in &info.hole_regions {
        println!("  offset: {}, length: {}", offset, len);
    }

    std::fs::remove_file(path)?;
    Ok(())
}
```

### Exercice 2.3: Punching Holes with fallocate (2.3.29.f, g, h)

```rust
use std::fs::{File, OpenOptions};
use std::io::{self, Write};
use std::os::unix::io::AsRawFd;

/// fallocate flags (2.3.29.g)
mod fallocate_flags {
    pub const FALLOC_FL_KEEP_SIZE: i32 = 0x01;
    pub const FALLOC_FL_PUNCH_HOLE: i32 = 0x02;  // 2.3.29.g
    pub const FALLOC_FL_COLLAPSE_RANGE: i32 = 0x08;
    pub const FALLOC_FL_ZERO_RANGE: i32 = 0x10;
}

/// Hole puncher for creating sparse files (2.3.29.f)
pub struct HolePuncher;

impl HolePuncher {
    /// Punch a hole in a file (2.3.29.f, g, h)
    /// This deallocates the specified range without changing file size
    pub fn punch_hole(file: &File, offset: u64, length: u64) -> io::Result<()> {
        // 2.3.29.h: Using nix::fcntl::fallocate
        let fd = file.as_raw_fd();
        let flags = fallocate_flags::FALLOC_FL_PUNCH_HOLE
                  | fallocate_flags::FALLOC_FL_KEEP_SIZE;

        let result = unsafe {
            libc::fallocate(fd, flags, offset as i64, length as i64)
        };

        if result < 0 {
            Err(io::Error::last_os_error())
        } else {
            Ok(())
        }
    }

    /// Allocate space without writing (preallocate)
    pub fn preallocate(file: &File, offset: u64, length: u64) -> io::Result<()> {
        let fd = file.as_raw_fd();

        let result = unsafe {
            libc::fallocate(fd, 0, offset as i64, length as i64)
        };

        if result < 0 {
            Err(io::Error::last_os_error())
        } else {
            Ok(())
        }
    }

    /// Zero a range (faster than writing zeros)
    pub fn zero_range(file: &File, offset: u64, length: u64) -> io::Result<()> {
        let fd = file.as_raw_fd();
        let flags = fallocate_flags::FALLOC_FL_ZERO_RANGE;

        let result = unsafe {
            libc::fallocate(fd, flags, offset as i64, length as i64)
        };

        if result < 0 {
            Err(io::Error::last_os_error())
        } else {
            Ok(())
        }
    }
}

/// Using nix crate for fallocate (2.3.29.h)
pub mod nix_fallocate {
    use std::fs::File;
    use std::io;
    use std::os::unix::io::AsRawFd;
    use nix::fcntl::{fallocate, FallocateFlags};

    /// Punch hole using nix (2.3.29.h)
    pub fn punch_hole_nix(file: &File, offset: i64, len: i64) -> io::Result<()> {
        let flags = FallocateFlags::FALLOC_FL_PUNCH_HOLE
                  | FallocateFlags::FALLOC_FL_KEEP_SIZE;

        fallocate(file.as_raw_fd(), flags, offset, len)
            .map_err(|e| io::Error::new(io::ErrorKind::Other, e))
    }
}

fn demonstrate_hole_punching() -> io::Result<()> {
    println!("\nHole Punching (2.3.29.f, g, h)");
    println!("==============================\n");

    let path = "/tmp/punch_test.dat";

    // Create file with data
    let mut file = File::create(path)?;
    let data = vec![0xAA; 1024 * 1024];  // 1MB of data
    file.write_all(&data)?;
    file.sync_all()?;

    println!("Created 1MB file filled with data");

    // Reopen for punching
    let file = OpenOptions::new()
        .read(true)
        .write(true)
        .open(path)?;

    // Punch holes at various offsets (2.3.29.g)
    println!("Punching holes...");
    HolePuncher::punch_hole(&file, 100_000, 200_000)?;  // 200KB hole at 100KB
    HolePuncher::punch_hole(&file, 500_000, 100_000)?;  // 100KB hole at 500KB

    // Analyze result
    let finder = HoleFinder::new(&file)?;
    let info = finder.map_regions()?;

    println!("\nAfter punching holes:");
    println!("  Logical size: {} bytes", info.logical_size);
    println!("  Physical size: ~{} bytes", info.physical_size);
    println!("  Holes: {}", info.hole_count);

    std::fs::remove_file(path)?;
    Ok(())
}
```

### Exercice 2.4: Efficient Sparse File Copy (2.3.29.i)

```rust
use std::fs::{File, OpenOptions};
use std::io::{self, Read, Seek, SeekFrom, Write};

/// Sparse-aware file copier (2.3.29.i)
/// Preserves holes during copy operations
pub struct SparseCopier {
    buffer_size: usize,
}

impl SparseCopier {
    pub fn new(buffer_size: usize) -> Self {
        Self { buffer_size }
    }

    /// Copy file preserving sparse regions (2.3.29.i)
    pub fn copy_sparse(&self, src_path: &str, dst_path: &str) -> io::Result<CopyStats> {
        let src = File::open(src_path)?;
        let mut dst = File::create(dst_path)?;

        let finder = HoleFinder::new(&src)?;
        let info = finder.map_regions()?;

        let mut stats = CopyStats {
            bytes_copied: 0,
            bytes_skipped: 0,
            data_regions: 0,
            holes_preserved: 0,
        };

        // Set final file size first (creates sparse file)
        dst.set_len(info.logical_size)?;

        // Only copy data regions (skip holes)
        let mut src = File::open(src_path)?;
        let mut buffer = vec![0u8; self.buffer_size];

        for (offset, length) in &info.data_regions {
            src.seek(SeekFrom::Start(*offset))?;
            dst.seek(SeekFrom::Start(*offset))?;

            let mut remaining = *length;
            while remaining > 0 {
                let to_read = remaining.min(self.buffer_size as u64) as usize;
                let bytes_read = src.read(&mut buffer[..to_read])?;
                if bytes_read == 0 {
                    break;
                }
                dst.write_all(&buffer[..bytes_read])?;
                remaining -= bytes_read as u64;
                stats.bytes_copied += bytes_read as u64;
            }

            stats.data_regions += 1;
        }

        // Calculate skipped bytes (holes)
        for (_, length) in &info.hole_regions {
            stats.bytes_skipped += *length;
            stats.holes_preserved += 1;
        }

        Ok(stats)
    }

    /// Detect if buffer is all zeros (potential hole)
    fn is_zero_buffer(buffer: &[u8]) -> bool {
        // Check word-aligned for efficiency
        let (prefix, aligned, suffix) = unsafe {
            buffer.align_to::<u64>()
        };

        prefix.iter().all(|&b| b == 0)
            && aligned.iter().all(|&w| w == 0)
            && suffix.iter().all(|&b| b == 0)
    }

    /// Copy with automatic hole detection
    pub fn copy_with_hole_detection(
        &self,
        src_path: &str,
        dst_path: &str,
    ) -> io::Result<CopyStats> {
        let mut src = File::open(src_path)?;
        let metadata = src.metadata()?;
        let file_size = metadata.len();

        let mut dst = File::create(dst_path)?;
        dst.set_len(file_size)?;  // Create sparse file

        let mut buffer = vec![0u8; self.buffer_size];
        let mut stats = CopyStats::default();
        let mut offset = 0u64;

        while offset < file_size {
            let to_read = (file_size - offset).min(self.buffer_size as u64) as usize;
            let bytes_read = src.read(&mut buffer[..to_read])?;

            if bytes_read == 0 {
                break;
            }

            if Self::is_zero_buffer(&buffer[..bytes_read]) {
                // Skip writing zeros - creates hole
                stats.bytes_skipped += bytes_read as u64;
            } else {
                dst.seek(SeekFrom::Start(offset))?;
                dst.write_all(&buffer[..bytes_read])?;
                stats.bytes_copied += bytes_read as u64;
            }

            offset += bytes_read as u64;
        }

        Ok(stats)
    }
}

#[derive(Debug, Default)]
pub struct CopyStats {
    pub bytes_copied: u64,
    pub bytes_skipped: u64,
    pub data_regions: usize,
    pub holes_preserved: usize,
}

impl CopyStats {
    pub fn efficiency(&self) -> f64 {
        let total = self.bytes_copied + self.bytes_skipped;
        if total == 0 {
            0.0
        } else {
            self.bytes_skipped as f64 / total as f64 * 100.0
        }
    }
}

fn demonstrate_sparse_copy() -> io::Result<()> {
    println!("\nSparse-Aware Copy (2.3.29.i)");
    println!("============================\n");

    // Create source sparse file
    let src = "/tmp/sparse_src.dat";
    let dst = "/tmp/sparse_dst.dat";

    create_sparse_file(src, 10 * 1024 * 1024)?;  // 10MB
    write_sparse_data(src, b"HEADER DATA HERE", 0)?;
    write_sparse_data(src, b"MIDDLE SECTION", 5 * 1024 * 1024)?;
    write_sparse_data(src, b"FOOTER", 10 * 1024 * 1024 - 10)?;

    let copier = SparseCopier::new(64 * 1024);  // 64KB buffer
    let stats = copier.copy_sparse(src, dst)?;

    println!("Copy statistics:");
    println!("  Data copied: {} bytes", stats.bytes_copied);
    println!("  Holes skipped: {} bytes", stats.bytes_skipped);
    println!("  Efficiency: {:.1}% space saved", stats.efficiency());

    std::fs::remove_file(src)?;
    std::fs::remove_file(dst)?;

    Ok(())
}
```

---

## Partie 3: Projet Final - XAttr & Sparse File Manager

### Exercice 3.1: Combined Tool

```rust
//! Combined Extended Attributes and Sparse File Management Tool

use std::path::PathBuf;
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "fstools")]
#[command(about = "Extended attributes and sparse file tools")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Extended attribute operations
    Xattr {
        #[command(subcommand)]
        action: XattrAction,
    },
    /// Sparse file operations
    Sparse {
        #[command(subcommand)]
        action: SparseAction,
    },
}

#[derive(Subcommand)]
enum XattrAction {
    /// List all extended attributes
    List { path: PathBuf },
    /// Get an attribute value
    Get { path: PathBuf, name: String },
    /// Set an attribute
    Set { path: PathBuf, name: String, value: String },
    /// Remove an attribute
    Remove { path: PathBuf, name: String },
    /// Add a tag
    Tag { path: PathBuf, tag: String },
    /// List tags
    Tags { path: PathBuf },
}

#[derive(Subcommand)]
enum SparseAction {
    /// Analyze sparse file
    Analyze { path: PathBuf },
    /// Create sparse file
    Create { path: PathBuf, size: u64 },
    /// Punch hole in file
    Punch { path: PathBuf, offset: u64, length: u64 },
    /// Copy preserving holes
    Copy { source: PathBuf, dest: PathBuf },
}

fn main() -> std::io::Result<()> {
    let cli = Cli::parse();

    match cli.command {
        Commands::Xattr { action } => handle_xattr(action),
        Commands::Sparse { action } => handle_sparse(action),
    }
}

fn handle_xattr(action: XattrAction) -> std::io::Result<()> {
    match action {
        XattrAction::List { path } => {
            println!("Extended attributes for {:?}:", path);
            for name in XAttrManager::list(&path)? {
                if let Some(value) = XAttrManager::get(&path, &name)? {
                    let display = if value.iter().all(|b| b.is_ascii_graphic() || *b == b' ') {
                        String::from_utf8_lossy(&value).to_string()
                    } else {
                        format!("{:02X?}", value)
                    };
                    println!("  {} = {}", name, display);
                }
            }
        }
        XattrAction::Get { path, name } => {
            match XAttrManager::get_string(&path, &name)? {
                Some(value) => println!("{}", value),
                None => println!("Attribute not found"),
            }
        }
        XattrAction::Set { path, name, value } => {
            XAttrManager::set_string(&path, &name, &value)?;
            println!("Set {} = {}", name, value);
        }
        XattrAction::Remove { path, name } => {
            XAttrManager::remove(&path, &name)?;
            println!("Removed {}", name);
        }
        XattrAction::Tag { path, tag } => {
            let tagger = FileTagger::new();
            tagger.add_tag(&path, &tag)?;
            println!("Added tag: {}", tag);
        }
        XattrAction::Tags { path } => {
            let tagger = FileTagger::new();
            let tags = tagger.get_tags(&path)?;
            if tags.is_empty() {
                println!("No tags");
            } else {
                println!("Tags: {}", tags.join(", "));
            }
        }
    }
    Ok(())
}

fn handle_sparse(action: SparseAction) -> std::io::Result<()> {
    match action {
        SparseAction::Analyze { path } => {
            let file = std::fs::File::open(&path)?;
            let finder = HoleFinder::new(&file)?;
            let info = finder.map_regions()?;

            println!("Sparse file analysis: {:?}", path);
            println!("  Logical size:  {} bytes", info.logical_size);
            println!("  Physical size: {} bytes", info.physical_size);
            println!("  Sparseness:    {:.1}%", info.sparseness_ratio() * 100.0);
            println!("  Data regions:  {}", info.data_regions.len());
            println!("  Hole regions:  {}", info.hole_count);
        }
        SparseAction::Create { path, size } => {
            create_sparse_file(path.to_str().unwrap(), size)?;
            println!("Created sparse file: {} bytes", size);
        }
        SparseAction::Punch { path, offset, length } => {
            let file = std::fs::OpenOptions::new()
                .write(true)
                .open(&path)?;
            HolePuncher::punch_hole(&file, offset, length)?;
            println!("Punched hole: offset={}, length={}", offset, length);
        }
        SparseAction::Copy { source, dest } => {
            let copier = SparseCopier::new(64 * 1024);
            let stats = copier.copy_sparse(
                source.to_str().unwrap(),
                dest.to_str().unwrap(),
            )?;
            println!("Copy complete:");
            println!("  Bytes copied: {}", stats.bytes_copied);
            println!("  Bytes skipped: {}", stats.bytes_skipped);
            println!("  Efficiency: {:.1}%", stats.efficiency());
        }
    }
    Ok(())
}
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| XAttr get/set/list/remove | 25 |
| Namespace handling | 10 |
| Low-level nix API | 10 |
| Sparse file creation | 15 |
| SEEK_HOLE/SEEK_DATA | 15 |
| fallocate hole punching | 15 |
| Sparse-aware copy | 10 |
| **Total** | **100** |

---

## Ressources

- [xattr crate](https://docs.rs/xattr/)
- [nix crate xattr](https://docs.rs/nix/latest/nix/sys/xattr/)
- [Linux xattr(7)](https://man7.org/linux/man-pages/man7/xattr.7.html)
- [fallocate(2)](https://man7.org/linux/man-pages/man2/fallocate.2.html)
- [lseek(2) SEEK_HOLE](https://man7.org/linux/man-pages/man2/lseek.2.html)
