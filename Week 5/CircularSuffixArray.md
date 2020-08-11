```java
// using Radixsort3way
public class CircularSuffixArray {
    private final int[] index;
    private final int N; // length of s
    private final String ss; // double s in order to make getting suffixes easier

    // circular suffix array of s
    public CircularSuffixArray(String s) {
        if (s == null) {
            throw new IllegalArgumentException();
        }
        N = s.length();
        index = new int[N];
        ss = s + s; // double s
        for (int i = 0; i < N; ++i) {
            index[i] = i;
        }
        sort();
    }

    // length of s
    public int length() {
        return N;
    }

    // returns index of ith sorted suffix
    public int index(int i) {
        if (i < 0 || i >= N) {
            throw new IllegalArgumentException();
        }
        return index[i];
    }

    private void sort() {
        sort(ss, 0, N - 1, 0);
    }
    // Radixsort3way
    // performance of LSD Radixsort is worse on short strings because it has to access count[] many times. But it's always better when strings are long
    // in order to pass all timing tests, We have to choose Radixsort3way instead of LSD
    private void sort(String s, int lo, int hi, int d) {
        if (hi <= lo) return;
        int lt = lo, gt = hi;
        char v = charAtSuffix(s, index[lo], d);
        int i = lo + 1;
        while (i <= gt) {
            char t = charAtSuffix(s, index[i], d);
            if (t < v) exch(lt++, i++);
            else if (t > v) exch(i, gt--);
            else ++i;
        }

        sort(s, lo, lt - 1, d);
        if (d < N) sort(s, lt, gt, d + 1); // the condition of going deeper can be specific on this case
        sort(s, gt + 1, hi, d);
    }

    private void exch(int i, int j) { // exchanging index is equivalent to exchanging suffix strings
        int t = index[i];
        index[i] = index[j];
        index[j] = t;
    }

    private char charAtSuffix(String s, int suffixBeginIndex, int d) {
        return s.charAt(suffixBeginIndex + d);
    }

    // unit testing (required)
    public static void main(String[] args) {
        String s = "ABRACADABRA!";
        CircularSuffixArray csa = new CircularSuffixArray(s);
        System.out.println("String: " + s);
        System.out.println("String length: " + s.length());
        System.out.println("csa length : " + csa.length());
        System.out.print("csa index array : ");
        for (int i = 0; i < s.length(); ++i) {
            System.out.print(" " + csa.index(i));
        }
        System.out.println();
    }
}
```