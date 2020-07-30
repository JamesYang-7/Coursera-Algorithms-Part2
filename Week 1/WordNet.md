'''java
import edu.princeton.cs.algs4.Digraph;
import edu.princeton.cs.algs4.In;
import edu.princeton.cs.algs4.StdIn;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.Topological;

import java.util.Map;
import java.util.HashMap;
import java.util.LinkedList;
import java.util.List;

public class WordNet {
    private final Map<String, List<Integer>> dict = new HashMap<>(); // a noun may have more than 1 id
    private final Map<Integer, String> reverseDict = new HashMap<>();
    private final Digraph graph;
    private final SAP sapGraph;

    // constructor takes the name of the two input files
    public WordNet(String synsets, String hypernyms) {
        if (synsets == null || hypernyms == null) {
            throw new IllegalArgumentException();
        }
        // read synsets
        In in = new In(synsets);
        int id = 0;
        int maxID = 0;
        String words = "";
        List<Integer> idList = null;
        while (in.hasNextLine()) {
            String[] segments = in.readLine().split(",");
            id = Integer.parseInt(segments[0]);
            words = segments[1]; // the second part contains the synset
            reverseDict.put(id, words);
            String[] arrayOfWords = words.split(" ");
            for (String s : arrayOfWords) {
                if (dict.containsKey(s)) {
                    idList = dict.get(s);
                }
                else {
                    idList = new LinkedList<>();
                    dict.put(s, idList);
                }
                idList.add(id);
            }
            maxID = Math.max(maxID, id);
        }
        graph = new Digraph(maxID + 1); // make a graph
        // read hypernyms
        String[] nums;
        int v = 0, w = 0;
        in = new In(hypernyms);
        while (in.hasNextLine()) {
            nums = in.readLine().split(",");
            v = Integer.parseInt(nums[0]);
            for (int i = 1; i < nums.length; ++i) {
                w = Integer.parseInt(nums[i]);
                graph.addEdge(v, w);
            }
        }
        if (!isRootedDAG()) {
            throw new IllegalArgumentException();
        }

        sapGraph = new SAP(graph);
    }
 
    // returns all WordNet nouns
    public Iterable<String> nouns() {
        return dict.keySet();
    }
 
    // is the word a WordNet noun?
    public boolean isNoun(String word) {
        if (word == null) {
            throw new IllegalArgumentException();
        }
        return dict.containsKey(word);
    }
 
    // distance between nounA and nounB (defined below)
    public int distance(String nounA, String nounB) {
        if (!isNoun(nounA) || !isNoun(nounB)) {
            throw new IllegalArgumentException();
        }
        return sapGraph.length(dict.get(nounA), dict.get(nounB));
    }
 
    // a synset (second field of synsets.txt) that is the common ancestor of nounA and nounB
    // in a shortest ancestral path (defined below)
    public String sap(String nounA, String nounB) {
        if (!isNoun(nounA) || !isNoun(nounB)) {
            throw new IllegalArgumentException();
        }
        int idAC = sapGraph.ancestor(dict.get(nounA), dict.get(nounB));
        return reverseDict.get(idAC);
    }

    private boolean isRootedDAG() {
        Topological tplgc = new Topological(graph); // use Topological (or DirectedCycle) as required
        if (!tplgc.hasOrder())
            return false;
        int rootCnt = 0;
        for (int i = 0; i < graph.V(); ++i) { // count roots
            if (!graph.adj(i).iterator().hasNext()) rootCnt += 1;
        }
        if (rootCnt != 1) 
            return false;
        return true;
    }
 
    // do unit testing of this class
    public static void main(String[] args) {
        WordNet wn = new WordNet(args[0], args[1]);
        while (!StdIn.isEmpty()) {
            String v = StdIn.readString();
            String w = StdIn.readString();
            int length   = wn.distance(v, w);
            String ancestor = wn.sap(v, w);
            StdOut.printf("distance = %d, ancestor = %s\n", length, ancestor);
        }
    }
 }
 ```
