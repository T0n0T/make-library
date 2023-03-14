#预定义变量
CC = gcc
#预定义编译目录
SUBDIRS = src \
        lib \
        main \
		build 
#预定义目标
OBJS = src.o lib.o main.o
BIN = hello
OBJS_DIR = build
BIN_DIR = bin
#传递预定义参数
export CC OBJS BIN OBJS_DIR BIN_DIR

all:CHECK_DIR $(SUBDIRS) 
CHECK_DIR:
	@mkdir -p $(BIN_DIR)
	@mkdir -p $(OBJS_DIR)
$(SUBDIRS):ECHO
	@make -C $@

# ./$(BIN_DIR)/$(BIN) : ./$(OBJS_DIR)/$(OBJS)
# 	@$(CC) $^ -o $@ 

ECHO:
	@echo $(SUBDIRS)
	@echo begin compile
clean:
	@$(RM) $(OBJS_DIR)/*.o
	@rm -rf $(BIN_DIR)
