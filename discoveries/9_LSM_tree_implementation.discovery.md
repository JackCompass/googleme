# Building an LSM Tree Storage Engine in Node.js

Whenever we talk about high-throughput, write-heavy storage systems link Cassandra, RocksDB, or other databases, LSM (Log-Structured Merge) trees are almost always there powering them under the hood. 

Recently, I decided to sit down and implement an LSM-tree storage engine from scratch to get my hands dirty and understand how the all the pieces actually interact. And because this was just a learning project, I decided to write it in **Node.js**.

Before you start rambling about why **node** in the comments: yes, Node.js is definitely **not** the best language to implement a database storage engine. We will get into why having a single-threaded runtime is a major bottleneck later, but building it in Node gave a super transparent view into the mechanics of WAL, MemTables, SSTables, Bloom Filters, and Compaction.

Breaking it down in how writes and reads work, how compaction cleans up the mess, and where my implementation differ from the real beasts running in production.

Checkout the repository [here](https://github.com/JackCompass/LSM-tree) for the code.

---

## The Big Picture: How Writes Flow

In traditional B-Tree databases, writes update pages in-place on disk, which means lots of random I/O. Random writes on disk are slow. LSM trees flip this paradigm upside down: **all writes are sequential**.

When a write (`put` or `delete`) comes in:

1. We append the change to an on-disk **Write-Ahead Log (**`wal.log`**)**.
2. We insert the key-value pair into an in-memory sorted structure called the **Active MemTable** (in RAM).
3. Once the MemTable reaches a size threshold in this case (4MB), it freezes into an **Immutable MemTable**, flushes to disk as a sorted file called an **SSTable (Sorted String Table)**, and the old WAL is deleted.
4. Background **Compaction** merges overlapping SSTables together to clean up deleted keys (tombstones) and stale updates.

```
       Client Write (put / delete)
              │
       ┌──────┴────────────────────────┐
       ▼                               ▼
 [ 1. Append ]                   [ 2. Insert ]
 ┌───────────────┐              ┌──────────────────┐
 │    wal.log    │ (fsyncSync)  │  Active MemTable │ (In RAM)
 └───────────────┘              └────────┬─────────┘
                                         │ (Hits threshold)
                                         ▼
                                ┌──────────────────┐
                                │ Immutable MemTable│
                                └────────┬─────────┘
                                         │ (Flush to disk)
                                         ▼
                                ┌──────────────────┐
                                │    L0 SSTables   │ [Data | Index | Bloom | Footer]
                                └────────┬─────────┘
                                         │ (Compaction merge)
                                         ▼
                                ┌──────────────────┐
                                │    L1 SSTables   │
                                └──────────────────┘
```

---

## 1. Why the WAL (`wal.log`) is important

The whole reason LSM writes are blazing fast is that the active MemTable lives purely in volatile RAM. But RAM has one tiny flaw: if someone pulls the plug or the Node process crashes, everything in memory vanishes instantly.

To guarantee durability (the "D" in ACID), we must record our intention *before* modifying memory. That is the **Write-Ahead Log (WAL)**. 

Every single mutation (PUT or DELETE) is encoded into a binary frame and appended sequentially to `wal.log`:

```
┌──────────────┬──────────────┬──────────────┬─────────────┬───────────────┐
│ CRC32 (4 B)  │ OpType (1 B) │ KeyLen (2 B) │ ValLen (4 B)│ Key + Value   │
└──────────────┴──────────────┴──────────────┴─────────────┴───────────────┘
```

When our engine starts up or recovers from a crash, it scans the surviving WAL files from oldest to newest:

1. Validates the CRC32 checksum for each frame (to detect partial/torn writes if the machine died mid-append).
2. Replays valid records back into a fresh MemTable.
3. Once recovered, flushes that table directly to a durable SSTable on disk and cleanly discards the old journal files.

No data lost. Simple enough RIGHT!!! 

---

## 2. Don't Trust the OS: Handing Data to the OS vs `fsyncSync`

I thing that suprised my is the file handling. Here is a trap that catches a lot of developers when working with file systems:

When you call `fs.writeFileSync()` or `fs.writeSync()` in Node.js, you might think: *"Great! My bytes are safely written into the storage disk!"* 

**Nope.**

Operating systems don't immediately write your data to disk. Doing physical disk I/O on every single write system call would bring the OS to a crawl. Instead, the OS kernel copies your bytes into its in-memory **page cache (buffer cache)** and marks those pages as "dirty". The OS then lazily flushes those dirty pages to the physical disk whenever its background kernel flusher gets around to it (or when memory runs low).

If the power dies or the kernel panics while those bytes are sitting in the OS page cache, **your write is gone**, even though your application thought it wrote successfully!

To ensure actual durability, we must force the OS to flush its kernel page cache straight down to physical storage. In Node.js, we do this using `fdatasyncSync(fd)` (or `fsyncSync(fd)`):

```typescript
// Encode the binary frame
const record: WalRecord = { op: OpType.Put, key, value: stringValue };
const frame = encode(record);

// 1. Hand bytes to the OS kernel
writeSync(this.walFd, frame);

// 2. FORCE the kernel to flush dirty pages down to physical disk NOW
fdatasyncSync(this.walFd);

// 3. ONLY THEN update the in-memory MemTable
this.activeMemTable.put(key, stringValue);
```

By calling `fdatasyncSync`, we block until the disk controller confirms the bytes have actually hit non-volatile media. Only after that guarantee. we safely touch the MemTable for updating.

---

## 3. The Other Half: How Reads Actually Work

Now, writes are great, but what happens when a client calls `get("user:42")`?

Because LSM trees don't update data in place, a newer update to a key might be in RAM while an older version of that same key is sitting in an SSTable on disk. So reads must search in strict reverse chronological order (newest to oldest):

```
Client get(key)
      │
      ▼
1. Active MemTable (RAM) ───────► Found? Return immediately!
      │ (Miss)
      ▼
2. Frozen / Immutable MemTable ──► Found? Return immediately!
      │ (Miss)
      ▼
3. Level 0 SSTables (Disk) ──────► Check newest to oldest L0 file
      │ (Miss)
      ▼
4. Level 1 SSTables (Disk) ──────► Check newest to oldest L1 file
      │ (Miss)
      ▼
Return null (Key not found)
```

Here is the step-by-step read ladder:

1. **Active MemTable (RAM)**: This has the absolute newest writes. If the key exists here, we stop right away and return it. (If it has a tombstone flag, it was deleted, so we return `null`).
2. **Frozen / Immutable MemTable (RAM)**: If a background flush is currently pouring data to disk, this frozen table holds the second newest writes. If found here, return it.
3. **On-Disk SSTables (Newest to Oldest)**:
   - First, we search Level 0 SSTables from the newest file to the oldest file.
   - Then, we search Level 1 SSTables.
   - **Crucial Rule**: The moment we find a match, **we stop searching**. Since we check newest files first, the first match is guaranteed to be the freshest value. If that match is a tombstone, the key is officially dead.

---

## 4. The SSTable: Bloom Filter, Sparse Index, and the Footer

Reading from disk sounds slow, especially if we have multiple SSTable files. What if we have to check a 1 GB file on disk for a key that isn't even in there?

To keep read latency low, our SSTable binary layout contains four critical components:

```
┌─────────────────────────────────────────────────────────────────────────┐
│ DATA BLOCK: Sequential Sorted Records                                  │
│   Record 1: [KeyLen (2B)][ValLen (4B)][TombstoneFlag (1B)][Key][Val]   │
│   Record 2: ...                                                         │
│   ...                                                                   │
│   Record N: ...                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│ SPARSE INDEX BLOCK: (offsets for every K-th key, e.g., every 16th key)  │
│   [KeyLen (2B)][KeyStr][OffsetInFile (8B)]                              │
├─────────────────────────────────────────────────────────────────────────┤
│ BLOOM FILTER BLOCK: Serialized Bit Array                                │
├─────────────────────────────────────────────────────────────────────────┤
│ FOOTER (Fixed Size - 28 Bytes):                                         │
│   [IndexOffset (8B)][IndexLen (4B)]                                     │
│   [BloomOffset (8B)][BloomLen (4B)]                                     │
│   [MagicNumber 0x4C534D31 (4B)]                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

Let's dissect each component:

### The Bloom Filter

Before doing any disk reads to look for data, we check the Bloom filter in memory.

A Bloom filter is a space-efficient probabilistic data structure. When checking if a key exists:

- If it says **No**, the key **definitely does not exist** in the SSTable. We instantly skip reading this entire file! No wasted disk reads.
- If it says **Yes**, the key *might* be in the file (there is a small 1% false positive rate). Only then do we proceed to inspect the index.

We construct the Bloom filter by running the key through multiple hash values using double-hashing (Kirsch-Mitzenmacher optimization) and setting the corresponding bits in a bitset Buffer.

### The Sparse Index

If we stored an index entry for *every single key*, our index would be almost as big as the data itself.

Instead, we use a **Sparse Index**: we only record the byte offset for every $K$-th key (for example, every 16th key).

- When looking for key `"user:42"`, we binary-search the sparse index in memory.
- The index tells us that `"user:42"` falls somewhere between offset `1024` and `3072`.
- We read only that tiny 2 KB slice from disk into RAM and scan through those few records locally.

### Why is the Footer at the End of the File?

Notice how the **Footer** is located at the very end of the file instead of the beginning. Why?

When writing an SSTable, you stream records sequentially: you don't know ahead of time how many bytes the data block will consume, nor where the sparse index or bloom filter will begin or end until you finish writing all the data!

So you write:

1. The sorted data records.
2. The sparse index (remembering its byte offset).
3. The serialized bloom filter (remembering its byte offset).
4. **The fixed-size Footer (28 bytes)**, containing the exact byte offsets and lengths of the index and bloom filter, ending with a magic number (`0x4C534D31`, which translates in ASCII to `"LSM1"`).

When reading the SSTable:

```typescript
// Seek to the last 28 bytes of the file
const footer = Buffer.allocUnsafe(28);
readSync(fd, footer, 0, 28, fileSize - 28);

// Verify magic bytes
const magic = footer.readUInt32BE(24);
if (magic !== 0x4c534d31) throw new Error("Invalid SSTable file!");

// Read offsets for bloom and sparse index
const bloomOffset = Number(footer.readBigUInt64BE(12));
const bloomLen = footer.readUInt32BE(20);
const indexOffset = Number(footer.readBigUInt64BE(0));
const indexLen = footer.readUInt32BE(8);
```

You read the footer first, load the bloom filter and sparse index, and you're ready to do lightning-fast point lookups!

---

## 5. The Ledger: What role does `MANIFEST.json` play?

With SSTables constantly being created by flushes and merged/deleted by compactions, how does the database keep track of what files are active? If your machine suddenly loses power, how does it know which SSTables actually count and which ones were halfway through being written?

Enter `MANIFEST.json`.

The manifest is the engine's master ledger. It tracks the exact list of valid SSTable filenames belonging to Level 0 and Level 1. Whenever a flush or compaction finishes:

1. The new SSTable is written and synced to disk.
2. The `MANIFEST.json` file is updated and immediately `fsync`'d.
3. The obsolete input files are safely deleted.

On restart, the engine reads `MANIFEST.json` first. If an SSTable is on disk but not in the manifest, it's considered uncommitted junk and ignored.

---

## 6. Compaction & K-Way Merge (AI Explanation)

As you keep writing to your database, you end up with dozens of Level 0 SSTables piling up on disk. 

Having too many files creates two big problems:

1. **Reads get slow**: You have to check multiple files for every lookup.
2. **Disk space gets wasted**: If you updated the key `"user:name"` 10 times, you are storing 9 old, useless versions on disk!

To fix this, we run **Compaction**. And under the hood, compaction uses an algorithm called **K-Way Merge**.

### The Desk Analogy

Imagine you have 4 separate stacks of paper receipts on your desk (these are our 4 L0 SSTables).  
Each stack is already sorted alphabetically from A to Z.

```
Stack 1: [ Apple ($5), Banana ($2), Grape ($3) ]
Stack 2: [ Banana ($4 - New), Cherry ($1), Orange ($6) ]
Stack 3: [ Apple ($7 - New), Mango ($5) ]
Stack 4: [ Cherry (DELETED), Watermelon ($8) ]
```

Searching through all 4 stacks every single time is exhausting. You want **one single master stack**.

Here is how you do the **K-Way Merge**:

1. You look at the **top card of each of the 4 stacks** at the same time:
   - Stack 1 top: `Apple`
   - Stack 2 top: `Banana`
   - Stack 3 top: `Apple`
   - Stack 4 top: `Cherry`
2. Who comes first alphabetically? `Apple`!
3. But notice `Apple` is on both Stack 1 and Stack 3. Which one is the real one? You pick the one from the **newer stack** (`Apple ($7)`), throw the older `Apple ($5)` into the trash can, and advance both stacks.
4. If a card says `Cherry (DELETED)`, and there's no older file behind it that needs to know it was deleted, you throw it straight into the trash too.
5. Repeat this comparison until all 4 stacks are emptied.

Boom! You now have a single, clean, perfectly sorted Level 1 file with zero duplicates and zero garbage.

In code, we use a **Min-Heap (Priority Queue)** so comparing the top items of $K$ files is super fast ($O(\log K)$).

---

## Toy vs Production: Where This Implementation Differs

Building a learning engine is awesome, but it's equally important to understand where our simplified version differs from production systems like LevelDB or RocksDB.

### 1. Choosing Node.js (and the Single-Thread Bottleneck)

Let's address the elephant in the room: Node.js runs on Google V8 with a **single-threaded event loop**.

In a real database engine, compaction (reading multiple large files, running a k-way merge sort, and writing new SSTables) is heavily CPU and I/O intensive. In our pure JavaScript implementation:

- Hashing CRC32 checksums, serializing buffers, and running compaction merges all execute right on the main JavaScript thread.
- If compaction is running, incoming `put()` and `get()` calls are blocked or delayed.
- V8's garbage collector pauses can introduce latency spikes when allocating millions of short-lived buffers.

In languages like C++, Rust, or Go, compaction runs quietly in background worker thread pools with zero-copy buffer pools and fine-grained concurrent locks.

### 2. Immediate Per-Write `fsync` vs Group Commit / Batching

In our implementation, every single `put()` call executes:

```typescript
writeSync(this.walFd, encode(record));
fdatasyncSync(this.walFd); // Forces physical disk write on EVERY write
```

While this guarantees maximum durability, physical disk syncs are slow (often limited to hundreds or low thousands of IOPS depending on the drive). 

Production engines use **Group Commit**:

- Incoming writes are queued up in an in-memory buffer.
- A single `fsync` is triggered periodically (e.g. every 10ms) or after a batch threshold (e.g., 64 KB of writes).
- This amortizes the cost of the disk sync across hundreds of writes simultaneously, boosting throughput by orders of magnitude.

### 3. Two Levels of Compaction vs Multi-Level Compaction

In our implementation, we kept things simple with **exactly two levels**:

- **Level 0 (L0)**: Created directly from MemTable flushes. Files can have overlapping key ranges.
- **Level 1 (L1)**: Merged output from L0 compactions. Deduplicated and sorted.

In real-world storage engines, leveled compaction typically has 6 or 7 levels ($L_0$ to $L_6$):

- Each level has an exponential size limit (e.g., $L_1 = 10\text{ MB}$, $L_2 = 100\text{ MB}$, $L_3 = 1\text{ GB}$, etc., growing by a 10x factor).
- Files in $L_1$ and beyond have strictly non-overlapping key ranges.
- This keeps both **read amplification** (how many files you have to check to find a key) and **space amplification** (how much space is wasted on stale/deleted keys) tightly bounded.

---

## Final Thoughts

Building an LSM tree engine from scratch explains so much of modern database internals. When you see how the Write-Ahead Log protects against memory crashes, how `fsyncSync` ensures data actually settles on physical platters, how the Bloom filter and Sparse Index work hand-in-hand with the Footer, and how K-way merge compaction tidies up the disk, the architecture feels remarkably elegant.

Node.js might not be what you'd pick to build the next SQLite or RocksDB, but watching your own custom binary format, MemTable swaps, and compaction merges work end-to-end is super satisfying.

PS: If you ever write your own file formats, remember: always put the footer at the end. Your streaming write pipeline will thank you later. 😉