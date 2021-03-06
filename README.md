<main class="col-md markdown-body">
<h1 id="tideman">Tideman</h1>

<p>Implement a program that runs a Tideman election, per the below.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>./tideman Alice Bob Charlie
Number of voters: 5
Rank 1: Alice
Rank 2: Charlie
Rank 3: Bob

Rank 1: Alice
Rank 2: Charlie
Rank 3: Bob

Rank 1: Bob
Rank 2: Charlie
Rank 3: Alice

Rank 1: Bob
Rank 2: Charlie
Rank 3: Alice

Rank 1: Charlie
Rank 2: Alice
Rank 3: Bob

Charlie
</code></pre></div></div>

<h2 id="background">Background</h2>

<p>You already know about plurality elections, which follow a very simple algorithm for determining the winner of an election: every voter gets one vote, and the candidate with the most votes wins.</p>

<p>But the plurality vote does have some disadvantages. What happens, for instance, in an election with three candidates, and the ballots below are cast?</p>

<p><img src="https://cs50.harvard.edu/x/2020/psets/3/fptp_ballot_1.png" alt="Five ballots, tie betweeen Alice and Bob"></p>

<p>A plurality vote would here declare a tie between Alice and Bob, since each has two votes. But is that the right outcome?</p>

<p>There’s another kind of voting system known as a ranked-choice voting system. In a ranked-choice system, voters can vote for more than one candidate. Instead of just voting for their top choice, they can rank the candidates in order of preference. The resulting ballots might therefore look like the below.</p>

<p><img src="https://cs50.harvard.edu/x/2020/psets/3/ranked_ballot_1.png" alt="Three ballots, with ranked preferences"></p>

<p>Here, each voter, in addition to specifying their first preference candidate, has also indicated their second and third choices. And now, what was previously a tied election could now have a winner. The race was originally tied between Alice and Bob. But the voter who chose Charlie preferred Alice over Bob, so Alice could here be declared the winner.</p>

<p>Ranked choice voting can also solve yet another potential drawback of plurality voting. Take a look at the following ballots.</p>

<p><img src="https://cs50.harvard.edu/x/2020/psets/3/condorcet_1.png" alt="Nine ballots, with ranked preferences"></p>

<p>Who should win this election? In a plurality vote where each voter chooses their first preference only, Charlie wins this election with four votes compared to only three for Bob and two for Alice. (Note that, if you’re familiar with the instant runoff voting system, Charlie wins here under that system as well). Alice, however, might reasonably make the argument that she should be the winner of the election instead of Charlie: after all, of the nine voters, a majority (five of them) preferred Alice over Charlie, so most people would be happier with Alice as the winner instead of Charlie.</p>

<p>Alice is, in this election, the so-called “Condorcet winner” of the election: the person who would have won any head-to-head matchup against another candidate. If the election had been just Alice and Bob, or just Alice and Charlie, Alice would have won.</p>

<p>The Tideman voting method (also known as “ranked pairs”) is a ranked-choice voting method that’s guaranteed to produce the Condorcet winner of the election if one exists.</p>

<p>Generally speaking, the Tideman method works by constructing a “graph” of candidates, where an arrow (i.e. edge) from candidate A to candidate B indicates that candidate A wins against candidate B in a head-to-head matchup. The graph for the above election, then, would look like the below.</p>

<p><img src="https://cs50.harvard.edu/x/2020/psets/3/condorcet_graph_1.png" alt="Nine ballots, with ranked preferences"></p>

<p>The arrow from Alice to Bob means that more voters prefer Alice to Bob (5 prefer Alice, 4 prefer Bob). Likewise, the other arrows mean that more voters prefer Alice to Charlie, and more voters prefer Charlie to Bob.</p>

<p>Looking at this graph, the Tideman method says the winner of the election should be the “source” of the graph (i.e. the candidate that has no arrow pointing at them). In this case, the source is Alice — Alice is the only one who has no arrow pointing at her, which means nobody is preferred head-to-head over Alice. Alice is thus declared the winner of the election.</p>

<p>It’s possible, however, that when the arrows are drawn, there is no Condorcet winner. Consider the below ballots.</p>

<p><img src="https://cs50.harvard.edu/x/2020/psets/3/no_condorcet_1.png" alt="Nine ballots, with ranked preferences"></p>

<p>Between Alice and Bob, Alice is preferred over Bob by a 7-2 margin. Between Bob and Charlie, Bob is preferred over Charlie by a 5-4 margin. But between Charlie and Alice, Charlie is preferred over Alice by a 6-3 margin. If we draw out the graph, there is no source! We have a cycle of candidates, where Alice beats Bob who beats Charlie who beats Alice (much like a game of rock-paper-scissors). In this case, it looks like there’s no way to pick a winner.</p>

