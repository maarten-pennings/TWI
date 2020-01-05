# TWI
Timeout added to Arduino TWI implementation

## Introduction
The standard I2C functions on Arduino can hang.
When `endTransaction` does not get an ack from the slave, the function does not return.
This is a feature of the avr library 1.8.2 that comes with Arduino IDE 1.8.10 (January 2020).

## Problem
The problem is in the implementation of the `twi` utility, more specifically the `twi_writeTo()` function.
On my machine you find it as a subdirectory of the `Wire` library.

```
C:\Users\Maarten\AppData\Local\Arduino15\packages\arduino\hardware\avr\1.8.2\libraries\Wire\src\utility
```

That function contains a while loop without time-out:

```
  // wait for write operation to complete
  while(wait && (TWI_MTX == twi_state)){
    continue;
  }
```

## Fix

I have changed the infinit while loop to

```
  // wait for write operation to complete
  now=micros();                                           // MAARTEN
  while(wait && (TWI_MTX == twi_state)){
    if( micros()-now > twi_timeout_us ) return 5;         // MAARTEN
    continue;
  }
```

The variable `twi_timeout_us` is introduced by me by adding to the top of the file

```
static uint32_t twi_timeout_us = 100000L; // 100ms   // MAARTEN
void twi_timeout_set_us(uint32_t timeout_us) {       // MAARTEN
    twi_timeout_us= timeout_us;                      // MAARTEN
}                                                    // MAARTEN
                                                     // MAARTEN
uint32_t twi_timeout_get_us(void) {                  // MAARTEN
    return twi_timeout_us;                           // MAARTEN
}                                                    // MAARTEN
```

## Diff

This project gives you
 - the original [twi.h](twi.org.h) from avr library 1.8.2 that comes with Arduino IDE 1.8.10 (January 2020).
 - the original [twi.c](twi.org.c) from avr library 1.8.2 that comes with Arduino IDE 1.8.10 (January 2020).
 - my [twi.h](twi.h) with the time out added
 - my [twi.c](twi.c) with the time out added
 - a diff screenshot for `twi.h` [twi.h](twi.h.diff.png)
 - the diff screenshots for `twi.c` [twi.c 1](twi.c.diff1.png) [twi.c 2](twi.c.diff2.png)
 
 ## Use
 
 If you want to use the fix
  - download the zip file
  - copy `twi.h` over `C:\Users\<you>\AppData\Local\Arduino15\packages\arduino\hardware\avr\1.8.2\libraries\Wire\src\utility\twi.h`
  - copy `twi.c` over `C:\Users\<you>\AppData\Local\Arduino15\packages\arduino\hardware\avr\1.8.2\libraries\Wire\src\utility\twi.c`
  - it is wise to first do a diff if you have a different version than 
    avr library 1.8.2 that comes with Arduino IDE 1.8.10 (January 2020).
  - if you upgrade Arduino (libs) this fix gets overwritten, so you have to redo it
  - Note that Wire has these errors by default
    - 0:success
    - 1:data too long to fit in transmit buffer
    - 2:received NACK on transmit of address
    - 3:received NACK on transmit of data
    - 4:other error
  - this library now adds one more
    - 5:timeout
 
 (end of doc)
