# Project: helloworld
# File:    Makefile
# Purpose: Build, run, and clean the helloworld C11 program.
# Author:  OpenClaw Agent
# Date:    2026-10-07

CC     ?= gcc
CFLAGS = -std=c11 -Wall -Wextra -O2

TARGET = helloworld
SRC    = src/main.c

.PHONY: all run clean

all: $(TARGET)

$(TARGET): $(SRC)
	$(CC) $(CFLAGS) -o $@ $<

run: all
	./$(TARGET)

clean:
	rm -f $(TARGET) $(TARGET).exe
