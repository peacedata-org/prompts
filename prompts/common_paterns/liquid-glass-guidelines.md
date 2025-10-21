# Liquid Glass Web Design Specification

## Design Philosophy

Create a web interface that embodies Apple's Liquid Glass design language—a dynamic, translucent material system that combines the optical properties of physical glass with fluid, responsive behavior. The design must feel alive, adaptive, and deeply integrated with content while maintaining exceptional clarity and hierarchy.

**Core Principle**: This is NOT traditional glassmorphism. Liquid Glass is a living, breathing meta-material that dynamically bends and shapes light, responds to interaction, and adapts intelligently to its environment.

---

## I. Material Properties

### A. Optical Characteristics

**Lensing & Refraction**
- Background content must be BENT and WARPED through the glass surface, not simply blurred
- Implement real-time dynamic lensing that creates depth perception through light distortion
- Edges must show pronounced refraction effects where light bends most dramatically
- The material should magnify, distort, and refract content beneath it like actual curved glass

**Translucency System**
- Base translucency allows 60-75% of background content to remain visible
- Never use flat transparency—always combine with blur, refraction, and tint
- Glass thickness must be visually communicated through shadow depth and refraction intensity
- Larger elements simulate thicker glass with deeper shadows and stronger lensing

**Light Behavior**
- Implement specular highlights that track user interaction and movement
- Light must scatter softly at glass edges while concentrating through the center
- Highlights should feel wet, glossy, and dynamic—not static gradients
- Ambient light from surrounding content should reflect and tint the glass surface

### B. Adaptive Intelligence

**Environmental Awareness**
- Glass must continuously sample colors from background content
- Tint adjusts dynamically to ensure text/icon legibility while revealing content beneath
- Automatically switch between light and dark appearance based on content luminance
- When text scrolls underneath, shadows intensify automatically for separation

**Contextual Morphing**
- Small controls (buttons, toggles) use lighter, more transparent glass
- Large surfaces (sheets, panels, sidebars) simulate thicker glass with richer shadows
- Glass properties transform smoothly when elements resize or morph
- Depth and refraction intensity scale proportionally with element size

---

## II. Visual Layers (Composited in Order)

**Layer 1: Base Illumination**
- Soft, adaptive glow that samples background colors
- Provides foundational depth and material presence
- Intensity varies with glass "thickness"

**Layer 2: Shadow System**
- Cast deep, soft shadows beneath glass elements
- Shadow depth/blur increases with element size and elevation
- Shadows must be colored (tinted by surroundings), never pure black
- Creates clear separation between glass layer and content below

**Layer 3: Refraction/Blur**
- Real-time background distortion (not simple Gaussian blur)
- Stronger refraction at edges, softer in center
- Must feel like light bending through curved glass

**Layer 4: Surface Highlights**
- Specular reflections along top and curved edges
- Subtle gradient overlays that simulate light catching glass
- Responsive to interaction—brighten/shift with hover and click

**Layer 5: Content Layer**
- Text, icons, and controls that sit ON the glass
- Must maintain WCAG AA contrast (4.5:1 for text, 3:1 for UI)
- Use adaptive shadows behind text if background contrast drops

---

## III. Motion & Interaction

### A. Fluid Responsiveness

**Core Motion Principles**
- Motion must feel liquid, gel-like, and elastic—never stiff or mechanical
- Elements morph and flex with a soft, organic quality
- All animations use ease curves that mimic fluid dynamics (cubic-bezier)

**Hover Behavior**
- Glass instantly energizes with brightened highlights on hover
- Subtle elastic flex—element slightly expands and contracts
- Lensing effect intensifies, making underlying content more visible
- Transition duration: 150-200ms with fluid easing

**Press/Active States**
- Element compresses slightly with a gel-like squish
- Light concentrates at press point
- Deeper shadow suggests element is being pushed into the surface
- Return bounce uses spring physics (overshoot + settle)

### B. Materialization

**Appearance**
- Elements don't fade in—they MATERIALIZE
- Gradually modulate lensing and refraction from 0% to full strength
- Light bending becomes visible first, then highlights emerge
- Duration: 300-400ms with ease-out curve

**Disappearance**
- Reverse the lensing modulation smoothly
- Glass dissolves by reducing refraction, not opacity
- Maintains optical integrity throughout transition

---

## IV. Typography & Color

### A. Text Treatment

**Hierarchy**
- Large headings (32px+) can use semi-bold weights with subtle text shadows
- Body text (16-18px) must be medium weight minimum for legibility
- Small text (14px) requires increased contrast and careful background handling

**Legibility Safeguards**
- Text ALWAYS has adaptive shadow or backdrop when over variable backgrounds
- If background luminance conflicts with text, glass tint automatically adjusts
- Minimum contrast ratio: 4.5:1 for body text, 7:1 for small text
- Use text-stroke (1px) on light text over light backgrounds for separation

### B. Color System

