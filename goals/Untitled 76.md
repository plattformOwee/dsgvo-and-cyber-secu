Currently i have a dating app with working backend but the profile creation flow is clunky.

While i do have a menu to create each type of profile element, they are just in sequence after one another.

The goal of this chat is, to create a unified profile creation flow.

The userflow should go something like this:

- user chooses a element-type to add first element

- they put in their answers and click "next"

(progressbar on top gets fuller)

- user chooses next element-type to add

- they put in their answers and click "next"

(progressbar on top gets fuller)

and so on.

task1:

Create a profile_creation.dart:

- progressbar at the top

- below just empty space for the different other pages (ie the pages where user chooses and adds the elements)

task2:

Create choose_element.dart:

Should show a list of clickable cards with rounded corners and each card holds the layout of on type of element

task3: