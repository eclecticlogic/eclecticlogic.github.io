---
layout: post
title: Solving Wordament 
draft: true
author: Karthik Abram
tags: java game puzzle
---

_Statutory Disclaimer: This post is simply an exercise in algorithm development. It is not a warrant to attempt to cheat while playing Wordament. Any attempt to cheat the game will get you booted. The booting will be well deserved and this blog takes no responsibility for it._

### Game Time
[Wordament](http://www.wordament.com/) is a very popular word puzzle that is available on all the mobile app stores. The object of the game is to identify as many words as possible from a 4x4 grid of letters. Words are formed by sliding your finger on the letters. You can form words by moving in any direction as long as you don't repeat the letters. You get points for your words - longer words earn more points and there are special bonuses for uncommon words and words that contain letter sequences that are specifically called out. 

![Wordament for XBox](http://www.wordament.com/wp-content/uploads/2013/01/wordament_512x512.png)

A first time casual player will be simultaneously astounded at the number of words identified by the winners of any given round and embarrassed by their own dismal  performance. In fact, the winners are jaw-droppingly fast. Have a look at this video of a face-off (or is that word-off?):

<iframe width="560" height="420" allowfullscreen="allowfullscreen" src="http://www.youtube.com/embed/zYeCYS7hk-g?color=white&amp;theme=light"> </iframe>

Some of us speed-challenged folks, especially the geeks at heart may consider instead the challenge of algorithmically finding all the words. The problem is relatively straightforward and with adequate setup may even make a great interview question. If you do use this as an interview question, I would strongly suggest letting the candidate program on a computer with an IDE/language of their choice and not on a whiteboard.

### Word Bank

First we need a database of valid words. Here are a couple of sources to get such a list of words:

- [SIL English Wordlist](http://www-01.sil.org/linguistics/wordlists/english/)
- [SCOWL](http://wordlist.aspell.net/) dictionary

At a high-level, our task will be to navigate the grid adding letters to a sequence and testing if what we have is a valid word. Our challenge is to:

1. Ensure that we navigate all permutations of the letters.
2. Not visit any letter more than once. 
3. Determine if the sequence of letters we have is a valid word.

The last step is straightforward. We can simply load the entire list of valid words into a HashMap and check to see if the word we've assembled is valid. However, for our purpose, we are going to eschew the HashMap based approach and instead construct a tree of valid words such that we can more efficiently figure out if a word is valid or not. From a practical computational standpoint, this extra optimization is not going to make a material difference. However, the optimization does have academic value.

### From Word Bank to Word Tree

We will construct our word tree by creating nodes that have two properties:

1. A letter value.
1. An index of up to 26 child nodes, one per letter that can follow the current node. 
2. A flag that indicates if the path from the root to the current node represents a valid word.

We use two classes to construct the nodes, a `NodeLoader` and the `Node` itself. The code for these classes is shown below:

{% highlight java linenos %}

public class NodeLoader {

	
	public static Node init() throws IOException {
        // LineIterator and FileUtils are from apache commons io.
		LineIterator iterator = FileUtils.lineIterator(new File("src/main/resources/dict.txt"));
		
		Node root = new Node();
		root.setWord(false);
		
		while (iterator.hasNext()) {
			String line = iterator.next().toLowerCase().trim();
			createNodes(line, root, 0).setWord(true);
		}
		
		return root;
	}
	

	private static Node createNodes(String line, Node current, int index) {
		if (index != line.length()) {
			Node next = current.createOrGet(line.charAt(index));
			return createNodes(line, next, index+1);
		} else {
			return current;
		}
	}
}

{% endhighlight %}

Notice that root node does not represent a letter. In fact, the Nodes don't explicitly represent a letter. The letter representation is implicit from their position in the parent. For example, the sequence from the root node of the first child node, its second child node and its third child node represent the letter sequence 'abc'.

The Node is a simple class of mostly boilerplate code (please Oracle, if you are listening, at least add [Lombok](http://projectlombok.org/) style @Getter/@Setter annotations to Java 9?)

{% highlight java linenos %}

public class Node {

    private boolean word;

    private Node[] branches = new Node[26];


    public boolean isWord() {
        return word;
    }


    public void setWord(boolean word) {
        this.word = word;
    }


    public Node getNode(char c) {
        int i = c - 'a';
        return branches[i];
    }


    public Node createOrGet(char c) {
        Node n = getNode(c);
        if (n == null) {
            n = new Node();
            branches[c - 'a'] = n;
        }

        return n;
    }
}

{% endhighlight %}

### "Wording" it together

Armed with our word-tree, we can now write the grid input and traverse logic. For reading in the input, we will require the user to type in four lines of letters with each line containing 4 letters. We can then parse the letters and put them into a two-dimensional array:

{% highlight java linenos %}

        	Node root = NodeLoader.init();
        Scanner scanner = new Scanner(System.in);
        char grid[][] = new char[4][4];
        String parts[] = new String[4];

        parts[0] = scanner.nextLine();
        parts[1] = scanner.nextLine();
        parts[2] = scanner.nextLine();
        parts[3] = scanner.nextLine();

        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                grid[i][j] = parts[i].charAt(j);
            }
        }

{% endhighlight %} 

With the grid initialized we now have to traverse the grid. Our grid traversal must consider every location in the grid as a starting point. As we traverse the grid and identify valid words, we add it to a set (thus eliminating duplicates). Once the words have all been identified, we can sort them by length and print them. At a high level, assuming a "traverse" method, the code required is:

{% highlight java linenos %}

       for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 4; j++) {
                traverse(grid, new boolean[4][4], i, j, root, "");
            }
        }

        List<String> temp = new ArrayList<String>(wordsFound);
        Collections.sort(temp, new Comparator<String>() {

            public int compare(String o1, String o2) {
                return o2.length() - o1.length();
            }
        });
        for (String word : temp) {
            System.out.println(word);
        }

{% endhighlight %}

The variable `wordsFound` is simple HashSet. The only thing left to flush out is the traverse routine. Grid traversal is easily written as a recursive routine. Our job is to consider the current location, add the letter to the sequence by grabbing the appropriate child node of the current node (indexed by the letter) and seeing if the node represents a complete word. If it does, we add it to our list of valid words and then try to traverse by moving in each of the 8 different directions we can head in (left, right, up, down, diagonally to the top-right, top-left, bottom-right and bottom-left). 

{% highlight java linenos %}

/**
     * @param grid The grid of letters representing the game.
     * @param visited A grid of indicators telling us which letters have already been visited.
     * @param i Our current location in the grid. x-axis
     * @param j Our current location in the grid. y-axis
     * @param node The node we have just visited.
     * @param current The currently formulated word.
     */
    private static void traverse(char[][] grid, boolean[][] visited, int i, int j, Node node, String current) {
        char c = grid[i][j];
        String newStr = current + Character.toString(c);
        Node next = node.getNode(c);

        if (next != null) {
            visited[i][j] = true;
            if (next.isWord() && newStr.length() > 2) {
                wordsFound.add(newStr);
            }

            if (j < 3 && !visited[i][j + 1]) {
                traverse(grid, visited, i, j + 1, next, newStr);
            }
            if (i < 3 && !visited[i + 1][j]) {
                traverse(grid, visited, i + 1, j, next, newStr);
            }
            if (j > 0 && !visited[i][j - 1]) {
                traverse(grid, visited, i, j - 1, next, newStr);
            }
            if (i > 0 && !visited[i - 1][j]) {
                traverse(grid, visited, i - 1, j, next, newStr);
            }
            if (i < 3 && j < 3 && !visited[i + 1][j + 1]) {
                traverse(grid, visited, i + 1, j + 1, next, newStr);
            }
            if (i < 3 && j > 0 && !visited[i + 1][j - 1]) {
                traverse(grid, visited, i + 1, j - 1, next, newStr);
            }
            if (i > 0 && j > 0 && !visited[i - 1][j - 1]) {
                traverse(grid, visited, i - 1, j - 1, next, newStr);
            }
            if (i > 0 && j < 3 && !visited[i - 1][j + 1]) {
                traverse(grid, visited, i - 1, j + 1, next, newStr);
            }
            visited[i][j] = false;
        }

    }

{% endhighlight %}

Here is a sample run. The letters entered are:

|a|b|c|d|
---------
|e|f|g|h|
---------
|i|j|k|l|
---------
|m|n|o|p|

This gives us the following words:

- knife
- plonk
- glop
- mink
- jink
- fink
- fab
- ink
- fin
- lop


The complete code is available at [Github](https://github.com/eclecticlogic/wordament). The code comes with a plea to use it to further academic interest in such puzzle solvers and not to get cheap thrills from cheating in the real game.   