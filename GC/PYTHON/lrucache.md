```python
import sys

  

# def timer(func):

#     import time

#     def wrapper(*args, **kwargs):

#         start_time = time.time()

#         result = func(*args, **kwargs)

#         end_time = time.time()

#         print(f"Execution time: {end_time - start_time:.6f} seconds")

#         return result

#     return wrapper

  

from collections import OrderedDict

from functools import wraps

from hashlib import blake2b

  

gstore = 0

def cache(max_size=15):

    body = OrderedDict()

  

    def wrapper(func):

  

        @wraps(func)

        def inner(*args,**kwargs):

            key_data = (args, tuple(sorted(kwargs.items())))

            k = blake2b(repr(key_data).encode(), digest_size=8).hexdigest()

  

            # print(args)

            if k in body:

                # print("hit", end=" ")

                body.move_to_end(k)

                global gstore

                gstore += 1

                return body[k]

            # print("miss", end=" ")

            sol = func(*args, **kwargs)

            # store in cache

            body[k] = sol

  

            # evictins strat

            if len(body) > max_size:

                body.popitem(last=False)

                # print("evict", end=" ")

            return sol

        return inner

    return wrapper

  
  

def solve():

    import random

    @cache()

    def calc(a, b):

        return a**b

  

    TOTAL = 20000

    for _ in range(TOTAL):

        a   = random.randrange(1, 10)

        top = random.randrange(1, 10)

        # print(f"CALULATING FOR THE {a} to pow {top} -> {calc(a, top)}")

        calc(a, top)

  

    print("percent hit", (gstore/TOTAL) * 100, "%")

if __name__ == "__main__":

    print(solve())
```

