# Texlive Manual Install

References:

1. https://wiki.archlinux.org/title/TeX_Live
2. https://en.wikibooks.org/wiki/LaTeX/Installation#Custom_installation_with_TeX_Live

Install the manual installer from AUR
```
# paru -S texlive-installer
```

Execute the installer and choose your options
```
# /opt/texlive-installer/install-tl
```

Run first test
```
$ pdftex '\empty Hello world!\bye'
```

The following message will appear
```
*************************************************************
*                                                           *
* WARNING: you are switching to fmtutil's per-user formats. *
*         Please read the following warnings!               *
*                                                           *
*************************************************************

You have run fmtutil-user (as opposed to fmtutil-sys) for the first time;
this has created format files which are local to your personal account.

From now on, any changes in system formats will *not* be automatically
reflected in your files; furthermore, running fmtutil-sys will no longer
have any effect for you.

As a consequence, you yourself have to rerun fmtutil-user after any
change in the system directories. For example, when one of the LaTeX or
other format source files changes, which happens frequently.
See https://tug.org/texlive/scripts-sys-user.html for details.

If you want to undo this, remove the files mentioned above.

Run mktexfmt --help for full documentation of fmtutil.
mktexfmt [INFO]: exiting with status 0
entering extended mode
[1{/usr/local/texlive/2022/texmf-var/fonts/map/pdftex/updmap/pdftex.map}]</usr/
local/texlive/2022/texmf-dist/fonts/type1/public/amsfonts/cm/cmr10.pfb>
Output written on texput.pdf (1 page, 11915 bytes).
Transcript written on texput.log.
```

Start a `screen` shell (to stay on the same shell)
```
$ screen
```

Become superuser
```
$ su
```

Run umask
```
# umask 022
```

Install LaTeX
```
# tlmgr install latex latex-bin
```

Make the corresponding symlinks
```
# ln -s /usr/local/texlive/2022/bin/x86_64-linux/pdftex /usr/local/bin/pdflatex
# ln -s /usr/local/texlive/2022/bin/x86_64-linux/pdftex /usr/local/bin/latex
```

Essential packages for work:
```
# tlmgr install \
wrapfig \
geometry \
fancyhdr \
babel-spanish \
amsmath \
mathtools \
tools \
hyperref \
pdftexcmds \
infwarerr \
kvoptions \
caption \
multirow \
newfloat \
pgfplots \
comment \
epstopdf-pkg 
```

Packages for pdf manipulation:
```
# tlmgr install \
pdfpages \
pdflscape \
pdfjam
```

Close the `screen` session.

Set-up directory for templates
```
$ TEXMF=/path/to/texmf
$ cp -r /usr/local/texlive/texmf-local/tex/ $TEXMF
```

Directory to put templates
```
TEXMF/tex/latex/local/
```
