# Plannotator Overview

## Overview

Plannotator is a plugin that can be used with OpenCode, and it appears to also be available for other coding agents such as Cloud Code.

Its main goal is to provide a richer GUI for reviewing AI-generated plans before implementation. Instead of reading a plain-text plan only in the terminal, Plannotator gives a more structured review experience that makes it easier to inspect the proposed work and provide feedback.

## Why It Is Useful

Plannotator improves the plan review phase by making the plan easier to understand and discuss.

Key benefits:

- Richer visual interface for plan review
- Ability to add annotations to specific parts of the plan
- Ability to leave general comments for the overall document
- Better collaboration during the planning phase before implementation starts

## How It Fits Into OpenCode

Plannotator is inserted into the `submit_plan` phase.

In practice, once the model has prepared a plan and reaches the point where it submits that plan for review, the plugin provides an enhanced interface for inspecting the plan and adding feedback.

To use it correctly, you need to be running in plan mode.

## Typical Workflow

1. Start the task in plan mode.
2. Let the model analyze the problem and prepare a plan.
3. When the model reaches the `submit_plan` step, Plannotator intercepts that phase.
4. Review the plan in the GUI.
5. Add targeted annotations on specific points, or add general comments for the whole plan.
6. Refine the plan based on the feedback before implementation begins.

## Troubleshooting

Due to Visa proxy behavior and restrictions introduced by Zscaler, installation may not always work out of the box.

In most cases, the problem is network-related rather than a problem with the plugin itself.

Things to check:

- Corporate proxy configuration
- Zscaler restrictions or certificate-related issues
- Network access to any remote installation sources or package registries
- Whether the installation behaves differently on a different network or after proxy-related adjustments

Check the screenshots for examples of the installation issues and error patterns.

## Notes

This tool is especially useful when the planning step matters and you want a clearer way to review and annotate AI-generated plans before moving into implementation.
