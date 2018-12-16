# tc39-proposal-dotstructuring
## Proposal: Object dotstructuring (dot property access + destructuring)

This proposal is also known as: Pick Notation, Extended Object Destructuring, Extended Dot Notation.
Inspired by Bob Myers' [proposal](https://github.com/rtm/js-pick-notation) released under MIT License.\

This is a proposal to allow read and write operations on multiple objects' properties, leveraging object destructuring syntax to support renaming and defaults. The suggested syntax would enable quick objects creation from existing object, to create subsets, and quick properties merging/replacing.
