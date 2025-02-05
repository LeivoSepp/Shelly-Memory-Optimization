# Shelly Script Memory Optimization

> [!TIP]
> This is a small collection of techniques for Shelly script memory optimization.  
> Shelly scripts require memory optimization as they grow.

Small scripts can run without optimization, but as they grow, they quickly hit the memory limit. When this happens, you will simply receive an "Out of Memory" message with no further details.

- [Shelly Script Memory Optimization](#shelly-script-memory-optimization)
- [Memory Optimization Recommendations in a Nutshell](#memory-optimization-recommendations-in-a-nutshell)
- [Variable names](#variable-names)
- [Values](#values)
- [Function names](#function-names)
- [print() or console.log()](#print-or-consolelog)
- [Variables and Lazy Load](#variables-and-lazy-load)
  - [Bad Practice](#bad-practice)
  - [Better Practice](#better-practice)
  - [Best Practice](#best-practice)
    - [Define data](#define-data)
    - [Accessing to the data.](#accessing-to-the-data)
- [Parsing JSON](#parsing-json)
- [Author](#author)


> [!IMPORTANT]
> Shelly devices have only 25KB of memory for scripts, shared between runtime and peak usage.

These techniques haven't been documented this way before, as we are dealing with an extremely limited amount of available memory.

Using these methods, such as short variable names or text-searching JSON instead of parsing it, makes the code less readable and harder to understand for the end user.

> [!TIP]
> Shelly doesn’t care about human readability—only efficiency.

# Memory Optimization Recommendations in a Nutshell

1. **Keep it short** – Use variable and function names with a maximum of 4 characters.
2. **Go local** – Prioritize local variables and minimize the use of global variables.
3. **Minimize variables** – If a value is used only once or twice, avoid creating a dedicated variable.
4. **Load smartly** – Extract only the necessary data instead of loading an entire dataset into memory.
5. **Fewer functions** – Merge smaller functions into larger, more efficient code blocks.
6. **Inline trivial logic** – Instead of separate functions for simple calculations, integrate them into the main function.
7. **Efficient data handling** – Use string search instead of JSON.parse() for handling large datasets, as parsing consumes a lot of memory.

# Variable names

1. Use short names (max 4 chars) → 14 bytes. :+1:
2. Names with 5–14 chars consume more memory → 28 bytes.
3. Names with 15–24 chars are memory-hungry → 56 bytes. :-1:

This memory allocation is related only to the variable name.
```js
// Short variable name 1-4 char
let shrt;
// Memory used: 14 bytes

// Normal variable name 5-14 char -> 28 bytes
let myNormalName;
// Memory used: 28 bytes

// Long variable name 15-24 char
let mySpecialExtraLongName;
// Memory used: 56 bytes
```

# Values
1. Use short values when possible: strings under 9 chars. :+1:
2. Arrays and JSON take up the same space if JSON keys are short.
3. Keep JSON key names to a maximum of 4 chars. :+1:

```js
// Int value until 8191
let int = 8191;
// Memory used: 14 bytes

// Int 8192 - some big number
let int = 123456789012345678901234567890;
// Memory used: 28 bytes

// String value 1-9 chars
let str = "abcdefghi";
// Memory used: 28 bytes

// String value 10-19 chars, every 9 chars -> 14 bytes
let str = "abcdefghijklmnopqrs";
// Memory used: 42 bytes

// Empty array
let arr=[];
// Memory used: 28 bytes

// Array each value int->14 byte, string->28
let arr=["volvo", 2016];
// Memory used: 70 bytes (array + string + int)

// Empty JSON
let json = {};
// Memory used: 28 bytes

// JSON each value int->14 byte, string->28
let json = {car: "volvo", year: 2016};
// Memory used: 70 bytes, same as Array

// JSON, long names > 4 chars
let json = {coolCar: "volvo", prodYear: 2016};
// Memory used: 98 bytes
```

# Function names
1. Use short names (max 4 chars) → 70 bytes. :thumbsup:
2. Names with 5–14 chars consume more memory → 84 bytes.
3. Names with 15–24 chars are memory hungry → 98 bytes. :thumbsdown:

> [!TIP]
> The initial amount of data or logic inside a function doesn’t impact memory usage unless it is actually used.

```js
// Short function name 1-4 chars
function strt() {
    return;
}
// Memory Used: 70 Peak: 70

// Normal name 5-14 chars 
function startLongName() {
    return;
}
// Memory Used: 84 Peak: 84

// Long name 15-24 chars
function startExtraVeryLongName() {
    return;
}
// Memory Used: 98 Peak: 98
```

# print() or console.log()
1. Use print(), always &rArr; 0 bytes. :thumbsup:
2. Using console.log() &rArr; 42 bytes. :thumbsdown:

```js
// Use always print()
print("Hello world!"); 
// Memory used: 0 byte

// Try to avoid console.log()
console.log("Hello World!"); 
// Memory used 42 bytes
```

# Variables and Lazy Load
## Bad Practice

Defining global variables this way is very costly from a memory perspective.

Long variable names and constant data loading into memory are inefficient. :-1:

```js
// Define data in multiple JSONs
const VORK1 = { dayRate: 77.2, nightRate: 77.2, dayMaxRate: 77.2, holidayMaxRate: 77.2 };
const VORK2 = { dayRate: 60.7, nightRate: 35.1, dayMaxRate: 60.7, holidayMaxRate: 35.1 };
const VORK4 = { dayRate: 36.9, nightRate: 21, dayMaxRate: 36.9, holidayMaxRate: 21 };
const VORK5 = { dayRate: 52.9, nightRate: 30.3, dayMaxRate: 81.8, holidayMaxRate: 47.4 };
const Partner24 = { dayRate: 60.7, nightRate: 60.7, dayMaxRate: 60.7, holidayMaxRate: 60.7 };
const Partner24Plus = { dayRate: 38.6, nightRate: 38.6, dayMaxRate: 38.6, holidayMaxRate: 38.6 };
const Partner12 = { dayRate: 72.4, nightRate: 42, dayMaxRate: 72.4, holidayMaxRate: 42 };
const Partner12Plus = { dayRate: 46.4, nightRate: 27.1, dayMaxRate: 46.4, holidayMaxRate: 27.1 };
const NONE = { dayRate: 0, nightRate: 0, dayMaxRate: 0, holidayMaxRate: 0 };
// Memory Used: 1764 Peak: 1764 

// Accessing the data
let totl = VORK1.dayRate + VORK1.nightRate + VORK1.dayMaxRate + VORK1.holidayMaxRate;
// Memory Used: 1806 Peak: 1806
```

## Better Practice
Short names, max 4 chars. Any name longer than 4 chars consistently consumes more memory. :+1:

Accessing any parameter doesn't change memory usage, as everything is already loaded.

```js
// Define data as JSON
const tRt = {
    VRK1: { dRt: 77.2, nRt: 77.2, dMRt: 77.2, hMRt: 77.2 },
    VRK2: { dRt: 60.7, nRt: 35.1, dMRt: 60.7, hMRt: 35.1 },
    VRK4: { dRt: 36.9, nRt: 21, dMRt: 36.9, hMRt: 21 },
    VRK5: { dRt: 52.9, nRt: 30.3, dMRt: 81.8, hMRt: 47.4 },
    P24: { dRt: 60.7, nRt: 60.7, dMRt: 60.7, hMRt: 60.7 },
    P24P: { dRt: 38.6, nRt: 38.6, dMRt: 38.6, hMRt: 38.6 },
    P12: { dRt: 72.4, nRt: 42, dMRt: 72.4, hMRt: 42 },
    P12P: { dRt: 46.4, nRt: 27.1, dMRt: 46.4, hMRt: 27.1 },
    NONE: { dRt: 0, nRt: 0, dMRt: 0, hMRt: 0 },
}
// Memory Used: 1176 Peak: 1176

// Accessing the data
let totl = tRt.VRK1.dRt + tRt.VRK1.nRt + tRt.VRK1.dMRt + tRt.VRK1.hMRt;
// Memory Used: 1218 Peak: 1218
```

## Best Practice 
### Define data

Data defined inside a function and accessed only when required is called the LazyLoad technique. :+1:

```js
// Define data inside a function
function tRt() {
    return {
        VRK1: { dRt: 77.2, nRt: 77.2, dMRt: 77.2, hMRt: 77.2 },
        VRK2: { dRt: 60.7, nRt: 35.1, dMRt: 60.7, hMRt: 35.1 },
        VRK4: { dRt: 36.9, nRt: 21, dMRt: 36.9, hMRt: 21 },
        VRK5: { dRt: 52.9, nRt: 30.3, dMRt: 81.8, hMRt: 47.4 },
        P24: { dRt: 60.7, nRt: 60.7, dMRt: 60.7, hMRt: 60.7 },
        P24P: { dRt: 38.6, nRt: 38.6, dMRt: 38.6, hMRt: 38.6 },
        P12: { dRt: 72.4, nRt: 42, dMRt: 72.4, hMRt: 42 },
        P12P: { dRt: 46.4, nRt: 27.1, dMRt: 46.4, hMRt: 27.1 },
        NONE: { dRt: 0, nRt: 0, dMRt: 0, hMRt: 0 },
    }
}
// Memory Used: 56 Peak: 56
```

### Accessing to the data.

**Bad Practice**

It's a bad practice to load the entire dataset into memory.  
The outcome is the same as the previous example. :-1:

```js
// Load full data into variable and use it partially
let tRt = tRt();
let totl = tRt.VRK1.dRt + tRt.VRK1.nRt + tRt.VRK1.dMRt + tRt.VRK1.hMRt;
// Memory Used: 1218 Peak: 1218
```

**Better Practice**

There is only a peak memory load when the program reads the data, and after that, the memory is released. :+1:  
However, creating variables from that data consumes more memory than without a variable.

```js
// Load only required data into variable
let vrk1 = tRt().VRK1;
let totl = vrk1.dRt + vrk1.nRt + vrk1.dMRt + vrk1.hMRt;
// Memory Used: 238 Peak: 994
```

**Best Practice**

Two important things to consider here:

1. The only variable should be the sum of all the values.
2. The variable name should be exactly the same as the function name to gain the biggest benefit. 
   - I just randomly figured this out :thumbsup:

```js
// Load data as needed, without creating variable
// Variable name as you like
let totl = tRt().VRK1.dRt + tRt().VRK1.nRt + tRt().VRK1.dMRt + tRt().VRK1.hMRt;
// Memory Used: 98 Peak: 1022

// Load data as needed, without creating variable
// Variable name === Function name
let tRt = tRt().VRK1.dRt + tRt().VRK1.nRt + tRt().VRK1.dMRt + tRt().VRK1.hMRt;
// Memory Used: 42 Peak: 1008
```

# Parsing JSON 
Use string search to extract data from large JSON. This is a significant memory improvement. :+1:  
Do not parse the entire JSON into memory. :-1:

```js
// Define data as string
let data = '{"isok":true,"data":{"online":true,"device_status":{"script:1":{"id":1,"running":false},"_updated":"2021-10-19 07:59:18","cloud":{"connected":true},"script:4":{"id":4,"running":false},"wifi":{"sta_ip":"192.168.2.16","status":"got ip","ssid":"linux-bg.org","rssi":-59}}}}'
// Memory Used: 406

// Parse the data as JSON and store in memory
let json = JSON.parse(data);
// Memory Used: 1120

// The whole data in memory, accessing rssi adds just 14 bytes
let rssi = json.data.device_status.wifi.rssi;
// Memory Used: 1134

// Search string 'rssi' inside of the string
// to get only this small part of value
let rsId = data.indexOf("rssi") + 6;
let rssi = data.substring(rsId, data.indexOf("}", rsId));
// Memory Used: 448

```

# Author

Created by Leivo Sepp, 07.01.2025

[Shelly Script Memory Optimization - GitHub Repository](https://github.com/LeivoSepp/Shelly-Memory-Optimization)