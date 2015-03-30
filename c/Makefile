DIRS := $(wildcard */)

$(info DIRS = $(DIRS))

all: $(addprefix sub-,$(DIRS))
	@:

%: $(addprefix sub-,$(DIRS))
	@:

sub-%:
	$(MAKE) -C $* $(MAKECMDGOALS)
