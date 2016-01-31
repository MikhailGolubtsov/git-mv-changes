#!/bin/bash

if [ "$#" -lt 3 ]; then
  echo -e "
NAME
    mv-changes - move specific changes from one commit to another.

SYNOPSIS
    mv-changes <source> <destination> <path>

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
" && exit 1
fi

BRANCH=`git rev-parse --symbolic-full-name --abbrev-ref HEAD`
FROM_PARAM=$1
TO_PARAM=$2
WHAT=${*:3}
FROM=`git rev-parse $FROM_PARAM`
TO=`git rev-parse $TO_PARAM`

if [[ `git log $FROM..$TO | wc -l` -gt 0 ]]; then
  START_REV=$FROM
  END_REV=$TO
  DIRECTION='up'
else
  START_REV=$TO
  END_REV=$FROM
  DIRECTION='down'
fi

if [[ $BRANCH == "HEAD" ]]; then
  echo "You are in detached state, which is not allowed for this command." && exit 1
fi
if [[ `git log ^HEAD $FROM $TO | wc -l` -gt 0 ]]; then
  echo "$FROM_PARAM, $TO_PARAM are unreachable from current HEAD, switch to the branch, please" && exit 1
fi
if [[ `git show --pretty="format:" --name-only $FROM -- $WHAT | wc -l` -eq 0 ]]; then
  echo "Nothing to move. There is no changes of '$WHAT' in $FROM_PARAM" && exit 0
fi

echo "Moving changes of '$WHAT' $DIRECTION to the tree"
echo "Source: " && git show --oneline --name-status $FROM -- $WHAT
echo "Destination: " && git show --oneline --cumulative $TO

echo -e "In case of troubles to revert any changes use:
    git reset --hard `git rev-parse --short HEAD`"

# from now on we are in detached state
git checkout -q $END_REV

# it's where the magic happens..
git reset -q $FROM~ -- $WHAT
# if move changes down in the history we don't need fixup commit for $FROM
# but anyway we have to commit to create a fixup commit for $TO
if [[ "$DIRECTION" == "down" ]]; then
  git commit -q -m 'tmp - will be deleted automatically'
else
  git commit -q --fixup=$FROM
fi
git add $WHAT && git commit -q --fixup=$TO
# do-nothing editor to get rid of user actions
EDITOR='echo'
git rebase --autosquash -q -i $START_REV~ || `git rebase --abort && git checkout $BRANCH && exit 1`

# and we need the second rebase to replay the rest of the branch on top of the edited history
# $ONTO pointer depends on whether we have fixed $FROM or not
[[ "$DIRECTION" == "down" ]] && ONTO=HEAD~ || ONTO=HEAD
# replay subsequent commits from original branch
git rebase -q --onto $ONTO $END_REV $BRANCH || exit 1
