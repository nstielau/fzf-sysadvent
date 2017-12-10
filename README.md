Due 12/11/17
Editor: sascha bates	sascha.bates@gmail.com	@sascha_d

-----
# Awesome bash/zsh autocompletion with fzf

## It's all about the CLIUX

Ok, CLIUX is not a word, but command-line interface user experience is definitely a worthy consideration, especially 
if you use the command line on a daily basis (or, hourly).  Web apps and graphical user interfaces usually get the 
design love, but this holiday season, treat yourself to a great command line user experience.   

Now that we're thinking about CLIUX, we can roll up our sleeves and see how we can improve some of our everyday command line 
operations.  This can be as simple as adding in some colors to our shell prompts or big as [changing over to a new 
shell](https://github.com/robbyrussell/oh-my-zsh/).  We're going to focus on a few ways to make finding things easier.

## Enter FZF
A lot of CLI is about finding stuff.  Sometimes that is finding the right files to edit, or searching through git logs 
to review, or finding a a process Id or user Id to use as a parameters for other commands (kill, chgrp, etc).

[Fzf](https://github.com/junegunn/fzf/) is a fuzzy finder, a CLI tool that is explicitly meant to bridge the technical 
need to find things with the user need to find them easily.  Written by [junegunn](https://github.com/junegunn), 
around for a few years.  But this is the year you use it to make your CLIUX awesome.

FZF adheres to the Unix Philosophy of [doing one thing, and doing it well](https://en.wikipedia.org/wiki/Unix_philosophy#Do_One_Thing_and_Do_It_Well) and valuing composability.  These two characteristics make it likely we can find lots of ways to use fuzzy finding, and to integrate it into our daily operations.

## Using FZF

You can install FZF with package managers, `brew install fzf` for OSX and `dnf install fzf` for Fedora, or check out 
the [full installation instructions](https://github.com/junegunn/fzf#installation) for other options.  Once you've got 
it installed, we'll start fuzzy finding! 

### Level 1: What is this thing, & Piping STDIN

We'll start off looking at how we can pipe a input set to `fzf` and then pipe the selected value to another utility.  Fzf can fuzzy search any input from stdin.  For starters, let's pipe our dictionary through `fzf`,

```
cat /usr/share/dict/words | fzf
```

and start typing to narrow down the list to matching words: `c` `l` `n` `j` `f` `i`.  Ok, so there is one word that 
contain those letters in that order.  Clanjamfrie.  

![Searching for clanjamfrie](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/clanjamfrie.png)

It's a Scottish word that means spoken nonsense, as in"Anyone who doesn’t like fuzzy finding is just spouting 
clanfamfrie". Who knew?  Now we are getting somewhere.  Pipe that to cowsay and alias it for future use:

![fzf and cowsay FTW](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/cowsay.png)

Now that we've got our vocabulary searching down, let's tackle another common need for searching, finding a file.  
Maybe we just remember it is a JSON file somewhere in our home directory.  We will know it when we see it, but don't 
quite remember the name.  So we'll use `find ` to search for files with a `.json` extension in our home directory, and 
pipe to `fzf` for fuzzy finding:

```
find ~ -name “*.json” | fzf
```

Fzf acts like an interactive realtime grep, so we can quickly narrow or broaden the search.  Search around and press enter to select a filename.  On OSX we can leverage the `open` command with `fzf` to quickly search and find files.

```
function open(){/usr/bin/open "$(fzf)"}
```

Where does `fzf` get it's input set if we don't pipe in anything?  From the `FZF_DEFAULT_COMMAND` variable. Maybe we want to ignore hidden files when we're searching, 

```
# Exclude hidden files by default, and anything in the 'Library' directory
export FZF_DEFAULT_COMMAND="find . -not -path '*/\.*' | grep -v '/Library/'" 
```

### Level 2: Advanced Searching
OK, fzf can fuzzy find from standard in.  cool.  Let’s take the next step and think of some useful aliases for stuff 
we regularly search through.  Also check out all the [examples on the fzf wiki](https://github.com/junegunn/fzf/wiki/examples) (some of these are taken from there).

This example makes it easy to search though git logs and examine a diff.
```
# fshow - git commit browser
fshow() {
  git log --graph --color=always \
      --format="%C(auto)%h%d %s %C(black)%C(bold)%cr" "$@" |
  fzf --ansi --no-sort --reverse --tiebreak=index --bind=ctrl-s:toggle-sort \
      --bind "ctrl-m:execute:
                (grep -o '[a-f0-9]\{7\}' | head -1 |
                xargs -I % sh -c 'git show --color=always % | less -R') << 'FZF-EOF'
                {}
FZF-EOF"
}
```

Here's the `fshow` function running against the `fzf` codebase:

~[Finding a git commit](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/fshow.png)


These helpers for kubernetes let you easily search for cluster config files and find namespaces.
```
# short alias that uses chosen namespace
k () {
    kubectl --namespace=${NS:-default} $@
}

# short alias for picking a Kube config
c () {
  export KUBECONFIG=$(find ~/.kube -type f -name '*'"$1"'*' -exec grep -q "clusters:" {} \; -print | fzf --select-1)
}

# helper for setting a namespace
ns () {
    namespaces=$(timeout 10s kubectl get ns -o=custom-columns=:.metadata.name)
    if [ "$?" -eq "124" ]; then
        printf "Could not connect to k8s cluster"
    fi
    export NS=`echo $namespaces | fzf --select-1`
    echo "Set namespace to $NS"
}
```

Cool stuff, and there are tons of other ideas on the example wikipage.

## Conclusion

If you want to spruce up your command line user experience, dig in with `fzf` and put the power of fuzzy finding to use.  Junegunn's `fzf` is a simple, composable tool that can make your command line more efficient and usable for 2018.


## References, Details and Inspiration

* [fzf Examples](https://github.com/junegunn/fzf/wiki/examples#processes)
* [Git Example, selecting a branch](https://stackoverflow.com/questions/36513310/how-to-get-a-gits-branch-with-fuzzy-finder)
* [Using fzf to create fuzzy shell bookmarks](https://dmitryfrank.com/articles/shell_shortcuts)
* [http://owen.cymru/fzf-ripgrep-navigate-with-bash-faster-than-ever-before/](Using fzf with ripgrep to search while 'faster than ever')

