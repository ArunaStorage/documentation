
TEXSRC		= storage
INPUT		= references.bib $(wildcard *.tex)

LATEX		= pdflatex
BIBTEX		= bibtex

.PHONY:		clean realclean $(SUBDIRS)

all:		$(SUBDIRS) $(TEXSRC).pdf

$(SUBDIRS):
		$(MAKE) -C $@

clean:
		rm -f *.aux *.log *.lof *.lot $(TEXSRC).bbl $(TEXSRC).blg $(TEXSRC).tdo $(TEXSRC).out $(TEXSRC).toc

realclean:	clean
		rm -f $(TEXSRC).pdf 

$(TEXSRC).aux:	main.tex $(SUBDIRS)
		$(LATEX) $(TEXSRC).tex

$(TEXSRC).pdf:	$(TEXSRC).tex *.tex
		$(eval STAMP := $(shell stat -c %Y main.aux))
		$(LATEX) $(TEXSRC).tex
		$(LATEX) $(TEXSRC).tex
		$(LATEX) $(TEXSRC).tex
		touch -cm main.aux
