---
name: flutter-pro-design
description: Create distinctive, production-grade Flutter/Dart mobile and web interfaces with exceptional UI/UX design, smooth animations, and clean architecture. Use when building Flutter screens, widgets, components, full apps, dashboards, onboarding flows, or any mobile/web interface in Flutter/Dart. Generates polished, animated, professional code that avoids generic AI aesthetics. Triggers on any mention of Flutter, Dart, mobile app design, or app UI.
---

# Flutter Pro Design Skill

Create distinctive, production-grade Flutter interfaces with exceptional design quality, fluid animations, and clean architecture. Every screen should feel intentionally crafted — never generic.

## Design Thinking (Before Coding)

Before writing any code, define the aesthetic direction:

- **Purpose**: What problem does this screen solve? Who is the user?
- **Tone**: Choose a bold direction — luxury/refined, soft/organic, brutally minimal, editorial/magazine, playful/vibrant, dark/immersive, glassmorphism, neomorphism, retro-futuristic, or another distinct aesthetic. Commit fully.
- **Signature Element**: What ONE thing makes this screen unforgettable? A hero animation? A custom scroll effect? A unique transition? An unexpected layout?
- **Color Strategy**: Dominant color + sharp accent. Never distribute colors timidly or evenly.

## Typography Excellence

Flutter gives access to Google Fonts. USE THEM CREATIVELY:

```dart
// NEVER use default or generic fonts. Always import distinctive typography.
import 'package:google_fonts/google_fonts.dart';

// Examples of distinctive pairings:
// Display/Headlines:
GoogleFonts.playfairDisplay()    // Elegant editorial
GoogleFonts.spaceGrotesk()       // Modern tech
GoogleFonts.cormorantGaramond()  // Luxury refined
GoogleFonts.clashDisplay()       // Bold contemporary
GoogleFonts.cabinetGrotesk()     // Clean geometric
GoogleFonts.generalSans()        // Versatile modern
GoogleFonts.satoshi()            // Friendly professional
GoogleFonts.zodiak()             // Art deco revival
GoogleFonts.gambetta()           // Sophisticated serif
GoogleFonts.switzer()            // Swiss precision

// Body text:
GoogleFonts.dmSans()             // Clean readable
GoogleFonts.plusJakartaSans()     // Modern warmth
GoogleFonts.outfit()             // Geometric clarity
GoogleFonts.manrope()            // Balanced neutral
GoogleFonts.lexend()             // Optimized readability
```

**Rules:**
- Always pair a distinctive display font with a refined body font
- Vary choices between projects — NEVER converge on the same fonts
- Use font weight contrast dramatically (Thin 100 vs Bold 800)
- Letter spacing matters: tight for headlines (-1.5 to -0.5), relaxed for labels (+1.0 to +2.0)

## Color & Theme System

Build a cohesive theme using `ThemeData` and `ColorScheme`:

```dart
// Create a complete, intentional color system
final theme = ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(
    seedColor: primaryColor,
    brightness: Brightness.light, // or dark
  ),
  // Override with custom palette for distinctiveness
  extensions: [
    CustomColors(
      accent: Color(0xFFFF6B35),
      surface: Color(0xFFF8F7F4),
      subtle: Color(0xFFE8E6E1),
      success: Color(0xFF2D9F6F),
      warning: Color(0xFFE8A838),
    ),
  ],
);
```

