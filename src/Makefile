# Compiler settings
CC = gcc
STDFLAGS = -std=c11 -Wall -Werror -Wextra 
TEST_CFLAGS = -g $(STDFLAGS) #-fsanitize=address
GCOV_FLAGS = -fprofile-arcs -ftest-coverage
VALGRIND_FLAGS = --trace-children=yes --track-fds=yes --track-origins=yes --leak-check=full --show-leak-kinds=all --verbose

# Libraries for different OS
LIBS = -lcheck
LINUX_FLAGS = -lpthread -lcheck -pthread -lrt -lm -lsubunit  # Added -lsubunit for Check with Subunit support

# Targets
TARGET = s21_string.a
TEST_TARGET = test_string.a
SRC = $(wildcard s21_*.c)
OBJ = $(patsubst %.c, %.o, ${SRC})

# Test Sources
PREF_TEST_SRC = ./unit_tests/
TEST_SRC = $(wildcard $(PREF_TEST_SRC)/s21_*.c)

NEW_TEST_SRC = $(wildcard ./tests/*.c)
NEW_TEST_OBJ = $(patsubst ./tests/%.c, %.o, $(NEW_TEST_SRC))

# OS Detection for Open Command
OS := $(shell uname -s)

ifeq ($(OS),Linux)
  OPEN_CMD = xdg-open
endif

ifeq ($(OS),Darwin)
  OPEN_CMD = open
endif

# Default target
all: clean $(TARGET) test

# Build static library
$(TARGET): ${SRC}
	$(CC) -c $(STDFLAGS) $(SRC)
	ar rcs $@ $(OBJ)

# Compile and run tests
test: $(TARGET)
ifeq ($(OS), Darwin)
	$(CC) $(STDFLAGS) ${NEW_TEST_SRC} $(TARGET) -o lib_test $(LIBS)
else
	$(CC) $(STDFLAGS) ${NEW_TEST_SRC} $(TARGET) -o lib_test $(LIBS) $(LINUX_FLAGS)
endif
	./lib_test

# Code coverage report
gcov_report: clean_test
ifeq ($(OS), Darwin)
	$(CC) $(STDFLAGS) $(GCOV_FLAGS) ${NEW_TEST_SRC} ${SRC} -o test $(LIBS)
else
	$(CC) $(STDFLAGS) $(GCOV_FLAGS) ${NEW_TEST_SRC} ${SRC} -o test $(LIBS) $(LINUX_FLAGS)
endif
	./test
	lcov -t "stest" -o s21_test.info -c -d .
	genhtml -o report s21_test.info
	$(OPEN_CMD) ./report/index.html

# Memory leak check with Valgrind
valgrind: $(TARGET)
	$(CC) $(TEST_CFLAGS) ${TEST_SRC} $< -o test $(LIBS) $(LINUX_FLAGS)
	CK_FORK=no valgrind $(VALGRIND_FLAGS) --log-file=RESULT_VALGRIND.txt ./lib_test

# Format code using Clang-Format
clang:
	clang-format -style=Google -n *.c *.h */*.c */*.h

# Check for memory leaks using leaks (macOS)
leaks:
	CK_FORK=no leaks --atExit -- ./lib_test

# Install dependencies for macOS
brew:
	cd ~
	curl -fsSL https://rawgit.com/kube/42homebrew/master/install.sh | zsh

# Install lcov for code coverage
lcov:
ifeq ($(OS),Linux)
	sudo apt install lcov
endif
ifeq ($(OS),Darwin)
	brew install lcov
endif

# Cleanup object files
clean_obj:
	rm -rf *.o

# Cleanup libraries
clean_lib: 
	rm -rf *.a

# Cleanup test files
clean_test:
	rm -rf *.gcda
	rm -rf *.gcno
	rm -rf *.info
	rm -rf test
	rm -rf report

# Cleanup all
clean: clean_lib clean_test clean_obj
	rm -rf lib_test
	rm -rf unit_test
	rm -rf RESULT_VALGRIND.txt

.PHONY: all clean check lcov
