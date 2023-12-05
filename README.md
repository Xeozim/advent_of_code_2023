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
| i | j | Substring | Result                                                             |
|---|---|-----------|--------------------------------------------------------------------|
| 0 | 1 | a         | No numbers start with the letter a so can immediately move i along |
| 1 | 2 | s         | Might be the start of "six" or "seven", keep going                 |
| 1 | 3 | si        | Can only be "six" now                                              |
| 1 | 4 | sit       | Definitely not a number, move i along                              |
| 2 | 3 | i         | No numbers start with the letter i, move i along                   |
| 3 | 4 | t         | Could be the start of "two" or "three"                             |
| 3 | 5 | tw        | Nearly there...                                                    |
| 3 | 6 | two       | This is a number! So we stop                                       |

The substring checking is easy because we can pre-calculate all the possible sub-strings of the
strings which represent the numbers one to nine - there aren't that many. e.g. these are all the
valid 1 and 2 letter starts of numbers (using python sets here so that the 't' at the start of 
'two' and at the start of 'three' doesn't create a duplication):

`
{'e', 'f', 'n', 'o', 's', 't'}
`

`
{'ei', 'fi', 'fo', 'ni', 'on', 'se', 'si', 'th', 'tw'}
`


Also whenever we move i we check if the char at i is a numeric (0-9) and stop straight away if so.

The reverse operation is effectively the same, but instead of starting at the left of the string
we start at the right, and we check against the valid ends of numbers instead of the valid starts.

Originally I was going to use the valid_starts for the reverse check as well, and change the
implementations of the functions like move_marker. But in the end it was much easier to just pre-
calculate the valid_ends as well and do everything literally in reverse.


## Day 2

### Part 1

Again parsing approach was pretty basic, split on `":"` and remove `"Game "` to get the game ID, 
split the remainder on `";"` to get each draw in the game, and split those on `","` to get each colour
in the draw.

Put a bit more effort into the data structures this time which paid off when I got to part 2. Stored
every draw in every game so we could do some reasoning later on.

`GetMaxDraw` just works out the maximum draw we can guarantee based on the draws we've seen so far, it
is just a combination of the highest # of each colour seen in any draw. Basically if we had seen the
actual amount of each colour drawn at some point (even if that was across different draws) and we
emptied the bag in one draw, this is what we would expect to see.

With the max draw information we can check if a game is possible for a given bag contents pretty
simply, if the maximum draw amount of any colour is higher than the bag contents, that game isn't
possible.

### Part 2

Having done the work up front to put the information into objects, this was easy. `GetMaxDraw` already
gives us the minimum number of cubes of each colour required to play any game, so we just need to
calculate the "power" of that draw for each game and sum them. Simples!


## Day 3

### Part 1

First approach that springs to mind is to copy the day 2 tactic of building a nice data structure
which will hopefully answer both parts. In this case we can view the schematic as a series of 
contiguous blocks of numbers, symbols (other than full stops), and blanks (full-stop symbols).
The blocks for the first two lines of the example schematic would look like this
467..114..      =       [467][ .. ][114][..]
...*......      =       [...][*][  ......  ]

We can generate blocks for a line simply enough. I bet there is a regex way to do this but I've
managed to avoid them so far so I'll just parse the line char by char. To do this there are two ints
to represent the start (i) and end (j) indices of the substring we're currently looking at. Every
iteration we need to check the type of the next letter (the one at j+1) to see if it's the same as
the contents of the current block we're building. If it isn't then we finalise the current block
and start a new one, setting i and j to j+1.

Just a note here that compared to previous days numbers are considered as a whole i.e. 467 is
four hundred and sixty seven not 4, 6, and 7.

When parsing a block we need to know the index at which it starts and ends, so that we can work out
which blocks are connected to it above and below. When we're calculating the blocks on a given line
we can populate the lateral (left + right) links as we go. We'll also assign a type to each block
(numeric, symbol, blank).

Once we've worked out all the blocks on each line we can populate the links between them by looking
at the blocks in the lines above and below, not forgetting to include blocks which cover the chars
which are diagonally adjacent.

Having all the links calculated we can find the answer to part one by looping over all the blocks
and summing the contents of numeric blocks which have any links to a symbol block.

This was quite fiddly to do in the end. Got majorly tripped up by having links as a list to start
with and ending up with duplication in those lists. Also messed up checking the lines below AND the
cell at the top right because mixing 0 indices and 1 indices confuses me.

### Part 2

Part 1 was worth all the effort, the nice data structure and inadvertendly well-debugged parsing
means that we can do this bit nice and easily. A couple of extra functions in the Block class can
check if any block is a gear and calculate the gear ratio, and we can get the sum the same way as we
did for summing the numeric values.


## Day 4

First impressions are that this is a fairly simple one, at least based on part 1. Planning to follow
a similar approach in terms of parsing the input into a bunch of useful objects which will give me
the answers. In addition will try to implement the example input as a test set.

### Part 1

Parsing approach is pretty simple, split on `:` to separate the card id from the numbers, then split
on `|` to separate the winning numbers from the player numbers, then split on ` ` (space) to get
each number.

Will have a single class which represents a card, containing the card ID, the set of winning numbers
and the set of player numbers. The class will also define a function to calculate the matching
numbers and one to calculate the score based on how many matching numbers there are.

The description provides us with 6 examples each of which has an answer for how many points it is
worth, so we can turn each of them into a test case. Because I'm just using jupyter notebooks I'm
not going to bother with a proper testing framework, will just have a specific testing section.

Went pretty well, worked 2nd time. Ironically the tests passed with the first implementation but
the real input had more space handling required, possibly due to the way it was split from a multi-
line string.

### Part 2

More of a twist than previous days! Explains the suspiciously reducing scores on the example cards...

The obvious but potentially quite slow approach is just to work down the cards, counting the amount
of copies we have of each one as we go. So we'll just build a list that contains an integer value
for the number of copies of each card, remembering of course that each card we process we have to
take account of how many copies of that card there are based on previously processed cards.

This worked nicely, and not really slow to be honest, VS code still registered it as 0.0s.
So I'm calling that a win!


## Day 5

### Part 1

On the face of it quite complex, but there's lots of tests provided and it's effectively the same
thing several times in a row (but not in such a way that it's combinatorially explosive).

Initial thinking was just to do what they do in the example and build an actual map for each step
i.e. have a list as long as the number of seeds where the corresponding entry in that list is the
soil type for that seed.

However, looking at the actual input the numbers get very large, so probably would be faster to take
what feels like a slightly cruder approach - store the map ranges in a class, then find and apply
the appropriate map range every time we want to map a specific number. We'll also have a class to
represent each mapping type i.e. seed-to-soil. These will store a collection of map ranges sorted
by input value, then when we want to do a mapping we can just find the appropriate range and apply
it (or reuse the input value if there isn't one).

It feels like there is probably some clever way to combine the mappings so that you could have a
single mapping from seed to location (which is all we really care about). But I can't think how to
do this and started this one early to get more points so not hanging around waiting for inspiration.

To parse things, was going to go line by line with stored state but turned out to be neater to split
the whole string into blocks based on empty lines, then we have one block per mapping and one at the
top for the seed numbers.
 - For getting the first line of input (the seed numbers)
   - All can be integers of arbitrary length
   - In keeping with previous days going to avoid regex. Will split on spaces then do a call to
     `int()` on all but the first (which should contain "`seeds:`" to extract the values
 - Similarly to the overall blocks, a mapping block can be split by lines into:
   - The first line of the block / beginning of a map i.e. "`x-to-y map:`"
      - Based on the input we can assume that x and y are just lower case alphabetical characters
      - In keeping with previous days going to avoid regex and just split the string a couple of times,
        once on spaces to get the "`x-to-y`" away from "`map:`", and then on hyphens to get "`x`",
        "`to`", and "`y`"
   - A series of lines containing map range definitions i.e. "`a b c`" where we want to capture a, b
     , and c. Parsing here is the same as for the seed numbers

Writing tests went pretty well yesterday so same again today, will have a couple of simple ones for
testing the parsing, then test the overall mapping logic with a bunch for each stage of
transformation (seed-to-soil, soil-to-fertilizer, etc.) using the example data.

