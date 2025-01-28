**The main proposal is located at [`proposal.md`](https://github.com/MatinDevsHere/LiveView-Prefetching/blob/master/proposal.md)**

# LiveView Prefetching Proposal

This repository contains a proposal to implement prefetching capabilities in Phoenix LiveView, which would enhance the user experience and performance of LiveView web apps.

## Why Prefetching?

Currently, LiveView loads content when users interact with elements or navigate between views. While this works well, there's potential for improvement. Users experience delays when navigating between views; Connection latency and the rendering process have the largest impact.

This proposal aims to introduce prefetching capabilities that would allow LiveView to:

- Preload likely-to-be-needed content before user interaction
- Significantly reduce perceived latency in view transitions
- Maintain LiveView's simplicity while adding powerful optimization options

## Getting Started

To learn more about this proposal:

1. Read the detailed `proposal.md` file in this repository
2. Check out the feedbacks and issues
3. Provide your insights and suggestions

## Contributing

We welcome community feedback and contributions to help shape this proposal. However, keep in mind that our goal is to help the Phoenix community and contributors by giving them a clear path of how this feature can be implemented, plus why it should exist. Thus, issues and discussions stating that prefetching isn't "beneficial" for their specific use cases, or there's no reason for having prefetching in Phoenix LiveView are not accepted. Instead, we encourage you to share the possible obstacles in our roadmap, so we can friendly discuss and try to address them.

Here's a list of contributions we accept:

- Open issues for questions or suggestions
- Submit pull requests with improvements
- Reflect the proposal in social media and other related spaces, helping it gain more attention and feedback.

## Status

This is a proposal in development and is subject to changed in the future.
