.DEFAULT_GOAL=all
REBAR3_URL=https://s3.amazonaws.com/rebar3/rebar3

# If there is a rebar in the current directory, use it
ifeq ($(wildcard rebar3),rebar3)
REBAR3 = $(CURDIR)/rebar3
endif

# Fallback to rebar on PATH
REBAR3 ?= $(shell which rebar3)

# And finally, prep to download rebar if all else fails
ifeq ($(REBAR3),)
REBAR3 = rebar3
endif

all: $(REBAR3)
	@$(REBAR3) do clean, compile, eunit, dialyzer

rel: all
	@$(REBAR3) release

$(REBAR3):
	curl -Lo rebar3 $(REBAR3_URL) || wget $(REBAR3_URL)
	chmod a+x rebar3

install: $(REBAR3) distclean
	$(REBAR3) update

travis: all ct rel
	@echo "Travis'd!"

ct: $(REBAR3)
	$(REBAR3) do ct --dir apps/chef_db/itest, ct --dir apps/oc_chef_authz/itest, ct --dir apps/oc_chef_wm/itest

version_clean:
	@rm -f VERSION

## echo -n only works in bash shell
SHELL=bash
REL_VERSION ?= $$(git log --oneline --decorate | grep -v -F "jenkins" | grep -F "tag: " --color=never | head -n 1 | sed  "s/.*tag: \([^,)]*\).*/\1/")-$$(git rev-parse --short HEAD)

VERSION: version_clean
	@echo -n $(REL_VERSION) > VERSION

## for Omnbibus
omnibus: $(REBAR3) distclean
	$(REBAR3) update
	$(REBAR3) do compile, release

## For dvm
DVM_HOOK = bundle
dvm_file = $(wildcard /vagrant/dvm.mk)
ifeq ($(dvm_file),/vagrant/dvm.mk)
	include /vagrant/dvm.mk
endif

bundle_clean:
	@cd apps/chef_objects/priv/depselector_rb; rm -rf .bundle

bundle:
	@echo bundling up depselector, This might take a while...
	@cd apps/chef_objects/priv/depselector_rb; bundle install --deployment --path .bundle

travis_env:
	@echo export CHEFDK_GECODE_PATH=/opt/chefdk/embedded/lib/ruby/gems/2.3.0/gems/dep-selector-libgecode-1.3.1/lib/dep-selector-libgecode/vendored-gecode/
	@echo export TRAVIS=1
	@echo export USE_SYSTEM_GECODE=1
	@echo export LIBRARY_PATH=$$CHEFDK_GECODE_PATH/lib
	@echo export LD_LIBRARY_PATH=$$CHEFDK_GECODE_PATH/lib
	@echo export CPLUS_INCLUDE_PATH=$$CHEFDK_GECODE_PATH/include

distclean:
	@rm -rf _build