<p>To handle this, the Tideman algorithm must be careful to avoid creating cycles in the candidate graph. How does it do this? The algorithm locks in the strongest edges first, since those are arguably the most significant. In particular, the Tideman algorithm specifies that matchup edges should be “locked in” to the graph one at a time, based on the “strength” of the victory (the more people who prefer a candidate over their opponent, the stronger the victory). So long as the edge can be locked into the graph without creating a cycle, the edge is added; otherwise, the edge is ignored.</p>

<p>How would this work in the case of the votes above? Well, the biggest margin of victory for a pair is Alice beating Bob, since 7 voters prefer Alice over Bob (no other head-to-head matchup has a winner preferred by more than 7 voters). So the Alice-Bob arrow is locked into the graph first.  The next biggest margin of victory is Charlie’s 6-3 victory over Alice, so that arrow is locked in next.</p>

<p>Next up is Bob’s 5-4 victory over Charlie. But notice: if we were to add an arrow from Bob to Charlie now, we would create a cycle! Since the graph can’t allow cycles, we should skip this edge, and not add it to the graph at all. If there were more arrows to consider, we would look to those next, but that was the last arrow, so the graph is complete.</p>

<p>This step-by-step process is shown below, with the final graph at right.</p>

<p><img src="https://cs50.harvard.edu/x/2020/psets/3/lockin.png" alt="Nine ballots, with ranked preferences"></p>

<p>Based on the resulting graph, Charlie is the source (there’s no arrow pointing towards Charlie), so Charlie is declared the winner of this election.</p>

<p>Put more formally, the Tideman voting method consists of three parts:</p>

<ul>
  <li data-marker="*"><strong>Tally</strong>: Once all of the voters have indicated all of their preferences, determine, for each pair of candidates, who the preferred candidate is and by what margin they are preferred.</li>
  <li data-marker="*"><strong>Sort</strong>: Sort the pairs of candidates in decreasing order of strength of victory, where strength of victory is defined to be the number of voters who prefer the preferred candidate.</li>
  <li data-marker="*"><strong>Lock</strong>: Starting with the strongest pair, go through the pairs of candidates in order and “lock in” each pair to the candidate graph, so long as locking in that pair does not create a cycle in the graph.</li>
</ul>

<p>Once the graph is complete, the source of the graph (the one with no edges pointing towards it) is the winner!</p>

<h2 id="getting-started">Getting Started</h2>

<p>Here’s how to download this problem’s “distribution code” (i.e., starter code) into your own CS50 IDE. Log into <a href="https://ide.cs50.io/">CS50 IDE</a> and then, in a terminal window, execute each of the below.</p>

<ul>
  <li data-marker="*">Execute <code class="highlighter-rouge">cd</code> to ensure that you’re in <code class="highlighter-rouge">~/</code> (i.e., your home directory).</li>
  <li data-marker="*">Execute <code class="highlighter-rouge">cd pset3</code> to change into (i.e., open) your <code class="highlighter-rouge">pset3</code> directory that should already exist.</li>
  <li data-marker="*">Execute <code class="highlighter-rouge">mkdir tideman</code> to make (i.e., create) a directory called <code class="highlighter-rouge">tideman</code> in your <code class="highlighter-rouge">pset3</code> directory.</li>
  <li data-marker="*">Execute <code class="highlighter-rouge">cd tideman</code> to change into (i.e., open) that directory.</li>
  <li data-marker="*">Execute <code class="highlighter-rouge">wget https://cdn.cs50.net/2019/fall/psets/3/tideman/tideman.c</code> to download this problem’s distribution code.</li>
  <li data-marker="*">Execute <code class="highlighter-rouge">ls</code>. You should see this problem’s distribution code, in a file called <code class="highlighter-rouge">tideman.c</code>.</li>
</ul>

<h2 id="understanding">Understanding</h2>

<p>Let’s open up <code class="highlighter-rouge">tideman.c</code> to take a look at what’s already there.</p>

<p>First, notice the two-dimensional array <code class="highlighter-rouge">preferences</code>. The integer <code class="highlighter-rouge">preferences[i][j]</code> will represent the number of voters who prefer candidate <code class="highlighter-rouge">i</code> over candidate <code class="highlighter-rouge">j</code>.</p>

