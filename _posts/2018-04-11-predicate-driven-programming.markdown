---
layout: post
title: Predicate-driven programming in C and C++
description: Using the libdo library
date: '2018-04-11 15:33:23'
tags: c c++ design-patterns predicate-driven-programming Arduino
---

## Predicate-driven programming in C and C++ using the libdo library.

***

### Event-driven programming

In an event-driven application, there is generally a main loop that listens for events, and then triggers a callback function when one of those events is detected.<sup>[1](#event-driven-programming-wikipedia)</sup> <br/>
Normally, when using an event-driven library or framework, you'd create an event handler and bind it to an event. This is done by registering the event handler with an event manager. The event manager then dispactches the events to the event handlers as they are received.

---

### Predicate-driven programming

In predicate-driven programming, predicates replace events. It is meant to complement event-driven programming and not compete with it.

> **Predicate**<br/>
> Something which is affirmed or denied concerning an argument of a proposition. 

In computing, a predicate can be thought of as an expression which evaluates to a boolean value (i.e. `true` or `false`). <br/>
Here, you bind callbacks to predicates, and these callbacks are triggered whenever the predicate evaluates to `true`. So, this is essentially a `if-this-then-that` paradigm. <br/>
Typically, all this happens when the library `loop()` method is called. This makes predicate-driven programming especially well-suited for deferred processing. 


Basically, this means a loop like this: 
```c
int main() {
	while (1) {
		if (disconnected()) {
			reconnect();
		}
		if (connected()) {
			start();
		}
		/* ... */
	}
	return 0;
}
```

Becomes something like this, where predicate functions (`disconnected()` and `connected()`) are paired with their respective handlers (`reconnect()` and `start()`).
```c
int main() {
	call_when(reconnect, disconnected);
	call_when(start, connected);
	/* ... */
	while (1) {
		loop();	
	}
	return 0;
}
```

Admittedly, this doesn't look very useful, but if you follow some best practices & patterns, and add a few goodies like priority based dispatch, expirable handlers, etc., then things start to get interesting. 

---

### libdo

[libdo](https://github.com/Sufi-Al-Hussaini/libdo) is an embeddable library for predicate-driven programming in C and C++. 

### Terminology

> **Doer**<br/>
> One who does something.

Represented by `struct do_doer` in the library. Its function is analogous to that of the event dispatcher or scheduler in event-driven programming. Each `doer` instance internally maintains a *queue* (a *vector* actually) of `works` to be done.
```c
struct do_doer *doer = do_init();
/* ... */
do_destroy(doer);
```
<br/>

> **Work**<br/>
> An activity involving effort done in order to achieve a result.

Represented by `struct do_work` in the library. It ties a `work function` (i.e. the event handler 
in event-driven programming) to a `predicate`. 

The `work function` is typedefd as:
```c 
typedef bool (*work_func)(void *);
```

So, it takes a `void *` as input argument and returns a boolean. The input argument can be user defined data (or context). 
The return value can be used by the `work function` to remove itself for the `doer's` work queue after it runs. 
```c
bool work_function(void *data) {
    /* ... */
    /* Returning true here will remove the associated work */
    /* from the doer's work queue */
    return false;
}
```
<br/>

> **Predicate**<br/>
> Something which is affirmed or denied concerning an argument of a proposition. 

##### 3 types of predicates are supported by libdo - 

* Pointer-to-boolean

Use this if you want to avoid the function call overhead of predicate functions or when you're using this in conjunction with an event-driven event handler that sets this boolean. 

```c
/* work1_func() will be called if run_work1 is true */
bool run_work1 = true;
struct do_work *work1 = do_work_if(work1_func, NULL, &run_work1);
```

* Function-pointer

```c
bool work2_predicate_func(void *data) {
    return false;
}

/* ... */

/* work2_func() will be called when work2_predicate_func() returns true */
struct do_work *work2 = do_work_when(work2_func, NULL, work2_predicate_func);
```

* Time (as a `time_t` variable) 

```c
/* work3_func() will be called after 3 seconds from now */
struct do_work *work3 = do_work_after(work3_func, NULL, time(NULL) + 3);
```
<br/>

> **Priority**<br/>
> The fact or condition of being regarded as more important than others.

All `works` are initially assigned a priority equal to `SIZE_MAX` and are normally called in the order they were added, with the one added first being called first. This can be changed by assiging a priority to the `work`. `0` is reserved, `1` is the highest and `SIZE_MAX` is the lowest priority.
**Note:** You must also inform the `doer` that priorities have changed by calling the `do_set_prio_changed()` method.
```c
do_work_set_prio(work1, 1);
do_set_prio_changed(doer);
```
<br/>

> **Expirable**<br/>
> That which may expire; capable of being brought to an end.

`works` added to the `doer` queue can be indefinite or expirable. Indefinite works can be removed by having the associated `work function` return true, or by explicitly calling the `do_not_do()` method - 
```c
do_so(doer, work1);
do_so_until(doer, work2, time(NULL) + 30);	/* Will be removed after 30s */
/* ... */
do_not_do(doer, work1);
```

---

### What happens in the `loop()`?

The loop first sorts `works` by priority. It then iterates over the `work` queue, calling the `work function` if and when its associated `predicate` evaluates to true. It also removes those `works` whose `work functions` return true.

---

### Best practices

* Don't sleep in your predicate or work functions. Instead, add a `work` with a time predicate, if you want to do something after a delay.
* Add a delay between subsequent `loop()` calls, to keep the processor happy. Ideally, have a blocking function call before the `loop()` call.
* Make sure you remove `works` that you don't need.

---

### Applications

Here are a few applications I could think of:

##### 1. Dealing with streams

When dealing with streams, it is common to wait for a particular input. For example, waiting for `OK` after sending an `AT\r\n` command in serial programming using a GSM modem like SIM900. 

```c
bool waitForResp(Stream *serial, const char *expected_response, unsigned int timeout) {
	/* ... */
	while(1) {
        while (serial->available()) {
            char c = serial->read();
            /* Append to previously read chars and match with expected response */
            /* Return true if matched */
        }
        /* Break if timed-out */ 
    }
    return false;
}

int main() {
	Stream serial(/* ... */);
	serial.write("AT\r\n");
	bool ok = waitForResp(&serial, "OK", 3000);
	/* ... */
	return 0;
}
```

But there's a problem here. Waiting on a single response ignores the fact that in the meantime another command or data may be received, for example a `RING` which indicates an incoming call, and it will be lost. <br/>
Using libdo, this problem can be solved. Below is a program which rejects all incoming calls and responds to other commands as well. New commands can be added easily.

```c
static char g_cmd[30];
static bool g_initialized;

static bool read_serial_cmd(Stream *stream, char *buf, const char *delimiter, unsigned int timeout);

/* Predicates */
bool incoming_call(void *data) {
    return !strcmp(g_cmd, "RING");
}

bool ok(void *data) {
    return !strcmp(g_cmd, "OK");
}

/* Work functions */
bool reject_incoming_call(void *data) {
    Stream *serial = data;
    serial->write("ATH\r\n");
    return false; /* Run always */
}

bool set_initialized(void *data) {
    g_initialized = true;
    return true; /* Run once */
}

int main() {
	Stream serial(/* ... */);
	struct do_doer *serial_doer = do_init();

	struct do_work *reject_incoming = do_work_when(reject_incoming_call, &serial, incoming_call);
    do_so(serial_doer, reject_incoming);

	/* ... */

	serial.write("AT\r\n");
	struct do_work *check_initialized = do_work_when(set_initialized, NULL, ok);
    do_so_until(serial_doer, check_initialized, time(NULL) + 3); /* Remove the work after 3 seconds */

	while (1) {
		bool cmd_recv = read_serial_cmd(&serial, g_cmd, "\r\n", 3000);
		if (cmd_recv) {
			do_loop(serial_doer);
		}
	}

	do_destroy(serial_doer);
	return 0;
}
```
<br/>

##### 2. Deferred processing

In system programming, event driven loops that block on mutexes, signals, etc. could use the respective timed variant syscalls (`pthread_cond_timedwait()`, `pselect()`, etc.) and call the `loop()` method when such a call times out.<br/> 
So, callbacks can be added from within event handlers and serviced when there's nothing much to do. 

For example: 
```c
static struct do_doer *g_doer;

bool long_running_job(void *data) {
    /* ... */
    return true;
}

bool not_busy(void *data) {
    /* ... */
    return true;
}

void event_handler() {
	/* Handle event */
    struct do_work *deferred = do_work_when(long_running_job, NULL, not_busy);
    do_so(g_doer, deferred); 
}

int main() {
	/* ... */
    g_doer = do_init();

	int ret = pthread_cond_timedwait(&condition, &mutex, &ts);
    if (ret == 0) {
        /* ... */
        event_handler();
    }
    else if (ret == ETIMEDOUT) {
    	do_loop(g_doer);
    }

    do_destroy(g_doer);
    exit(EXIT_SUCCESS);
}
```

---

##### References:
<a name="event-driven-programming-wikipedia">1. </a>[Wikipedia: Event-driven programming](https://en.wikipedia.org/wiki/Event-driven_programming)<br/>

