```java
import java.util.HashMap;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;

import edu.princeton.cs.algs4.FlowEdge;
import edu.princeton.cs.algs4.FlowNetwork;
import edu.princeton.cs.algs4.FordFulkerson;
import edu.princeton.cs.algs4.In;
import edu.princeton.cs.algs4.Queue;
import edu.princeton.cs.algs4.StdOut;

public class BaseballElimination {
    private final int n;
    private final Map<String, Integer> teamIndex;
    private final String[] teams;
    private final int[] w;
    private final int[] l;
    private final int[] r;
    private final int[][] g;
    private final int s;
    private final int t;
    private final int[] vteam;
    private boolean isLastEliminated;
    private String lastProcessedTeam;
    private List<String> certificateOfElimination;

    // create a baseball division from given filename in format specified below
    public BaseballElimination(String filename) {
        In in = new In(filename);
        n = in.readInt();
        teamIndex = new HashMap<>();
        String teamName = "";
        teams = new String[n];
        w = new int[n];
        l = new int[n];
        r = new int[n];
        g = new int[n][n];
        s = n*n + 1;
        t = n*n + 2;
        vteam = new int[n];
        lastProcessedTeam = null;
        certificateOfElimination = null;

        for (int i = 0; i < n; ++i) {
            teamName = in.readString();
            teamIndex.put(teamName, i);
            teams[i] = teamName;
            w[i] = in.readInt();
            l[i] = in.readInt();
            r[i] = in.readInt();
            vteam[i] = i*n + i;
            for (int j = 0; j < n; ++j) {
                g[i][j] = in.readInt();
            } 
        }
    }

    // number of teams              
    public int numberOfTeams()  {
        return n;
    }

    // all teams                      
    public Iterable<String> teams() {
        return teamIndex.keySet();
    }

    // number of wins for given team                      
    public int wins(String team) {
        validateTeam(team);
        return w[teamIndex.get(team)];
    }   

    // number of losses for given team         
    public int losses(String team) {
        validateTeam(team);
        return l[teamIndex.get(team)];
    }

    // number of remaining games for given team             
    public int remaining(String team) {
        validateTeam(team);
        return r[teamIndex.get(team)];
    }       

    // number of remaining games between team1 and team2
    public int against(String team1, String team2) {
        validateTeam(team1);
        validateTeam(team2);
        return g[teamIndex.get(team1)][teamIndex.get(team2)];
    }

    // is given team eliminated?
    public boolean isEliminated(String team) {
        validateTeam(team);
        if (!team.equals(lastProcessedTeam)) {
            constructFF(team);
        }
        return isLastEliminated;
    }

    // subset R of teams that eliminates given team; null if not eliminated
    public Iterable<String> certificateOfElimination(String team) {
        validateTeam(team);
        if (isEliminated(team)) {
            return new LinkedList<>(certificateOfElimination);
        }
        else return null;
    }

    private void constructFF(String team) {
        isLastEliminated = false;
        certificateOfElimination = new LinkedList<>();
        lastProcessedTeam = team;
        FlowNetwork graph = new FlowNetwork(n*n + 3);
        int teamNum = teamIndex.get(team);
        int maxWin = w[teamNum] + r[teamNum];
        int vmatch = 0;
        boolean[] isVisited = new boolean[n*n + 3];

        // check for trivial elimination
        for (int i = 0; i < n; ++i) {
            if (i == teamNum)
                continue;
            if (w[i] > maxWin) {
                isLastEliminated = true;
                certificateOfElimination.add(teams[i]);
            }
        }
        if (isLastEliminated) {
            return;
        }

        // check for nontrivial elimination
        for (int i = 0; i < n; ++i) { // construct the graph
            if (i == teamNum) // no need to add the team being questioned
                continue;
            for (int j = i + 1; j < n; ++j) {
                if (j == teamNum) // no need to add the team being questioned
                    continue;
                vmatch = getVMatch(i, j); // get vertex of match
                FlowEdge eL = new FlowEdge(s, vmatch, g[i][j]); // edge points from s to a match
                FlowEdge eM1 = new FlowEdge(vmatch, vteam[i], Double.POSITIVE_INFINITY); // edges point from a match to teams
                FlowEdge eM2 = new FlowEdge(vmatch, vteam[j], Double.POSITIVE_INFINITY);
                graph.addEdge(eL);
                graph.addEdge(eM1);
                graph.addEdge(eM2);
            }
            FlowEdge eR = new FlowEdge(vteam[i], t, maxWin - w[i]); // edge points from a team to t
            graph.addEdge(eR);
        }
        FordFulkerson FF = new FordFulkerson(graph, s, t); // get the max flow

        // get certificateOfElimination
        Queue<Integer> q = new Queue<>();
        q.enqueue(s);
        isVisited[s] = true;
        int head, other;
        while (!q.isEmpty()) { // a little bit complicated...but it works
            head = q.dequeue();
            for (FlowEdge e : graph.adj(head)) {
                other = e.other(head);
                if (!isVisited[other] && e.residualCapacityTo(other) > 0) {
                    q.enqueue(other);
                    isVisited[other] = true;
                    if (isVTeam(other)) {
                        certificateOfElimination.add(teams[vToTeam(other)]);
                    }
                }
            }
        }
        if (!certificateOfElimination.isEmpty()) {
            isLastEliminated = true;
        }
        else {
            certificateOfElimination = null;
        }

    }

    // validate whether it's an existing team
    private void validateTeam(String team) {
        if (team == null || !teamIndex.containsKey(team)) {
            throw new IllegalArgumentException();
        }
    }

    // return the vertex which represents the games between team1 and team2
    private int getVMatch(int team1, int team2) {
        if (team1 > team2) { // in order to eliminate the influece of numerical order, we have to make sure team1 < team2
            int team = team1;
            team1 = team2;
            team2 = team;
        }
        return team1*n + team2;
    }

    // check is the vertex a team vertex
    private boolean isVTeam(int v) {
        int field1 = v/n;
        int field2 = v % n;
        if (field1 >= 0 && field1 < n && field1 == field2)
            return true;
        return false;
    }

    // convert a team vertex to team id (array index)
    private int vToTeam(int v) {
        return v/n;
    }

    public static void main(String[] args) {
        BaseballElimination division = new BaseballElimination(args[0]);
        for (String team : division.teams()) {
            if (division.isEliminated(team)) {
                StdOut.print(team + " is eliminated by the subset R = { ");
                for (String t : division.certificateOfElimination(team)) {
                    StdOut.print(t + " ");
                }
                StdOut.println("}");
            }
            else {
                StdOut.println(team + " is not eliminated");
            }
        }
    } 
}
```
