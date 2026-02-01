```css
/* animation is used with properties that can change over time 
   (like transform, opacity, background, position, etc.) */

.example {
  animation-name: bounce;
  animation-duration: 1.5s;
  animation-timing-function: ease-in-out;
  animation-delay: 5s;
  animation-iteration-count: infinite;
  animation-direction: alternate;
  animation-fill-mode: both;
  animation-play-state: running;
}
```

- `animation-name: bounce;` → links to the keyframes definition.
    
- `animation-duration: 1.5s;` → each cycle lasts 1.5 seconds.
    
- `animation-timing-function: ease-in-out;` → smooth acceleration & deceleration.
    
- `animation-delay: 5s;` → starts after 5 seconds.
    
- `animation-iteration-count: infinite;` → runs forever.
    
- `animation-direction: alternate;` → reverses on every cycle.
    
- `animation-fill-mode: both;` → retains styles before start & after end.
    
- `animation-play-state: running;` → ensures the animation is active.
    

---

```css
@keyframes bounce {
  0%   { transform: translateY(0); }
  100% { transform: translateY(-100px); }
}
```

Defines the `bounce` animation: element starts at its original position (0%) and moves up by 100px (100%).

---


