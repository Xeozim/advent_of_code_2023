# Advent of Code 2023

My workings for AoC 2023. Generally I use vscode's built in Jupyter notebooks.

## Day 1

### Part 1

Parsing approach is just to go through the line strings char by char from left to right and right to
left (simultaneously), and recording the first char we find that is a numeric character. The left to
right scan gives us the first one in the string, the right to left one gives us the last one.

Pythons handling of strings as arrays of chars is really nice here.

### Part 2

Much trickier. Approach was as follows: Slide two markers over the string which define the start and
end of a substring which we'll check to see if it represents a number. The first marker (i) starts
at 0 and moves along as soon as we establish that the character at that index is not part of a sub-
string which represents a number. The second marker (j) starts at i+1 and moves along after the sub-
string is checked, or is reset to i+1 if i moves.

So for example, given this string: "asitwone"
i   j   Substring   Result
0   1   a           No numbers start with the letter a so can immediately move i along
1   2   s           Might be the start of "six" or "seven", keep going
1   3   si          Can only be "six" now
1   4   sit         Definitely not a number, move i along
2   3   t           Could be the start of "two" or "three"
2   4   tw          Nearly there...
2   5   two         This is a number! So we stop

The substring checking is easy because we can pre-calculate all the possible sub-strings of the
strings which represent the numbers one to nine - there aren't that many. e.g. these are all the
valid 1 and 2 letter starts of numbers (using python sets here so that the 't' at the start of 
'two' and at the start of 'three' doesn't create a duplication):
{'e', 'f', 'n', 'o', 's', 't'}
{'ei', 'fi', 'fo', 'ni', 'on', 'se', 'si', 'th', 'tw'}

Also whenever we move i we check if the char at i is a numeric (0-9) and stop straight away if so.

The reverse operation is effectively the same, but instead of starting at the left of the string
we start at the right, and we check against the valid ends of numbers instead of the valid starts.

Originally I was going to use the valid_starts for the reverse check as well, and change the
implementations of the functions like move_marker. But in the end it was much easier to just pre-
calculate the valid_ends as well and do everything literally in reverse.


## Day 2

### Part 1

Again parsing approach was pretty basic, split on the colon and remove "Game " to get the game ID, 
split the remainder on ";" to get each draw in the game, and split those on "," to get each colour
in the draw.

Put a bit more effort into the data structures this time which paid off when I got to part 2. Stored
every draw in every game so we could do some reasoning later on.

GetMaxDraw just works out the maximum draw we can guarantee based on the draws we've seen so far, it
is just a combination of the highest # of each colour seen in any draw. Basically if we had seen the
actual amount of each colour drawn at some point (even if that was across different draws) and we
emptied the bag in one draw, this is what we would expect to see.

With the max draw information we can check if a game is possible for a given bag contents pretty
simply, if the maximum draw amount of any colour is higher than the bag contents, that game isn't
possible.

### Part 2

Having done the work up front to put the information into objects, this was easy. GetMaxDraw already
gives us the minimum number of cubes of each colour required to play any game, so we just need to
calculate the "power" of that draw for each game and sum them. Simples! 