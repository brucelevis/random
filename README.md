# Random for modern C++ with convenient API
[![Build Status](https://travis-ci.org/effolkronium/random.svg?branch=develop)](https://travis-ci.org/effolkronium/random)
[![Build status](https://ci.appveyor.com/api/projects/status/vq1kodqqxwx16rfv/branch/develop?svg=true)](https://ci.appveyor.com/project/effolkronium/random/branch/develop)
[![Coverage Status](https://coveralls.io/repos/github/effolkronium/random/badge.svg?branch=develop)](https://coveralls.io/github/effolkronium/random?branch=develop)
<a href="https://scan.coverity.com/projects/effolkronium-random">
  <img alt="Coverity Scan Build Status"
       src="https://scan.coverity.com/projects/12707/badge.svg"/>
</a>
- [Design goals](#design-goals)
- [Integration](#integration)
- [Five-minute tutorial](#five-minute-tutorial)
  - [Number range](#number-range)
  - [Common type number range](#common-type-number-range)
  - [Bool](#bool)
  - [Random value from std::initilizer_list](#random-value-from-stdinitilizer_list)
  - [Shuffle](#shuffle)
  - [Seeding](#seeding)
  - [min-value](#min-value)
  - [max-value](#max-value)
  - ['get' without arguments](#get-without-arguments)
  - [Discard](#discard)
  - [isEqual](#isequal)
  - [Serialize](#serialize)
  - [Deserialize](#deserialize)
  - [Thread local random](#thread-local-random)
  - [Local random](#local-random)
## Design goals
There are few ways to get working with random in C++:
- **C style**
```cpp
  srand( time(NULL) ); // seed with time since epoch
  auto random_number = rand() % (9 - 1)) + 1; // get a pseudo-random integer between 1 and 9
```
* Problems
  * should specify seed before using rand() function
  * should write your own distribution algorihtm
  * [There are no guarantees as to the quality of the random sequence produced.](http://en.cppreference.com/w/cpp/numeric/random/rand#Notes)
- **C++11 style**
```cpp
  std::random_device random_device; // create object for seeding
  std::mt19937 engine{random_device()}; // create engine and seed it
  std::uniform_int_distribution<> dist(1,9); // create distribution for integers with [1, 9] range
  auto random_number = dist(engine); // finally get a random number
```
* Problems
  * should specify seed
  * should choose, create and use a chain of various objects like engines and distributions
  * mt19937 use 5000 bytes of memory for each creation
  * Uncomfortable and not intuitively clear
- **effolkronium random style**
```cpp
  // auto seeded
  auto random_number = Random::get(1, 9); // invoke 'get' method to generate  a pseudo-random integer between 1 and 9
  // yep, that's all.
```
* Advantages
  * **Intuitive syntax**. You can do almost everything with random by simple 'get' method, like getting simple numbers, bools, random object from given set or using custom distribution.
  * **Trivial integration**. All code consists of a single header file [`random.hpp`](https://github.com/effolkronium/random/blob/develop/source/random.hpp). Tahat's it. No library, no subproject, no dependencies, no complex build system. The class is written in vanilla C++11. All in all, everything should require no adjustment of your compiler flags or project settings.
  * **Usability**. There are 3 versions of random: 
    * *random_static* which has static methods and static internal state. It's not thread safe but more efficient
    * *random_thread_local* which has static methods and [thread_local](http://en.cppreference.com/w/cpp/keyword/thread_local) internal state. It's thread safe but less efficient
    * *random_local* which has non static methods and local internal state. It can be created at local scope
## Integration
The single required source, file `random.hpp` is in the `source` directory.
All you need to do is add
```cpp
#include "effolkronium/random.hpp"

// get base random alias which is auto seeded and has static API and internal state
using Random = effolkronium::random_static;
```
to the files you want to use effolkronium random class. That's it. Do not forget to set the necessary switches to enable C++11 (e.g., `-std=c++11` for GCC and Clang).
## Five-minute tutorial
### Number range
Returns a random number between first and second argument.
```cpp
auto val = Random::get(-1, 1) // decltype(val) is int
```
```cpp
// specify explicit type
auto val = Random::get<uint8_t>(-1, 1) // decltype(val) is uint8_t
```
```cpp
// you able to use range from greater to lower
auto val = Random::get(1.l, -1.l) // decltype(val) is long double
```
```cpp
auto val = Random::get(1.f, -1) // Error: implicit conversions are not allowed here.
```
### Common type number range
Choose common type of two range arguments by std::common_type.
```cpp
auto val = Random::get<Random::common>(1, 0.f) // decltype(val) is float
```
```cpp
auto val = Random::get<Random::common>(0ul, 1ull) // decltype(val) is unsigned long long
```
```cpp
auto val = Random::get<Random::common>(1.2l, 1.5f) // decltype(val) is long double
```
```cpp
auto val = Random::get<Random::common>(1u, -1) // Error: prevent conversion from signed to unsigned
```
### Bool
Generate bool with [0; 1] probability
```cpp
auto val = Random::get<bool>(0.7) // true with 70% probability
```
```cpp
auto val = Random::get<bool>() // true with 50% probability by default
```
```cpp
auto val = Random::get<bool>(-1) // Error: assert occurred! Out of [0; 1] range
```
### Random value from std::initilizer_list
Return random value from values in std::initilizer_list
```cpp
auto val = Random::get({1, 2, 3}) // val = 1 or 2 or 3
```
### Shuffle
[ref](http://en.cppreference.com/w/cpp/algorithm/random_shuffle)

Reorders the elements in a given range or in all container
```cpp
std::array<int, 3> array{ {1, 2, 3} };
```
```cpp
Random::shuffle( array.begin( ), array.end( ) )

// or just
Random::shuffle( array )
```
### Seeding
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/seed)

You able to reseed internal random engine.
```cpp
Random::seed( 10 ); // 10 is new seed number

std::seed_seq sseq{ 1, 2, 3 };
Random::seed( sseq ); // use seed sequence here
```
### Min value
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/min)

Returns the minimum value potentially generated by the internal random-number engine
```cpp
auto minVal = Random::min( );
```
### Max value
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/max)

Returns the maximum value potentially generated by the internal random-number engine
```cpp
auto maxVal = Random::max( );
```
### get without arguments
Returns the rundom number in [ Random::min( ), Random::max ] range
```cpp
auto val = Random::get( );
// val is rundom number in [ Random::min( ), Random::max ] range
```
### Discard
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/discard)

Advances the internal engine's state by a specified amount.
Equivalent to calling Random::get() N times and discarding the result.
```cpp
Random::discard( 500 );
```
### IsEqual
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/operator_cmp)

Compares internal pseudo-random number engine with other pseudo-random number engine.
```cpp
Random::Engine otherEngine;
bool isSame = Random::isEqual( otherEngine );
```
### Serialize
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/operator_ltltgtgt)

Serializes the internal state of the internal pseudo-random number engine as a sequence of decimal numbers separated by one or more spaces, and inserts it to the output stream. The fill character and the formatting flags of the stream are ignored and unaffected.
```cpp
std::stringstream strStream;
Random::serialize( strStream ); // the strStream now contain internal state of the Random internal engine
```
### Deserialize
[ref](http://en.cppreference.com/w/cpp/numeric/random/mersenne_twister_engine/operator_ltltgtgt)

Restores the internal state of the internal pseudo-random number engine from the serialized representation, which was created by an earlier call to 'serialize' using a stream with the same imbued locale and the same CharT and Traits. If the input cannot be deserialized, internal engine is left unchanged and failbit is raised on input stream.
```cpp
std::stringstream strStream;
Random::serialize( strStream );

// ...

Random::deserialize( strStream ); // Restore internal state of internal Random engine
```
### Thread local random
### Local random
