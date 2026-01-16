# Exercise 03: Two Pointers Technique

## Concepts Covered
- **1.1.i** Two pointers pattern
- Pair sum problems
- Partition problems
- Container with most water
- Three-sum variants

## Objective

Master the two pointers technique for efficient array manipulation. Two pointers reduce O(n^2) brute force to O(n) or O(n log n).

## Requirements

### Rust Implementation

```rust
pub mod two_pointers {
    /// Find pair with given sum in sorted array
    /// Returns indices (i, j) such that arr[i] + arr[j] == target
    pub fn pair_with_sum(arr: &[i32], target: i32) -> Option<(usize, usize)>;

    /// Find pair with given sum in unsorted array
    /// Returns indices of any valid pair
    pub fn pair_with_sum_unsorted(arr: &[i32], target: i32) -> Option<(usize, usize)>;

    /// Find all unique triplets that sum to zero
    pub fn three_sum(arr: &mut [i32]) -> Vec<[i32; 3]>;

    /// Find triplet closest to target sum
    pub fn three_sum_closest(arr: &mut [i32], target: i32) -> i32;

    /// Container with most water problem
    /// Returns maximum area between two lines
    pub fn max_area(heights: &[i32]) -> i64;

    /// Trapping rain water problem
    /// Returns total water that can be trapped
    pub fn trap_water(heights: &[i32]) -> i64;

    /// Remove duplicates from sorted array in-place
    /// Returns new length
    pub fn remove_duplicates(arr: &mut [i32]) -> usize;

    /// Dutch National Flag: partition into 3 groups
    /// Rearranges arr so that: all 0s < all 1s < all 2s
    pub fn dutch_flag(arr: &mut [i32]);

    /// Move all zeros to end, maintaining relative order
    pub fn move_zeros(arr: &mut [i32]);

    /// Check if array is palindrome
    pub fn is_palindrome(arr: &[i32]) -> bool;

    /// Reverse array segment [start, end]
    pub fn reverse_segment<T>(arr: &mut [T], start: usize, end: usize);

    /// Merge two sorted arrays into one sorted array
    /// Assumes arr1 has enough space at end for arr2
    pub fn merge_sorted(arr1: &mut [i32], len1: usize, arr2: &[i32]);

    /// Find subarray with given sum (positive numbers only)
    pub fn subarray_sum(arr: &[i32], target: i32) -> Option<(usize, usize)>;
}
```

### Python Implementation

```python
def pair_with_sum(arr: list[int], target: int) -> tuple[int, int] | None: ...
def pair_with_sum_unsorted(arr: list[int], target: int) -> tuple[int, int] | None: ...
def three_sum(arr: list[int]) -> list[tuple[int, int, int]]: ...
def three_sum_closest(arr: list[int], target: int) -> int: ...
def max_area(heights: list[int]) -> int: ...
def trap_water(heights: list[int]) -> int: ...
def remove_duplicates(arr: list[int]) -> int: ...
def dutch_flag(arr: list[int]) -> None: ...
def move_zeros(arr: list[int]) -> None: ...
def is_palindrome(arr: list[int]) -> bool: ...
def reverse_segment(arr: list, start: int, end: int) -> None: ...
def merge_sorted(arr1: list[int], len1: int, arr2: list[int]) -> None: ...
def subarray_sum(arr: list[int], target: int) -> tuple[int, int] | None: ...
```

## Two Pointers Patterns

### Pattern 1: Opposite Ends
```
left = 0
right = len - 1
while left < right:
    process(arr[left], arr[right])
    # Move pointers based on condition
```

### Pattern 2: Same Direction
```
slow = 0
for fast in range(len):
    if condition(arr[fast]):
        arr[slow] = arr[fast]
        slow += 1
return slow  # new length
```

