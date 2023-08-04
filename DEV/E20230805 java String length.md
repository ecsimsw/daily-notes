## Java String length

### ANS 1
Strings are backed by char[] arrays, so they are limited to the size of an array structure. An array can hold Integer.MAX_VALUE elements and it also depends on how much memory you've allocated to the JVM. So, in theory a String can be upwards of two billion characters

### ANS 2
Since the maximum size a string can hold is Integer.MAX_VALUE i.e. 2^31-1 bytes which is 2GB-1byte ~ 2GB , a String can hold 1400 Kindle e-book contents
