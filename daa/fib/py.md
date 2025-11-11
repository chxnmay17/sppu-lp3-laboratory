## 🐍 **Python — Fibonacci Step Count**

```python
def fib_recursive(n, counter):
    counter[0] += 1
    if n <= 1:
        return n
    return fib_recursive(n - 1, counter) + fib_recursive(n - 2, counter)


n = int(input("Enter n: "))

# ----- Iterative -----
first, second = 0, 1
iter_steps = 0
for i in range(n):
    iter_steps += 1
    if i > 1:
        iter_steps += 1
    first, second = second, (i if i <= 1 else first + second)
    iter_steps += 2

print(f"Iterative Steps: {iter_steps}")

# ----- Recursive -----
count = [0]
fib_recursive(n - 1, count)
print(f"Recursive Steps: {count[0]}")
```


