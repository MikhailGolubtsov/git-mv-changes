# git-mv-changes
NAME
    
    mv-changes - move specific changes from one commit to another.

SYNOPSIS
    
    mv-changes <source> <destination> <path>...

OPTIONS

    <source>, <destination> - git revision pointers in any format

    <path> - file or directory path(s), wildcards can be used

DESCRIPTION

    This command moves specific changes from one commit to another.

    It assumes that there the branch is checked out and there are no local changes.

    It works in both directions - moving changes from an older commit to a newer one and vice versa.

    Internally it uses *rebase* so that history is changed.

    May or will NOT work in case of:
      * present changes in <path> between source and destination commits
      * renames
      * merge-commits
      * emptying source commit (just squash/fixup then)
      

INSTALLATION
    
    1. git clone https://github.com/MikhailGolubtsov/git-mv-changes.git
    2. cd git-mv-changes
    3. sudo cp mv-changes /usr/bin/
