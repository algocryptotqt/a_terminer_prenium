# ex14: RAID Simulator

**Module**: 2.3 - File Systems
**Difficulte**: Difficile
**Duree**: 8h
**Score qualite**: 97/100

## Concepts Couverts

### 2.3.30: RAID Concepts (9 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | RAID concept | Redundant Array |
| b | RAID 0 | Striping |
| c | RAID 1 | Mirroring |
| d | RAID 5 | Distributed parity |
| e | RAID 6 | Double parity |
| f | RAID 10 | Mirror + stripe |
| g | Software vs hardware | Trade-offs |
| h | mdadm | Linux software RAID |
| i | Implementation | Simple striping en Rust |

---

## Sujet

Implementer un simulateur RAID complet supportant les niveaux 0, 1, 5, 6 et 10 avec gestion des pannes et reconstruction.

### Structures

```c
// Virtual disk
typedef struct {
    char name[64];
    uint8_t *data;
    size_t size;
    bool failed;            // j: Degraded mode
    bool is_hot_spare;      // i: Hot spare
    uint64_t reads;
    uint64_t writes;
} vdisk_t;

// RAID array
typedef struct {
    int level;              // a-e: RAID level (0,1,5,6,10)
    vdisk_t **disks;
    size_t disk_count;
    size_t stripe_size;     // g: Chunk size
    size_t total_size;
    size_t usable_size;

    // Hot spares (i)
    vdisk_t **hot_spares;
    size_t spare_count;

    // State
    bool degraded;          // j: Degraded mode active
    int failed_disk;        // Index of failed disk (-1 if none)
    bool rebuilding;        // h: Rebuild in progress
    float rebuild_progress;
} raid_array_t;

// RAID configuration
typedef struct {
    int level;
    size_t disk_count;
    size_t disk_size;
    size_t stripe_size;     // g
    size_t spare_count;     // i
} raid_config_t;

// Performance metrics
typedef struct {
    double read_mbps;
    double write_mbps;
    double iops;
    double rebuild_time_sec;
    double fault_tolerance;  // Number of disks that can fail
} raid_perf_t;
```

### API

```c
// Array lifecycle
raid_array_t *raid_create(raid_config_t *config);
void raid_destroy(raid_array_t *array);

// I/O operations
ssize_t raid_read(raid_array_t *array, void *buf, size_t count, off_t offset);
ssize_t raid_write(raid_array_t *array, const void *buf, size_t count, off_t offset);

// 2.3.30.a: RAID 0 - Striping
void raid0_stripe_write(raid_array_t *array, const void *buf, size_t count, off_t offset);
void raid0_stripe_read(raid_array_t *array, void *buf, size_t count, off_t offset);

// 2.3.30.b: RAID 1 - Mirroring
void raid1_mirror_write(raid_array_t *array, const void *buf, size_t count, off_t offset);
void raid1_mirror_read(raid_array_t *array, void *buf, size_t count, off_t offset);

// 2.3.30.c: RAID 5 - Distributed parity
void raid5_write(raid_array_t *array, const void *buf, size_t count, off_t offset);
void raid5_read(raid_array_t *array, void *buf, size_t count, off_t offset);
int raid5_parity_disk(raid_array_t *array, off_t stripe_num);

// 2.3.30.d: RAID 6 - Double parity (P + Q)
void raid6_write(raid_array_t *array, const void *buf, size_t count, off_t offset);
void raid6_read(raid_array_t *array, void *buf, size_t count, off_t offset);
void raid6_calc_pq(raid_array_t *array, off_t stripe, uint8_t *P, uint8_t *Q);

// 2.3.30.e: RAID 10 - Mirror + Stripe
void raid10_write(raid_array_t *array, const void *buf, size_t count, off_t offset);
void raid10_read(raid_array_t *array, void *buf, size_t count, off_t offset);

// 2.3.30.f: Parity calculation
uint8_t raid_calc_parity(uint8_t **data, size_t disk_count, size_t offset);
void raid_xor_blocks(uint8_t *dst, const uint8_t *src, size_t len);

// 2.3.30.g: Stripe management
size_t raid_get_stripe_size(raid_array_t *array);
void raid_set_stripe_size(raid_array_t *array, size_t size);
int raid_offset_to_disk(raid_array_t *array, off_t offset);
off_t raid_offset_to_stripe(raid_array_t *array, off_t offset);

// 2.3.30.h: Rebuild
int raid_rebuild(raid_array_t *array, int disk_index);
int raid_rebuild_from_spare(raid_array_t *array);
float raid_get_rebuild_progress(raid_array_t *array);
void raid_cancel_rebuild(raid_array_t *array);

// 2.3.30.i: Hot spare
int raid_add_hot_spare(raid_array_t *array, size_t disk_size);
int raid_remove_hot_spare(raid_array_t *array, int spare_index);
int raid_activate_spare(raid_array_t *array);

// 2.3.30.j: Degraded mode
int raid_fail_disk(raid_array_t *array, int disk_index);
bool raid_is_degraded(raid_array_t *array);
int raid_degraded_read(raid_array_t *array, void *buf, size_t count, off_t offset);
int raid_get_failed_disks(raid_array_t *array, int *disks, size_t max);

// Comparison and analysis
void raid_compare_levels(raid_perf_t *perf, size_t disk_count, size_t disk_size);
void raid_benchmark(raid_array_t *array, raid_perf_t *result);

// Statistics
typedef struct {
    uint64_t total_reads;
    uint64_t total_writes;
    uint64_t parity_updates;      // f
    uint64_t stripe_operations;   // g
    uint64_t rebuild_operations;  // h
    uint64_t degraded_reads;      // j
} raid_stats_t;

void raid_get_stats(raid_array_t *array, raid_stats_t *stats);
void raid_print_layout(raid_array_t *array);
```

