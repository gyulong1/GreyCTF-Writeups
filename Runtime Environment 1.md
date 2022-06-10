Since this is a RE challenge, we start by loading the binary into a disassembler like IDA. Looking at the `main.main` function, we see that the bulk of the action happens in the `main.encode` function. It looks really scary but after distilling the important parts we get this.

```
qmemcpy(sub_str, "NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe", sizeof(sub_str));
  len = 3 * ((__int64)a11 / 3);
  index = 0LL;
  out_index = 0LL;
  while ( (__int64)index < len )
  {
    v14 = *(unsigned __int8 *)(a10 + index);

    // Step 1
    v15 = (v14 << 16) | ((a10 + index + 1) << 8) | (index + a10 + 2);

    // Step 2 & 3
    v16 = sub_str + ((v15 >> 18) & 63));

    (a7 + out_index) = v16;
    v17 = v15;
    v18 = sub_str + ((v15 >> 12) & 63));

    (out_index + a7 + 1) = v18;
    v19 = v17;
    v20 = sub_str + ((v17 >> 6) & 63));

    (out_index + a7 + 2) = v20;
    a6 = out_index + 3;
    v21 = sub_str + (v19 & 63));

    (a7 + out_index + 3) = v21;
    index += 3LL;
    out_index += 4LL;
  }
  v22 = a11 - index;
  if ( a11 == index )
    return output;
```

We deduced that the encoding algorithm goes like this:
1. In groups of 3, convert each character to a 8-digit binary number and concatenate them, resulting in a 24-bit number.
2. Split the binary number into 4 groups of 6 and convert each 6-digit binary back to decimal.
3. Use each decimal number as the index for which character to output using the string `NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe`,
so 0 would become `N`, 1 would become `a` ...

The remaining parts is to handle cases where the output isn't a multiple of 3 which requires padding '-'. Great, now we can reverse the encoding using a python script.
```
substitution = "NaRvJT1B/m6AOXL9VDFIbUGkC+sSnzh5jxQ273d4lHPg0wcEpYqruWyfZoM8itKe"

def decode(challenge):
    arr = []

    // change each character to a 6 digit binary number according to its index in the substitution string
    for i in challenge:
        if i != '-':
            arr.append('{0:06b}'.format(substitution.find(i)))

    // concatenate the binary string array
    arr = "".join(arr)

    out = []
    // split into groups of 8 and convert back to ASCII
    for i in range(0, len(arr), 8):
        if(int(arr[i:i + 8], 2) != 0):
            out.append(chr(int(arr[i:i + 8], 2)))

    return "".join(out)

start = "GvVf+fHWz1tlOkHXUk3kz3bqh4UcFFwgDJmUDWxdDTTGzklgIJ+fXfHUh739+BUEbrmMzGoQOyDIFIz4GvTw+j--"

while True:
    start = decode(start)
    print(start)
```

The last step took us quite a bit of guessing which was to decode `challenge.txt` several times to get the flag. We get the flag `grey{B4s3d_G0Ph3r_r333333}`.