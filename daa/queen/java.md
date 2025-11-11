---

## 🟨 **Java — N-Queens with First Queen Fixed (Scanner version)**

```java
import java.util.*;

public class NQueensFixedFirst {
    static int n, r0, c0;
    static int[][] board;
    static boolean[] col, d1, d2;

    static void place(int r, int c, boolean v) {
        board[r][c] = v ? 1 : 0;
        col[c] = v;
        d1[r - c + (n - 1)] = v;
        d2[r + c] = v;
    }

    static boolean solve(int r) {
        if (r == n) return true;
        if (r == r0) return solve(r + 1); // skip fixed row
        for (int c = 0; c < n; c++) {
            if (col[c] || d1[r - c + (n - 1)] || d2[r + c]) continue;
            place(r, c, true);
            if (solve(r + 1)) return true;
            place(r, c, false);
        }
        return false;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter n r0 c0: ");
        n = sc.nextInt();
        r0 = sc.nextInt();
        c0 = sc.nextInt();

        if (n <= 0 || r0 < 0 || r0 >= n || c0 < 0 || c0 >= n) {
            System.out.println("Invalid input");
            return;
        }

        board = new int[n][n];
        col = new boolean[n];
        d1 = new boolean[2 * n - 1];
        d2 = new boolean[2 * n - 1];

        place(r0, c0, true);

        if (!solve(0)) {
            System.out.println("No solution");
            return;
        }

        System.out.println("\nFinal N-Queens Matrix:");
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++)
                System.out.print(board[i][j] + " ");
            System.out.println();
        }
    }
}
```



