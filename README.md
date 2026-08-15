## Mini Layout Engine

### A C++ implementation of core layout algorithms demonstrating understanding understanding of browser rendering fundamentals. Built as preparation for the WebKit Layout and Rendering team internship

## Project Overview

### This project implements a simplified layout engine in C++ that:

➢ Parses a simplified layout description format (CSS-like properties)
➢ Construct a layout tree
➢ Implements fundamental layout algorithms (Block layout + Flexbox basics)
➢ Performs text layout with line breaking and alignment
➢ Calculates positions and dimensions
➢ Includes comprehensive testing

## Relavance to WebKit

## The WebKit Layout and Rendering team works on the core engine that determines how web content appears on billions of Apple devices.


## This project demonstrates:

### 1. **Layout Tree Structure**

➢ Node hierarchy (Box model)
➢ Property inheritance
➢ Tree traversal


### 2. **Block Layout ALgorithm**

➢ Normal flow layout
➢ Box model calculations (margin, border, padding, content)
➢ Containing block logic

### 3. **Flexbox Layout Algorithm (Simplified)**

➢ Main axis / cross axis
➢ Flex item sizing
➢ Justification and alignment basics

### 4. **Text Layout Engine**

➢ Word wrapping and line breaking algorithm
➢ Text alignment (left, center, right, justify)
➢ Simple font metrics (character width, line height, letter spacing)
➢ Justified text with even word spacing distribution

### 5. **Layout Engine**

➢ Coordinates Layout passes
➢ Handles dependencies
➢ Computes final positions

### 6. **Testing Framework**

➢ Unit tests for layout calculations
➢ Regression test suite
➢ Performance benchmarks

### Project Scope 


## Included Features: 

➢ **Box model:** margin, border, padding, content 
➢ **Block layout:** Vertical stacking, width/height calculation
➢ **Basic flexbox:** flex-direction (row/column), justify-content, align-items
➢ **Text layout:** Word wrapping, line breaking, left/center/right/justify alignment
➢ **Font metrics:** Configure character width, line height, letter spacing
➢ **Positioning:** Normal flow only (no absolute/fixed)
➢ **Properties:** Width, height, margin, padding, display, flex properties
➢ Testing: Comprehensive unit and integration tests





