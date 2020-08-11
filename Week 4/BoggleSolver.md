```java
import edu.princeton.cs.algs4.In;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.TrieST;
/**
 * the solver itself is a trie (dictionary)
 */
public class BoggleSolver {
    private final String[] dict; 
    private final int[] scores; 
    private int row;
    private int col;
    private TrieST<Boolean> validWordTrie;
    private BoggleBoard board;
    private boolean[][] marked;
    

    // Initializes the data structure using the given array of strings as the dictionary.
    // (You can assume each word in the dictionary contains only the uppercase letters A through Z.)
    public BoggleSolver(String[] dictionary) {
        if (dictionary == null) {
            throw new IllegalArgumentException();
        }
        dict = dictionary; 
        scores = new int[dict.length]; 
        for (int i = 0; i < dictionary.length; ++i) {
            scores[i] = getScore(dict[i]); 
        }
    }

    // Returns the set of all valid words in the given Boggle board, as an Iterable.
    public Iterable<String> getAllValidWords(BoggleBoard xboard) {
        if (xboard == null) {
            throw new IllegalArgumentException();
        }
        board = xboard;
        row = board.rows();
        col = board.cols();
        validWordTrie = new TrieST<>();
        marked = new boolean[row][col];
        for (int i = 0; i < row; ++i) {
            for (int j = 0; j < col; ++j) {
                dfs(i, j, new StringBuilder(), 0, 0, dict.length - 1);
            }
        }
        return validWordTrie.keysWithPrefix("");
    }

    private void dfs(int i, int j, StringBuilder prefix, int d, int lo, int hi) {
        if (!validateCoord(i, j) || marked[i][j]) {
            return;
        }
        boolean isQ = false;
        char c = board.getLetter(i, j);
        if (c == 'Q') {
            isQ = true;
            while ((dict[lo].length() < d + 1 || dict[lo].charAt(d) < c) && lo <= hi) {
                ++lo;
            }
            while ((dict[hi].length() >= d + 1 && dict[hi].charAt(d) > c) && lo <= hi) {
                --hi;
            }
            if (lo > hi || dict[lo].charAt(d) > c || dict[hi].charAt(d) < c) {
                return;
            }
            d += 1;
            c = 'U';
        }
        while ((dict[lo].length() < d + 1 || dict[lo].charAt(d) < c) && lo <= hi) {
            ++lo;
        }
        while ((dict[hi].length() >= d + 1 && dict[hi].charAt(d) > c) && lo <= hi) {
            --hi;
        }
        if (lo > hi || dict[lo].charAt(d) > c || dict[hi].charAt(d) < c) {
            return;
        }
        marked[i][j] = true;
        if (isQ) {
            prefix.append('Q').append('U');
        }
        else {
            prefix.append(c);
        }
        String s = prefix.toString();
        // if (s.equals(dict[lo])) {
        if (d > 1 && dict[lo].length() == d + 1) {
            validWordTrie.put(s, true);
        }
        
        for (int di = -1; di <= 1; ++di) { // go deeper
            for (int dj = -1; dj <= 1; ++dj) {
                dfs(i + di, j + dj, prefix, d + 1, lo, hi);
            }
        }
        if (isQ) {
            prefix.delete(prefix.length() - 2, prefix.length());
        }
        else {
            prefix.deleteCharAt(prefix.length() - 1);
        }
        marked[i][j] = false;
    }

    private boolean validateCoord(int i, int j) { // check if the coordinate valid
        if (i >= 0 && i < row && j >= 0 && j < col) {
            return true;
        }
        return false;
    }

    // Returns the score of the given word if it is in the dictionary, zero otherwise.
    // (You can assume the word contains only the uppercase letters A through Z.)
    public int scoreOf(String word) {
        if (word == null) {
            throw new IllegalArgumentException();
        }
        int idx = contains(word);
        if (word.length() < 3 || idx < 0) {
            return 0;
        }
        return scores[idx];
    }

    private int contains(String word) {
        if (word == null) {
            throw new IllegalArgumentException();
        }
        int lo = 0;
        int hi = dict.length - 1;
        int mid, cmp;
        while (lo <= hi) {
            mid = lo + (hi - lo)/2;
            cmp = dict[mid].compareTo(word);
            if (cmp > 0) hi = mid - 1;
            else if (cmp < 0) lo = mid + 1;
            else return mid;
        }
        return -1;
    }

    private int getScore(String s) {
        int n = s.length();
        int score = 0;
        if (n < 3) {
            score = 0;
        }
        else if (n == 3 || n == 4) {
            score = 1;
        }
        else if (n == 5) {
            score = 2;
        }
        else if (n == 6) {
            score = 3;
        }
        else if (n == 7) {
            score = 5;
        }
        else {
            score = 11;
        }
        return score;
    }

    public static void main(String[] args) {
        In in = new In(args[0]);
        String[] dictionary = in.readAllStrings();
        BoggleSolver solver = new BoggleSolver(dictionary);
        BoggleBoard board = new BoggleBoard(args[1]);
        int score = 0;
        int entries = 0;
        for (String word : solver.getAllValidWords(board)) {
            // StdOut.println(word);
            ++entries;
            score += solver.scoreOf(word);
        }
        System.out.println("Entries = " + entries);
        StdOut.println("Score = " + score);
    }
}
```