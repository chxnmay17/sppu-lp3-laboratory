---

## 🐍 **Python — N-Queens with First Queen Fixed**

```python
def solve_nqueens_fixed(n, r0, c0):
    if n <= 0 or not (0 <= r0 < n and 0 <= c0 < n):
        return None, "Invalid input"

    board = [[0]*n for _ in range(n)]
    col = [False]*n
    d1 = [False]*(2*n-1)
    d2 = [False]*(2*n-1)

    def place(r, c, v):
        board[r][c] = 1 if v else 0
        col[c] = v
        d1[r - c + (n - 1)] = v
        d2[r + c] = v

    place(r0, c0, True)

    def backtrack(r):
        if r == n:
            return True
        if r == r0:
            return backtrack(r + 1)
        for c in range(n):
            if col[c] or d1[r - c + (n - 1)] or d2[r + c]:
                continue
            place(r, c, True)
            if backtrack(r + 1):
                return True
            place(r, c, False)
        return False

    if not backtrack(0):
        return None, "No solution"
    return board, None


if __name__ == "__main__":
    n, r0, c0 = map(int, input("Enter n r0 c0: ").split())
    res, err = solve_nqueens_fixed(n, r0, c0)
    if err:
        print(err)
    else:
        print("\nFinal N-Queens Matrix:")
        for row in res:
            print(*row)
```




