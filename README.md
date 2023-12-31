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

### Part 2

Looks like a relatively simple change, most of the code for part 1 will still be useful we just
need to generate some more seed numbers. So the plan is to change the function that gets the seeds
from the inputs to call out to another function if we set a flag that says the seed string is
representing ranges (basically meaning use for part 2).

However, now it looks like it might be worth investing in a more complicated mapping approach,
because we're not applying each mapping to a dozen or so seed numbers, but to huge numbers of them!
For example in my input the first pair of numbers describes nearly 100 million seeds.

Hmmmmmm....

OK how about this, we only care about the lowest result, and the mappings are defined in fairly big
non-overlapping chunks. So why not explore the edges of each mapping chunk and trace a path to the
lowest location number. In order to guarantee that the number we find is the lowest we can work
backwards from location to seed, trying different chunks of location numbers until we find one that
maps to a valid seed number.

So, we know from the humidity-to-location map which blocks of locations map to which blocks of
humidity. In the example data this is fairly simple, locations 0 -> 55 are unaffected by any mapping
and so map to humidity values 0 -> 55. The next highest block of locations is 56 -> 59 and maps to
humidities 93 -> 96 as defined by the line "`56 93 4`".

Say we want to explore the temperatures that map to the lowest location numbers. We can use the
humidity to location map block with the lowest output (0 -> 55 default mapping to humidity 0 -> 55)
then we know if we can find the temperature values which map to that block, they will have the
lowest possible location values. The temperature-to-humidity map in turn tells us that humidity 0
maps to temperature 69 ("`0 69 1`"), and the others in the block (1 -> 55) map to the number above
them ("`1 0 69`").

We can repeat this process all the way back to the seed side of the mappings. Once we have a block
of candidate seed numbers we can check if any of them are valid relatively easily. If none exist we
go back one level and try the next branch, repeating until we find a path through our mappings which
starts in the lowest possible block of locations and finds a valid seed number. Because the ordering
of numbers is unchanged by a particular mapping, we can just find the lowest valid seed number and
trace back through the mapping path to get the corresponding location, which should be the lowest...
right?

This process is obviously quite complex and will involve a fair bit of recursive branching, but the
number of operations should be way lower, especially as it's basically impossible that we'll have to
actually check all the possible branches.

The first thing that will probably be helpful is to create RangedMapper objects for the default
map behaviour (where a number maps to itself). We can do this when we create the Mapping objects
by having the code fill in any gaps left by the mappers passed to the constructor.

... To be continued ...


## Day 6

This looks much more friendly! Nice physicsy question even if the physics are a bit quirky in this
world.

### Part 1

#### Quadratic Formulas
This feels like solving equations to me. The distance travelled $s$ for a given race is given by
$(s = (t - h) * h)$ where $t$ is the time available, and $h$ is how long we hold down the button. We
want to find all the solutions where $s > r$ where $r$ is the current record.

Lucky for us we know (either by instinct or using the example) that the behaviour is quadratic in
nature - it has a single maximum point and falls away equally in both directions. So all we have to
do is find the point where the equation for $s$ crosses the value of $r$, it will do this twice if
there is more than one solution (once on the way up, and again on the way down).

If we rearrange a bit we can come up with a quadratic equation in the form $ax^2 + bx + c = 0$:

$r = (t - h) * h$ => $r = -h^2 + th$ => $-h^2 + th - r = 0$

