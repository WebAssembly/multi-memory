W3C_STATUS    = ED
BUILDDIR      = _build
STATICDIR     = _static
DOWNLOADDIR   = _download
NAME          = WebAssembly
DECISION_URL  = https://github.com/WebAssembly/meetings/blob/main/main/2017/WG-12-06.md
TAR           = tar

.PHONY: all
all:
	mkdir -p $(BUILDDIR)/html
	bikeshed spec --md-status=$(W3C_STATUS) index.bs $(BUILDDIR)/html/index.html
	@echo "Build finished. The HTML pages are in `pwd`/$(BUILDDIR)/html."

.PHONY: publish
publish:
	(cd ..; make publish-js-api)

.PHONY: clean
clean:
	rm -rf $(BUILDDIR)
	rm -rf $(STATICDIR)

.PHONY: diff
diff: all
	@echo "Downloading the old single-file html spec..."
	curl `grep "^TR" index.bs | cut -d' ' -f2` -o $(BUILDDIR)/html/old.html
	@echo "Done."
	@echo "Diffing new against old..."
	perl ../util/htmldiff.pl $(BUILDDIR)/html/old.html $(BUILDDIR)/html/index.html $(BUILDDIR)/html/diff.html
	@echo "Done. The diff is at $(BUILDDIR)/html/diff.html"

.PHONY: WD-tar
# macOS tar has no “--transform” option (only GNU tar does), so on macOS,
# do “brew install tar” & run “make” like this: “TAR=gtar make -e WD-tar”
WD-tar: all
	bikeshed spec --md-status=$(W3C_STATUS) index.bs $(BUILDDIR)/html/index.html
	$(TAR) -C $(BUILDDIR)/html --transform="s/index.html/Overview.html/" -cf $(BUILDDIR)/WD.tar index.html
	@echo "Built $(BUILDDIR)/WD.tar."

.PHONY: WD-echidna
WD-echidna: WD-tar
	@if [ -z $(W3C_USERNAME) ] || \
	    [ -z $(W3C_PASSWORD) ] || \
			[ -z $(DECISION_URL) ] ; then \
		echo "Must provide W3C_USERNAME, W3C_PASSWORD, and DECISION_URL environment variables"; \
		exit 1; \
	fi
	curl 'https://labs.w3.org/echidna/api/request' \
		--user '$(W3C_USERNAME):$(W3C_PASSWORD)' \
		-F "tar=@$(BUILDDIR)/WD.tar" \
		-F "decision=$(DECISION_URL)" | tee $(BUILDDIR)/WD-echidna-id.txt

.PHONY: WD-echidna-CI
WD-echidna-CI: WD-tar
	@if [ -z $(W3C_ECHIDNA_TOKEN_JSAPI) ] || \
			[ -z $(DECISION_URL) ] ; then \
		echo "Must provide W3C_ECHIDNA_TOKEN_JSAPI and DECISION_URL environment variables"; \
		exit 1; \
	fi
	curl 'https://labs.w3.org/echidna/api/request' \
		-F "tar=@$(BUILDDIR)/WD.tar" \
		-F "token=$(W3C_ECHIDNA_TOKEN_JSAPI)" \
		-F "decision=$(DECISION_URL)" | tee $(BUILDDIR)/WD-echidna-id.txt
	@echo
	@echo "Published w$(W3C_STATUS). Check its status at https://labs.w3.org/echidna/api/status?id=`cat $(BUILDDIR)/WD-echidna-id.txt`"
