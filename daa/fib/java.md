---

## 🟨 **Java — Fibonacci Step Count**

```java
import java.util.Scanner;

public class FibonacciStepsOnly {
    static int recursiveSteps = 0;

    static int fibRecursive(int n) {
        recursiveSteps++; // count one step per call
        if (n <= 1)
            return n;
        return fibRecursive(n - 1) + fibRecursive(n - 2);
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter n: ");
        int n = sc.nextInt();

        // ----- Iterative -----
        long a = 0, b = 1, c;
        int iterativeSteps = 0;

        System.out.print("Fibonacci Series (Iterative): ");
        if (n >= 1) System.out.print(a + " ");
        if (n >= 2) System.out.print(b + " ");

        for (int i = 2; i < n; i++) {
            iterativeSteps++; // one step per iteration
            c = a + b;
            System.out.print(c + " ");
            a = b;
            b = c;
        }

        System.out.println("\nIterative Steps: " + iterativeSteps);

        // ----- Recursive -----
        recursiveSteps = 0;
        fibRecursive(n); // calculate recursively for step counting
        System.out.println("Recursive Steps: " + recursiveSteps);
    }
}


```



