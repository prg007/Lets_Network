# CS 389 HW_04 - Let's Network...!
### Names : Prasun and Hrishee

## File Overview
`hw_4_network.zip` : zip file submission of our repository. See below as to why we needed to use a zip file. Inside the `zip` file we have:

We'll go over the new files (we elide descriptions of anything from HW3 and HW2, since those files are pretty much the same. Except `cache.hh` which now has an additional constructor):

`cache_server.cc` : Implementation of an asynchronous HTTP cache server using the [Crow](https://github.com/ipkn/crow) framework.

`cache_client.cpp` : Implementation of a synchronous cache client that makes API requests to a cache server. It uses the [cpr](https://github.com/whoshuu/cpr) framework.

`testy_client.cc` : Our testing file that has unit tests for the cache client functionality. It also uses the [Catch2](https://github.com/catchorg/Catch2) framework.

`CMakeLists.txt` : For building. It should build all three executables `testy_cache`, `server`, `testy_client`. Since we also did our evictor tests inside the `testy_cache.cc` file for HW3, we don't have a `test_evictors.cc` file.

## Getting Started
You can compile and run this on a Mac or in the Linux VM, although the steps are a bit different in each case. See the instructions for details.
### What you need
Catch2, Crow, cpr, cmake, make, Boost (must have boost version 1.69 specifically. Doesn't work with boost versions greater than 1.69 due to [this issue](https://github.com/ipkn/crow/issues/340)).


We'll go over how to manage these dependencies and compile in the next section. We've provided a `zip` file containing all our
 homework implementation as well as the necessary libraries (except boost 1.69.0, which you have to download and configure yourself - we help you below). Managing these dependencies was a nightmare for us. We hope that our zip file will make your life easier instead of having you manually download and configure the crow and `cpr` libraries which caused
 us a massive headache. (We had to zip the repository because the `cpr` library was too large to upload otherwise).

### How to compile
We have very specific compilation instructions. Please read carefully (sorry about this). 

- If you currently have the correct version of Boost move onto the next step. Otherwise download boost 1.69 from this link: [boost_1_69_0](https://www.boost.org/users/history/). Extract it anywhere you want on your machine by either by double clicking on the file or using the following Terminal Command `tar xvzf boost_1_69_0.tar.bz2`.

- Unzip our zip file. Wherever you unzipped it, `cd` into it in the command line.

- In the `CMakeLists.txt` file, manually change all paths to the `boost_1_69_0` folder to match the path on your machine/VM. Specifically, you should change it 4 times in lines 16,18, 23 and 25 in the `set CMAKE_CXX_FLAGS` part. **Important:** if you're on the Linux VM, you must add the flag `-pthread` to the same lines. 

- Enter the build folder, `cd build`. If the build folder is not empty, empty it out (i.e. do a clean all in the build directory). You don't have to empty it out if you have already compiled it once before. But for your first time compiling, **empty it out**.   

- From the build folder, do `cmake ..` then `make`. This should produce the three executables `testy_cache`, `server`, and `testy_client` which will also be in the build folder. Since this Makefile produced by the CMakefile has a lot of bloat (it was provided by the cpr library, which we modified for our purposes), once it says that it has built all of the three executables above you can do a keyboard error to stop it from building the other junk. 

### How to run
First off, make sure the server is **running** on the same IP and port as the client (in `testy_client.cc`) before you test the client. Otherwise, the client will (understandably) fail.
To run the server, do `./server` if you just want to use the default parameters (explained below), or do `./server -m X -s Y -p Z -t W` where X is maxmem, Y is the host IP address, Z is the port number, and W is the number of threads. (Currently, the server is single threaded even if you give  W > 1).

Then, to test the client, run `./testy_client`. All the test cases should pass (if you use the default server configuration). You can also make manual HTTP requests on a separate command line window using something like `curl`.
Sample `curl` command line usage is :

`curl -X HEAD localhost:2001/`

`curl -X PUT localhost:2001/key/value`

`curl -X POST localhost:2001/reset`

`curl -X DELETE localhost:2001/key`

`curl -X GET localhost:2001/key`

## TCP Server
Our asynchronous TCP server uses Crow, as mentioned above. Our default settings for the `SERVER_CACHE` are `maxmem = 30`, `server = "127.0.0.1"`, `port = 2001`, `threads = 1`. These can be overwritten via the command line 

(e.g. `server -m 1024 -s 123.4.5.6 -p 2020 -t 10`). 

Our server cache uses a default load factor of 0.75, as well as the default hash function (`std::hash`) and default eviction policy (refuse exceeding cache puts). Refer to `cache_server.cc` for further implementation details.

## TCP Client
Refer to `cache_client.cpp` for implementation details. We used a library called C++ Requests, or cpr, for short (linked above). If the TCP client detects that its host server is not running, it gracefully exits. As for the issue of memory ownership of the pointer from `Cache::get()`, we maintained a vector of returned char pointers in the implementation and deleted each one in the destructor of the actual `Cache` object. This means that the returned pointers will be tied to the life and death of the Cache client object. 

## Testing and Valgrind
When testing (i.e. running `testy_client`) make sure that the server is configured in its default state, as that's what the test cases are designed for. Running **valgrind** on `./testy_client` while the `./server` is up assures us that there are no memory leaks.  Valgrind on `testy_cache` also gives us no memory leaks, although that doesn't depend on whether the server is up or not. 
