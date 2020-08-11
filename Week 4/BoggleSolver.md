```java
import edu.princeton.cs.algs4.In;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.TrieST;
/**
 * the solver itself is a trie (dictionary)
 */
public class BoggleSolver {
    private static final int R = 26; // number of uppercase characters
    private int row;
    private int col;
    private Node root;      // root of trie
    private TrieST<Boolean> validWordTrie;
    private BoggleBoard board;
    private boolean[][] marked;
    

    // Initializes the data structure using the given array of strings as the dictionary.
    // (You can assume each word in the dictionary contains only the uppercase letters A through Z.)
    public BoggleSolver(String[] dictionary) {
        if (dictionary == null) {
            throw new IllegalArgumentException();
        }
        for (int i = 0; i < dictionary.length; ++i) {
            put(dictionary[i], getScore(dictionary[i]));
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
                dfs(root, i, j, new StringBuilder());
            }
        }
        return validWordTrie.keysWithPrefix("");
    }

    private void dfs(Node pre, int i, int j, StringBuilder prefix) {
        if (!validateCoord(i, j) || marked[i][j]) {
            return;
        }
        char c = board.getLetter(i, j);
        Node x = pre.next[c - 'A'];
        if (x == null) { // if the dictionary doesn't contain such a prefix, then there is no need to go forward
            return;
        }
        if (c == 'Q') { // special case 'Q'
            x = x.next['U' - 'A'];
            if (x == null) {
                return;
            }
            prefix.append(c).append('U');
        }
        else {
            prefix.append(c);
        }
        marked[i][j] = true;
        String s = prefix.toString();
        if (contains(s)) {
            validWordTrie.put(s, true);
        }

        for (int di = -1; di <= 1; ++di) { // go deeper
            for (int dj = -1; dj <= 1; ++dj) {
                dfs(x, i + di, j + dj, prefix);
            }
        }

        // restore StringBuilder and mark
        if (c == 'Q') {
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
        Node x = getNode(word);
        if (x == null || x.val == 0) {
            return 0;
        }
        return x.val;
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

    private static class Node { // trie
        private int val;
        private Node[] next = new Node[R];
    }

    private int get(String key) { // trie
        if (key == null) throw new IllegalArgumentException();
        Node x = get(root, key, 0);
        if (x == null) return 0;
        return x.val;
    }

    private Node getNode(String key) { // trie
        return get(root, key, 0);
    }

    private boolean contains(String key) { // trie
        if (key == null) throw new IllegalArgumentException();
        return get(key) != 0;
    }

    private Node get(Node x, String key, int d) { // trie
        if (x == null) return null;
        if (d == key.length()) return x;
        int c = key.charAt(d) - 'A';
        return get(x.next[c], key, d+1);
    }

    private void put(String key, int val) { // trie
        if (key == null) throw new IllegalArgumentException();
        root = put(root, key, val, 0);
    }

    private Node put(Node x, String key, int val, int d) { // trie
        if (x == null) x = new Node();
        if (d == key.length()) {
            x.val = val;
            return x;
        }
        int c = key.charAt(d) - 'A';
        x.next[c] = put(x.next[c], key, val, d+1);
        return x;
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