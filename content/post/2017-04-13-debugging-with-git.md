---
categories: 
  - apprenticeship
date: "2017-04-13T08:30:00Z"
title: Debugging with git
---

When I was researching git rebasing earlier this week I came across a couple interesting git features that I hadn’t run into before. In addition to git's core version control functionality, it also has several powerful debugging features. One of those is `git blame`. When you run `blame` on a file git annotates each line of the file with information about the last revision.

![debugging1](/assets/2017-04-13-git1.png)

You can even restrict `blame` to only look at specific lines of a file using the -L flag like below.

![debugging2](/assets/2017-04-13-git2.png)

The other debugging feature, which I thought was pretty neat was `git bisect`. With `bisect`, you provide git with a commit where a bug or unwanted behavior is present, along with a commit where you know that bug or unwanted behavior was not present. Git will then perform a binary search over the relevant portion of the commit history. At each step in the search, it asks the user if that commit is “good” or “bad” until it can determine which commit introduced the defect. So with relatively little effort you can quickly figure out which commit was the problem commit.

This entire process can even be automated as long as you provide git with a script that exits with a return code of 0 if the current commit is “good” and 1-127 if the commit is “bad”. This obviously doesn’t just have to apply to bugs, it could be a slowdown or some other unwanted behavior. As long as you can automate the testing for that behavior you could automate the search of the commit history using `git bisect` with the `run` command. 

In the example below, my test script just checks if `file2.txt` exists (admittedly kind of silly as there better ways to do this in git, but just for illustration). We can see from the log that `file2.txt` was introduced at `4ecaafb`, so just think of the addtion of `file2.txt` as the introduction of some unwanted behavior that our test script is testing for.

![debugging3](/assets/2017-04-13-git-log.png)



``` shell
# test.sh
if [[ -f file2.txt ]] ; then
    echo 'Still broken'
    exit 1
fi
    echo 'Fixed!'
    exit 0
```

Below I just ran through the bisect process manually, each time running my test script and then telling git if that commit was "good" or "bad". 

![debugging4](/assets/2017-04-13-git-bisect1.png)

However, given that the test script returns a valid error code the entire process can be automated using `git bisect run`. So if you have a long running test script, just kick off `git bisect run` and go grab a coffee!

![debugging5](/assets/2017-04-13-git-bisect2.png)

## Resources
[https://lwn.net/Articles/317154/](https://lwn.net/Articles/317154/)

[https://git-scm.com/book/en/v2/Git-Tools-Debugging-with-Git](https://git-scm.com/book/en/v2/Git-Tools-Debugging-with-Git)

[http://stackoverflow.com/questions/4713088/how-to-use-git-bisect](http://stackoverflow.com/questions/4713088/how-to-use-git-bisect)
