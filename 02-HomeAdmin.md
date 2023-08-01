# Home administrarion

## User binary folder

Create a binary location for user
```
mkdir ~/.local/bin
```

Add this location to path
```
~/.bashrc
.....
++ # set PATH so it includes user's private ~/.local/bin if it exists
++ if [ -d "$HOME/.local/bin" ] ; then
++    PATH="$HOME/.local/bin:$PATH"
++ fi 
```