We can use the [quadratic formula](https://en.wikipedia.org/wiki/Quadratic_formula) to solve this,
substituting:
$$ x = {-b \pm \sqrt{b^2-4ac} \over 2a} $$

In our case $x$ is $h$, $a=-1$, $b=t$, and $c=-r$*.


The $\pm$ in the equation will give us two answers (most of the time), which will correspond to the
two points where the line $s = (t - h) * h$ crosses $s = r$.

Let's try this out with the example they use. So $t=7$, and $r=9$. Solving the quadratic formula
gives us two roots: $5.3$ and $1.7$ (rounded to 1 d.p.). Because our toy boat race only works in
integer values we have to round these. In order to get integers which will beat the record we have
to round up the lower value and round down the higher one. For the example this gives us 2 and 5 as
the edges of our record setting zone, the same as they got :tada:. Then we can just work out the
number of possible record setting values by doing 5 - 2 + 1 (+1 makes the range inclusive) to get 4.

*: This was updated to $c=-(r+1)$, see note in [Testing](#testing)

#### Code
Very long explanation, lets get started on the code.

We'll parse the input into a series of Race objects, each one with a time and record distance. The
Race class will implement our logic, calling out to a utility function for solving a quadratic
equation

The input is arranged slightly differently to previous days, each object we want to create is a
column rather than a row. So we'll have to parse all the rows to get the numbers in each then
iterate over them to create objects from the values in each column.

Everything else is just utilities and testing stuff.

#### Testing
On testing it turns out that race 3 is a good indicator (a well chosen example Eric!).
The solutions to it's quadratic are exactly 10 and 20, so we calculate the range as 10,11,...19,20.
But these will give us record *equalling* distances, not record beating. Slight tweak to the logic
to use $c=-(r+1)$ should fix this.

### Part 2

Vindication! Our approach scales really well to this sort of thing. There's only one a couple of
tweaks to make. Firstly we shouldn't generate the list of possible winning races and count its
length, we can just take the answers at the limit away from one another to find the same thing.
Secondly we need to update the parsing to ignore spaces. Both easy to do, and we'll add another test
to make sure we do it right.


## Day 7

### Part 1

Bit of a classic this, ranking poker hands. I happen to know that this is somewhat of a solved
problem. At scale the best thing to do is use a pre-calculated list of ordered poker hands and an
efficient technique for looking up two items in that list. For regular poker hands there is a well
known tool called the Two Plus Two hand evaluator which uses this approach.On initial reading I was
tempted to just download the datafile for 2+2 and use that, however Eric is one step ahead as always
and the ordering system for hands is slightly different here, so leaving this idea in the back
pocket for now, might come back to this in part 2.

In regular 5 card poker games, the order of the cards is not important. If two hands have the same
type (full house / two pair / high card etc.) then the constituent card ranks come into play i.e. a
four of a kind (2AAAA) is better than (33332) because the card that defines the type is higher (A>3).

In camel cards, the order of the cards is the 2nd most important thing after hand type. This is
much simpler for us to deal with, all we have to do is find out what type each hand is, and then we
can do a 6-step sort over the whole list of cards. Note that we sort by the least important factor
first:
 1. Value of 5th card
 2. Value of 4th card
 ...
 5. Value of 1st card
 6. Hand type

Also worth noting that we can have different hand types to normal poker in camel cards, no flushes
or straights, and 5 of a kinds but that won't really affect the implementation as far as I can tell.

As always we'll have container classes to parse things into, in this case Hand which tells us what
cards are in the hand and will contain the function for ranking two hands, and a Card enum which
stores the value of a single card defines the card ordering which we can use for sorting.

Because we're using python, the easiest sorting method is to define a function which turns our
object into something python already knows how to sort. So we'll do that in a function called
SortKey in the Hand class.

The approach here is to turn every hand into an integer that represents the rank of that hand. We
can guarantee the order of these integers is what we want by using a base-13 system. Each digit in
this system instead of representing ten values (from 0 to 9 like the standard base-10 system) can
represent thirteen values (0 to 13), the value of each digit increases by a multiplie of by 13
rather than by 10. So in base-13 the number `200` is $2*13^2 = 338$. We choose 13 because that's just
enough resolution to include all our card values in a single digit, and because there are fewer hand
types than card values it can encompass that too.

So we effectively generate a 6 digit base-13 number to represent each hand, with the first digit
representing the hand type, the second digit the value of the first card, etc. For example the hand
`KK677` is a two pair (enum value = 2) with a king at card 1 (enum value 11), another king at
card 2, a 6 at card 3 (value 4), and sevens at cards 4 and 5 (value 5). So the representation of
this hand is $2*13^5 + 11*13^4 + 11*13^3 + 4*13^2 + 5*13^1 + 5*13^0 = 1081670$. In the code we don't
actually bother with storing the representation in base 13, we just store it as an int, the highest
value possible is 2599050 (AAAAA) which easily fits into int representation.

### Part 2

Looks like a fairly manageable change, making JACK into JOKER and re-arranging the card strengths is
a quick change to the `Card` enum, and we can just maintain a separate joker count when working out
the card type. The basic approach I took is helpful here because we can just update the logic for
each check to include the relevant joker consideration e.g. the `occurrences == 5` check for a
five-of-a-kind can just become `occurrences + joker_count == 5`.

Most annoying bit is we can't make these changes in a way that preserves the original logic, so will
have to make a whole new notebook for part 2 so that we can still see how part 1 worked. Also have
to rewrite some tests and make sure we're checking the new joker card a bit.

Well that's frustrating, the new tests pass and we get the right answer for the example, but not the
real input apparently!

Looking at the hand rankings, hands with jokers at the start are consistently ranked very low, which
doesn't make sense. The lowest rank hand has two jokers in!

The logic adjustment for calculating hand type was too simple, for trips and pairs it's not enough
to just include the jokers against everything, because then we end up double counting. Need to count
the pairs and triplets as normal then include the jokers when we're working out if a hand is a full
house / three of a kind / two pair / one pair. We can ignore the case where there are three or four
jokers, because that will give us a five of a kind, but we do need to check some other cases such as
a pair of jokers making a three-of-a-kind.

That wasn't quite right, we're still mis-classifying full-houses that include a Joker as two pair
e.g. `44KKJ` was classified as two pair. Need to check for any pairs and a joker when converting
pairs to triples using a joker (not just one pair).

Also turns out that our logic for early returns when we find a four-of-a-kind is wrong, if there are
four jokers e.g. `J8JJJ` the first card we check (2) can have an occurrence count of 0 but we'll
still early exit thinking there's quads, but later in the occurrence array one of the cards (8) will
have a count of 1 and we'll find a quintet so we should be waiting to see if that's the case.

Yatta! Success at last, that was harder than it ought to have been.
