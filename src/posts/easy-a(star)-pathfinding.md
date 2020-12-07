---
title: Easy A* (star) Pathfinding
date: '2020-12-07'
tags:
  - algorithms
  - data structures
  - programming
---
Today we’ll be going over the A* pathfinding algorithm, how it works, and its implementation in Python 🐍.

- - -

If you’re a game developer, you might have always wanted to implement A* as character (or enemy) pathfinding. I know a couple of years ago I did, but with my level of programming at the time I had issues with the current articles out there. I wanted to write this as an easy introduction with a clear source code example to A* pathfinding so anyone can easily pick it up and use it in their game.

One important aspect of A* is `f=g+h`. The f, g, and h variables are in our Node class and get calculated every time we create a new node. Quickly I’ll go over what these variables mean.

* F is the total cost of the node.
* G is the distance between the current node and the start node.
* H is the heuristic — estimated distance from the current node to the end node.

Let’s take a look at a quick graphic to help illustrate this.

![Graphic to help illustrate this](https://firebasestorage.googleapis.com/v0/b/dakiya-23d2c.appspot.com/o/A_Star%20Algo.svg?alt=media&token=bb86a441-66aa-4628-97d9-b3fe528c9ece)

Awesome! Let’s say `node(0)` is our starting position and `node(19)` is our end position. Let’s also say that our current node is at the the red square `node(4)`.

### G
> G is the distance between the current node and the start node.

If we count back we can see that `node(4)` is 4 spaces away from our start node.
We can also say that G is 1 more than our parent node (`node(3)`). So in this case for `node(4)`, `currentNode.g = 4`.

### H
> H is the heuristic — estimated distance from the current node to the end node.

So let’s calculate the distance. If we take a look we’ll see that if we go over 7 spaces and up 3 spaces, we’ve reached our end node (`node(19)`).

Let’s apply the Pythagorean Theorem! a² + b² = c². After we’ve applied this, we’ll see that `currentNode.h = 7² + 3²`. Or `currentNode.h = 58`.

But don’t we have to apply the square root to 58? Nope! We can skip that calculation on every node and still get the same output. Clever!

With a heuristic, we need to make sure that we can actually calculate it. It’s also very important that the heuristic is always an underestimation of the total path, as an overestimation will lead to A* searching for through nodes that may not be the ‘best’ in terms of f value.

### F
> F is the total cost of the node.

So let’s add up h and g to get the total cost of our node. `currentNode.f = currentNode.g + currentNode.h`. Or `currentNode.f = 4+ 58`. Or `currentNode.f = 62`.

## Time to use `f=g+h`
Alright, so that was a lot of work. Now with all that work, what am I going to use this f value for?

With this new f value, we can look at all our nodes and say, “Hey, is this the best node I can pick to move forward with right now?”. Rather than running through every node, we can pick the ones that have the best chance of getting us to our goal.

## A⭐️ Method Steps — by Patrick Lester
I’ve pasted the steps for A* from Patrick Lester’s article that you can check out here. The same website is also listed below in resources. This is an insanely good explanation, and is why I decided to go with it rather than writing it again.
### a) Add the starting square (or node) to the open list.
### b) Repeat the following:
1. Look for the lowest F cost square on the open list. We refer to this as the current square.
2. Switch it to the closed list.
3. For each of the 8 squares adjacent to this current square-
* If it is not walkable or if it is on the closed list, ignore it. Otherwise do the following.
* If it isn’t on the open list, add it to the open list. Make the current square the parent of this square. Record the F, G, and H costs of the square.
* If it is on the open list already, check to see if this path to that square is better, using G cost as the measure. A lower G cost means that this is a better path. If so, change the parent of the square to the current square, and recalculate the G and F scores of the square. If you are keeping your open list sorted by F score, you may need to resort the list to account for the change.
4. Stop when you-
* Add the target square to the closed list, in which case the path has been found, or
* Fail to find the target square, and the open list is empty. In this case, there is no path.
### c) Save the path. Working backwards from the target square, go from each square to its parent square until you reach the starting square. That is your path.

## Python code

Feel free to use this code in your own projects.

```python
class Node():
    """A node class for A* Pathfinding"""

    def __init__(self, parent=None, position=None):
        self.parent = parent
        self.position = position

        self.g = 0
        self.h = 0
        self.f = 0

    def __eq__(self, other):
        return self.position == other.position


def astar(maze, start, end):
    """
    Returns a list of tuples as a path from the 
    given start to the given end in the given maze
    """

    # Create start and end node
    start_node = Node(None, start)
    start_node.g = start_node.h = start_node.f = 0
    end_node = Node(None, end)
    end_node.g = end_node.h = end_node.f = 0

    # Initialize both open and closed list
    open_list = []
    closed_list = []

    # Add the start node
    open_list.append(start_node)

    # Loop until you find the end
    while len(open_list) > 0:

        # Get the current node
        current_node = open_list[0]
        current_index = 0
        for index, item in enumerate(open_list):
            if item.f < current_node.f:
                current_node = item
                current_index = index

        # Pop current off open list, add to closed list
        open_list.pop(current_index)
        closed_list.append(current_node)

        # Found the goal
        if current_node == end_node:
            path = []
            current = current_node
            while current is not None:
                path.append(current.position)
                current = current.parent
            return path[::-1] # Return reversed path

        # Generate children
        children = []
        for new_position in [(0, -1), (0, 1), (-1, 0), (1, 0), (-1, -1), (-1, 1), (1, -1), (1, 1)]: # Adjacent squares

            # Get node position
            node_position = (current_node.position[0] + new_position[0], current_node.position[1] + new_position[1])

            # Make sure within range
            if node_position[0] > (len(maze) - 1) or node_position[0] < 0 or node_position[1] > (len(maze[len(maze)-1]) -1) or node_position[1] < 0:
                continue

            # Make sure walkable terrain
            if maze[node_position[0]][node_position[1]] != 0:
                continue

            # Create new node
            new_node = Node(current_node, node_position)

            # Append
            children.append(new_node)

        # Loop through children
        for child in children:

            # Child is on the closed list
            for closed_child in closed_list:
                if child == closed_child:
                    continue

            # Create the f, g, and h values
            child.g = current_node.g + 1
            child.h = ((child.position[0] - end_node.position[0]) ** 2) + ((child.position[1] - end_node.position[1]) ** 2)
            child.f = child.g + child.h

            # Child is already in the open list
            for open_node in open_list:
                if child == open_node and child.g > open_node.g:
                    continue

            # Add the child to the open list
            open_list.append(child)


def main():

    maze = [[0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 1, 0, 0, 0, 0, 0],
            [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]]

    start = (0, 0)
    end = (7, 6)

    path = astar(maze, start, end)
    print(path)


if __name__ == '__main__':
    main()
```

## Wrapping up

Hopefully this will help you to understand A* (star) Pathfinding algorithm.
