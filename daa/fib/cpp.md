

---

## 🟦 **C++ — Fibonacci Step Count**

```cpp
#include <iostream>
using namespace std;

int recursiveSteps = 0;

int fibRecursive(int n) {
    recursiveSteps++;  // Count each call
    if (n <= 1)
        return n;
    return fibRecursive(n - 1) + fibRecursive(n - 2);
}

int main() {
    int n;
    cout << "Enter n: ";
    cin >> n;

    // ----- Iterative -----
    long long first = 0, second = 1, next;
    int iterativeSteps = 0;
    for (int i = 0; i < n; i++) {
        iterativeSteps++;
        if (i > 1) iterativeSteps++; // addition step
        first = second;
        second = (i <= 1) ? i : first + second;
        iterativeSteps += 2;
    }

    cout << "Iterative Steps: " << iterativeSteps << endl;

    // ----- Recursive -----
    recursiveSteps = 0;
    fibRecursive(n - 1);
    cout << "Recursive Steps: " << recursiveSteps << endl;

    return 0;
}
```