**Light Theme**
- Base glass tint: 5-15% opacity of white with subtle warm cast
- Shadows: Soft black at 15-25% opacity with 8-16px blur
- Text: Rich black (#1a1a1a) with adaptive shadows
- Highlights: White at 40-60% opacity, concentrated at top edges

**Dark Theme**
- Base glass tint: 8-18% opacity of white, cooler temperature
- Shadows: Deep black at 20-30% opacity with 12-20px blur
- Text: Off-white (#f5f5f5) with subtle glow for separation
- Highlights: Bright white at 50-70%, more pronounced than light theme

**Accent Colors**
- Can tint glass with brand colors at 10-20% opacity
- Colored glass ONLY for primary actions or navigation
- Maintain legibility—increase tint opacity for smaller elements
- Avoid rainbow explosion—use 1-2 accent colors maximum per view

---

## V. Component Architecture

### A. Structural Hierarchy

**Content Layer (Z-index: 1)**
- Main content, images, text, data displays
- Should never use Liquid Glass material
- Provides the "below glass" context that glass reacts to

**Glass Navigation Layer (Z-index: 100)**
- Toolbars, tab bars, sidebars, navigation controls
- Always uses Liquid Glass material
- Floats above content with clear separation
- Should be persistent across views when appropriate

**Modal/Sheet Layer (Z-index: 200)**
- Dialogs, sheets, popovers, menus
- Uses thicker glass simulation with deeper shadows
- Can stack, but avoid glass-on-glass (see Section VI)

### B. Shape Language

**Rounded Geometry**
- All glass elements use rounded corners (12-24px radius depending on size)
- Maintain concentricity—nested elements follow parent curves
- Shapes should feel friendly, finger-sized, touch-optimized
- Avoid sharp corners—they break the liquid metaphor

**Floating Forms**
- Elements don't pin to screen edges—they float with margins
- Navigation bars can shrink/expand contextually (e.g., on scroll)
- Bubbles appear/disappear based on context—not always visible
- Maintain generous spacing (16-24px) between glass elements

---

## VI. Implementation Rules

### A. What TO Do

**Use Liquid Glass For:**
- Navigation bars, tab bars, sidebars
- Toolbars and control panels that float above content
- Buttons, toggles, sliders, and input fields
- Modal sheets, dialogs, and popovers
- Media controls and transport bars
- Context menus and selection tooltips

**Material Variants:**
- **Regular Glass**: Full adaptive behavior, dynamically adjusts tint and contrast
- **Clear Glass**: More transparent, less adaptive, used when content below is visually rich

### B. What NOT To Do

**Avoid Liquid Glass In:**
- Primary content areas (text blocks, images, data tables)
- List items or scrollable content rows—breaks hierarchy
- Backgrounds of entire screens—glass should float, not fill
- Cards that sit within the content layer—reserve glass for controls

**Anti-Patterns:**
- **Glass-on-Glass**: Never stack multiple translucent layers—creates visual mud
- **Over-tinting**: Don't colorize everything—reserve color for emphasis
- **Static Blur**: Glass must be dynamic, not a fixed Photoshop effect
- **Poor Contrast**: Never sacrifice legibility for aesthetics

---

## VII. Accessibility Requirements

### A. Inclusive Design

**Respect System Preferences:**
- `prefers-reduced-transparency`: Increase glass opacity, reduce blur
- `prefers-reduced-motion`: Disable elastic/fluid animations, use simple fades
- `prefers-contrast`: Boost tint opacity, strengthen shadows, increase text weight
- Ensure fallback styles that maintain design intent without effects

**Keyboard & Focus:**
- Glass elements must show clear focus states (2px ring with high contrast)
- Focus indicators should glow subtly, not just outline
- Tab order follows visual hierarchy top-to-bottom, left-to-right

### B. Performance Considerations

**Optimization Targets:**
- Maintain 60fps for all animations and interactions
- Use CSS/GPU acceleration where possible: `transform`, `opacity`, `filter`
- Avoid layout thrashing—batch reflows and repaints
- Consider reduced glass complexity on lower-end devices

---

## VIII. Light & Dark Theme Requirements

### Theme Switching

**Adaptive Behavior:**
- Glass must INDEPENDENTLY adapt light/dark based on local background luminance
- A single view can contain light glass over dark content and dark glass over light content
- Theme switching should transition smoothly over 300ms with synchronized changes

**Theme-Specific Adjustments:**
- Dark theme requires MORE prominent highlights and shadows for depth perception
- Light theme uses subtler effects—daylight provides natural contrast
- Text contrast ratios must be validated independently for each theme

---

## IX. Web Implementation Guidance

### A. CSS Properties to Leverage

**Core Effects:**
```
backdrop-filter: blur() saturate();
background: rgba() with gradients;
box-shadow: multiple layered shadows;
filter: drop-shadow() for text legibility;
transform: for fluid motion;
transition/animation: with cubic-bezier curves;
```

**Advanced Techniques:**
- Combine multiple backdrop-filters for complex lensing
- Use SVG filters for custom refraction effects
- Implement CSS custom properties for theme adaptability
- Consider CSS Houdini (if supported) for advanced glass effects

### B. Responsive Behavior

**Breakpoints:**
- Mobile (<640px): Lighter glass, simplified shadows, reduced blur
- Tablet (640-1024px): Full glass effects with moderate complexity
- Desktop (1024px+): Maximum glass richness, deepest shadows, strongest lensing

**Touch vs. Hover:**
- Touch devices: Slightly larger touch targets, more pronounced press states
- Hover devices: Subtle hover effects, refined cursor feedback

---

## X. Quality Checklist

Before considering the design complete, verify:

- [ ] Glass elements dynamically refract/bend background content
- [ ] Specular highlights respond to interaction
- [ ] Text maintains 4.5:1 contrast in all states
- [ ] Motion feels fluid, elastic, and organic—never robotic
- [ ] Elements materialize/dematerialize via lensing modulation
- [ ] Shadows are soft, colored, and depth-appropriate
- [ ] No glass-on-glass stacking anywhere
- [ ] System accessibility preferences are respected
- [ ] Both light and dark themes are equally polished
- [ ] Performance is smooth (60fps) across interactions
- [ ] Hierarchy is clear: content → glass controls → modals

---

## Design Intent Summary

This specification demands a web interface where glass is not decoration—it is a functional, intelligent layer that bridges content and controls. The material must feel tangible yet ethereal, responsive yet refined. It should delight users while maintaining absolute clarity of purpose.

**The glass should feel alive. Every interaction should confirm: this is a living, breathing material that exists in harmony with content, light, and motion.**