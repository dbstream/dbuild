# SPDX-License-Identifier: MIT
# DBuild version 1.0
# Root-level Makefile for the DBuild build system. This is the "heart" of the
# build system, implementing most of the logic used.

# Stabilise the environment
# -------------------------
MAKEFLAGS += -Rr --no-print-directory

override dbuild-makefile := $(abspath $(lastword $(MAKEFILE_LIST)))
override dbuild-files := $(dir $(dbuild-makefile))DBuildFiles

# Useful variables to have defined.
# ---------------------------------
override empty :=
override comma := ,
override space := $(empty) $(empty)

# Useful functions
# ----------------

override dbuild-debug = $(empty)
# uncomment the following if you want to enable DBuild debug prints:
# override dbuild-debug = $(info [DBuild] $1)

override dbuild-reverse = $(if $(wordlist 2,2,$(1)),$(call dbuild-reverse,$(wordlist 2,$(words $(1)),$(1))) $(firstword $(1)),$(1))

override do-submake = $(MAKE) -C $1 -f $(dbuild-makefile) relsrc=$(relsrc)$1/ src=$(src)/$1

# Let the user specify verbosity (whether or not commands are printed).
# ---------------------------------------------------------------------
ifeq ($(V),$(empty))
V := 0
endif

ifeq ($(V),0)
Q := @
else
Q := $(empty)
endif

# Are we a submake?
ifeq ($(src),$(empty))

# We are not a submake.
# Prepare to call the first submake.
# ----------------------------------

# By specifying O=..., the user can choose an alternative output directory.
ifneq ("$(origin O)", "command line")
O := build
endif

# Set some variables.
# -------------------
relsrc := $(empty)
src := $(CURDIR)
export src relsrc

export dbuild-pre-files dbuild-post-files

# Do we have multiple MAKECMDGOALS?
# ---------------------------------
override split-build := 0
ifneq ($(MAKECMDGOALS),$(firstword $(MAKECMDGOALS)))
$(call dbuild-debug,Split build.)
split-build := 1
endif

ifeq ($(split-build),1)

# Build targets individually
# --------------------------
$(MAKECMDGOALS): FORCE
	$(Q)set -e; \
	for goal in $(MAKECMDGOALS); do \
		$(MAKE) -f $(dbuild-makefile) $$goal; \
	done

else # split-build

# Pivot into the build directory. This starts the "real" build.
# -------------------------------------------------------------
all $(filter-out all clean,$(MAKECMDGOALS)): FORCE
	$(Q)mkdir -p $(O)
	$(Q)$(MAKE) -C $(O) -f $(dbuild-makefile) $(filter-out clean,$(MAKECMDGOALS))

# Define a clean target if the build directory is the standard one.
# -----------------------------------------------------------------
ifeq ($(O),build)
clean: FORCE
	@echo CLEAN
	$(Q)rm -r $(O)
endif
endif # !split-build

else # $(src) == $(empty)

# We are a submake.
# -----------------

$(call dbuild-debug,Invoking DBuild files of directory $(src))

# Include pre-included files.
# ---------------------------
include $(dbuild-files)/pre.include
$(foreach file,$(dbuild-pre-files),$(eval include $(file)))

# Include the actual DBuild file.
# -------------------------------
include $(src)/DBuild

# Include post-included files.
# ----------------------------
$(foreach file,$(call dbuild-reverse,$(dbuild-post-files)),$(eval include $(file)))
include $(dbuild-files)/post.include

endif # $(src) != $(empty)

# Useful to have around.
# ----------------------
.PHONY: FORCE
FORCE:
