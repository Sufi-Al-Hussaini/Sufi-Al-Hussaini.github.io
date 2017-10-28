---
layout: post
title: Convert string to C++11 fixed width integer type
description: C++11 version of .NET's TryParse
date: '2017-10-28 16:19:23'
tags: c++ 
---


## A collection of functions that allow to convert a string representation of a number to its equivalent C++11 fixed width integer type (eg. uint8_t, int16_t, etc.).

***

I was working on [NXP's JN-AN-1216-Zigbee-3-0-IoT-ControlBridge](https://www.nxp.com/products/wireless-connectivity/proprietary-ieee-802.15.4-based/zigbee/zigbee-3.0:ZIGBEE-3-0) serial protocol, trying to implement it in C++11, when I was faced with a problem. The .NET protocol implementation uses the `TryParse` family of methods to convert string number representations to their equivalent integer types.
```c#
private bool bStringToUint8(string inputString, out byte u8Data)
{
    bool bResult = true;

    if (Byte.TryParse(inputString, NumberStyles.HexNumber, CultureInfo.CurrentCulture, out u8Data) == false)
    {
        // Show error message
        MessageBox.Show("Invalid Parameter");
        bResult = false;
    }
    return bResult;
}
```

I wanted similar functionality in C++ as I wanted my implementation to resemble NXP's .NET based API.
So here's my solution based off of [this Stack Overflow answer](https://stackoverflow.com/a/8715855/3490458):
```c++
uint8_t stou8(std::string const &str, size_t *idx = 0, int base = 10) {
    auto result = std::stoul(str, idx, base);
    if (result > std::numeric_limits<uint8_t>::max()) { throw std::out_of_range("stou8"); }
    return static_cast<uint8_t>(result);
}
```

The interface is very much like that of `std::stoi` and the base arg allows to parse decimal, hexadecimal, etc..
Ofcourse this is not exactly like .NET's `TryParse` as it uses exceptions for error handling, but it is quite straightforward to wrap it to get a `TryParse` like interface.
```c++
bool stou8(std::string const &str, uint8_t &u8, size_t *idx = 0, int base = 10) {
    try {
        u8 = stou8(str, idx, base);
    } catch (...) {
    	u8 = 0;		// Just as TryParse does
        return false;
    }
    return true;
}
```

You could write the other variants like so:
```c++
uint16_t stou16(std::string const &str, size_t *idx = 0, int base = 10) {
    auto result = std::stoul(str, idx, base);
    if (result > std::numeric_limits<uint16_t>::max()) { throw std::out_of_range("stou16"); }
    return static_cast<uint16_t>(result);
}

uint32_t stou32(std::string const &str, size_t *idx = 0, int base = 10) {
    auto result = std::stoul(str, idx, base);
    if (result > std::numeric_limits<uint32_t>::max()) { throw std::out_of_range("stou32"); }
    return static_cast<uint32_t>(result);
}

uint64_t stou64(std::string const &str, size_t *idx = 0, int base = 10) {
    auto result = std::stoull(str, idx, base);
    if (result > std::numeric_limits<uint64_t>::max()) { throw std::out_of_range("stou64"); }
    return static_cast<uint64_t>(result);
}

// ...
```

Or use some template magic to do this:
```c++
template<
        typename T,
        typename = typename std::enable_if<std::is_arithmetic<T>::value, T>::type
> T stou(std::string const &str, size_t *idx, int base) {
    auto result = std::stoul(str, idx, base);
    if (result > std::numeric_limits<T>::max()) { throw std::out_of_range("stou"); }
    return static_cast<T>(result);
}
```

The above function may or may not work for `uint64_t` depending on whether or not the width of `unsigned long` is 64 bits or more because `std::stoul` returns an `unsigned long`. The standard only requires it to be atleast 32 bit wide.<br/>
So a better solution would be to call `std::stoul` or `std::stoull` based on the size of the fixed width integer type:
```c++
template<
        typename T,
        typename = typename std::enable_if<std::is_arithmetic<T>::value, T>::type
> T stou(std::string const &str, size_t *idx, int base) {
    auto result = (sizeof(T) > sizeof(unsigned long))
                  ? std::stoull(str, idx, base) : std::stoul(str, idx, base);
    if (result > std::numeric_limits<T>::max()) { throw std::out_of_range("stou"); }
    return static_cast<T>(result);
}

auto stou8 = stou<uint8_t>;
auto stou16 = stou<uint16_t>;
auto stou32 = stou<uint32_t>;
auto stou64 = stou<uint64_t>;
```

