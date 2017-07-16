---
title: Advanced Integrators Summary
summary : "Summary of what we've learned about time integrators"
sidebar: mydoc_sidebar
permalink: mydoc_advanced_integrator_summary.html
folder: mydoc
toc : true
---

## Overview
Here's a handy table of what we've learned over the last few articles.

| Method | Position Error | Velocity Error | A-stable |
|-------|--------|---------|---------|
| Explicit Euler | $$\mathcal{O}(h)$$ | $$\mathcal{O}(h)$$ | No |
| Symplectic Euler | $$\mathcal{O}(h)$$ | $$\mathcal{O}(h)$$ | No |
| Velocity Verlet | $$\mathcal{O}(h^{2})$$ | $$\mathcal{O}(h^{2})$$ | No |
| Backward Euler | $$\mathcal{O}(h)$$ | $$\mathcal{O}(h)$$ | Yes |
| Implicit Midpoint Method | $$\mathcal{O}(h)$$ | $$\mathcal{O}(h)$$ | Yes |

So if your simulation is not very heavy on physics Symplectic Euler is probably okay. 
Otherwise you'll want to go with Verlet. 
Implicit methods like the Gauss–Legendre methods (of which implicit midpoint is the simplest) should be reserved for particularly stubborn 
differential equations as they are expensive.
