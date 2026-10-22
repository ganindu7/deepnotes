---
layout: default
title: Attribute macros
nav_order: 2
permalink: /topics/Code/rust/attribute_macros
parent: Rust
grand_parent: Development
---

## Attribute macros

Status: Draft (work in progress) · Updated: {{ page.path | file_date | date_to_string }}

In Rust the `#[derive(...)]` attribute is a macro that generates default implementations of specific traits for the struct or enum defined after the attribute line.

The Rust compiler has built-in code generation for a specific, exhaustive list of standard library traits, including `Debug`.

The `#[derive]` macro accepts both procedural macros and traits compatible with `#[derive]`.
