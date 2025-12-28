# clj-rocksdb

A self-contained Clojure wrapper around [RocksDB](https://rocksdb.org), providing all necessary binaries via [RocksDB Java API](https://github.com/facebook/rocksdb/tree/master/java/src/main/java/org/rocksdb).

This is a fork of [kotyo/clj-rocksdb](https://github.com/kotyo/clj-rocksdb) with:
- Updated RocksDB to 9.7.3
- Added `multi-get` support for efficient batch key retrieval

The source code and documentation is based on [Factual/clj-leveldb](https://github.com/Factual/clj-leveldb).

## Usage

**deps.edn:**
```clj
io.replikativ/clj-rocksdb {:mvn/version "0.1.65"}
```

**Leiningen:**
```clj
[io.replikativ/clj-rocksdb "0.1.65"]
```

To create or access a database, use `clj-rocksdb/create-db`.
The returned database object can be used with `clj-rocksdb/get`, `multi-get`, `put`, `delete`, `batch`, and `iterator`.

By default, RocksDB stores keys and values as byte arrays. You can define custom encoders and decoders in `create-db`:

```clj
clj-rocksdb> (def db (create-db "/tmp/rocksdb" 
                       {:key-encoder taoensso.nippy/freeze :key-decoder taoensso.nippy/thaw 
                        :val-encoder taoensso.nippy/freeze :val-decoder taoensso.nippy/thaw}))
#'clj-rocksdb/db
clj-rocksdb> (put db "a" "b")
nil
clj-rocksdb> (get db "a")
"b"
```

Both `put` and `delete` can take multiple values, which will be written in batch:

```clj
clj-rocksdb> (put db "a" "b" "c" "d" "e" "f")
nil
clj-rocksdb> (delete db "a" "c" "e")
nil
```

If you need to batch a collection of puts and deletes, use `batch`:

```clj
clj-rocksdb> (batch db {:put ["a" "b" "c" "d"] :delete ["j" "k" "l"]})
```

To efficiently read multiple keys in a single call, use `multi-get`:

```clj
clj-rocksdb> (put db "a" "b" "c" "d" "e" "f")
nil
clj-rocksdb> (multi-get db ["a" "c" "e" "missing"])
{"a" "b", "c" "d", "e" "f"}
```

Note: `multi-get` returns a sparse map - keys that don't exist are not included in the result. This uses RocksDB's `multiGetAsList` API for efficient batch retrieval.

We can also get a sequence of all key/value pairs, either in the entire database or within a given range using `iterator`:

```clj
clj-rocksdb> (put db "a" "b" "c" "d" "e" "f")
nil
clj-rocksdb> (iterator db)
(["a" "b"] ["c" "d"] ["e" "f"])
clj-rocksdb> (iterator db "c" nil)
(["c" "d"] ["e" "f"])
clj-rocksdb> (iterator db nil "c")
(["a" "b"] ["c" "d"])
```

Syncing writes to disk can be forced via `sync`, and compaction can be forced via `compact`.


## License

Originally by [kotyo](https://github.com/kotyo/clj-rocksdb).

EPL-v1.0 - Distributed under the Eclipse Public License, the same as Clojure.
