eselectdir = $(datadir)/eselect/modules
dist_eselect_DATA = kotlin.eselect

dist_pkglibexec_SCRIPTS = \
	run-kotlin-tool.sh
EXTRA_DIST = \
	run-kotlin-tool.sh.in

tools = \
	kapt \
	kotlin \
	kotlinc \
	kotlinc-js \
	kotlinc-jvm \
	kotlin-dce-js

$(tools): $(DESTDIR)$(bindir)
	$(LN_S) $(pkglibexecdir)/run-kotlin-tool.sh $</$@

$(DESTDIR)$(bindir):
	$(MKDIR_P) $@

install-exec-hook: $(tools)

.PHONY: $(tools)