### Pattern 3: Dutch Flag (3-way partition)
```
low = 0
mid = 0
high = len - 1
while mid <= high:
    if arr[mid] == 0:
        swap(arr, low, mid)
        low += 1
        mid += 1
    elif arr[mid] == 1:
        mid += 1
    else:
        swap(arr, mid, high)
        high -= 1
```

## Test Cases

```rust
#[test]
fn test_pair_sum() {
    let arr = vec![2, 7, 11, 15];
    assert_eq!(pair_with_sum(&arr, 9), Some((0, 1)));
    assert_eq!(pair_with_sum(&arr, 18), Some((1, 2)));
    assert_eq!(pair_with_sum(&arr, 5), None);
}

#[test]
fn test_three_sum() {
    let mut arr = vec![-1, 0, 1, 2, -1, -4];
    let result = three_sum(&mut arr);
    // Should contain [-1, -1, 2] and [-1, 0, 1]
    assert_eq!(result.len(), 2);
    assert!(result.contains(&[-1, -1, 2]));
    assert!(result.contains(&[-1, 0, 1]));
}

#[test]
fn test_max_area() {
    let heights = vec![1, 8, 6, 2, 5, 4, 8, 3, 7];
    assert_eq!(max_area(&heights), 49);

    let heights = vec![1, 1];
    assert_eq!(max_area(&heights), 1);
}

#[test]
fn test_trap_water() {
    let heights = vec![0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1];
    assert_eq!(trap_water(&heights), 6);

    let heights = vec![4, 2, 0, 3, 2, 5];
    assert_eq!(trap_water(&heights), 9);
}

#[test]
fn test_remove_duplicates() {
    let mut arr = vec![1, 1, 2, 2, 2, 3, 4, 4, 5];
    let len = remove_duplicates(&mut arr);
    assert_eq!(len, 5);
    assert_eq!(&arr[..len], &[1, 2, 3, 4, 5]);
}

#[test]
fn test_dutch_flag() {
    let mut arr = vec![2, 0, 2, 1, 1, 0];
    dutch_flag(&mut arr);
    assert_eq!(arr, vec![0, 0, 1, 1, 2, 2]);

    let mut arr = vec![2, 0, 1];
    dutch_flag(&mut arr);
    assert_eq!(arr, vec![0, 1, 2]);
}

#[test]
fn test_move_zeros() {
    let mut arr = vec![0, 1, 0, 3, 12];
    move_zeros(&mut arr);
    assert_eq!(arr, vec![1, 3, 12, 0, 0]);
}

#[test]
fn test_merge_sorted() {
    let mut arr1 = vec![1, 2, 3, 0, 0, 0];
    let arr2 = vec![2, 5, 6];
    merge_sorted(&mut arr1, 3, &arr2);
    assert_eq!(arr1, vec![1, 2, 2, 3, 5, 6]);
}

#[test]
fn test_subarray_sum() {
    let arr = vec![1, 4, 20, 3, 10, 5];
    assert_eq!(subarray_sum(&arr, 33), Some((2, 4))); // 20 + 3 + 10 = 33
    assert_eq!(subarray_sum(&arr, 100), None);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Pair sum (sorted and unsorted) | 15 |
| Three sum (no duplicates in result) | 15 |
| Container with most water | 10 |
| Trapping rain water | 15 |
| Remove duplicates | 10 |
| Dutch flag partitioning | 10 |
| Move zeros | 5 |
| Merge sorted arrays | 10 |
| Subarray sum | 10 |
| **Total** | **100** |

## Complexity Requirements

| Function | Time | Space |
|----------|------|-------|
| pair_with_sum | O(n) | O(1) |
| pair_with_sum_unsorted | O(n) | O(n) |
| three_sum | O(n^2) | O(1) aux |
| max_area | O(n) | O(1) |
| trap_water | O(n) | O(1) |
| remove_duplicates | O(n) | O(1) |
| dutch_flag | O(n) | O(1) |
| move_zeros | O(n) | O(1) |
| merge_sorted | O(n+m) | O(1) |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `two_pointers.py`
