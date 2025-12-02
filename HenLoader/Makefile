###############################################################################
# Makefile cheat sheet:
# https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html
###############################################################################

# =======================
# Configurable Variables
# =======================
DISC_LABEL		?= HenLoader
MAKEFILE_DIR	:= $(dir $(realpath $(lastword $(MAKEFILE_LIST))))
BDJSDK_HOME		?= $(MAKEFILE_DIR)/../../bdj-sdk
BDJSDK_HOST		:= $(BDJSDK_HOME)/host
JAVA8_HOME		?= $(BDJSDK_HOST)/jdk8
BDJSDK_LIB		:= $(BDJSDK_HOME)/target/lib
AVCHD			:= $(BDJSDK_HOME)/resources/AVCHD

# =======================
# Tool Paths
# =======================
BDSIGNER		:= $(BDJSDK_HOST)/bin/bdsigner
MAKEFS			:= $(BDJSDK_HOST)/bin/makefs
JAVAC			:= $(JAVA8_HOME)/bin/javac
JAR				:= $(JAVA8_HOME)/bin/jar

export JAVA8_HOME

# =======================
# Build Variables
# =======================
CLASSPATH		:= $(BDJSDK_LIB)/interactive-stubs.zip:$(BDJSDK_LIB)/bdjstack.jar:$(BDJSDK_LIB)/rt.jar:src
JFLAGS			:= -Xlint:-options -source 1.4 -target 1.4
LOADER_DSTDIR	:= build/henloader
SOURCES	   		:= $(wildcard src/org/bdj/*.java) \
					$(wildcard src/org/bdj/sandbox/*.java) \
					$(wildcard src/org/bdj/api/*.java)

DISC			:= build/disc
BD_JAR			:= $(DISC)/BDMV/JAR/00000.jar
BD_HEN			:= $(DISC)/BDMV/AUXDATA/aiofix_USBpayload.elf
BD_META			:= $(DISC)/BDMV/META/DL/bdmt_eng.xml
BD_BANNER		:= $(DISC)/BDMV/META/DL/logo.png

TMPL_FILES		:= $(shell find $(AVCHD)/ -type f)
DISC_FILES		:= $(patsubst $(AVCHD)%,$(DISC)%,$(TMPL_FILES)) \
					$(BD_JAR) $(BD_META) $(BD_BANNER) $(BD_HEN)

TOOLCHECKS		:= $(JAVAC) $(JAR) $(MAKEFS) $(BDSIGNER)

# =======================
# Main Targets
# =======================
.PHONY: all help clean distclean check-tools

all: check-tools $(DISC_LABEL).iso

check-tools:
	@for tool in $(TOOLCHECKS); do \
		command -v $$tool >/dev/null 2>&1 || { echo "Error: $$tool not found"; exit 1; }; \
	done

help:
	@echo "Available targets:"
	@echo "  all	  - Build the disc image (default)"
	@echo "  clean	- Remove build artifacts"
	@echo "  distclean  - Remove all generated files"
	@echo "  help	 - Show this help message"
	@echo "Variables can be overridden, e.g. make DISC_LABEL=MyDisc"

$(DISC):
	@mkdir -p $(dir $(DISC_FILES)) $(LOADER_DSTDIR)

# Copy template files
$(DISC)/%: $(DISC)
	cp $(AVCHD)/$* $@

# Build JAR
$(BD_JAR): $(DISC) $(SOURCES)
	@if [ -z "$(SOURCES)" ]; then echo "Error: No source files found."; exit 1; fi
	@echo "Compiling Java sources..."
	$(JAVAC) -d $(LOADER_DSTDIR) -sourcepath src $(JFLAGS) -cp $(CLASSPATH) $(SOURCES)
	@echo "Creating JAR..."
	$(JAR) cf $@ -C $(LOADER_DSTDIR) .
	@echo "Signing JAR..."
	$(BDSIGNER) -keystore $(BDJSDK_HOME)/resources/sig.ks $@

# Pattern rules for metadata and payload
$(DISC)/BDMV/AUXDATA/%.elf: bd-metadata/%.elf $(DISC)
	cp $< $@

$(DISC)/BDMV/META/DL/%.xml: bd-metadata/%.xml $(DISC)
	cp $< $@

$(DISC)/BDMV/META/DL/%.png: bd-metadata/%.png $(DISC)
	cp $< $@

# ISO generation
$(DISC_LABEL).iso: $(MAKEFS) $(DISC_FILES)
	@echo "Generating ISO image..."
	$(MAKEFS) -m 16m -t udf -o T=bdre,v=2.50,L=$(DISC_LABEL) $@ $(DISC)

clean:
	@echo "Cleaning build artifacts..."
	$(RM) -rf META-INF build $(DISC_LABEL).iso src/org/bdj/*.class src/org/bdj/sandbox/*.class src/org/bdj/api/*.class

distclean: clean
	@echo "Removing all generated files..."
	@$(RM) -rf $(DISC) $(LOADER_DSTDIR)