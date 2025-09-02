# [Feature Request]: Optimized DOM Cloning Strategy per Animation Type

## Overview

Currently, SSGOI uses `element.cloneNode(true)` to deep clone the entire DOM tree during OUT animations. While this works reliably for all animation types, it may introduce unnecessary overhead for simple animations like fade out.

## Current Implementation Analysis

```typescript
// packages/core/src/lib/create-transition-callback.ts:147
const cloned = element.cloneNode(true) as HTMLElement; // Clones all child elements
```

### Observed Behavior

The current approach prioritizes **stability and versatility**:
- ✅ Works perfectly with all animation types (fade, scale, slide, hero, etc.)
- ✅ Guarantees visually identical results to the original
- ✅ Simple and predictable implementation

However, there's room for improvement:
- ⚠️ Even simple animations like fade out clone the entire DOM tree
- ⚠️ Potential performance impact with complex DOM structures (hundreds to thousands of child elements)
- ⚠️ Internal content that won't be visible is still fully cloned

## Proposed Improvement

Introducing an optimization strategy that selectively clones only necessary elements based on animation type could be beneficial.

### Core Concept

**For Fade Out:**
- What's actually needed: Element dimensions, position, background styles
- What's unnecessary: Inner text, child elements (invisible anyway due to fading)

**For Scale/Slide:**
- What's needed: All visual elements (maintain current approach)

**For Hero Transitions:**
- What's needed: Only specific elements with `data-hero-key`

### Expected Benefits

1. **Memory Efficiency**: Reduced unnecessary DOM cloning, especially for complex components (tables, lists, forms, etc.)

2. **Performance Improvement**: Unlike the current linear time complexity with DOM tree size, copying only what's needed maintains consistent performance

3. **Mobile Optimization**: Smoother animations in resource-constrained environments

## Implementation Suggestion

Rather than complex implementation, I propose a gradual improvement approach:

1. **Phase 1**: Detect animation type and choose between shallow/deep clone
2. **Phase 2**: Provide options for developers to specify cloning strategy
3. **Phase 3**: Analyze actual usage patterns for automatic optimization

## Considerations

- **Backward Compatibility**: Default behavior remains unchanged (deep clone)
- **Stability**: Thorough testing needed to prevent visual issues
- **Gradual Adoption**: Start with opt-in approach, consider changing defaults after stabilization

## Related Context

This optimization would be particularly meaningful in scenarios such as:
- Page transitions with large tables or lists
- Admin pages with complex form elements
- Mobile web applications
- SPAs with frequent page transitions

## Discussion Points

I'd like to hear the community's thoughts on whether this feature would be truly useful and if the benefits justify the implementation complexity. Feedback from those who have experienced performance issues due to DOM cloning in production environments would be especially valuable.