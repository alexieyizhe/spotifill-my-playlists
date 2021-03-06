__CS 241 |__ January 31, 2019

# More Finite Automata

### State Sets

Take this example of an NFA for $L = \{cab\} \cup \{\text{strings with even number of a's}\}$ where $\Sigma = \{a, b, c\}$.

![image](./assets/image.png)

If we want to **convert it into a DFA**, we assign each state a number like so:

![image (assets/image (1)-8947689.png)](./assets/image (1)-8947689.png)

and then we ask the following:

1. What states can I be in at the start?
2. From that state, where can I go on an input of:
   - `a`?
   - `b`?
   - `c`?
   - Do this for all reachable states
3. Our **accepting states** are those states with a state set that contains 1 or 6 (our accepting states in the NFA)

which will give us the starting state set of:

<svg width="800" height="300" version="1.1" xmlns="http://www.w3.org/2000/svg">
	<ellipse stroke="black" stroke-width="1" fill="none" cx="296.5" cy="148.5" rx="30" ry="30"/>
	<text x="281.5" y="154.5" font-family="Times New Roman" font-size="20">{1}</text>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="296.5" cy="148.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="377.5" cy="87.5" rx="30" ry="30"/>
	<text x="362.5" y="93.5" font-family="Times New Roman" font-size="20">{5}</text>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="500.5" cy="134.5" rx="30" ry="30"/>
	<text x="485.5" y="140.5" font-family="Times New Roman" font-size="20">{6}</text>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="500.5" cy="134.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="357.5" cy="235.5" rx="30" ry="30"/>
	<text x="332.5" y="241.5" font-family="Times New Roman" font-size="20">{2, 6}</text>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="357.5" cy="235.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="522.5" cy="235.5" rx="30" ry="30"/>
	<text x="497.5" y="241.5" font-family="Times New Roman" font-size="20">{3, 5}</text>
	<polygon stroke="black" stroke-width="1" points="236.5,148.5 266.5,148.5"/>
	<polygon fill="black" stroke-width="1" points="266.5,148.5 258.5,143.5 258.5,153.5"/>
	<polygon stroke="black" stroke-width="1" points="320.464,130.453 353.536,105.547"/>
	<polygon fill="black" stroke-width="1" points="353.536,105.547 344.137,106.366 350.153,114.354"/>
	<text x="342.5" y="138.5" font-family="Times New Roman" font-size="20">a</text>
	<polygon stroke="black" stroke-width="1" points="313.723,173.064 340.277,210.936"/>
	<polygon fill="black" stroke-width="1" points="340.277,210.936 339.778,201.516 331.59,207.256"/>
	<text x="312.5" y="211.5" font-family="Times New Roman" font-size="20">c</text>
	<polygon stroke="black" stroke-width="1" points="326.43,146.446 470.57,136.554"/>
	<polygon fill="black" stroke-width="1" points="470.57,136.554 462.247,132.113 462.932,142.09"/>
	<text x="394.5" y="163.5" font-family="Times New Roman" font-size="20">b</text>
	<polygon stroke="black" stroke-width="1" points="382.004,218.193 475.996,151.807"/>
	<polygon fill="black" stroke-width="1" points="475.996,151.807 466.577,152.338 472.346,160.507"/>
	<text x="434.5" y="205.5" font-family="Times New Roman" font-size="20">b, c</text>
	<polygon stroke="black" stroke-width="1" points="387.5,235.5 492.5,235.5"/>
	<polygon fill="black" stroke-width="1" points="492.5,235.5 484.5,230.5 484.5,240.5"/>
	<text x="435.5" y="256.5" font-family="Times New Roman" font-size="20">a</text>
</svg>

Note that each of the three states on the left have more connected paths and states, but the amount increases exponentially as we increase the number of states, since we have to iterate through each other possible state to go to.

==**Every DFA is an NFA.**==

==__Every NFA can be converted into a DFA.__==



### $\epsilon$- NFAs

Soemtimes, we want to change the state without consuming an input symbol. This allows us to move to another state  at any time, and it makes it easier for us to glue smaller automata together.

![image (assets/image (2)-8949287.png)](./assets/image (2)-8949287.png)

The above would be the simpler NFA for the very first NFA we defined above.

To **convert these $\epsilon$-NFAs to a DFA,** we ask similar questions as above, but we ask one additional one:

- Where can I go on an input of $x$ while also following any $\epsilon$'s before or after $x$?
- An **epsilon closure** is the set of states reachable from the current set of states (also an epsilon closure - default is the epsilon closure of the start state) through 0 (no epsilon, normal finite automata) or more epsilon transitions.

We can implement an $\epsilon$-NFA like so:

```pseudocode
states = epsilon_closure({q_0}) // epsilon closure is the states from the set that can be reached by following epsilon
while not EOF do
	ch <- read()
	states <- epsilon_closure(PRODUCT (q belonging to all states) of delta(q, ch))
end while
return states UNION A =/= 0
```



$\epsilon$-NFAs exactly recognize regular language. 

| regexp      | $\epsilon$-NFA                                               | DFA  |
| ----------- | ------------------------------------------------------------ | ---- |
| $\empty$    | ![image-20190131105929024](assets/image-20190131105929024.png) |      |
| $ \epsilon$ | ![image-20190131110017009](assets/image-20190131110017009.png) |      |
| $a$         | ![image-20190131110047914](assets/image-20190131110047914.png) |      |
| $E_1 | E_2$ | Connect a new start state to each start state of $E_1, E_2$ with an $\epsilon$ |      |
| $E_1E_2$    | Connect all of the accept states of $E_1$ to the start state of $E_2$ with $\epsilon$, and change the accept states to regular states. |      |
| $E^*$       | Create a new start state, make it accepting, connect it to the old one with an $\epsilon$, and connect accept states of $E$ to this new start state.<br />**We have to create a new state because the old start state may have other transitions back to it, and making it accepting could end up accepting unwanted strings.** |      |



### Scanning

When you are scanning a program to parse it as tokens, you are essentially traversing a finite automata:

<svg width="800" height="450" version="1.1" xmlns="http://www.w3.org/2000/svg">
	<ellipse stroke="black" stroke-width="1" fill="none" cx="295.5" cy="226.5" rx="30" ry="30"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="430.5" cy="184.5" rx="30" ry="30"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="430.5" cy="184.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="464.5" cy="308.5" rx="30" ry="30"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="464.5" cy="308.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="336.5" cy="392.5" rx="30" ry="30"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="336.5" cy="392.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="336.5" cy="100.5" rx="30" ry="30"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="336.5" cy="100.5" rx="24" ry="24"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="187.5" cy="381.5" rx="30" ry="30"/>
	<ellipse stroke="black" stroke-width="1" fill="none" cx="187.5" cy="381.5" rx="24" ry="24"/>
	<polygon stroke="black" stroke-width="1" points="278.349,251.114 204.651,356.886"/>
	<polygon fill="black" stroke-width="1" points="204.651,356.886 213.326,353.18 205.122,347.464"/>
	<text x="247.5" y="323.5" font-family="Times New Roman" font-size="20">register</text>
	<polygon stroke="black" stroke-width="1" points="302.693,255.625 329.307,363.375"/>
	<polygon fill="black" stroke-width="1" points="329.307,363.375 332.242,354.41 322.534,356.808"/>
	<text x="323.5" y="311.5" font-family="Times New Roman" font-size="20">keyword</text>
	<polygon stroke="black" stroke-width="1" points="322.491,239.596 437.509,295.404"/>
	<polygon fill="black" stroke-width="1" points="437.509,295.404 432.495,287.413 428.129,296.41"/>
	<text x="316.5" y="288.5" font-family="Times New Roman" font-size="20">comma</text>
	<polygon stroke="black" stroke-width="1" points="324.146,217.588 401.854,193.412"/>
	<polygon fill="black" stroke-width="1" points="401.854,193.412 392.73,191.014 395.701,200.563"/>
	<text x="366.5" y="226.5" font-family="Times New Roman" font-size="20">#</text>
	<polygon stroke="black" stroke-width="1" points="304.783,197.972 327.217,129.028"/>
	<polygon fill="black" stroke-width="1" points="327.217,129.028 319.987,135.088 329.496,138.182"/>
	<text x="269.5" y="162.5" font-family="Times New Roman" font-size="20">label</text>
	<path stroke="black" stroke-width="1" fill="none" d="M 268.792,213.403 A 66.487,66.487 0 0 1 307.199,95.372"/>
	<polygon fill="black" stroke-width="1" points="268.792,213.403 265.729,204.48 259.43,212.246"/>
	<text x="230.5" y="140.5" font-family="Times New Roman" font-size="20">&#949;</text>
	<path stroke="black" stroke-width="1" fill="none" d="M 167.524,359.286 A 94.699,94.699 0 0 1 267.742,215.454"/>
	<polygon fill="black" stroke-width="1" points="267.742,215.454 261.029,208.826 258.843,218.585"/>
	<text x="154.5" y="246.5" font-family="Times New Roman" font-size="20">&#949;</text>
	<path stroke="black" stroke-width="1" fill="none" d="M 314.319,203.265 A 106.017,106.017 0 0 1 401.821,176.042"/>
	<polygon fill="black" stroke-width="1" points="314.319,203.265 323.583,201.484 316.777,194.157"/>
	<text x="342.5" y="170.5" font-family="Times New Roman" font-size="20">&#949;</text>
	<path stroke="black" stroke-width="1" fill="none" d="M 325.467,226.431 A 184.046,184.046 0 0 1 446.009,284.919"/>
	<polygon fill="black" stroke-width="1" points="325.467,226.431 333.046,232.048 333.837,222.079"/>
	<text x="396.5" y="235.5" font-family="Times New Roman" font-size="20">&#949;</text>
</svg>

So, to parse a program written in `C`, we need to:

- Take in an input string $w$.
- Break $w$ into $w_1, w_2, .., w_n$ such that $w_i \in L$, where $L$ is the language of `C` tokens
- Output each $w_i$.

$L = \{\text{valid C tokens}\}$ is regular. $LL^* = \{\text{1 or more valid c tokens concatenated} \}$.