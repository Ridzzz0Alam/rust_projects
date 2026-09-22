# rust_csv — notes

Reading CSV files in Rust with the `csv` crate.

## Bugs I hit and how they were fixed

### 1. Relative paths resolve from where you run `cargo run`, not from the project folder

The path was `"./rust_csv/customers.csv"`, but `cargo run` is invoked from inside
`rust_csv/`, so it looked for `rust_csv/rust_csv/customers.csv` and failed with
`No such file or directory (os error 2)`.

```rust
// before
read_from_file("./rust_csv/customers.csv")
// after
read_from_file("customers.csv")
```

The error was printing the whole time via `eprintln!`, but stderr is easy to miss
in the noise of Cargo's build output. When nothing seems to happen, look for a
single quiet line above the prompt.

### 2. `csv::Reader` eats the first row as a header by default

`customers.csv` has no header row — it is data from line 1. The default reader
treated `John,Doe,...` as column names, so that record silently vanished and only
5 of 6 rows printed. Missing data, no error message.

```rust
// before
let mut reader = csv::Reader::from_path(path)?;
// after
let mut reader = csv::ReaderBuilder::new().has_headers(false).from_path(path)?;
```

`ReaderBuilder` is the configurable version of `Reader`. Also has `.delimiter(b';')`,
`.flexible(true)`, and others.

## Optional cleanups, not done

- `use csv;` on line 3 is unnecessary — since edition 2018, crates listed in
  `Cargo.toml` are already in scope.
- `main` returns `()`, so the process exits 0 even when the read fails. Returning
  `Result<(), Box<dyn Error>>` from `main` would make the exit code reflect failure.

## Things worth remembering

- No `target/` appears in this folder. The repo shares one build dir at
  `../.build/` via `../.cargo/config.toml`, so the binary is at
  `../.build/debug/rust_csv`.
- The `csv` crate correctly handles escaped quotes (`"John ""Da Man"""`), commas
  inside quoted fields, and empty fields. That is why you do not `split(',')`.
- Next step: `cargo add serde --features derive`, then `reader.deserialize::<T>()`
  to get real structs and typed fields instead of string indexing.
