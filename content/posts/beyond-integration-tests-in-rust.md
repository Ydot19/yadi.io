+++
date = '2025-01-09'
draft = false
title = 'Beyond Integration Tests in Rust'
Summary = "Exploring how to group and execute a collection of related tests in Rust"
categories = ["testing"]
tags = ["rust", "testing", "tutorial"]
ShowBreadCrumbs = true
ShowReadingTime = true
searchHidden = false
ShowToc = true
+++


## Overview

Exploring how to group and execute different categories of tests using rust and cargo.
<!-- Details in the post-->

## Background

In the second half of 2024, I spent sometime going over the [`Zero To Production In Rust` Book](https://www.zero2prod.com/index.html?country_code=US) while learning how to use rust to build services. Naturally, I took some of the ideas in the book and gave it my own slight spin based on my past experience. One of these areas was in testing and namely how to test the services at different levels.

Before we go any further, lets talk about the nomenclature in this blog. This nomenclature is not meant to be 100% industry standard but rather a way to have common language around what is that I am describing.

### Nomenclature

- `integration tests`

A test that evaluates more than one code unit / module in a rust project. In rust, these tests typically reside in the `tests/` directory. The parent directory is the one that contains the `cargo.toml` file

Example:

```text
service/
    lib/ <-- the application logic as a "library"
    bin/ <-- the executable binary of the service. Instantiates the lib (more on this later)
    tests/ <-- integration tests live here
    Cargo.toml
```

Theses tests call some entry point in the lib/ directory and all the subsequent code paths from that entry-point. 


- `end-to-end tests` 

This form of testing does not rely on invoking the rust entry-point directly (like with integration tests). Rather, it relies on calling the running service[s]. This form of testing is helpful to make sure that the pipes across the different components of your services work as expected when the service is running.

If we take the example of http-based service and say it had the following attributes

```text
service
    http://localhost:3030/GetUser
        - Under the hood, this endpoint calls 
        the entrypoint method `pub get_user` method in the `lib/` directory
```

Integration tests would call that entry-point method (because it is public). End-to-end tests would call the `localhost:3030/GetUser` of the running service

### What is the issue?

The cargo tool-chain comes with good, sane defaults. Writing lib integration tests is as easy as running

```zsh
$ cargo test --test '*'
```
<small>Note: The above command runs all integration tests in the `tests/` sub-directory.</small>

This works for almost all use cases, but for the case we described above, its not as intuitive how to denote some tests as `integration tests` separate from `end-to-end tests`

## How can we get around this?

We can define a test harness in the `Cargo.toml` file for each type of test we care about. As an example say we had the following folder structure

```text
service/
    ...
    tests/
        integration/
            mod.rs
            test_integration_a.rs
            test_integration_b.rs
        endtoend/
            mod.rs
            test_e2e_a.rs
            test_e2e_b.rs
    Cargo.toml
```
<small>Note: test file names are only for example purposes</small>


And each of the test files has a function marked as

```rust
#[test]
fn testSomething() {
    // ...
}
```

and/or

```rust
#[cfg(test)]
mod group_of_tests {
    //..
}
```

These tests can be part of mod of there respective directory. In the case of `endtoend/` in our example, the mod.rs file would look like so

```rust
mod test_e2e_a;
mod test_e2e_b;
```

And the test harness in `Cargo.toml` would be defined as

```toml
[[test]]
name = "integration"
path = "tests/integration/mod.rs"
test = true

[[test]]
name = "endtoend"
path = "tests/endtoend/mod.rs"
test = true
```

### Executing each kind of test

With plain cargo

```bash
$ cargo test --test integration
```

With nextest

```bash
$ cargo nextest run --test integration
```

## Final thoughts

Some of this may sound overkill. For most projects this is likely the case but it is a good strategy to keep in your back pocket if you want to logically group and execute tests.

