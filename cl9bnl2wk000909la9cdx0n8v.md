# Mega Sekurak Hacking Party CTF October 2022

> First things first - I'm tight on schedule so keep eyes on updated version with a lot more details, more in-depth analysis in my style ;)

# postgres


![2022-10-15-12-26-12.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665941561757/0vrcyS3oT.png align="left")

Solve:

Table `flag` has flag as a title on one of its columns.

```sql
>> SELECT to_json((SELECT t FROM public.flag t LIMIT 1))
({'val1': 'Close', 'CTF_a2*****************************4dd89': 'but not there yet'},)
```

# deobf

Solve:
Tried deobfuscate and understand what this code is doing, but due to lack of time I've slapped few `console.log(..)` functions and got the flag in the output.

```js
console.log(window[_0x553b6f(0xca, 'xKir')](_0x553b6f(0xc2, 'rW2u')))
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665941939535/9jVyo-C5_.png align="left")

# traversal

Solve: 

When second argument in `Path.Combine` is rooted path, first one is ignored.

https://learn.microsoft.com/en-us/dotnet/api/system.io.path.combine?view=net-6.0

![2022-10-15-23-07-39.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665942203007/k56cz5kKg.png align="left")

# math-basic


![2022-10-15-08-55-13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665942425345/FTX035B-g.png align="left")

Solve:

Read lines with `telnetlib` library and write solves accordingly.

![2022-10-15-10-38-13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665942440392/_DvLAKaxV.png align="left")

```py
#!/usr/bin/env python3`

import string
import math
from telnetlib import Telnet

host = "192.46.238.159"
port = 1337

def tcSend(tc, msg):
    tc.write(msg.encode('ascii') + b"\n")

def tcReadUntil(tc, end):
    #print(end)
    return tc.read_until(end.encode('ascii'), 5)

def tcReadNextTask(tc):
    lines = []
    try:
        tcOutput = tcReadUntil(tc, ">>")
        lines = tcOutput.splitlines()
        return lines[-2]
    except:
        print(lines)
        return ''


# https://geekflare.com/prime-number-in-python/
def isPrime(n):
  for i in range(2,int(math.sqrt(n))+1):
    if (n%i) == 0:
      return False
  return True

def solveTask(text):
    tokens = [x.decode('ascii') for x in text.split()]
    try:
        if 'perfect' in tokens:
            #print('perfect')
            n = int(tokens[1])
            return math.sqrt(n).is_integer()
        if 'prime?' in tokens:
            #print('prime')
            n = int(tokens[1])
            return isPrime(n)
        if 'divisible' in tokens:
            #print('divisible')
            n1 = int(tokens[1])
            n2 = int(tokens[4][:-1])
            return n1 % n2 == 0
        else:
            #print('??')
            return print(tokens)
    except Exception as e:
        print(tokens)
        print(e)

if __name__ == "__main__":
    accepted_characters = ''
    with Telnet(host, port) as tc:
        while True:
            taskText = tcReadNextTask(tc)
            if taskText == '':
                break
            answer = 'Y' if solveTask(taskText) else 'N'
            tcSend(tc, answer)

```

* https://blog.cyberethical.me/google-ctf-2021-filestore#exploit
* https://www.mathsisfun.com/definitions/perfect-square.html
* https://geekflare.com/prime-number-in-python/