<p>The file also defines another two-dimensional array, called <code class="highlighter-rouge">locked</code>, which will represent the candidate graph. <code class="highlighter-rouge">locked</code> is a boolean array, so <code class="highlighter-rouge">locked[i][j]</code> being <code class="highlighter-rouge">true</code> represents the existence of an edge pointing from candidate <code class="highlighter-rouge">i</code> to candidate <code class="highlighter-rouge">j</code>; <code class="highlighter-rouge">false</code> means there is no edge. (If curious, this representation of a graph is known as an “adjacency matrix”).</p>

<p>Next up is a <code class="highlighter-rouge">struct</code> called <code class="highlighter-rouge">pair</code>, used to represent a pair of candidates: each pair includes the <code class="highlighter-rouge">winner</code>’s candidate index and the <code class="highlighter-rouge">loser</code>’s candidate index.</p>

<p>The candidates themselves are stored in the array <code class="highlighter-rouge">candidates</code>, which is an array of <code class="highlighter-rouge">string</code>s representing the names of each of the candidates. There’s also an array of <code class="highlighter-rouge">pairs</code>, which will represent all of the pairs of candidates (for which one is preferred over the other) in the election.</p>

<p>The program also has two global variables: <code class="highlighter-rouge">pair_count</code> and <code class="highlighter-rouge">candidate_count</code>, representing the number of pairs and number of candidates in the arrays <code class="highlighter-rouge">pairs</code> and <code class="highlighter-rouge">candidates</code>, respectively.</p>

<p>Now onto <code class="highlighter-rouge">main</code>. Notice that after determining the number of candidates, the program loops through the <code class="highlighter-rouge">locked</code> graph and initially sets all of the values to <code class="highlighter-rouge">false</code>, which means our initial graph will have no edges in it.</p>

<p>Next, the program loops over all of the voters and collects their preferences in an array called <code class="highlighter-rouge">ranks</code> (via a call to <code class="highlighter-rouge">vote</code>), where <code class="highlighter-rouge">ranks[i]</code> is the index of the candidate who is the <code class="highlighter-rouge">i</code>th preference for the voter. These ranks are passed into the <code class="highlighter-rouge">record_preference</code> function, whose job it is to take those ranks and update the global <code class="highlighter-rouge">preferences</code> variable.</p>

<p>Once all of the votes are in, the pairs of candidates are added to the <code class="highlighter-rouge">pairs</code> array via a called to <code class="highlighter-rouge">add_pairs</code>, sorted via a call to <code class="highlighter-rouge">sort_pairs</code>, and locked into the graph via a call to <code class="highlighter-rouge">lock_pairs</code>. Finally, <code class="highlighter-rouge">print_winner</code> is called to print out the name of the election’s winner!</p>

<p>Further down in the file, you’ll see that the functions <code class="highlighter-rouge">vote</code>, <code class="highlighter-rouge">record_preference</code>, <code class="highlighter-rouge">add_pairs</code>,<code class="highlighter-rouge">sort_pairs</code>, <code class="highlighter-rouge">lock_pairs</code>, and <code class="highlighter-rouge">print_winner</code> are left blank. That’s up to you!</p>

<h2 id="specification">Specification</h2>

<p>Complete the implementation of <code class="highlighter-rouge">tideman.c</code> in such a way that it simulates a Tideman election.</p>

