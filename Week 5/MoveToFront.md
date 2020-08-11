```java
import edu.princeton.cs.algs4.BinaryStdIn;
import edu.princeton.cs.algs4.BinaryStdOut;

public class MoveToFront {
    private static final char R = 256;
    private static final char[] SEQUENCE = new char[R];

    private static void restore() {
        for (char ch = 0; ch < R; ++ch) {
            SEQUENCE[ch] = ch;
        }
    }

    // apply move-to-front encoding, reading from standard input and writing to standard output
    public static void encode() {
        restore();
        char c;
        while (!BinaryStdIn.isEmpty()) {
            c = BinaryStdIn.readChar(8);
            for (char i = 0; i < R; ++i) {
                if (SEQUENCE[i] == c) {
                    BinaryStdOut.write(i);
                    for (int j = i; j > 0; --j) {
                        SEQUENCE[j] = SEQUENCE[j - 1];
                    }
                    SEQUENCE[0] = c;
                    break;
                }
            }
        }
        BinaryStdOut.flush(); // has to flush
    }

    // apply move-to-front decoding, reading from standard input and writing to standard output
    public static void decode() {
        restore();
        char index;
        char c;
        while (!BinaryStdIn.isEmpty()) {
            index = BinaryStdIn.readChar(8);
            c = SEQUENCE[index];
            BinaryStdOut.write(c);
            for (int i = index; i > 0; --i) {
                SEQUENCE[i] = SEQUENCE[i - 1];
            }
            SEQUENCE[0] = c;
        }
        BinaryStdOut.flush();
    }

    // if args[0] is "-", apply move-to-front encoding
    // if args[0] is "+", apply move-to-front decoding
    public static void main(String[] args) {
        if (args[0].equals("-")) {
            encode();
        }
        else if (args[0].equals("+")) {
            decode();
        }
    }
}
```