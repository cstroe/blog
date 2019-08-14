---
title: "Rust Logging"
date: 2019-08-13T22:53:49-05:00
tags: [ "rust" ]
---

One key component of any robust application is being
able to configure a logging framework and produce logs.

For [Rust][6], we have a lightweight logging facade crate named [log][1].
In order for logging to be produced, an actual logging framework must
be used that is compatible with the `log` crate.

Since I'm coming from the JVM and familiar with great logging frameworks
like [log4j2][3], I was drawn to the [log4rs][2] crate.

## Project Setup

After [creating a new Rust project with cargo][4] via `crate new <your_project_name>`,
we need to [specify our dependencies][5] on the crates by editing `Cargo.toml` and
updating the `dependencies` section:

```toml
[dependencies]
# https://github.com/rust-lang-nursery/log
log = "0.4.8"
# https://github.com/sfackler/log4rs
log4rs = "0.8.3"
```

I like to document links to my libraries as [comments][7] for quick reference.

## Logging Configuration

The first thing we need to do is to configure our logging framework.
There is more than one way to do this and I opted to have an external `log4rs.yaml` file for
ease of use.

The downside is that I have to distribute this file with my binary, but the upside is
that it also allows me to change the configuration while my application is running by simply
editing the `log4rs.yaml`.

For a simple configuration, create the `log4rs.yaml` file at the root of your project:

```yaml
# Scan this file for changes every 30 seconds
refresh_rate: 30 seconds

appenders:
  # An appender named "stdout" that writes to stdout
  stdout:
    kind: console

# Set the default logging level to "warn" and attach the "stdout" appender to the root
root:
  level: info
  appenders:
    - stdout
```

More [examples here][9].


## Logging

By default Cargo creates a `src` directory containing a single `main.rs`.
We need to edit this file and reference both the external crates:

```rust
extern crate log;
extern crate log4rs;
```

Next, to actually log messages, we have to `use` methods from the `log` crate.  For example,
to log at the `info` level (see the [docs][8]):

```rust
use log::info;
```

Next, in the `main` function, we need to initialize our `log4rs` framework and log
a test message:

```rust
fn main() {
    log4rs::init_file("log4rs.yml", Default::default()).unwrap();
    info!("Hello Rust!");
}
```

Putting it all together:

```rust
extern crate log;
extern crate log4rs;

use log::info;

fn main() {
    log4rs::init_file("log4rs.yml", Default::default()).unwrap();
    info!("Hello Rust!");
}
```

Let's test out the logging by running our program with `cargo -q run`:

```
2019-08-13T23:53:39.193548108-05:00 INFO rust_logging - Hello Rust!
```

The logging macros accept format strings too:

```rust
let world = "World";
info!("Hello {}!", world);
```

Which outputs:

```
2019-08-13T23:57:32.405480832-05:00 INFO rust_logging - Hello World!
```


[1]: https://docs.rs/log/0.4.8/log/
[2]: https://docs.rs/log4rs/0.8.3/log4rs/
[3]: https://logging.apache.org/log4j/2.x/
[4]: https://doc.rust-lang.org/cargo/getting-started/first-steps.html#first-steps-with-cargo
[5]: https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies
[6]: https://www.rust-lang.org/
[7]: https://github.com/toml-lang/toml#comment
[8]: https://docs.rs/log/0.4.8/log/#use
[9]: https://docs.rs/log4rs/0.8.3/log4rs/#examples
