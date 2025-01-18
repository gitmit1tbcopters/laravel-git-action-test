#!/bin/bash

# Define the target installation directory
INSTALL_DIR="${INSTALL_DIR}"
SYMLINK_DIR="${SYMLINK_DIR}"

# Check if INSTALL_DIR && SYMLINK_DIR not empty, and INSTALL_DIR exists
if [ -n "$INSTALL_DIR" ] && [ -n "$SYMLINK_DIR" ] && [ -d "$INSTALL_DIR" ]; then
    # Create symlinks for all directories in the installation directory
    echo "Creating symlinks for all $INSTALL_DIR directories in $SYMLINK_DIR..."
    mkdir -p "$SYMLINK_DIR"
    for dir in "$INSTALL_DIR"/*/; do
        if [ -d "$dir" ]; then
            ln -sf "$dir" "$SYMLINK_DIR/"
        fi
    done
fi
