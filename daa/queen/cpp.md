


---

## 🟦 **C++ — N-Queens with First Queen Fixed**

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, r0, c0;
    cout << "Enter n r0 c0: ";
    cin >> n >> r0 >> c0;

    if (n <= 0 || r0 < 0 || r0 >= n || c0 < 0 || c0 >= n) {
        cout << "Invalid input\n";
        return 0;
    }

    vector<vector<int>> board(n, vector<int>(n, 0));
    vector<int> col(n, 0), d1(2*n-1, 0), d2(2*n-1, 0);

    auto place = [&](int r, int c, int v) {
        board[r][c] = v;
        col[c] = v;
        d1[r - c + (n - 1)] = v;
        d2[r + c] = v;
    };

    place(r0, c0, 1);

    function<bool(int)> solve = [&](int r) -> bool {
        if (r == n) return true;
        if (r == r0) return solve(r + 1); // skip fixed row
        for (int c = 0; c < n; ++c) {
            if (col[c] || d1[r - c + (n - 1)] || d2[r + c]) continue;
            place(r, c, 1);
            if (solve(r + 1)) return true;
            place(r, c, 0);
        }
        return false;
    };

    if (!solve(0)) {
        cout << "No solution\n";
        return 0;
    }

    cout << "\nFinal N-Queens Matrix:\n";
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j)
            cout << board[i][j] << " ";
        cout << "\n";
    }
    return 0;
}
```


