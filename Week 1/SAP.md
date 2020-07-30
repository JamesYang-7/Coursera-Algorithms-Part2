```java
import edu.princeton.cs.algs4.Digraph;
import edu.princeton.cs.algs4.In;
import edu.princeton.cs.algs4.Queue;
import edu.princeton.cs.algs4.StdIn;
import edu.princeton.cs.algs4.StdOut;

public class SAP {
    private final Digraph graph;
    private final int nv;
    private int length; // use member variable to store the result
    private int ca;

    // constructor takes a digraph (not necessarily a DAG)
    public SAP(Digraph G) {
        if (G == null) {
            throw new IllegalArgumentException();
        }
        graph = new Digraph(G);
        nv = graph.V();
    }

    private void examinePoint(Integer v) {
        if (v == null || v < 0 || v >= nv) {
            throw new IllegalArgumentException();
        }
    }

    private void bfs(int v, int w) {
        examinePoint(v);
        examinePoint(w);
        length = -1;
        ca = -1;
        Queue<Integer> q = new Queue<>();
        boolean[] markedv = new boolean[nv];
        boolean[] markedw = new boolean[nv];
        int[] distv = new int[nv];
        int[] distw = new int[nv];
        int current = 0;

        q.enqueue(v);
        markedv[v] = true;
        distv[v] = 0;
        while (!q.isEmpty()) { // mark all ancestors of v
            current = q.dequeue();
            for (int next : graph.adj(current)) {
                if (!markedv[next]) { // if not visited, enqueue
                    q.enqueue(next);
                    markedv[next] = true;
                    distv[next] = distv[current] + 1;
                }
            }
        }

        q = new Queue<>();
        q.enqueue(w);
        markedw[w] = true;
        distw[w] = 0;
        while (!q.isEmpty()) { // find all common ancestors and check
            current = q.dequeue();
            if (markedv[current]) { // if it is a common ancestor, check whether it shortens the length
                int dist = distv[current] + distw[current];
                if (dist < length || length == -1) {
                    length = dist;
                    ca = current;
                }
            }
            for (int next : graph.adj(current)) {
                if (!markedw[next]) { // if not visited, enqueue
                    q.enqueue(next);
                    markedw[next] = true;
                    distw[next] = distw[current] + 1;
                }
            }
        }
    }
    private void bfs(Iterable<Integer> v, Iterable<Integer> w) {
        if (v == null || w == null) {
            throw new IllegalArgumentException();
        }
        length = -1;
        ca = -1;
        Queue<Integer> q = new Queue<>();
        boolean[] markedv = new boolean[nv];
        boolean[] markedw = new boolean[nv];
        int[] distv = new int[nv];
        int[] distw = new int[nv];
        int current = 0;
        
        for (Integer t : v) {
            examinePoint(t);
            q.enqueue(t);
            markedv[t] = true;
            distv[t] = 0;
        }
        while (!q.isEmpty()) { // mark all ancestors of v
            current = q.dequeue();
            for (int next : graph.adj(current)) {
                if (!markedv[next]) { // if not visited, enqueue
                    q.enqueue(next);
                    markedv[next] = true;
                    distv[next] = distv[current] + 1;
                }
            }
        }

        q = new Queue<>();
        for (Integer t : w) {
            examinePoint(t);
            q.enqueue(t);
            markedw[t] = true;
            distw[t] = 0;
        }
        while (!q.isEmpty()) { // find all common ancestors and check
            current = q.dequeue();
            if (markedv[current]) { // if it is a common ancestor, check whether it shortens the length
                int dist = distv[current] + distw[current];
                if (dist < length || length == -1) {
                    length = dist;
                    ca = current;
                }
            }
            for (int next : graph.adj(current)) {
                if (!markedw[next]) { // if not visited, enqueue
                    q.enqueue(next);
                    markedw[next] = true;
                    distw[next] = distw[current] + 1;
                }
            }
        }
    }

    // length of shortest ancestral path between v and w; -1 if no such path
    public int length(int v, int w) {
        bfs(v, w);
        return length;
    }
 
    // a common ancestor of v and w that participates in a shortest ancestral path; -1 if no such path
    public int ancestor(int v, int w) {
        bfs(v, w);
        return ca;
    }
 
    // length of shortest ancestral path between any vertex in v and any vertex in w; -1 if no such path
    public int length(Iterable<Integer> v, Iterable<Integer> w) {
        bfs(v, w);
        if (length >= 0) {
            return length;
        }
        return -1;
    }
 
    // a common ancestor that participates in shortest ancestral path; -1 if no such path
    public int ancestor(Iterable<Integer> v, Iterable<Integer> w) {
        bfs(v, w);
        if (ca >= 0) {
            return ca;
        }
        return -1;
    }
 
    // do unit testing of this class
    public static void main(String[] args) {
        In in = new In(args[0]);
        Digraph G = new Digraph(in);
        SAP sap = new SAP(G);
        while (!StdIn.isEmpty()) {
        int v = StdIn.readInt();
        int w = StdIn.readInt();
        int length   = sap.length(v, w);
        int ancestor = sap.ancestor(v, w);
        StdOut.printf("length = %d, ancestor = %d\n", length, ancestor);
        }
    }
 }
```