**Color Rules:**
- NEVER use pure black (#000000) or pure white (#FFFFFF) — use tinted neutrals
- Create depth with subtle background variations (2-3 surface levels)
- Accent colors should POP against the dominant palette
- Use opacity strategically: `.withOpacity(0.08)` for subtle fills, `.withOpacity(0.6)` for secondary text
- Dark themes: use deep blues/grays (#0A0E1A, #1A1F2E) instead of pure black

## Animation & Motion Design

This is the CORE differentiator. Every screen should have purposeful, fluid motion.

### Animation Principles

1. **Staggered Entrances** — Elements appear sequentially, not all at once
2. **Spring Physics** — Use spring curves for natural, organic feel
3. **Micro-interactions** — Every tap, swipe, scroll should have feedback
4. **Hero Transitions** — Seamless navigation between screens
5. **Scroll-driven Effects** — Parallax, fade, scale on scroll

### Staggered Animation Pattern (USE THIS EVERYWHERE)

```dart
class StaggeredListScreen extends StatefulWidget {
  @override
  State<StaggeredListScreen> createState() => _StaggeredListScreenState();
}

class _StaggeredListScreenState extends State<StaggeredListScreen>
    with TickerProviderStateMixin {
  late final AnimationController _controller;
  final int _itemCount = 8;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1200),
    )..forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Animation<double> _getStaggeredAnimation(int index) {
    final start = (index * 0.1).clamp(0.0, 1.0);
    final end = (start + 0.4).clamp(0.0, 1.0);
    return CurvedAnimation(
      parent: _controller,
      curve: Interval(start, end, curve: Curves.easeOutCubic),
    );
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: _itemCount,
      itemBuilder: (context, index) {
        final animation = _getStaggeredAnimation(index);
        return AnimatedBuilder(
          animation: animation,
          builder: (context, child) => Transform.translate(
            offset: Offset(0, 30 * (1 - animation.value)),
            child: Opacity(
              opacity: animation.value,
              child: child,
            ),
          ),
          child: _buildListItem(index),
        );
      },
    );
  }
}
```

### Spring Animation (Natural Feel)

```dart
// Use spring simulations for organic motion
final springAnimation = SpringSimulation(
  const SpringDescription(
    mass: 1.0,
    stiffness: 200.0,
    damping: 20.0,
  ),
  0.0,  // start
  1.0,  // end
  0.0,  // velocity
);

// Or use Curves for simpler cases:
Curves.easeOutBack      // Slight overshoot — feels alive
Curves.easeOutCubic     // Smooth deceleration — professional
Curves.elasticOut        // Bouncy — playful apps
Curves.easeInOutQuart   // Dramatic entrance/exit
```

### Micro-Interactions

```dart
// Animated button with scale + haptic feedback
class AnimatedPressButton extends StatefulWidget {
  final Widget child;
  final VoidCallback onPressed;
  
  const AnimatedPressButton({
    required this.child,
    required this.onPressed,
  });
  
  @override
  State<AnimatedPressButton> createState() => _AnimatedPressButtonState();
}

class _AnimatedPressButtonState extends State<AnimatedPressButton>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scale;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 100),
    );
    _scale = Tween<double>(begin: 1.0, end: 0.95).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
    );
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: (_) => _controller.forward(),
      onTapUp: (_) {
        _controller.reverse();
        HapticFeedback.lightImpact();
        widget.onPressed();
      },
      onTapCancel: () => _controller.reverse(),
      child: AnimatedBuilder(
        animation: _scale,
        builder: (context, child) => Transform.scale(
          scale: _scale.value,
          child: child,
        ),
        child: widget.child,
      ),
    );
  }
}
```

### Page Transitions

```dart
// Custom page route with slide + fade
class SlideUpRoute<T> extends PageRouteBuilder<T> {
  final Widget page;
  
  SlideUpRoute({required this.page})
      : super(
          transitionDuration: const Duration(milliseconds: 400),
          reverseTransitionDuration: const Duration(milliseconds: 350),
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            final curve = CurvedAnimation(
              parent: animation,
              curve: Curves.easeOutCubic,
            );
            return SlideTransition(
              position: Tween<Offset>(
                begin: const Offset(0, 0.15),
                end: Offset.zero,
              ).animate(curve),
              child: FadeTransition(
                opacity: Tween<double>(begin: 0, end: 1).animate(curve),
                child: child,
              ),
            );
          },
        );
}

// Shared element / Hero transitions
Hero(
  tag: 'item-$id',
  flightShuttleBuilder: (flightContext, animation, direction, fromHero, toHero) {
    return Material(
      color: Colors.transparent,
      child: toHero.widget,
    );
  },
  child: yourWidget,
)
```

### Scroll-Driven Animations

```dart
// Parallax effect in a CustomScrollView
class ParallaxHeader extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return SliverAppBar(
      expandedHeight: 300,
      flexibleSpace: LayoutBuilder(
        builder: (context, constraints) {
          final percent = 1 - (constraints.maxHeight - kToolbarHeight) /
              (300 - kToolbarHeight);
          return Stack(
            fit: StackFit.expand,
            children: [
              Transform.scale(
                scale: 1 + (percent * 0.1),
                child: Image.asset('assets/hero.jpg', fit: BoxFit.cover),
              ),
              // Gradient overlay that intensifies on scroll
              DecoratedBox(
                decoration: BoxDecoration(
                  gradient: LinearGradient(
                    begin: Alignment.topCenter,
                    end: Alignment.bottomCenter,
                    colors: [
                      Colors.transparent,
                      Colors.black.withOpacity(0.3 + percent * 0.4),
                    ],
                  ),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

## Spatial Composition & Layout

### Rules for Professional Layout:
- **8dp grid system** — All spacing should be multiples of 8 (8, 16, 24, 32, 48, 64)
- **Generous padding** — Mobile screens need breathing room. Minimum 20-24 horizontal padding
- **Visual hierarchy through scale** — Headlines 28-40sp, body 14-16sp, captions 11-12sp
- **Asymmetry is welcome** — Not everything needs to be centered
- **Depth through elevation** — Use subtle shadows, not Material's harsh defaults

### Custom Shadows (Not Material Defaults)

```dart
// Soft, layered shadows instead of Material elevation
BoxDecoration(
  borderRadius: BorderRadius.circular(16),
  color: Colors.white,
  boxShadow: [
    BoxShadow(
      color: Colors.black.withOpacity(0.04),
      blurRadius: 8,
      offset: const Offset(0, 2),
    ),
    BoxShadow(
      color: Colors.black.withOpacity(0.02),
      blurRadius: 24,
      offset: const Offset(0, 8),
    ),
  ],
)

// Colored glow shadow for accent elements
BoxDecoration(
  color: accentColor,
  borderRadius: BorderRadius.circular(12),
  boxShadow: [
    BoxShadow(
      color: accentColor.withOpacity(0.3),
      blurRadius: 20,
      offset: const Offset(0, 8),
    ),
  ],
)
```

## Visual Details & Polish

### Glassmorphism

```dart
ClipRRect(
  borderRadius: BorderRadius.circular(16),
  child: BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 12, sigmaY: 12),
    child: Container(
      decoration: BoxDecoration(
        color: Colors.white.withOpacity(0.15),
        borderRadius: BorderRadius.circular(16),
        border: Border.all(
          color: Colors.white.withOpacity(0.2),
        ),
      ),
      child: content,
    ),
  ),
)
```

### Gradient Meshes & Decorative Backgrounds

```dart
// Multi-stop gradient background with atmosphere
Container(
  decoration: const BoxDecoration(
    gradient: LinearGradient(
      begin: Alignment(-1.0, -1.0),
      end: Alignment(1.0, 1.0),
      colors: [
        Color(0xFF0A0E1A),
        Color(0xFF1A1040),
        Color(0xFF0D1520),
      ],
      stops: [0.0, 0.5, 1.0],
    ),
  ),
  child: Stack(
    children: [
      // Ambient glow orbs
      Positioned(
        top: -100,
        right: -50,
        child: Container(
          width: 300,
          height: 300,
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            gradient: RadialGradient(
              colors: [
                Color(0xFF6C63FF).withOpacity(0.3),
                Colors.transparent,
              ],
            ),
          ),
        ),
      ),
      // Your content here
    ],
  ),
)
```

### Shimmer Loading Effect

```dart
class ShimmerWidget extends StatefulWidget {
  final double width;
  final double height;
  final double borderRadius;

