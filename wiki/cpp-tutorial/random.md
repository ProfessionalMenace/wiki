---
bot_article: |
  # Generating Random Numbers in C++
  ## Example: Printing Ten Random Dice Rolls

  ```cpp
  #include <random>
  #include <iostream>
  int main() {
    std::random_device dev; // for seeding
    std::default_random_engine gen{dev()};
    std::uniform_int_distribution<int> dis{1, 6};
    for (int i = 0; i < 10; ++i) {
      std::cout << dis(gen) << ' ';
    }
  }
  ```

  ## Possible Output (will be different each time)
  ```cpp
  1 1 6 5 2 2 5 5 6 2
  ```
---

# Generating Random Numbers

A [Pseudorandom Number Generator (PRNG)](https://en.wikipedia.org/wiki/Pseudorandom_number_generator) is an algorithm
for generating a deterministic sequence of numbers that appear random. PRNGs maintain an internal state that is updated
each time a new number is generated. The initial state of the PRNG is called the seed and the process of setting the
initial state is called seeding. Two instances of the same PRNG initialized with the same seed will produce identical
sequences of numbers, which is important for reproducibility of statistical simulations.

In the [C++ standard library](https://en.cppreference.com/w/cpp/header/random), random engines are callable objects that
implement PRNG algorithms behind a
[shared interface](https://en.cppreference.com/w/cpp/named_req/RandomNumberEngine.html). A common example is
`std::default_random_engine`, which serves as a standard, general-purpose generator.

It should be noted that none of these PRNGs are cryptographically secure (should not be used for security-sensitive
applications). This means that the state of the random engine can be figured out and predicted given enough values.

### Example: Printing Ten Random Dice Rolls

```cpp
#include <random>
#include <iostream>

int main() {
  // initialize a random device
  std::random_device dev;

  // seed default_random_engine
  std::default_random_engine gen{dev()};

  // initialize a uniform integer distribution
  std::uniform_int_distribution<int> dis{1, 6};

  // roll the dice
  for (int i = 0; i < 10; ++i) {
    std::cout << dis(gen) << ' ';
  }
}
```

## Seeding PRNGs

### Random Device

A random device ([std::random_device](https://en.cppreference.com/w/cpp/numeric/random/random_device)) is random number
generator that attempts to utilize randomness from a non-deterministic source, typically provided by the operating
system (e.g. reading from `/dev/random` or `/dev/urandom` on UNIX-like systems). If no source of randomness is available
then it might fall back to using a deterministic [random engine](https://eel.is/c++draft/rand.device#2). This means that
two instances of `std::random_device` can produce the same sequence of numbers. However it is rarely the case on the
three major platforms (Windows, Linux, and macOS) in modern environments.

The random device should primarily be used for seeding random engines as it is slow and usually requires system calls.
On some UNIX-like systems, if `/dev/random` is used as the source of random numbers, it may block when the entropy pool
is exhausted (though this is generally no longer the case on modern systems). Additionally, the behavior of
`std::random_device` is implementation-defined.

### Example: Creating a random device out of `/dev/urandom` on UNIX-like systems

Using `/dev/random` might be preferred in the
[majority of cases](https://unix.stackexchange.com/questions/324209/when-to-use-dev-random-vs-dev-urandom). It is
however unclear if there is any actual difference between `/dev/random` and `/dev/urandom` as is implementation defined
and can vary system to system.

```cpp
#include <random>
#include <iostream>

int main() {
  // initialize a random device
  std::random_device dev{"/dev/urandom"};

  // generate seeds directly
  for (int i = 0; i < 10; ++i) {
    auto seed = dev();
    std::cout << seed << '\n';
  }
}
```

### Seed Sequence

Seed sequence (`std::seed_seq`) is a utility in the standard library for converting a small number of inputs into a
higher quality seed (does not contain large areas of zeros/ones) suitable for seeding PRNGs with large internal state
(e.g. `std::mt19937`).

## Predefined Generators

| Name                  | Generator                 |
| --------------------- | ------------------------- |
| default_random_engine | implementation defined    |
| minstd_rand0          | linear congruential       |
| minstd_rand           | linear congruential       |
| mt19937               | mersenne twister          |
| mt19937_64            | mersenne twister          |
| ranlux24              | subtract with carry       |
| ranlux48              | subtract with carry       |
| knuth_b               | minstd_rand0 with shuffle |
| philox4x32 (C++26)    | counter-based philox      |
| philox4x64 (C++26)    | counter-based philox      |

[source](https://timsong-cpp.github.io/cppwp/n4868/rand.predef)

### Linear Congruential Generator

[Linear congruential generator (LCG)](https://en.wikipedia.org/wiki/Linear_congruential_generator) is a very simple
pseudo PRNG with a small internal state (`sizeof(std::int_fast32_t)` bytes). The C++ standard library provides three
predefined LCG-based engines [`minstd_rand0`](https://timsong-cpp.github.io/cppwp/n4868/rand.predef#lib:minstd_rand0),
[`minstd_rand`](https://timsong-cpp.github.io/cppwp/n4868/rand.predef#lib:minstd_rand) and
[`knuth_b`](https://timsong-cpp.github.io/cppwp/n4868/rand.predef#lib:knuth_b). The statistical quality of these
generators is not considered good by modern standards.

```cpp
#include <iostream>
#include <random>
#include <cstdint>

int main() {
    // Musl rand() reconstruction
    using musl_rand = std::linear_congruential_engine<
        std::uint64_t,
        6364136223846793005,
        1,
        0 // 2^64
    >;

    // initialize generator with seed
    musl_rand gen(123456789);

    // generate random numbers
    for (int i = 0; i < 10; ++i) {
        auto random_number = gen();
        std::cout << random_number << std::endl;
    }
}
```

### Mersenne Twister

[Mersenne Twister (MT)](https://en.wikipedia.org/wiki/Mersenne_Twister) is a general-purpose PRNG with decent
statistical properties. The C++ standard library provides two predefined MT-based engines
[`mt19937`](https://timsong-cpp.github.io/cppwp/n4868/rand.predef#lib:mt19937) and
[`mt19937_64`](https://timsong-cpp.github.io/cppwp/n4868/rand.predef#lib:mt19937_64). The main limitations of the
Mersenne Twister engine are large internal state (624 \* `sizeof(std::uint_fast32_t)` bytes) and difficulty of its
seeding.

```cpp
#include <iostream>
#include <random>

int main() {
    // initialize seed sequence
    std::seed_seq seq{1, 2, 3, 4};

    // initialize Mersenne Twister engine with seed sequence
    std::mt19937 gen(seq);

    // initialize a uniform real distribution
    std::uniform_real_distribution<double> dis{0.0, 1.0};

    // generate random numbers in the interval [0, 1)
    for (int i = 0; i < 100; ++i) {
        double random_number = dis(gen);
        std::cout <<  random_number << std::endl;
    }
}
```

### Ranlux (Subtract With Carry)

[Ranlux](https://luscher.web.cern.ch/luscher/ranlux/) is a PRNG with a high statistical quality. The C++ standard
library provides two predefined ranlux-based engines `ranlux24` and `ranlux48`. The ranlux PRNGs are widely used in
[Monte Carlo simulation](https://en.wikipedia.org/wiki/Monte_Carlo_method) programs.

### Philox

Two philox random engines (philox4x32 and philox4x64) just got added into the
[C++26 standard](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2075r1.pdf). As of the time of writing this
article neither of them are implemented in both libstdc++ and libc++.

### Adapters

The standard library also provides engine adapters, which improve the statistical qualities of random engines.

- **[std::discard_block_engine](https://en.cppreference.com/w/cpp/numeric/random/discard_block_engine.html)**
- **[std::independent_bits_engine](https://en.cppreference.com/w/cpp/numeric/random/independent_bits_engine.html)**
- **[std::shuffle_order_engine](https://en.cppreference.com/w/cpp/numeric/random/shuffle_order_engine.html)**

### Distributions

- **[std::uniform_int_distribution](https://en.cppreference.com/w/cpp/numeric/random/uniform_int_distribution)**
- **[std::uniform_real_distribution](https://en.cppreference.com/w/cpp/numeric/random/uniform_real_distribution)**
- **[std::normal_distribution](https://en.cppreference.com/w/cpp/numeric/random/normal_distribution)**
- **[all distributions](https://en.cppreference.com/w/cpp/named_req/RandomNumberDistribution)**

### See Also

- **[UniformRandomBitGenerators](https://en.cppreference.com/w/cpp/named_req/UniformRandomBitGenerator)**
- **[Pseudo-random number generation](https://en.cppreference.com/w/cpp/numeric/random)**
- **[Generate random numbers using C++11 random library](https://stackoverflow.com/q/19665818/5740428)**
- **[Why is the use of rand() considered bad?](https://stackoverflow.com/q/52869166/5740428)**
- **[ChaCha20](https://cr.yp.to/chacha.html)**
- **[A PRNG Shootout](https://prng.di.unimi.it/)**
- **[PCG Random](https://pcg-random.org/)**
- **[myths about urandom](https://www.2uo.de/myths-about-urandom/)**
- **[boost::random](https://www.boost.org/library/latest/random/)**
