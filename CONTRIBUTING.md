# Contributing to CI/CD Solutions

Thank you for your interest in contributing to this project! This document provides guidelines and steps for contributing.

## How to Contribute

### Reporting Bugs

If you find a bug, please open an issue with:
- A clear, descriptive title
- Steps to reproduce the issue
- Expected behavior vs actual behavior
- Your environment (OS, CI/CD platform version, Terraform version)

### Suggesting Enhancements

Enhancement suggestions are welcome! Please open an issue with:
- A clear description of the enhancement
- Use case(s) for the enhancement
- Any relevant examples or references

### Pull Requests

1. **Fork the repository** and create your branch from `main`
2. **Make your changes** following the code style guidelines below
3. **Test your changes** locally if possible
4. **Update documentation** if needed
5. **Submit a pull request** with a clear description of the changes

## Code Style Guidelines

### YAML Files (Workflows, Pipelines)

- Use 2-space indentation
- Include comments for complex logic
- Use meaningful job and step names
- Group related steps together

### General

- Keep lines under 120 characters when possible
- Use consistent naming conventions (kebab-case for file names)
- Add comments to explain non-obvious configurations

## Testing Your Changes

Before submitting a PR:

1. **Validate YAML syntax** using tools like `yamllint`
2. **Verify Terraform workflow syntax** if applicable
3. **Test workflows** in a fork if possible

## Questions?

Feel free to open an issue for any questions about contributing.
