# Guide to x86-32 Kernel

## Introduction

### About this Tutorial

The hardest thing about System programming especially kernel development is to find out how to start. This document serves as a starting point for anyone who is interested in the kernel development and a review material for myself. I sincerely hope this document will be useful for anyone who reads it.

### What you should know before reading this tutorial

This tutorial expect you to have some basic programming skills -- you should be comfortable with at least one programming language. You should also have some knowledge about computers, includes but not limited to the function of CPU and RAM. Note You do not need to have an advanced programming skills or expertise on computer architecture to read this tutorial.

### Tools will be used Frequently

#### GNU GCC

GNU GCC is the compiler collection which is developed by GNU. In this document, we will only use its capability to compile C code into x86 assembly code.

#### GNU Assembler

GNU assembler or known as gas is the assembler we will be using for this document to convert x86 assembly code to machine code.

#### GNU Linker

GNU Linker or known as ld is the linker we will be using for this document to control linking process and produce final binary code.

#### VirtualBox

VirtualBox is an open source virtualization software. We will use it to run a 32 bit OS.

#### QEMU

QEMU is a generic and open source machine emulator and virtualizer. We will use QEMU to run our kernel code.
