```java
import edu.princeton.cs.algs4.BinaryStdIn;
import edu.princeton.cs.algs4.BinaryStdOut;

public class BurrowsWheeler {
    // apply Burrows-Wheeler transform,
    // reading from standard input and writing to standard output 
    public static void transform() {
        String s = BinaryStdIn.readString();
        int first = 0;
        int n = s.length();
        char[] t = new char[n];
        CircularSuffixArray csa = new CircularSuffixArray(s);
        
        for (int i = 0; i < n; ++i) {
            if (csa.index(i) == 0) {
                first = i;
                t[i] = s.charAt(n - 1);
            }
            else {
                t[i] = s.charAt(csa.index(i) - 1);
            }
        }
        BinaryStdOut.write(first);
        for (int i = 0; i < n; ++i) {
            BinaryStdOut.write(t[i]);
        }
        BinaryStdOut.flush(); // has to flush
    }
    
    // apply Burrows-Wheeler inverse transform,
    // reading from standard input and writing to standard output
    public static void inverseTransform() {
        StringBuilder builder = new StringBuilder();
        int first = BinaryStdIn.readInt();
        while (!BinaryStdIn.isEmpty()) {
            builder.append(BinaryStdIn.readChar());
        }
        int n = builder.length(); 
        char[] t = new char[n]; // tail
        char[] h = new char[n]; // head
        int[] next = new int[n];
        t = builder.toString().toCharArray();

        // use radix sort to determine next[]
        int R = 256;
        int[] count = new int[R + 1];
        for (int i = 0; i < n; ++i)
            ++count[t[i] + 1];
        for (int r = 0; r < R; ++r)
            count[r + 1] += count[r];
        for (int i = 0; i < n; ++i) {
            next[count[t[i]]] = i;
            h[count[t[i]]++] = t[i];
        }

        BinaryStdOut.write(h[first]);
        int cnt = 1;
        for (int i = next[first]; i != first; i = next[i]) {
            BinaryStdOut.write(h[i]);
            ++cnt;
        }
        if (cnt < n) { // special case: if periodic
            int i = first;
            while (cnt < n) {
                BinaryStdOut.write(h[i]);
                i = next[i];
                ++cnt;
            }
        }
        BinaryStdOut.flush(); 
    }

    // if args[0] is "-", apply Burrows-Wheeler transform
    // if args[0] is "+", apply Burrows-Wheeler inverse transform
    public static void main(String[] args) {
        if (args[0].equals("-")) {
            transform();
        }
        else if (args[0].equals("+")) {
            inverseTransform();
        }
    }
}
```
