---
state: Draft
start-date: 2024-07-07
author: Rodrigo Arias Mallo <rodarima@gmail.com>
---

# Dillo RFC 001: RFC documents

This document is reserved to describe how to write RFC proposals.

We should consider some aspects such as
[bike-shedding](https://en.wikipedia.org/wiki/Law_of_triviality) and how to
avoid blocking RFCs for too long.

We may follow some other project policies as a quick start.

We may change the prefix to something else than "RFC" as not to confuse them
with IETF RFCs.

## Motivation

Some proposals require more thought to be carried out, in particular those that
involve design of new features.

By having a single document that describes the motivation behind the change, how
and why it is done that way and not in another helps having a clear picture to
decide if the change should be done or not and to avoid potential pitfalls.

## Design

To implement a RFC proposal system, we will use several pieces.

### Markdown files

The proposals will be written in Markdown and stored in a git repository so they
can be tracked with version control. This also allows users to easily write RFCs
without having to invest a lot of time learning a new language.

### RFC name

Each RFC should have an unique name prefixed with a unique number. For now we
will use the printf format "RFC-%3d" to pad the number with zeros, so they are
sorted alphanumerically.

The name of the RFC document should be composed of lowercase letters joined by
hyphens (-), for example:

    001-rfc-documents.md

### Structure of a RFC

An RFC should contain at least the following sections:

- Abstract: Summary of the proposal in a few lines.
- Motivation: Describes the current problems and provides a justification on why
  a change is needed.
- Design: Explains how to implement the proposal. The content here may be split
  in several sections with other section names.
- Validation: Describes how to determine if the proposal has been implemented
  correctly.

### Acceptance procedure

A RFC is submitted by opening a MR in the RFC repository or by sending a patch
via email to the Dillo mailing list.

A new RFC begins as a "draft" while it gathers feedback and while it is still in
development. After enough consensus is gathered, the RFC is accepted or
rejected.

Once accepted, it will become the decision on how to implement the proposal
that should be followed.

### Criteria to create new RFCs

Smaller changes that can be done in a few weeks should not become RFCs, but
rather just issues. A practical approach is to try to cover it using a single
issue, and if it becomes too big then consider drafting an RFC.

## Validation

Once implemented we should be able to go to a git repository or web and read
the RFCs.
