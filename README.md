Due 12/11/17
Editor: sascha bates	sascha.bates@gmail.com	@sascha_d

-----
# Awesome bash/zsh autocompletion with fzf

## It's all about the CLIUX

Ok, CLIUX is not a word, but command-line interface user experience is definitely a worthy consideration, especially 
if you use the command line on a daily basis (or, constantly).  Web apps and graphical user interfaces usually get the 
design love, but this holiday season, treat yourself to a great command line user experience.   Now that we're 
thinking about CLIUX, we can roll up our sleeves and see how we can improve some of our everyday command line 
operations.  This can be as simple as adding in some colors to our shell prompts or big as [changing over to a new 
shell](https://github.com/robbyrussell/oh-my-zsh/).  We're going to focus on a few ways to make finding things easier.

## Enter FZF
A lot of CLI is about finding stuff.  Sometimes that is finding the right files to edit, or searching through git logs 
to review, or finding a a process Id or user Id to use as a parameters for other commands (kill, chgrp, etc).  
[Fzf](https://github.com/junegunn/fzf/) is a fuzzy finder, a CLI tool that is explicitly meant to bridge the technical 
need to find things with the user need to find them easily.  Written by [junegunn](https://github.com/junegunn), 
around for a few years.  But this is the year you use it to make your CLIUX awesome.
FZF adheres to the Unix Philosophy of [doing one thing, and doing it 
well](https://en.wikipedia.org/wiki/Unix_philosophy#Do_One_Thing_and_Do_It_Well).  It is composable.  FZF fits right 
in the mix.  A little DIY and you compose great UIs for common CLI tasks.

## Using FZF

You can install FZF with package managers, `brew install fzf` for OSX and `dnf install fzf` for Fedora, or check out 
the [full installation instructions](https://github.com/junegunn/fzf#installation) for other options.  Once you've got 
it installed, we'll start fuzzy finding! 

### Level 1: What is this thing, & Piping STDIN

Fzf can fuzzy search any input from stdin.  For starters, let's pipe our dictionary through `fzf`.  
```
cat /usr/share/dict/words | fzf
```
And start typing to narrow down the list to matching words: `c` `l` `n` `j` `f` `i`.  Ok, so there is one word that 
contain those letters in that order.  Clanjamfrie.  
[IMAGE]

It's a Scottish word that means spoken nonsense, as in"Anyone who doesn’t like fuzzy finding is just spouting 
clanfamfrie". Who knew?  Now we are getting somewhere.  Pipe that to cowsay and alias it for future use:

[IMAGE]

Now that we've got our vocabulary searching down, let's tackle another common need for searching, finding a file.  
Maybe we just remember it is a JSON file somewhere in our home directory.  We will know it when we see it, but don't 
quite remember the name.  So we'll use `find ` to search for files with a `.json` extension in our home directory, and 
pipe to `fzf` for fuzzy finding:

```find ~ -name “*.json” | fzf```

Fzf acts like an interactive realtime grep, so we can quickly narrow or broaden the search.  If we want to set the 
default command for `fzf` we can set the `FZF_DEFAULT_COMMAND` variable.  This command is used when there is nothing 
piped to `fzf`.  Maybe we want to ignore hidden files

```
export FZF_DEFAULT_COMMAND="find . -not -path '*/\.*'"
```

On OSX, we can set up a simple function so that we open the selected file with OSX's default application for that 
filetype:

```
function open(){/usr/bin/open "$(fzf)"}
```

### Level 2: Advanced Searching
OK, fzf can fuzzy find from standard in.  cool.  Let’s take the next step and think of some useful aliases for stuff 
we regularly search through.
Git.  Logs.  Search through commit messages and show commit.
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


K8s search through cluster configs, search through namespaces, search through pods to get logs for.
```
# short alias for picking a Kube config
c () {
  export KUBECONFIG=$(find ~/.kube -type f -name '*'"$1"'*' -exec grep -q "clusters:" {} \; -print | fzf --select-1)
}
```
Cool stuff, ton of other ideas on the example page.

### Level 3: Shell integration
Sky is the limit.  Go nuts
Autocompletion for CD, for 
Reverse history search?

## Conclusion
* You are worth it
* Simple, composable
* DIY


## References, Details and Inspiration

https://stackoverflow.com/questions/36513310/how-to-get-a-gits-branch-with-fuzzy-finder Git Example, select a branch

https://github.com/junegunn/fzf/wiki/examples#processes
Tons, killa  process

https://dmitryfrank.com/articles/shell_shortcuts

http://owen.cymru/fzf-ripgrep-navigate-with-bash-faster-than-ever-before/

