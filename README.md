# Performance Comparison Analysis: Wasmer vs. WASMTime

## Author: Mohammadreza Ashouir <ashourics@gmail.com>

## Video:
https://youtu.be/0c5SmgT1_P8


Recently I had a project regarding the security and the performance of WASM compilers for a blockchain ecosystem. I realized there are two major WASM interpreters/JITs, especially popular among the blockchain networks, i.e., WASMER and WASMTIME.

- wasmer - Interpreter/JIT made at wasmer.io. Based on Cranelift but has other compiler backends, too (LLVM, standalone)

- wasmtime - Interpreter/JIT. It does the same job wasmer but made by different developers. Based on Cranelift. Wasmtime doesn't seem to be involved with wapm at all. Note that wapm is a package manager similar to npm.

Installation:

### Wasmer Installation:
```
 curl https://get.wasmer.io -sSfL | sh
or
brew install wasmer
```

### Wasmtime Installation: 
```
curl https://wasmtime.dev/install.sh -sSf | bash
```

For the benchmarking, I initially wrote a couple of algorithms, such as sortings, but apparently, they need to be CPU-challenging on my machine. Thus, I finally ended up with Fibonacci to challenge the machine (Apple M1). This algorithm can be pretty slow for larger values of n as it involves lots of calculations and has a time complexity of O(2^n).

```
fn main() {
    println!("{}",fibo(42));
}

fn fibo(n: u32) -> u32 {

    if n < 2 {
        1
    } else {
        fibo(n-1) + fibo(n-2)
    }
}
```


### Results:

```
# Build native code
cargo build - release
time ./target/release/fibo
433494437
./target/release/fibo 0.83s user 0.00s system 81% cpu 1.021 total
# Build wasm code
cargo build - release - target wasm32-wasi
# Now on Wasmer
wasmer target/wasm32-wasi/release/fibo.wasm 0.99s user 0.01s system 96% cpu 1.040 total
# Now on Wasmtime
wasmtime target/wasm32-wasi/release/fibo.wasm 0.90s user 0.00s system 99% cpu 0.906 total
Second round
# Target:
./target/release/fibo 0.80s user 0.01s system 98% cpu 0.818 total
# Wasmer:
wasmer target/wasm32-wasi/release/fibo.wasm 1.05s user 0.03s system 98% cpu 1.096 total
# Wasmtime:
wasmtime target/wasm32-wasi/release/fibo.wasm 0.96s user 0.03s system 101% cpu 0.982 total
Third round
./target/release/fibo 0.81s user 0.01s system 98% cpu 0.833 total
wasmer target/wasm32-wasi/release/fibo.wasm 0.98s user 0.02s system 95% cpu 1.044 total
wasmtime target/wasm32-wasi/release/fibo.wasm 0.90s user 0.01s system 98% cpu 0.930 total
Fourth round
./target/release/fibo 0.80s user 0.01s system 98% cpu 0.818 total
wasmer target/wasm32-wasi/release/fibo.wasm 0.98s user 0.01s system 97% cpu 1.018 total
wasmtime target/wasm32-wasi/release/fibo.wasm 0.91s user 0.01s system 96% cpu 0.957 total
```

### Conclusion
Since both compilers use Cranelift for the backend, the performance for wasmtime and wasmer is noticeably different. wasmtime looks faster (at least for this small benchmark)-also, the memory use: native code, max 1.8 MB max resident memory. The wasm versions have a memory overhead of 10–15 megabytes.

- wasmer: 24 MB max resident size

- wasmtime: 12 MB max resident size

JIT starts up slower than native code because they need extra time to load and compile the code before running it.
wasmer and wasmtime both cache the results of the JIT compilation automatically. For both runtimes, the cached files are opaque blobs and named only by hash. For wasmer we can clear the cache with wasmer cache clean. However, wasmtime doesn't seem to have a way to tell it to clean the cache, but it does have some settings for how the cache is managed, and deleting the files by hand makes it shrug and re-create them.

- Both wasmer and wasmtime are easy to embed (in Rust programs) or easy-to-use standalone.
- They use almost similar codebases written in Rust.
- wasmer is run by a corporation that also runs the package manager wapm, while wasmtime is run by a non-profit backed mainly by Mozilla and Fastly.
