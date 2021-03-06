#
# Copyright 2011-2015 Jeff Bush
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

TOPDIR=../../../

include $(TOPDIR)/build/target.mk

FB_WIDTH=640
FB_HEIGHT=480
MEMORY_SIZE=4000000

CFLAGS+=-fno-rtti -std=c++11 -I$(TOPDIR)/software/libs/librender -DFB_WIDTH=$(FB_WIDTH) -DFB_HEIGHT=$(FB_HEIGHT) -Werror
LIBS=-lrender -lc -los-bare

SRCS=main.cpp PakFile.cpp MeshBuilder.cpp LevelRenderer.cpp

OBJS := $(CRT0_BARE) $(SRCS_TO_OBJS)
DEPS := $(SRCS_TO_DEPS)

all: $(OBJ_DIR)/program.hex

$(OBJ_DIR)/program.hex: $(OBJ_DIR)/program.elf
	$(ELF2HEX) -o $@ $<

$(OBJ_DIR)/program.elf: $(DEPS) $(OBJS)
	$(LD) -o $@ $(OBJS) $(LIBS) $(LDFLAGS)

# Make disassembly file
program.lst: $(OBJ_DIR)/program.elf FORCE
	$(OBJDUMP) --disassemble $(OBJ_DIR)/program.elf > program.lst 2> /dev/null

clean:
	rm -rf $(OBJ_DIR)

# Run in emulator
run: $(OBJ_DIR)/program.hex fsimage.bin
	$(EMULATOR) -c 0x$(MEMORY_SIZE) -f $(FB_WIDTH)x$(FB_HEIGHT) -b fsimage.bin $(OBJ_DIR)/program.hex

# Run in emulator under debugger
debug: $(OBJ_DIR)/program.hex fsimage.bin
	$(EMULATOR) -m gdb -c 0x$(MEMORY_SIZE) -f $(FB_WIDTH)x$(FB_HEIGHT) -b fsimage.bin $(OBJ_DIR)/program.hex &
	$(LLDB) --arch nyuzi $(OBJ_DIR)/program.elf -o "gdb-remote 8000"

fpgarun: $(OBJ_DIR)/program.hex fsimage.bin
	$(SERIAL_BOOT) $(SERIAL_PORT) $(OBJ_DIR)/program.hex fsimage.bin

verirun: $(OBJ_DIR)/program.hex fsimage.bin
	@rm -f $(OBJ_DIR)/output.bin output.png
	$(VERILATOR) +memdumpfile=$(OBJ_DIR)/output.bin +memdumpbase=0x200000 +memdumplen=0x12C000 +bin=$(OBJ_DIR)/program.hex +block=fsimage.bin
	@convert -depth 8 -size 640x480 rgba:$(OBJ_DIR)/output.bin output.png

fsimage.bin: pak0.pak
	$(MKFS) $@ pak0.pak

pak0.pak:
	@echo "Need to provide pak0.pak. See README"
	exit 1

FORCE:

-include $(DEPS)

