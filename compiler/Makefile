PCC_ROOT = ..
include $(PCC_ROOT)/bigloo-rules.mk

BCOMMONFLAGS	= $(READLINE_SRFI) -L $(BGL_DEFAULT_LIB_DIR) -L ../libs -I ../libs 
CCOMMONFLAGS	= -I. -I$(BGL_DEFAULT_LIB_DIR) 

# for pcc
SOURCE_LIST	= ast lexers parser driver evaluate commandline containers \
	         generate basic-blocks declare cfa target config include

SOURCE_FILES     := $(patsubst %,%.scm,$(SOURCE_LIST))
POPULATION       := $(patsubst %,%_$(SU).o,$(SOURCE_LIST))
STATIC_POPULATION       := $(patsubst %,%_$(SU)$(STATIC_SUFFIX).o,$(SOURCE_LIST))

C_SOURCE_LIST   = registry

C_POPULATION     := $(patsubst %,%_$(SU).o,$(C_SOURCE_LIST))
C_STATIC_POPULATION     := $(patsubst %,%_$(SU)$(STATIC_SUFFIX).o,$(C_SOURCE_LIST))

# for libphpeval
EVAL_SOURCE_LIST = ast lexers parser driver evaluate containers generate basic-blocks \
	            declare cfa debugger target config include synhighlight
EVAL_SOURCE_FILES 		= $(patsubst %,%.scm,$(EVAL_SOURCE_LIST))
EVAL_POPULATION 		:= $(patsubst %,%_$(SU).o,$(EVAL_SOURCE_LIST))
EVAL_STATIC_POPULATION       := $(patsubst %,%_$(SU)$(STATIC_SUFFIX).o,$(EVAL_SOURCE_LIST))

CLEFTOVERS	 := $(patsubst %.o,%.c,$(POPULATION)) $(patsubst %.o,%.c,$(STATIC_POPULATION)) \
		    $(patsubst %.o,%.c,$(EVAL_POPULATION)) $(patsubst %.o,%.c,$(EVAL_STATIC_POPULATION))

PHPEVAL_LIBS	= ../libs/libphpeval_$(SUV).a ../libs/libphpeval_$(SUV).$(SOEXT) 

all: unsafe

# NOTE: pdb is not in this list because it depends on mhttpd
#        it gets built from the top level make through "make debugger"
all-run: $(PHPEVAL_LIBS) clean-bin pcc pcctags

debug: safe

safe:
	UNSAFE=f $(MAKE) all-run

unsafe: 
	UNSAFE=t $(MAKE) all-run

profile:
	PROFILE=t $(MAKE) debug

clean-bin:
	-rm pcc$(WEXE) pcctags$(WEXE) pdb$(WEXE)

debugger: $(PHPEVAL_LIBS) pdb 

pcc_$(SU): $(PHPEVAL_LIBS) $(C_POPULATION)  commandline_$(SU).o
	bigloo $(SAFETY) $(PCC_LINK_OPTIONS) $(BCOMMONFLAGS) -o pcc_$(SU) commandline_$(SU).o -library php-runtime -library profiler -library webconnect -library phpeval $(READLINE_BL_LIB)

pcc: pcc_$(SU)
	cp pcc_$(SU) pcc

../libs/libphpeval_$(SUV).$(SOEXT) : ../libs/phpeval.heap $(EVAL_POPULATION) $(C_POPULATION) 
	$(call dllcmd,../libs/libphpeval_$(SUV).$(SOEXT)) $(EVAL_POPULATION) $(C_POPULATION) $(COMPILER_DLL_LIBS)

../libs/libphpeval_$(SUV).a : ../libs/phpeval.heap $(EVAL_STATIC_POPULATION) $(C_STATIC_POPULATION)
	ar ruv ../libs/libphpeval_$(SUV).a $(EVAL_STATIC_POPULATION) $(C_STATIC_POPULATION)

../libs/phpeval.heap : ast.scm lexers.scm parser.scm driver.scm evaluate.scm containers.scm generate.scm include.scm
	$(BIGLOO) $(BHEAPFLAGS) -heap-library phpeval make-lib.scm -addheap ../libs/phpeval.heap
	@find . -maxdepth 1 -name \*.init -exec cp '{}' ../libs \;

lexers_$(SU).o: ../runtime/php-runtime.sch ast_$(SU).o

containers_$(SU).o: declare_$(SU).o ast_$(SU).o ../runtime/php-runtime.sch

#this is to override -O3 with -O for parser.scm, since -O3 takes perversely long.
#parser_$(SU).o: ast_$(SU).o parser.scm
#	$(BIGLOO) $(BSAFEFLAGS) -mkaddlib -unsafe -saw -copt -fPIC -c parser.scm -o parser_$(SU).o

#parser_$(SU)t.o: ast_$(SU).o parser.scm
#	$(BIGLOO) -static-bigloo $(BUNSAFEFLAGS) -c parser.scm -o parser_$(SU)t.o


evaluate_$(SU).o: ast_$(SU).o declare_$(SU).o ../runtime/php-runtime.sch

debugger_$(SU).o: ast_$(SU).o declare_$(SU).o ../runtime/php-runtime.sch

driver_$(SU).o: declare_$(SU).o containers_$(SU).o evaluate_$(SU).o ast_$(SU).o parser_$(SU).o lexers_$(SU).o basic-blocks_$(SU).o cfa_$(SU).o debugger_$(SU).o # generate_$(SU).o

generate_$(SU).o: declare_$(SU).o containers_$(SU).o ast_$(SU).o ../runtime/php-runtime.sch

commandline_$(SU).o: lexers_$(SU).o #driver_$(SU).o  ast_$(SU).o

cfa_$(SU).o: ast_$(SU).o declare_$(SU).o basic-blocks_$(SU).o

basic-blocks_$(SU).o: ast_$(SU).o declare_$(SU).o

declare_$(SU).o: ast_$(SU).o

signatures_$(SU).o: php-hash_$(SU).o

signatures.o: php-hash.o

clean: 
	-rm -rf pcc.cgi pcc pcctags pdb pcc_[us] pdb_[us] pcctags_[us] testoutput $(CLEFTOVERS)
	-rm -f *_[us].o *.so

pcctags_$(SU): tags_$(SU).o $(PHPEVAL_LIBS)
	bigloo $(SAFETY) $(BCOMMONFLAGS) -O6 -o pcctags_$(SU) tags_$(SU).o -library phpeval -library php-runtime -library profiler -library webconnect

pcctags: pcctags_$(SU)
	cp pcctags_$(SU) pcctags

pdb_$(SU): pdb_$(SU).o $(PHPEVAL_LIBS)
	bigloo $(SAFETY) $(BCOMMONFLAGS) -O6 -o pdb_$(SU) pdb_$(SU).o -library phpeval -library php-runtime -library profiler -library webconnect -library mhttpd -lwebserver $(READLINE_BL_LIB)

pdb: pdb_$(SU)
	cp pdb_$(SU) pdb
