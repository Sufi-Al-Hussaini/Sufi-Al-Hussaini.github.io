---
layout: post
title: The Monitor pattern in modern C++
description: Implemented as a wrapper class
date: '2017-10-14 19:56:49'
tags: c++ design-patterns
---


## The Monitor design pattern is a synchronization construct used in concurrent programming to get mutual exclusion when accessing a shared resource. This post discusses its evolution, use cases and a (slightly modified) C++11 version of its implementation provided by Herb Sutter.

***

## The Evolution

The essence of the monitor pattern is that it uses some sort of a synchronization primitive (usually mutex) to protect access to a shared resource. The mutex is locked before accessing the resource and released thereafter, thereby serializing access to it and providing mutual exclusion.

The traditional way to use a mutex to serialize access to a shared resources (using POSIX threads' C API) would be as below:

```c
#include <pthread.h>

struct shared_resource s;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

static void *pthread_start_routine(void *arg) {
	/* ... */
	pthread_mutex_lock(&m);
	do_something_with(&s);
	pthread_mutex_unlock(&m);
	/* ... */
}
```


Now, because the mutex is only ever used to protect access to this one shared resource, it would be better to wrap the mutex and the shared resource together in a `synchronized` object. It would also be nice if this object provided functions to lock/unlock the mutex as that would give us the benefits of object oriented programming concepts (encapsulation and abstraction).

```c++
class synchronized_string {
	std::string s;
	std::mutex m;
public:
	// ...
	void lock() { m.lock(); }
	void unlock() { m.unlock(); }
};
``` 

We can now use this as follows:

```c++
synchronized_string s("Hello");
try {
	s.lock();
	s.do_something();
	s.unlock();
} catch (/* ... */) {
	// ...
}
```

The same solution can be realized in C using a structure with function pointers and hiding the mutex using a `void*`, although it usually isn't a good idea to do OOP is C.

But there's still a problem here, the onus of locking and unlocking the mutex is still on the caller. And (*most*) callers are stupid. 
The solution to this problem is to have the class take over the responsibility of locking/unlocking the mutex. 
To do this, we wrap all of functions in the public API with lock/unlock calls. The right way to do this ofcourse would be to use a RAII scoped lock like `std::lock_guard`.

```c++
class synchronized_string {
	std::string s;
	std::mutex m;
public:
	// ...
	void do_something() {
		m.lock();
		// Insert original `do_something()` implementation here
		m.unlock();
	}
}
};
```

This is the monitor pattern in its simplest form. 

Another way to implement this is to use callbacks, where the caller can set a locking and unlocking function. See [libcurl's opensslthreadlock example](https://curl.haxx.se/libcurl/c/opensslthreadlock.html) to see how this is done in C.

***

## The Problem

There's a fundamental problem with the monitor pattern (or the way it has been implemented above). Usually, providing mutual exclusion for every function call is not the best idea. Take for example the fund transfer problem in database applications - Bob wants to transfer a certain amount to Mary. Borrowing an example from Herb's presentation:

```c++
int amount = 300;
accounts.debit(bob, amount);
accounts.credit(mary, amount);
```

We want the above piece of code to run ***atomically***. But even if the `accounts` object follows the monitor pattern above, all our current implementation guarantees us is that the calls to `debit` and `credit` are thread-safe.
So, per-member-function locking doesn't seem to be the right granularity after all. 

***

## The Solution

One way to solve this problem would be to extend the `accounts` class to add a `transfer` function:

```c++
void transfer(user& sndr, user& rcvr, int amount) {
	m.lock();
	// Do everything that `debit()` & `credit()` did internally 
	// without the locking
	m.unlock();
}
```

Clearly this approach doesn't scale and becomes a pain to maintain.

A better solution to this problem would be to allow the caller to decide the granularity of the locking.
Using reflection or Aspect oriented programming (and the interceptor pattern) could allow us to solve this problem differently, but the solution Herb proposed is apt for C++.

```c++
template<class T>
class monitor {
    T t;
    mutable std::mutex m;

public:
    monitor(T t_ = T{}) : t(t_) {}

    template<typename F>
    auto operator()(F f) -> decltype(f(t)) {
        std::lock_guard<std::mutex> _{m};
        return f(t);
    }
};
```

This is a slightly modified version of the solution that Herb proposed as C++11 forbids apllying mutable to references.

>[7.1.1 Para 9]:
>
>"The mutable specifier can be applied only to names of class data members (9.2) and cannot be applied to names declared const or static, and cannot be applied to reference members."

Here, we override the function call operator, to which the caller passes an invokable (functor, lambda, etc.). And this invokable is given mutually exclusive access to the shared resource using the mutex. So now, the caller can decide the granularity of the locking.
A more complete implementation `monitor` for C++14 can be found [here](https://gist.github.com/etam/9019865).

Here's how it can be used:

```c++
#include <iostream>
#include <future>
#include <vector>

int main() {
    monitor<std::ostream &> sync_cout(std::cout);
    monitor<std::string> s("Start\n");
    std::vector<std::future<void>> v;

    for (int i = 0; i < 5; ++i) {
        v.push_back(std::async([&, i] {
            s([=](std::string &s) {
                s += std::to_string(i);
                s += "\n";
            });

            s([&](std::string &s) {
                sync_cout([=](std::ostream &c) { c << s; });
            });
        }));
    }

    for (auto &f: v) f.wait();
    sync_cout([=](std::ostream &c) { c << "Done" << std::endl; });
    return 0;
}
```


Most of this post is based on Herb Sutter's brilliant presentation C++ and Beyond 2012: Herb Sutter - C++ Concurrency.
<iframe src="https://channel9.msdn.com/Shows/Going+Deep/C-and-Beyond-2012-Herb-Sutter-Concurrency-and-Parallelism/player" width="720" height="405" allowFullScreen frameBorder="0"></iframe>

