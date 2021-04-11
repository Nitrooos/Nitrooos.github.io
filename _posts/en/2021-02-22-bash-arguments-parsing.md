---
title: Parsing arguments in Bash
date: 2021-03-25 20:00:00.000000000 +02:00
identifier: bash-parsing-arguments
categories:
- en
- devops
tags:
- Bash
- script
excerpt: 
  Bored of copy-pasting the same, long commands while working with project?
  This post will show you how to simplify your daily job using a simple,
  dedicated bash script. Enjoy!
---
Almost each of you, from time to time meets some simple scripts, automating
repetitive tasks. You can often see them as interfaces, making project 
management easier. They can execute some actions in just one command, while
previously you needed to copy or type manually many lines of code. Bash, so
the popular shell of Unix-like systems makes it possible to write such scripts.

Today I show you how to parse the arguments passed into Bash script in a
straightforward manner.

## One assumption before we begin 🙂

We assume the arguments are passed into script in a form of:

    --argument-name value

or in a shorter way:

    -A value

The script shown in this post handles both cases. I wrote about this, because
there exists also other, alternative syntax:

    --argument-name=value

Such syntax, as also any others, is not supported.

## Let's add some context

Let's say the script allows user to deploy some code changes to server. And
because of that, we name it deploy.sh and add some sensible parameters, like:

    -E, --environment   The environment, on which the script should deploy the 
                        changes (e.g. testing, preprod, prod etc.)
    -B, --branch        Branch name, where the changes come from
    --no-cache          If supplied, build the application without using any cache
                        (can be helpful e.g. for building a fresh, new Docker image, 
                        without using any cache)
    -h, --help          Show help menu

Given flags are sample, adjusted to our agreed context of creating a deploy.sh script.

## Finally Bash script!

{% highlight bash %}
STATUS_OK=0
DEPLOY_NO_CACHE="false"
INFO_DEPLOY_HELP=$(cat <<-EOF
  -E, --environment   Environment to make a deployment (e.g. testing, preprod, prod etc.)
  -B, --branch        Branch name to make a deployment from
  --no-cache          If given, build a new app version without using any cache
                      (can be useful for e.g. building a new, clean Dockera image with the app, without using cache)
  -h, --help          Show help menu
EOF
)

...

_skip=0
for opt in $@; do
    [ $_skip -ne 0 ] && { ((_skip--)); shift; continue; }
    case $opt in
        -E|--env)
            DEPLOY_ENV=$2
            _skip=1
            shift ;;
        -B|--branch)
            DEPLOY_BRANCH_NAME=$2
            _skip=1
            shift ;;
        --no-cache)
            DEPLOY_NO_CACHE="true"
            shift ;;
        -h|--help)
            echo -e "$INFO_DEPLOY_HELP"
            exit $STATUS_OK
            ;;
        *)
            shift
    esac
done
{% endhighlight %}

The code above does a simple argument parsing of deploy.sh script. How exactly
does it work? It's time to clarify this!

### Bash – $@ variable

This variable contains all the arguments passed into the script. That's why we
iterate them in a for loop - to check all arguments in order.

### Helper variable $_skip

Thanks to that variable we skip a particular number of arguments on the 
beginning of each for-loop iteration. That's because we check only arguments'
names (and they are caught in the case instruction) and skip their values, using
them only inside the case instruction. In order to skip an argument we use the
"shift" instruction. Each of arguments can have different number of values 
(e.g. for -E/--env and -B/--branch have 1 value, but the --no-cache argument
doesn't have any value). Setting the value of $_skip variable to 1 means 
skipping value of that argument on the beginning of next for loop iteration.

### Bash – case instruction

The case instruction matches arguments' names passed into script. The notation 
-E|--env means the match is made when the argument "-E" or "--env" is recognized.
The "shift" instruction causes going to the next argument and setting the 
$_skip variable with value > 0 causes skipping the list of values of that 
argument. Let's notice, in the code sample above we don't call any actions, we
only initialize the DEPLOY_ENV, DEPLOY_BRANCH and DEPLOY_NO_CACHE variables 
with proper values. In the case of DEPLOY_NO_CACHE, it's just a text value
"true", while for the others it's a $2 variable value. It means the argument's
value, given after its name (available nota bene in $1 variable).

### Case *)

When the checked argument's name doesn't match any of defined options, we just
ignore it. "shift" just skips the argument, going to the next one.

## Pro tip: bash and the Ctrl+R shortcut

Sometimes creating a dedicated script for repetitive tasks is not necessary.
The good examples are single commands you often use, containing quite a few
number of arguments. Painful to manually type everytime into terminal, they
can be actually easily found in shell's history and called in a simple way. You
only need to press Ctrl+R key combination and then type a piece of searched
expression!

For example: we work on the tests of User model, defined in the UserModelTest
class, inside the package auth.test.test_models. For the Django framework, the
example run of such tests can be sth like this:

    $ python manage.py test --settings=myawesomeapp.test_settings auth.test.test_models.UserModelTest

While we work on the tests and the associated code, we want to execute them
often. Of course we don't want to type that command again and again, searching by 
pressing the &#8593; arrow can be also not convenient. Instead, we can just press
Ctrl + R, type e.g. "UserModel"... and bash will find proper command in the 
history! Then, only one "Enter" press and we're done!

## Summary

In this post I showed you how to parse Bash's script arguments. After successful 
parsing, the proper values are available inside defined variables for
further processing (in the example from the post DEPLOY_ENV, DEPLOY_BRANCH_NAME and
DEPLOY_NO_CACHE variables). Thanks to that, we can extend the script to make any actual
tasks, with the possibility to parametrize them. Please note this post is the 
first in the mini-cycle about Bash on my blog. I hope you'll enjoy it!
