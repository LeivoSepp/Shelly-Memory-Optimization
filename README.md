# Shelly Script Memory Optimization

> [!TIP]
> Shelly scripts require memory optimization as they grow.

Small scripts can run without optimization, but as they grow, they quickly hit the memory limit. When this happens, you will simply receive an "Out of Memory" message with no further details.

- [Shelly Script Memory Optimization](#shelly-script-memory-optimization)
- [Variables and Lazy Load](#variables-and-lazy-load)
  - [Bad Practice :thumbsdown:](#bad-practice-thumbsdown)
  - [Better Practice :thumbsup:](#better-practice-thumbsup)
  - [Best Practice :thumbsup::thumbsup:](#best-practice-thumbsupthumbsup)
    - [Define data](#define-data)
    - [Accessing to the data.](#accessing-to-the-data)
- [Variable names](#variable-names)
- [Values](#values)
- [Function names](#function-names)
- [print() or console.log()](#print-or-consolelog)
- [Parsing JSON](#parsing-json)


> [!IMPORTANT]
> Shelly devices have only 25KB of memory for scripts, shared between runtime and peak usage.

Below is a small collection of techniques for Shelly script memory optimization.

These techniques haven't been documented this way before, as we are dealing with an extremely limited amount of available memory.

Using these methods, such as short variable names or text-searching JSON instead of parsing it, makes the code less readable and harder to understand for the end user.

> [!TIP]
> Shelly doesn’t care about human readability—only efficiency.

# Variables and Lazy Load
## Bad Practice :thumbsdown:
Defining global variables in this way is very costly from memory point of view.

Long variable names and all the data loaded constantly into memory.

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

## Better Practice :thumbsup:
Short names, max 4 chars. Any name longer than 4 char consumes constantly more memory. 

Accessing to any parameter doesnt change the memory usage, as everything is already there. 


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

## Best Practice :thumbsup::thumbsup:
### Define data
Data is defined inside a function and accessed only when it required, this is called LazyLoad technique.

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

**Bad Practice** :thumbsdown:

This is bad practice as the whole dataset loaded into memory and now the outcome is same with the previous example.
```js
// Load full data into variable and use it partially
let tRt = tRt();
let totl = tRt.VRK1.dRt + tRt.VRK1.nRt + tRt.VRK1.dMRt + tRt.VRK1.hMRt;
// Memory Used: 1218 Peak: 1218
```

**Better Practice** :thumbsup:

There is only peak memory load when the program reads the data and after that the memory released.
However, creating variables from that data is consuming more memory than without a variable.

```js
// Load only required data into variable
let vrk1 = tRt().VRK1;
let totl = vrk1.dRt + vrk1.nRt + vrk1.dMRt + vrk1.hMRt;
// Memory Used: 238 Peak: 994
```

**Best Practice** :thumbsup: :thumbsup:

Two important things here to consider:
1. The only variable is the the sum of all the values.
2. The variable name must be exactly as the function name to gain the biggest benefit. I just randomly figured this out.

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
# Variable names
1. Use short names, max 4 char -> 14 bytes. :thumbsup:
2. Names 5-14 char consumes more memory -> 28 bytes.
3. Names 15-24 char consumes even more memory -> 56 bytes.

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
1. Use short values if possible, strings less than 9 chars.
2. Array and JSON are in same size if json names are short.
3. Keep names inside a JSON max 4 chars.

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
// Memory used: 70 bytes (json + string + int)

// JSON, long names > 4 chars
let json = {coolCar: "volvo", prodYear: 2016};
// Memory used: 98 bytes
```

# Function names
1. Use short names, max 4 char -> 70 bytes.
2. Names 5-14 char consumes more memory -> 84 bytes.
3. Names 15-24 char consumes even more memory -> 98 bytes.

The initial amount of the data or logic inside of the function doesn't make any sense from memory point of view unless it is not used. 
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
1. Use print(), always &rArr; 0 bytes.
2. Using console.log() &rArr; 42 bytes.

```js
// Use always print()
print("Hello world!"); 
// Memory used: 0 byte

// Try to avoid console.log()
console.log("Hello World!"); 
// Memory used 42 bytes
```

# Parsing JSON 
1. Do not parse the whole JSON into memory.
2. Use string search to get data from large JSON. This is significant memory improvement. 

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