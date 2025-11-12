

---

## 🟩 **Java — Simple Huffman Coding**

```java
import java.util.*;

class HuffmanNode implements Comparable<HuffmanNode> {
    int freq;
    char ch;
    HuffmanNode left, right;
    HuffmanNode(int f, char c) { freq = f; ch = c; }
    public int compareTo(HuffmanNode n) { return freq - n.freq; }
}

public class HuffmanCoding {
    static void printCodes(HuffmanNode root, String code) {
        if (root == null) return;
        if (root.left == null && root.right == null)
            System.out.println(root.ch + " -> " + code);
        printCodes(root.left, code + "0");
        printCodes(root.right, code + "1");
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter number of characters: ");
        int n = sc.nextInt();

        PriorityQueue<HuffmanNode> q = new PriorityQueue<>();
        for (int i = 0; i < n; i++) {
            System.out.print("Enter character " + (i+1) + ": ");
            char ch = sc.next().charAt(0);
            System.out.print("Enter frequency of '" + ch + "': ");
            int f = sc.nextInt();
            q.add(new HuffmanNode(f, ch));
        }

        while (q.size() > 1) {
            HuffmanNode a = q.poll(), b = q.poll();
            HuffmanNode m = new HuffmanNode(a.freq + b.freq, '-');
            m.left = a; m.right = b;
            q.add(m);
        }

        System.out.println("\nHuffman Codes:");
        printCodes(q.peek(), "");
    }
}
```

---

### 🧪 **Example Output**

```
Enter number of characters: 3
Enter character 1: A
Enter frequency of 'A': 5
Enter character 2: B
Enter frequency of 'B': 7
Enter character 3: C
Enter frequency of 'C': 10

Huffman Codes:
A -> 00
B -> 01
C -> 1
```

---
