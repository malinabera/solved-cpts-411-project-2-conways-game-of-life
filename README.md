Download Link: https://assignmentchef.com/product/solved-cpts-411-project-2-conways-game-of-life
<br>
<h1></h1>

<u>Preamble:</u> You are allowed to work individually or in teams of size two each, for this project. Make only one full submission for each team project (including the Cover Sheet). The other person makes an “empty” submission with just the Cover Sheet (from the website) indicating what your team is and any other appropriate acknowledgements.

<h1>Project Description</h1>

In this project you will implement the <em>Conway’s Game of Life</em> in MPI. The game is a cellular automaton containing <em>m*n</em> cells arranged as <em>m</em> rows and <em>n</em> columns.

<u>Cell Neighborhood:</u> Since the automaton is structured as a grid (or mesh), each internal cell will contain exactly 8 neighbors: {N,S,E,W,NE,NW,SE,SW}. For cells that are on the boundary of the automaton (first row/column, last row/column), neighborhood will include wrapped around neighbors from the other end of the matrix – for instance, the first row will treat the last row as its preceding row, while the last column will treat the first column as its preceding column. The corner cells will use the corresponding cell in the diagonally opposite corner as substitute for their corresponding missing diagonal neighbor entries.

<u>Cell States and Generations:</u> Each cell can be in one of the two states at any given point of time – <em>alive</em> or <em>dead</em>. In the initialization step, we assign the state for each cell by a toss of a coin (i.e., random with equal probability).  After that, game simply keeps progressing from one “generation” to another without needing any further user input.

During the <em>k<sup>th</sup></em> generation, all cells determine their respective states based on the following simple <strong>rules</strong>:

<ol>

 <li>i) A living cell that has less than 3 living neighbors dies (as if caused by underpopulation, or loneliness, or simply boredom!); ii) A living cell with more than 5 living neighbors also dies (as if caused by overcrowding); iii)   A living cell with between 3 and 5 living neighbors lives on to the next generation; iv)      A dead cell will come (back) to life, if it has between 3 and 5 living neighbors.</li>

</ol>

Therefore, the decision to live or die obviously depends on the states of a cell’s neighbors and its own current state.

As a rule, we will use the states from <em>the</em> <em>previous </em>generation to make the decisions for the current generation. This implies that the states of cells in the current generation can be updated independent of one another and in any arbitrary order without affecting the output.

<em>Your task for this assignment is to write an efficient implementation for the Conway’s Game of Life in C/C++/MPI and analyze its scalability (both analytically and empirically). </em>For the purpose of this assignment, we will make the following assumptions:

<ul>

 <li>that the automaton’s matrix is always a <strong>square</strong> with <em>n</em> rows and <em>n</em> columns;</li>

 <li>that <em>n&gt;p</em> and <em>n</em> is divisible by <em>p</em> (the number of processes);</li>

 <li>that the number of generations to simulate is specified by the user at input. Let this parameter be denoted by <em>G</em>;</li>

</ul>

<strong>Parallel implementation: </strong>

There are multiple ways in which the Game of Life can be parallelized. One way is to decompose the input matrix into roughly equal sized smaller, nonoverlapping squares (or grids) such that each process owns a distinct, part of the matrix. <em>Another way is to decompose the matrix into disjoint batches of rows (or columns) such that each process gets n/p rows (or columns).</em> The latter approach, which is typically referred to as <em>block decomposition</em>, is better suited for implementation in distributed memory for these kinds of problems (why?). So we will use that approach in the implementation.   Note that in both approaches, each rank will get <em>n<sup>2</sup>/p</em> cells.

The pseudocode for your algorithm is as follows:

<u>Input:</u> User specifies parameters <em>n, G </em>

<u>Initialization:</u>

