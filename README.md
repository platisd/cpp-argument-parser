# Command Parser

Command Parser is a C++17 header-only utility library for parsing command line "commands".

We define a _command_ as a string passed to a binary as a command line argument in the form of:

```bash
./your_binary command_name <arg1> <arg2> [arg3]
```

A command may optionally take some (boolean) options

```bash
./your_binary command_name --option1 --option2 <arg1> <arg2> [arg3]
```

## Usage

See [example_main.cpp](example_main.cpp) for a simple example of how to use the library. In a nutshell:

```cpp
const auto all = UnparsedCommand::create("all", "Print current configuration");
const auto get = UnparsedCommand::create("get", "Get configuration key", "[-xyz] <key> [default]")
                         .withArgs<std::string, std::optional<std::string>>();
const auto encrypt = UnparsedCommand::create(
                         "encrypt"
                         "Encrypt the given files with the specified policy",
                         "<policy> [file...]")
                         .withArgs<std::string, std::vector<std::string>>();
const std::tuple commands { all, get, encrypt };
const auto parsedCommand = UnparsedCommand::parse(argc, argv, commands);

if (parsedCommand.is(all)) {
    std::cout << "all" << std::endl;
    // auto config = parsedCommand.getArgs(all); // Does not compile because all has no args
} else if (parsedCommand.is(get)) {
    const auto [key, defaultValue] = parsedCommand.getArgs(get);
    const auto x = parsedCommand.hasOption("x");
    const auto y = parsedCommand.hasOption("y");
    const auto z = parsedCommand.hasOption("z");
    std::cout << "get " << key << " " << x << " " << y << " " << z << std::endl;
    if (defaultValue) {
        std::cout << "default " << defaultValue.value() << std::endl;
    }
} else if (parsedCommand.is(encrypt)) {
    const auto [policy, files] = parsedCommand.getArgs(encrypt);
    std::cout << "encrypt " << policy << std::endl;
    for (const auto& file : files) {
        std::cout << "file " << file << std::endl;
    }
} else {
        parsedCommand.help();
}
```

### Allowed types

The following types are permitted as arguments. They are mandatory unless otherwise specified and their usage rules are
enforced during compilation:

* `std::string`
* `bool`
* `int`
* `long`
* `long long`
* `unsigned long`
* `unsigned long long`
* `float`
* `double`
* `long double`
* `std::optional<T>` where `T` is any of the above types
    * Optional argument: May not be provided but must be at the end of the argument list
* `std::vector<T>` where `T` is any of the above types
    * Zero or more optional arguments: May be of any number or not provided, but cannot precede a mandatory argument and
      cannot be combined with a `std::optional` argument

## Why not `insert your favorite CLI parsing library here`?

In all fairness, this library was created under the misconception that [cxxopts](https://github.com/jarro2783/cxxopts)
was not able to satisfy our use-case out of the box, which is **not** true. Some of its advantages are:

* Compile-time check of argument count and types (i.e. you cannot "forget" to parse an argument or parse one that was
  not expected)
* Header-only
* No dependencies
* No macros
* No exceptions

## Acknowledgements

This library was developed internally at [neat.no](https://neat.no) and is now open-sourced.