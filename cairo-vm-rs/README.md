<div align="center">
<img src="./bonaparte.webp" height="150">

### ⚡ Cairo-vm ⚡

A faster and safer implementation of the Cairo VM in Rust

[Report Bug](https://github.com/lambdaclass/cairo-vm/issues/new?assignees=&labels=bug&projects=&template=bug_report.md&title=bug%3A+) · [Request Feature](https://github.com/lambdaclass/cairo-vm/issues/new?labels=enhancement&title=feat%3A+)

[![rust](https://github.com/lambdaclass/cairo-vm/actions/workflows/rust.yml/badge.svg)](https://github.com/lambdaclass/cairo-vm/actions/workflows/rust.yml)
[![codecov](https://img.shields.io/codecov/c/github/lambdaclass/cairo-vm)](https://codecov.io/gh/lambdaclass/cairo-vm)
[![license](https://img.shields.io/github/license/lambdaclass/cairo-vm)](/LICENSE)
[![pr-welcome]](#-contributing)
[![Telegram Chat][tg-badge]][tg-url]

[pr-welcome]: https://img.shields.io/static/v1?color=orange&label=PRs&style=flat&message=welcome
[tg-badge]: https://img.shields.io/static/v1?color=green&logo=telegram&label=chat&style=flat&message=join
[tg-url]: https://t.me/starknet_rs

</div>

## Table of Contents

- [Disclaimer](#%EF%B8%8F-disclaimer)
- [About](#-about)
  - [The Cairo language](#the-cairo-language)
- [Getting Started](#-getting-started)
  - [Dependencies](#dependencies)
- [Usage](#-usage)
  - [Adding cairo-vm as a dependency](#adding-cairo-vm-as-a-dependency)
  - [Running cairo-vm from the CLI](#running-cairo-vm-from-cli)
  - [Using hints](#using-hints)
  - [Running a function in a Cairo program with arguments](#running-a-function-in-a-cairo-program-with-arguments)
  - [WebAssembly Demo](#webassembly-demo)
  - [Testing](#testing)
- [Benchmarks](#-benchmarks)
- [Changelog](#-changelog)
- [Contributing](#-contributing)
- [Related Projects](#-related-projects)
- [Documentation](#-documentation)
  - [Cairo](#cairo)
  - [Original Cairo VM Internals](#original-cairo-vm-internals)
  - [Compilers and Interpreters](#compilers-and-interpreters)
  - [StarkNet](#starknet)
  - [Computational Integrity and Zero-Knowledge Proofs](#computational-integrity-and-zero-knowledge-proofs)
- [License](#%EF%B8%8F-license)

## ⚠️ Disclaimer

🚧 `cairo-vm` is still being built therefore breaking changes might happen often so use it at your own risk. 🚧
Cargo doesn't comply with [semver](https://semver.org/), so we advise to pin the version to 0.1.0. This can be done adding `cairo-vm = "0.1.0"` to your Cargo.toml

## 📖 About

Cairo VM is the virtual machine for the [Cairo language](https://www.cairo-lang.org/).

There's an older version of [Cairo VM](https://github.com/starkware-libs/cairo-lang) written in Python, which is **currently in production**.

This repository contains the newer version, written in Rust. It's faster and has safer and more expressive typing. Once completed, it will replace the older one as the sole Cairo VM.

### The Cairo language

Cairo is the first production-grade platform for generating [STARK](https://vitalik.ca/general/2017/11/09/starks_part_1.html) proofs for general computation.

It's Turing-complete and it was created by [Starkware](https://starkware.co/) as part of the [Starknet](https://starkware.co/starknet/) ecosystem.

## 🌅 Getting Started

### Dependencies

#### Required

These are needed in order to compile and use the project.

- [Rust 1.69.0 or newer](https://www.rust-lang.org/tools/install)
- Cargo

#### Optional

These dependencies are only necessary in order to run the original VM, compile Cairo programs, and run tests.

- make
- PyEnv

#### Installation script

You can install all of the required and optional dependencies by running the script `install.sh` while in the repository root.

## 🚀 Usage

### Adding cairo-vm as a dependency

You can add the following to your rust project's `Cargo.toml`:

```toml
cairo-vm = { version = '0.7.0', features = ["lambdaworks-felt"] }
```

The `features = ["lambdaworks-felt"]` part adds usage of [`lambdaworks-math`](https://github.com/lambdaclass/lambdaworks) as the backend for `Felt252`. This improves performance by more than 20%, and will be the default in the future.

### Running cairo-vm from CLI

To run programs from the command line, first compile the repository from the cairo-vm-cli folder:

```bash
cd cairo-vm-cli; cargo build --release -F lambdaworks-felt; cd ..
```

The `-F lambdaworks-felt` part adds usage of [`lambdaworks-math`](https://github.com/lambdaclass/lambdaworks) as the backend for `Felt252`. This improves performance by more than 20%, and will be the default in the future.

Once the binary is built, it can be found in `target/release/` under the name `cairo-rvm-cli`.

To compile a program, use `cairo-compile [path_to_the_.cairo_file] --output [desired_path_of_the_compiled_.json_file]`. For example:

```bash
cairo-compile cairo_programs/abs_value_array.cairo --output cairo_programs/abs_value_array_compiled.json
```

To run a compiled .json program through the VM, call the executable giving it the path and name of the file to be executed. For example:

```bash
target/release/cairo-vm-cli cairo_programs/abs_value_array_compiled.json --layout all_cairo
```

The flag `--layout` determines which builtins can be used. More info about layouts [here](https://www.cairo-lang.org/docs/how_cairo_works/builtins.html#layouts).

To sum up, the following code will get you from zero to running a Cairo program:

```bash
git clone https://github.com/lambdaclass/cairo-vm.git

cd cairo-vm

cargo build --release

cairo-compile cairo_programs/abs_value_array.cairo --output cairo_programs/abs_value_array_compiled.json

target/release/cairo-vm-cli cairo_programs/abs_value_array_compiled.json --layout all_cairo
```

#### Other CLI arguments

The cairo-vm-cli supports the following optional arguments:

- `--trace_file <TRACE_FILE>`: Receives the name of a file and outputs the relocated trace into it

- `--memory_file <MEMORY_FILE>` : Receives the name of a file and outputs the relocated memory into it

- `--print_output` : Prints the program output

- `--proof_mode`: Runs the program in proof_mode

- `--secure_run`: Runs security checks after execution. Enabled by default when not in proof_mode

- `--air_public_input <AIR_PUBLIC_INPUT>`: Receives the name of a file and outputs the AIR public inputs into it. Can only be used if proof_mode is also enabled.

For example, to obtain the air public inputs from a fibonacci program run, we can run :

```bash
  target/release/cairo-vm-cli cairo_programs/proof_programs/fibonacci.json --layout all_cairo --proof_mode --air_public_input fibonacci_public_input.json
```

### Using hints

Currently, as this VM is under construction, it's missing some of the features of the original VM. Notably, this VM only implements a limited number of Python hints at the moment, while the [Python Cairo VM](https://github.com/starkware-libs/cairo-lang) allows users to run any Python code.

There are two ways to use non-standard hints in this VM:

- Extend the cairo-vm code and build your own binary using the interface [HintProcessor](docs/hint_processor/README.md).
- Use [cairo-vm-py](https://github.com/lambdaclass/cairo-vm-py) which supports running any hint in a Python interpreter.

### Running a function in a Cairo program with arguments

When running a Cairo program directly using the Cairo-vm repository you would first need to prepare a couple of things.

1. Specify the Cairo program you want to run

  ```rust
  let program =
          Program::from_file(Path::new(&file_path), None);
  ```

2. Instantiate the VM, the cairo_runner, the hint processor, and the entrypoint

  ```rust
  let mut vm = VirtualMachine::new(false);

  let mut cairo_runner = CairoRunner::new(&program, "all_cairo", false);

  let mut hint_processor = BuiltinHintProcessor::new_empty();

  let entrypoint = program
          .identifiers
          .get(&format!("__main__.{}", &func_name))?
          .pc;
  ```

3. Lastly, initialize the builtins and segments.

  ```rust
  cairo_runner.initialize_builtins(&mut vm)?;
  cairo_runner.initialize_segments(&mut vm, None);
  ```

When using cairo-vm with the Starknet devnet there are additional parameters that are part of the OS context passed on to the `run_from_entrypoint` method that we do not have here when using it directly. These parameters are, for example, initial stacks of the builtins, which are the base of each of them and are needed as they are the implicit arguments of the function.

```rust
 let _var = cairo_runner.run_from_entrypoint(
            entrypoint,
            vec![
                &MaybeRelocatable::from(2).into(),  //this is the entry point selector
                &MaybeRelocatable::from((2,0)).into() //this would be the output_ptr for example if our cairo function uses it
                ],
            false,
            &mut vm,
            &mut hint_processor,
        );
```

### WebAssembly Demo

A demo on how to use `cairo-vm` with WebAssembly can be found in [`examples/wasm-demo`](examples/wasm-demo/)

### Testing

To run the test suite you'll need `cargo-llvm-cov` dependency so make sure to run this command beforehand:

```bash
make deps
```

Now that you have the dependencies necessary to run the test suite you can run:

```bash
make test
```

## 📊 Benchmarks

Running a [Cairo program](./cairo_programs/benchmarks/big_fibonacci.cairo) that gets the 1.5 millionth Fibonacci number we got the following benchmarks:

- Execution time with [Criterion](./docs/benchmarks/criterion_benchmark.pdf)
- [Flamegraph](./docs/benchmarks/flamegraph.svg)
- Github action [results](https://lambdaclass.github.io/cairo-vm/)

Note before running the benchmark suite: the benchmark named [iai_benchmark](https://github.com/lambdaclass/cairo-vm/blob/8dba86dbec935fa04a255e2edf3d5d184950fa22/Cargo.toml#L59) depends on Valgrind. Please make sure it is installed prior to running the `iai_benchmark` benchmark.

Run the complete benchmark suite with cargo:

```bash
cargo bench
```

Run only the `criterion_benchmark` benchmark suite with cargo:

```bash
cargo bench --bench criterion_benchmark
```

Run only the `iai_benchmark` benchmark suite with cargo:

```bash
cargo bench --bench iai_benchmark
```

## 📜 Changelog

Keeps track of the latest changes [here](CHANGELOG.md).

## 🛠 Contributing

The open-source community is a fantastic place for learning, inspiration, and creation, and this is all thanks to contributions from people like you. Your contributions are **greatly appreciated**.

If you have any suggestions for how to improve the project, please feel free to fork the repo and create a pull request, or [open an issue](https://github.com/lambdaclass/starknet_in_rust/issues/new?labels=enhancement&title=feat%3A+) with the tag 'enhancement'.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feat/AmazingFeature`)
3. Commit your Changes (`git commit -m 'feat: add some AmazingFeature'`)
4. Push to the Branch (`git push origin feat/AmazingFeature`)
5. Open a Pull Request

And don't forget to give the project a star! ⭐ Thank you again for your support.

You can find more detailed instructions in the [CONTRIBUTING.md](CONTRIBUTING.md) document.

## 🌞 Related Projects

- [starknet_in_rust](https://github.com/lambdaclass/starknet_in_rust): implementation of Starknet in Rust, powered by the cairo-vm.
- [cairo-vm-py](https://github.com/lambdaclass/cairo-vm-py): Bindings for using cairo-vm from Python code.

## 📚 Documentation

### Cairo

- From Cairo Documentation: [How Cairo Works](https://www.cairo-lang.org/docs/how_cairo_works/index.html#how-cairo-works)
- [Cairo – a Turing-complete STARK-friendly CPU architecture](https://eprint.iacr.org/2021/1063)
- [A Verified Algebraic Representation of Cairo Program Execution](https://arxiv.org/pdf/2109.14534.pdf)
- [Cairo Verifier](https://github.com/patrickbiel01/Cairo_Verifier) in Rust

### Original Cairo VM Internals

We wrote a document explaining how the Cairo VM works. It can be found [here](./docs/python_vm/README.md).

### Compilers and Interpreters

This is a list of recommended books to learn how to implement a compiler or an interpreter.

- [How I wrote my own "proper" programming language - Mukul Rathi](https://mukulrathi.com/create-your-own-programming-language/intro-to-compiler/)
- [Introduction to Compilers and Language Design - Douglas Thain](http://compilerbook.org)
- [Beautiful Racket - Matthew Flatt](https://beautifulracket.com)
- [Crafting interpreters - Robert Nystrom](https://craftinginterpreters.com)
- [Engineering a Compiler - Keith D. Cooper, Linda Torczon](https://www.goodreads.com/en/book/show/1997607.Engineering_a_Compiler)

### StarkNet

- [StarkNet's Architecture Review](https://david-barreto.com/starknets-architecture-review/)

### Computational Integrity and Zero Knowledge Proofs

#### Basics

- [Intro to zero-knowledge proofs](https://www.youtube.com/watch?v=HUs1bH85X9I)
- [Security and Privacy for Crypto with Zero-Knowledge Proofs](https://www.youtube.com/watch?v=3NL0ThdvWMU)
- [A Hands-On Tutorial for Zero-Knowledge Proofs Series](http://www.shirpeled.com/2018/09/a-hands-on-tutorial-for-zero-knowledge.html)

#### ZK SNARKs

- [What are zk-SNARKs?](https://z.cash/technology/zksnarks/)
- [Vitalik's introduction to how zk-SNARKs are possible](https://vitalik.ca/general/2021/01/26/snarks.html)
- [Vitalik's post on quadratic arithmetic programs](https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649)
- [Why and How zk-SNARK Works - Maksym Petkus](https://arxiv.org/abs/1906.07221)
- [Comparing General Purpose zk-SNARKs](https://medium.com/coinmonks/comparing-general-purpose-zk-snarks-51ce124c60bd)
- [Dark forest's intro + circuits PART 1](https://blog.zkga.me/intro-to-zksnarks)
- [Dark forest's intro + circuits PART 2](https://blog.zkga.me/df-init-circuit)

#### STARKs

Introduction:

- [Cryptography Stack Exchange Answer](https://crypto.stackexchange.com/questions/56327/what-are-zk-starks)
- [Hasu gets STARK-pilled - with Eli Ben-Sasson](https://youtu.be/-6BtBUbiUIU)
- [Cairo for Blockchain Developers](https://www.cairo-lang.org/cairo-for-blockchain-developers/)
- [Why STARKs are the key to unlocking blockchain scalability](https://twitter.com/0xalec/status/1529915544324800512?s=12&t=FX6TgXCZY1iWcWmbc7oqSw)
- STARKs whitepaper: [Scalable, transparent, and post-quantum secure computational integrity](https://eprint.iacr.org/2018/046)
- STARKs vs. SNARKs: [A Cambrian Explosion of Crypto Proofs](https://nakamoto.com/cambrian-explosion-of-crypto-proofs/)

Vitalik Buterin's blog series on zk-STARKs:

- [STARKs, part 1: Proofs with Polynomials](https://vitalik.ca/general/2017/11/09/starks_part_1.html)
- [STARKs, part 2: Thank Goodness it's FRI-day](https://vitalik.ca/general/2017/11/22/starks_part_2.html)
- [STARKs, part 3: Into the Weeds](https://vitalik.ca/general/2018/07/21/starks_part_3.html)

Alan Szepieniec's STARK tutorial:

- [Anatomy of a STARK](https://aszepieniec.github.io/stark-anatomy/)

StarkWare's STARK Math blog series:

- [STARK Math: The Journey Begins](https://medium.com/starkware/stark-math-the-journey-begins-51bd2b063c71)
- [Arithmetization I](https://medium.com/starkware/arithmetization-i-15c046390862)
- [Arithmetization II](https://medium.com/starkware/arithmetization-ii-403c3b3f4355)
- [Low Degree Testing](https://medium.com/starkware/low-degree-testing-f7614f5172db)
- [A Framework for Efficient STARKs](https://medium.com/starkware/a-framework-for-efficient-starks-19608ba06fbe)

## ⚖️ License

This project is licensed under the Apache 2.0 license.

See [LICENSE](/LICENSE) for more information.