<ul>

 <li><strong><em>GenerateInitialGoL()</em></strong><a href="#_ftn1" name="_ftnref1"><em><sup><strong>[1]</strong></sup></em></a><em>:</em> This function should generate the initial matrix in parallel so that by the end of the call, the entire matrix is generated and stored in a distributed manner (with rank <em>i</em> generating and subsequently, also owning rows [i*(n/p) … (i+1)*(n/p)-1 ]. To do this, there are two steps:</li>

</ul>

o Rank 0 first generates <em>p</em> random numbers (in the interval of 1 and BIGPRIME<sup>2</sup>) and <strong>distributes</strong> them such that the <em>i<sup>th</sup></em> random number is handed over to rank <em>i</em>.  (Think of what MPI function you would use to do this distribution.) o Next, using the assigned random number as the “seed”, each rank (locally) generates a distinct sequence of <em>n<sup>2</sup>/p</em> random values, again in the same interval. Each generated random number is used in the following manner to fill the initial matrix:

if the <em>k<sup>th</sup></em> random number is even, then the <em>k<sup>th</sup></em> cell being filled in the local portion of the matrix is marked with status=Alive;  otherwise its state=Dead.

(This is one way to generate the matrix randomly. If you prefer to do it other ways, that’s fine too but make sure you use different random seeds in different processes to avoid the problem of generating matrix replicas across processes.) Also note that if you initialize this way, the initially filled matrix is not necessarily compliant with the rules of the GoL. That’s okay. You will fix it in the first iteration of your simulation. Read on…

<ul>

 <li><strong><em>Simulate()</em></strong><em>:</em> This function actually runs the Game of Life. The simulation is done for <em>G</em> generations (i.e., iterations). Within each iteration, rank <em>i</em> determines the new states for all the cell that it owns. To determine the new state of a cell, write a function called <em>DetermineState() </em>that takes a cell coordinate and returns its new state (alive or dead) based on the GoL rules described above. For this to happen, however, you need to first make sure all cell states have access to their neighboring entries – i.e., do the necessary communication to satisfy these dependencies <em>a priori</em>. You also need to ensure that all processes are executing the same generation at any given time. This can be done using a simple <em>MPI_Barrier</em> at the start of every generation.</li>

 <li><strong><em>DisplayGoL()</em></strong><em>:</em> As the simulation proceeds it is desirable to display the contents of the entire matrix to visualize the evolution of the cellular automaton. However, doing this after every step of simulation could become cumbersome for large <em>n</em>. Therefore, we will do this infrequently –</li>

</ul>

i.e., display once after every <em>x</em> number of generations. I will let you decide the value of <em>x</em> based on the timings you see for a specific input size. To do the display, write a function called <em>DisplayGoL()</em> which first aggregates (i.e., gathers) the entire matrix in rank 0 (“root”) and then displays it.  (Note, displaying from every separate rank independently is possible but could possibly shuffle up the prints out of order.) As an alternative to each process sending its entire local matrix to the root you could choose to send only the alive entries. This approach would provide some communication savings if the matrix is sparse with very few alive entries. However for simplicity if you want to just send the whole local matrix that is OK for this assignment.

<h1>Reporting</h1>

<ul>

 <li><u>Timing:</u>  Your code should measure the following times for reporting purposes:</li>

 <li>Total runtime (excluding the time for display) to run the simulation for the userspecified <em>G</em> generations</li>

 <li>Average time taken to execute a single generation (excluding time for display)</li>

 <li>Sum up the time for all communication steps (i.e., if you are using say <em>MPI_Barrier</em>, <em>MPI_Send, MPI_Recv</em>, <em>MPI_Gather</em>, <em>MPI_Scatter</em>, <em>MPI_Bcast</em>, <em>MPI_Reduce,</em>), select the maximum of that time across all processes, and report that as the “total communication time”.</li>

 <li>=&gt; “Total computation time” will then be (Total runtime – Total communication time).</li>

</ul>

These timings should be reported before the program terminates.

<h1>Experiments</h1>

To test the scalability of your code, devise the following experiments: Measure the average time per generation (excluding display time) as a function of the number of processes, for varying input sizes but a fixed number of generations (G). Think of building a table for this purpose, with rows for different input sizes (values of <em>n<sup>2</sup></em>) and columns for different number of processes (<em>p</em>). To change the input size, grow <em>n</em> in powers of 2, starting from an input that is as small as 4×4 and going as large as 2<sup>10</sup>x2<sup>10</sup> or more as dictated by your runtime. Vary the number of processes in powers of 2: e.g., {1,2,4,8,…, 64} in our cluster.

Using the above table, generate three plots:

<ol>

 <li>i) <strong>Speedup</strong> in the Average runtime per generation (Y-axis) vs. Number of processes (X-axis) – with different curves for different input sizes (nxn); ii) <strong>Efficiency</strong> curves for the above chart;</li>

</ol>

iii)        Plot the breakdown (in %) of the total runtime to indicate how much of it was spent in “Computation” (i.e., the time to update the local matrix and anything else that is not included any communication) vs.  “Communication” (i.e., the sum of the times spent in communicating primitives). Show this breakdown for varying number of processes for a fixed input size.

Please make sure you follow the templates provided in the lecture notes, to show your runtime/speedup/efficiency results: <a href="https://eecs.wsu.edu/%7Eananth/CptS411/Lectures/ParallelPerformance_example.pdf">https://eecs.wsu.edu/~ananth/CptS411/Lectures/ParallelPerformance_example.pdf</a>




<u>Interpretation:</u>  Briefly state your observations about your results – do they meet your analytical expectations? If not, why not? Do you see ways to optimize this further?

<strong>Deliverables (zipped into one zip file with your name(s) on it): </strong>

<ol>

 <li>Cover page mandatory</li>

 <li>Full source code inside a source_code folder</li>

</ol>

<ul>

 <li>Report in PDF (preferred) that shows all scaling results and your interpretation of those results.</li>

</ul>

<a href="#_ftnref1" name="_ftn1">[1]</a> Note that I have just mentioned the function names here. For consistency, I would like all of you to use the same function names. You are free to decide on the arguments as necessary in your implementation. <sup>2</sup> Some reasonably big prime number – e.g. 93563, 68111, etc.