---

## Exemple

```c
int main(void) {
    // Create RAID 5 array with 4 disks
    raid_config_t config = {
        .level = 5,             // c: RAID 5
        .disk_count = 4,
        .disk_size = 1024 * 1024,  // 1MB per disk
        .stripe_size = 64 * 1024,  // g: 64KB stripes
        .spare_count = 1           // i: 1 hot spare
    };

    raid_array_t *array = raid_create(&config);
    printf("Usable size: %zu bytes (%.1f%% efficiency)\n",
           array->usable_size,
           100.0 * array->usable_size / (config.disk_count * config.disk_size));

    // Write data
    char data[4096] = "RAID test data...";
    raid_write(array, data, sizeof(data), 0);

    // Read back
    char buf[4096];
    raid_read(array, buf, sizeof(buf), 0);

    // 2.3.30.f: Show parity calculation
    printf("Parity at offset 0: 0x%02x\n",
           raid_calc_parity((uint8_t**)array->disks, array->disk_count - 1, 0));

    // 2.3.30.j: Simulate disk failure
    printf("Simulating disk 1 failure...\n");
    raid_fail_disk(array, 1);

    if (raid_is_degraded(array)) {
        printf("Array is now degraded!\n");

        // Read still works via parity reconstruction
        raid_degraded_read(array, buf, sizeof(buf), 0);
        printf("Degraded read successful\n");

        // 2.3.30.i: Activate hot spare
        if (array->spare_count > 0) {
            raid_activate_spare(array);

            // 2.3.30.h: Rebuild
            printf("Starting rebuild...\n");
            raid_rebuild_from_spare(array);

            while (array->rebuilding) {
                printf("Rebuild progress: %.1f%%\n",
                       raid_get_rebuild_progress(array) * 100);
                usleep(100000);
            }
            printf("Rebuild complete!\n");
        }
    }

    // Compare all RAID levels
    printf("\n=== RAID Level Comparison ===\n");
    raid_perf_t perf[5];
    int levels[] = {0, 1, 5, 6, 10};
    const char *names[] = {"RAID 0", "RAID 1", "RAID 5", "RAID 6", "RAID 10"};

    for (int i = 0; i < 5; i++) {
        raid_config_t test_cfg = {
            .level = levels[i],
            .disk_count = 4,
            .disk_size = 1024 * 1024,
            .stripe_size = 64 * 1024,
            .spare_count = 0
        };
        raid_array_t *test = raid_create(&test_cfg);
        raid_benchmark(test, &perf[i]);

        printf("%s: Read=%.1f MB/s, Write=%.1f MB/s, Fault tolerance=%.0f disks\n",
               names[i], perf[i].read_mbps, perf[i].write_mbps,
               perf[i].fault_tolerance);
        raid_destroy(test);
    }

    // Print layout visualization
    printf("\n=== RAID 5 Layout ===\n");
    raid_print_layout(array);
    /*
    Stripe | Disk0 | Disk1 | Disk2 | Disk3
    -------|-------|-------|-------|-------
       0   |  D0   |  D1   |  D2   |  P0
       1   |  D3   |  D4   |  P1   |  D5
       2   |  D6   |  P2   |  D7   |  D8
       3   |  P3   |  D9   |  D10  |  D11
    */

    raid_destroy(array);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_raid0_striping()       // 2.3.30.a
#[test] fn test_raid1_mirroring()      // 2.3.30.b
#[test] fn test_raid5_parity()         // 2.3.30.c
#[test] fn test_raid6_double_parity()  // 2.3.30.d
#[test] fn test_raid10_combined()      // 2.3.30.e
#[test] fn test_parity_calculation()   // 2.3.30.f
#[test] fn test_stripe_size()          // 2.3.30.g
#[test] fn test_rebuild()              // 2.3.30.h
#[test] fn test_hot_spare()            // 2.3.30.i
#[test] fn test_degraded_mode()        // 2.3.30.j
#[test] fn test_data_integrity()
#[test] fn test_performance()
#[test] fn test_multiple_failures()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| RAID 0 striping (2.3.30.a) | 10 |
| RAID 1 mirroring (2.3.30.b) | 10 |
| RAID 5 distributed parity (2.3.30.c) | 15 |
| RAID 6 double parity (2.3.30.d) | 15 |
| RAID 10 combined (2.3.30.e) | 10 |
| Parity calculation (2.3.30.f) | 10 |
| Stripe management (2.3.30.g) | 5 |
| Rebuild process (2.3.30.h) | 10 |
| Hot spare (2.3.30.i) | 5 |
| Degraded mode (2.3.30.j) | 10 |
| **Total** | **100** |

---

## Fichiers

```
ex14/
├── raid.h
├── raid_core.c
├── raid0.c
├── raid1.c
├── raid5.c
├── raid6.c
├── raid10.c
├── rebuild.c
└── Makefile
```