  const ShimmerWidget({
    this.width = double.infinity,
    required this.height,
    this.borderRadius = 12,
  });

  @override
  State<ShimmerWidget> createState() => _ShimmerWidgetState();
}

class _ShimmerWidgetState extends State<ShimmerWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1500),
    )..repeat();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Container(
          width: widget.width,
          height: widget.height,
          decoration: BoxDecoration(
            borderRadius: BorderRadius.circular(widget.borderRadius),
            gradient: LinearGradient(
              begin: Alignment(-1.0 + 2 * _controller.value, 0),
              end: Alignment(-0.5 + 2 * _controller.value, 0),
              colors: [
                Color(0xFFE8E8E8),
                Color(0xFFF5F5F5),
                Color(0xFFE8E8E8),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

## Architecture Rules

### File Structure
```
lib/
├── core/
│   ├── theme/
│   │   ├── app_theme.dart         # ThemeData + ColorScheme
│   │   ├── app_typography.dart    # Text styles with Google Fonts
│   │   └── app_spacing.dart       # Spacing constants (8dp grid)
│   ├── animations/
│   │   ├── page_transitions.dart  # Custom route transitions
│   │   ├── stagger_animation.dart # Reusable stagger helpers
│   │   └── micro_interactions.dart # Button press, toggle, etc.
│   └── widgets/
│       ├── animated_card.dart     # Cards with entrance animation
│       ├── animated_button.dart   # Press-scale buttons
│       └── shimmer_loader.dart    # Loading placeholders
├── features/
│   └── feature_name/
│       ├── presentation/
│       │   ├── screens/
│       │   └── widgets/
│       ├── domain/
│       └── data/
└── main.dart
```

### Widget Rules
- **Small, focused widgets** — Each widget does ONE thing
- **const constructors** everywhere possible
- **Separate business logic from UI** completely
- **Responsive**: Use `MediaQuery`, `LayoutBuilder`, `Flexible`, `FractionallySizedBox`
- **Named parameters** with descriptive names
- **Extract magic numbers** into theme constants

## Packages to Leverage

```yaml
dependencies:
  google_fonts: ^6.0.0          # Beautiful typography
  flutter_animate: ^4.0.0       # Declarative animations (excellent!)
  shimmer: ^3.0.0               # Loading effects
  cached_network_image: ^3.3.0  # Image loading with placeholders
  flutter_svg: ^2.0.0           # SVG support
  lottie: ^3.0.0                # Lottie animations for complex motion
  smooth_page_indicator: ^1.1.0 # Beautiful page indicators
  animations: ^2.0.0            # Material motion (Google)
  iconsax_flutter: ^1.0.0       # Beautiful icon set (alternative to default)
  hugeicons: ^0.0.7             # Another premium icon set

dev_dependencies:
  flutter_lints: ^3.0.0
```

### flutter_animate (HIGHLY RECOMMENDED)

This package makes animations declarative and composable:

```dart
// Staggered list with flutter_animate
ListView.builder(
  itemBuilder: (context, index) => Card(...)
    .animate(delay: Duration(milliseconds: index * 100))
    .fadeIn(duration: 400.ms)
    .slideY(begin: 0.3, end: 0, curve: Curves.easeOutCubic),
)

// Complex entrance animations
Column(
  children: [
    Text('Welcome')
      .animate()
      .fadeIn(delay: 200.ms)
      .slideX(begin: -0.1),
    Text('Subtitle')
      .animate()
      .fadeIn(delay: 400.ms)
      .slideX(begin: -0.1),
    ElevatedButton(...)
      .animate()
      .fadeIn(delay: 600.ms)
      .scaleXY(begin: 0.9, end: 1.0),
  ],
)
```

## Critical DON'Ts

- **NEVER** use default Material colors without customization
- **NEVER** use default AppBar without styling
- **NEVER** leave screens without entrance animations
- **NEVER** use generic icon sets when better alternatives exist
- **NEVER** use pure black text on pure white backgrounds
- **NEVER** use uniform spacing — vary rhythm for visual interest
- **NEVER** create flat, lifeless layouts — every screen needs depth
- **NEVER** skip loading states — always show shimmer or skeleton screens
- **NEVER** use default Material elevation shadows — create custom soft shadows
- **NEVER** use boring, predictable screen transitions

## Implementation Checklist

For every screen you build, verify:

- [ ] Distinctive font pairing (display + body from Google Fonts)
- [ ] Custom color palette with tinted neutrals (no pure black/white)
- [ ] Staggered entrance animation for content
- [ ] Micro-interactions on buttons and interactive elements
- [ ] Custom page transition (not default Material)
- [ ] Proper visual hierarchy (size, weight, color, spacing)
- [ ] 8dp grid spacing throughout
- [ ] Custom shadows (not Material elevation)
- [ ] Loading/shimmer states
- [ ] Responsive layout considerations
- [ ] const constructors where possible
- [ ] Clean widget separation (small, focused)

Remember: Every screen should feel like it was designed by a senior UI designer and implemented by a senior Flutter developer. No generic outputs. Make it memorable.