<ul>
  <li data-marker="*">Complete the <code class="highlighter-rouge">vote</code> function.
    <ul>
      <li data-marker="*">The function takes arguments <code class="highlighter-rouge">rank</code>, <code class="highlighter-rouge">name</code>, and <code class="highlighter-rouge">ranks</code>. If <code class="highlighter-rouge">name</code> is a match for the name of a valid candidate, then you should update the <code class="highlighter-rouge">ranks</code> array to indicate that the voter has the candidate as their <code class="highlighter-rouge">rank</code> preference (where <code class="highlighter-rouge">0</code> is the first preference, <code class="highlighter-rouge">1</code> is the second preference, etc.)</li>
      <li data-marker="*">Recall that <code class="highlighter-rouge">ranks[i]</code> here represents the user’s <code class="highlighter-rouge">i</code>th preference.</li>
      <li data-marker="*">The function should return <code class="highlighter-rouge">true</code> if the rank was successfully recorded, and <code class="highlighter-rouge">false</code> otherwise (if, for instance, <code class="highlighter-rouge">name</code> is not the name of one of the candidates).</li>
      <li data-marker="*">You may assume that no two candidates will have the same name.</li>
    </ul>
  </li>
  <li data-marker="*">Complete the <code class="highlighter-rouge">record_preferences</code> function.
    <ul>
      <li data-marker="*">The function is called once for each voter, and takes as argument the <code class="highlighter-rouge">ranks</code> array, (recall that <code class="highlighter-rouge">ranks[i]</code> is the voter’s <code class="highlighter-rouge">i</code>th preference, where <code class="highlighter-rouge">ranks[0]</code> is the first preference).</li>
      <li data-marker="*">The function should update the global <code class="highlighter-rouge">preferences</code> array to add the current voter’s preferences. Recall that <code class="highlighter-rouge">preferences[i][j]</code> should represent the number of voters who prefer candidate <code class="highlighter-rouge">i</code> over candidate <code class="highlighter-rouge">j</code>.</li>
      <li data-marker="*">You may assume that every voter will rank each of the candidates.</li>
    </ul>
  </li>
  <li data-marker="*">Complete the <code class="highlighter-rouge">add_pairs</code> function.
    <ul>
      <li data-marker="*">The function should add all pairs of candidates where one candidate is preferred to the <code class="highlighter-rouge">pairs</code> array. A pair of candidates who are tied (one is not preferred over the other) should not be added to the array.</li>
      <li data-marker="*">The function should update the global variable <code class="highlighter-rouge">pair_count</code> to be the number of pairs of candidates. (The pairs should thus all be stored between <code class="highlighter-rouge">pairs[0]</code> and <code class="highlighter-rouge">pairs[pair_count - 1]</code>, inclusive).</li>
    </ul>
  </li>
  <li data-marker="*">Complete the <code class="highlighter-rouge">sort_pairs</code> function.
    <ul>
      <li data-marker="*">The function should sort the <code class="highlighter-rouge">pairs</code> array in decreasing order of strength of victory, where strength of victory is defined to be the number of voters who prefer the preferred candidate. If multiple pairs have the same strength of victory, you may assume that the order does not matter.</li>
    </ul>
  </li>
  <li data-marker="*">Complete the <code class="highlighter-rouge">lock_pairs</code> function.
    <ul>
      <li data-marker="*">The function should create the <code class="highlighter-rouge">locked</code> graph, adding all edges in decreasing order of victory strength so long as the edge would not create a cycle.</li>
    </ul>
  </li>
  <li data-marker="*">Complete the <code class="highlighter-rouge">print_winner</code> function.
    <ul>
      <li data-marker="*">The function should print out the name of the candidate who is the source of the graph. You may assume there will not be more than one source.</li>
    </ul>
  </li>
</ul>

<p>You should not modify anything else in <code class="highlighter-rouge">tideman.c</code> other than the implementations of the <code class="highlighter-rouge">vote</code>, <code class="highlighter-rouge">record_preferences</code>, <code class="highlighter-rouge">add_pairs</code>, <code class="highlighter-rouge">sort_pairs</code>, <code class="highlighter-rouge">lock_pairs</code>, and <code class="highlighter-rouge">print_winner</code> functions (and the inclusion of additional header files, if you’d like). You are permitted to add additional functions to <code class="highlighter-rouge">tideman.c</code>, so long as you do not change the declarations of any of the existing functions.</p>

<h2 id="usage">Usage</h2>

<p>Your program should behave per the example below:</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>./tideman Alice Bob Charlie
Number of voters: 5
Rank 1: Alice
Rank 2: Charlie
Rank 3: Bob

Rank 1: Alice
Rank 2: Charlie
Rank 3: Bob

Rank 1: Bob
Rank 2: Charlie
Rank 3: Alice

Rank 1: Bob
Rank 2: Charlie
Rank 3: Alice

Rank 1: Charlie
Rank 2: Alice
Rank 3: Bob

Charlie
</code></pre></div></div>

<h2 id="testing">Testing</h2>

<p>Be sure to test your code to make sure it handles…</p>

<ul>
  <li data-marker="*">An election with any number of candidate (up to the <code class="highlighter-rouge">MAX</code> of <code class="highlighter-rouge">9</code>)</li>
  <li data-marker="*">Voting for a candidate by name</li>
  <li data-marker="*">Invalid votes for candidates who are not on the ballot</li>
  <li data-marker="*">Printing the winner of the election</li>
</ul>

<p>Execute the below to evaluate the correctness of your code using <code class="highlighter-rouge">check50</code>. But be sure to compile and test it yourself as well!</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>check50 cs50/problems/2020/x/tideman
</code></pre></div></div>

<p>Execute the below to evaluate the style of your code using <code class="highlighter-rouge">style50</code>.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>style50 tideman.c
</code></pre></div></div>

<h2 id="how-to-submit">How to Submit</h2>

<p>Execute the below, logging in with your GitHub username and password when prompted. For security, you’ll see asterisks (<code class="highlighter-rouge">*</code>) instead of the actual characters in your password.</p>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code>submit50 cs50/problems/2020/x/tideman
</code></pre></div></div>


</main>
