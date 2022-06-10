We first examine the source code provided,


```
#include <iostream>
#include <unordered_map>
#include <ctime>
#include <string>
using namespace std;

#define LIMIT 25000

string flag = "<REDACTED>";

int main() {

    long long count = 0;
    // id => price
    unordered_map<long long, int> map;

    cout << "Blazinn fazz hashmapp price tracker" << endl;
    cout << "I'll give you something nice if it's too slow" << endl;

    double time = 0;
    int op = LIMIT;
    long long sum;
    while (op--) {
        int action; cin >> action;

        clock_t begin;

        switch (action) {

            case 0:
                long long id; int amount;
                cin >> id >> amount;
                if (count == LIMIT) {
                    cout << "This is too much..." << endl;
                    break;
                }
                count++;

                begin = clock();
                map[id] = amount;
                time += (double) (clock() - begin) / CLOCKS_PER_SEC;

                break;

            case 1:
                sum = 0;

                begin = clock();
                for (auto &item : map) {
                    sum += item.second;
                }
                time += (double) (clock() - begin) / CLOCKS_PER_SEC;

                cout << "The total amount is " << sum << endl;

                break;

            case 2:
                cout << "The total time is " << time << endl;
        }

        if (time > 5.0) {
            cout << flag << endl;
            return 1;
        }
    }
}

```

The server creates an `unordered_map` which is a hashmap and then sums up the time taken for each operation up to `LIMIT = 25000` operations. There are 3 types of operations, inserting into the map (0), summing values in map (1) and checking the current time (3). We obtain the flag when `time > 5.0`.

It is clear that ignoring operation 3 is optimal as it doesn't add to the time. There are some other optimisations that helped us solve the problem:

1. Doing all the inserts before the summing to ensure we sum up the maximum number of elements each time.
2. Using a large number for each value in the map so that the summing would take longer.
3. Using a large random number for each key in the map. The idea is to make the hashing take longer and use as many buckets in the hashmap as possible so the summing process later on takes longer as the values are further apart in memory.
4. Using an equal number of operation 1's and operation 2's. If we denote the number of operation 2's as `x`,  the number of sums made would be `(25000 - x) * x`, which has a maximum value when `x = 25000 / 2`. 

Using these optimisations, we can write a python script that connects to the server and sends the data required.

```
from pwn import *

conn = remote("challs.nusgreyhats.org", 10527)
for i in range(2):
    text = conn.recvline()

// fills the hashmap with random large values
for i in range(25000//2):
    conn.send(bytes("0\n", encoding="utf-8"))
    conn.send(bytes(f"{randint(0, 9223372036854775807)} 2147483647\n", encoding="utf-8"))

// starts the summing and prints the flag if it is received
for i in range(25000//2):
    conn.send(bytes("1\n", encoding="utf-8"))
    text = conn.recv(1000)
    if "grey" in text.decode(encoding='utf-8'):
        print(text)
```

We obtain the flag `grey{h4sHmaP5_r_tH3_k1nG_of_dAt4_strUcTuRe5}`.