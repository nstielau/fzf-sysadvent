# Awesome command-line fuzzy finding with fzf

## It's all about the CLIUX

Ok, CLIUX is not a word, but command-line interface user experience is definitely a worthy consideration, especially 
if you use the command line on a daily basis (or, hourly).  Web apps and graphical user interfaces usually get the 
design love, but this holiday season, treat yourself to a great command line user experience.   

Now that we're thinking about CLIUX, we can roll up our sleeves and see how we can improve some of our everyday command line 
operations.  This can be as simple as [adding in some colors](http://ethanschoonover.com/solarized) to our shell prompts or big as [changing over to a new shell](https://github.com/robbyrussell/oh-my-zsh/).  

We're going to focus on a few ways to make finding things easier.

## Enter FZF
A lot of CLI is about finding stuff.  Sometimes that is finding the right files to edit, or searching through git logs 
to review, or finding a a process Id or user Id to use as a parameters for other commands (kill, chgrp, etc).

[Fzf](https://github.com/junegunn/fzf) is a fuzzy finder, a CLI tool that is explicitly meant to bridge the technical 
need to find things with the user need to find them easily.  It has been around for a few years, but this is the year you use it to make your CLIUX awesome.

![FZF Logo](https://raw.githubusercontent.com/junegunn/i/master/fzf.png)

FZF adheres to the Unix Philosophy of [doing one thing, and doing it well](https://en.wikipedia.org/wiki/Unix_philosophy#Do_One_Thing_and_Do_It_Well) and valuing composability.  These two characteristics make it likely we can find lots of ways to use fuzzy finding, and to integrate it into our daily operations.

Quick shoutout to [junegunn](https://github.com/junegunn/fzf) for authoring `fzf` and [Tschuy](https://tschuy.com/) for introducing `fzf` to me.

## Using FZF

### Getting started

You can install FZF with package managers, `brew install fzf` for OSX and `dnf install fzf` for Fedora, or check out 
the [full installation instructions](https://github.com/junegunn/fzf#installation) for other options.  Once you've got 
it installed, we'll start fuzzy finding! 

### Step 1) Piping data into FZF

We'll start off looking at how we can pipe an input set to `fzf` and doing some fuzzy finding.  Fzf can fuzzy search any input from stdin.  For starters, let's pipe our dictionary through `fzf`,

```
cat /usr/share/dict/words | fzf
```

and start typing to narrow down the list to matching words: `c` `l` `n` `j` `f` `i`.  Ok, so there is one word out of the 235,886 in the dictionary that 
contains those letters in that order.  Clanjamfrie.  

![Searching for clanjamfrie](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/clanjamfrie.png)

It's a Scottish word that means spoken nonsense, as in"Anyone who doesn’t like fuzzy finding is just spouting 
clanfamfrie." Who knew?  


### Step 2) Piping data out of FZF

Now we are getting somewhere.  Selecting a dictionary word is pretty useful, but how can we make this really great?  That's right, piping our selected word to `cowsay`.

```
cat /usr/share/dict/words | fzf | cowsay
```

![fzf and cowsay FTW](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/cowsay.png)

### Step 3) Live preview

Sysadmins everywhere are on the edge of their seats.  Cowsay, dictionaries, fuzzy finding.  It's almost too much.  How can we take this to the next level?

Using the `--preview` flag, we can specify a program for a live preview of our `fzf` selection.  In this case, we can get a preview of exactly how the cow will say it:

```
cat /usr/share/dict/words | fzf --preview "cowsay {}" | cowsay
```

![Preview that moo](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/imagine.png)

### Step 4) Gitting More Pragmatic

Not that cowsay isn't a real world use-case, but let's get into something more pragmatic, like... git!  This gem of an example is from the [ample examples on the fzf wiki](https://github.com/junegunn/fzf/wiki/examples) (although paired down a bit for consumability here).  It shows how to search though git logs and examine a diff.
```
# fshow - git commit browser
fshow() {
  git log --graph --color=always \
      --format="%C(auto)%h%d %s %C(black)%C(bold)%cr" "$@" |
  fzf --ansi --bind "enter:execute:
                (grep -o '[a-f0-9]\{7\}' | head -1 |
                xargs -I % sh -c 'git show --color=always % | less -R') << 'FZF-EOF'
                {}
FZF-EOF"
}
```

The big new addition here is the `--bind` option, which can specify a program to execute upon key press or selection.  The `fshow` function uses this functionality to view the git diff with `less` when enter is pressed.

Here's the `fshow` function running against the `fzf` codebase:

![Finding a git commit](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/fshow.png)

### Step 5) Make viewing diffs easy

This is pretty cool, but we already know how to make it cooler.  With `--preview`!

```
fshow() {
  git log --graph --color=always \
      --format="%C(auto)%h%d %s %C(black)%C(bold)%cr" "$@" |
  fzf --ansi --preview "echo {} | grep -o '[a-f0-9]\{7\}' | head -1 | xargs -I % sh -c 'git show --color=always %'" \
             --bind "enter:execute:
                (grep -o '[a-f0-9]\{7\}' | head -1 |
                xargs -I % sh -c 'git show --color=always % | less -R') << 'FZF-EOF'
                {}
FZF-EOF"
}
```
![Previewing diffs](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/fshow_preview.png)

Wow! Does anyone else feel like we just implemented Github in like 6 lines?!?!

### Step 6) Man Explorer

Or, whip up a man page explorer with search and live-preview.  Once you have the format down, you can imagine more use-cases that boil down to fuzzy finding with a detailed preview that you can select to execute additional commands, optionally falling back to the beginning. 

```
# This is ugly.  Refactoring left as exercise to reader...
function  mans(){
man -k . | fzf -n1,2 --preview "echo {} | cut -d' ' -f1 | sed 's# (#.#' | sed 's#)##' | xargs -I% man %" --bind "enter:execute: (echo {} | cut -d' ' -f1 | sed 's# (#.#' | sed 's#)##' | xargs -I% man % | less -R)"
}
```

![Man Page Explorer](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/selinux_man.png)

## Why I can't live without FZF

Ok, I guess I could live.  But my command line useage would be a little sadder.  For example, I use these kubernetes config helpers every day.  In addition to saving some time, I get a little bit more joy out of the `fzf` implementations.  I love the command line, but that doesn't mean I don't want a great user experience.  Heck, I *deserve* one.

```
# short alias for picking a Kube config
# Find cluster definitions in ~/.kube and save one as variable.
c () {
  export KUBECONFIG=$(find ~/.kube -type f -name '*'"$1"'*' -exec grep -q "clusters:" {} \; -print | fzf --select-1)
}

# helper for setting a namespace
# List namespaces, preview the pods within, and save as variable
ns () {
    namespaces=$(kubectl get ns -o=custom-columns=:.metadata.name)
    export NS=`echo $namespaces | fzf --select-1 --preview "kubectl --namespace {} get pods"`
    echo "Set namespace to $NS"
}

# short alias that uses chosen namespace
k () {
    kubectl --namespace=${NS:-default} $@
}

```



Choosing a namespace:

![Kubernetes Namespace Chooser](https://raw.githubusercontent.com/nstielau/fzf-sysadvent/master/images/choose_namespace.png)

## Conclusion

If you want to spruce up your command line user experience, dig in with `fzf` and put the power of fuzzy finding to use.  Junegunn's `fzf` is a simple, composable tool that can make your command line more efficient and usable for 2018.

## References, Details and Inspiration

* [fzf Examples](https://github.com/junegunn/fzf/wiki/examples#processes)
* [Git Example, selecting a branch](https://stackoverflow.com/questions/36513310/how-to-get-a-gits-branch-with-fuzzy-finder)
* [Using fzf to create fuzzy shell bookmarks](https://dmitryfrank.com/articles/shell_shortcuts)
* [Using fzf with ripgrep to search while 'faster than ever'](http://owen.cymru/fzf-ripgrep-navigate-with-bash-faster-than-ever-before/)

